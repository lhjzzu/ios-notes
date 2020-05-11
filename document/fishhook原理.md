## fishhook原理

### 概述

+ fishhook只能做外部符号的替换
  - 替换代码段引用的定义在动态库的变量
  - 替换代码段引用的定义在动态库的函数

+ 根据前面对动态库的分析，我们可知， 当dyld加载了动态库之后，动态库会把符号表合并到全局符号表中，此时可以确定所有的符号的位置。此时我们可以进行重定位，更新原来引用定义在外部动态库中的符号
+ 引用的定义在动态库中的符号最终存放在
  - 定义在外部动态库中的全局变量, 存放在`section(__DATA, __got)`区域
  - 定义在外部动态库中的函数, 存放在`section(__DATA, __la_symbol_ptr)`, `section(__DATA, __nl_symbol_ptr)`区域

### 使用示例

```objc
static int (*orig_printf)(const char *, ...);
static void (*orig_nslog)(NSString *format,...);

void new_nslog(NSString *format,...) {
    orig_nslog(@"NSLog\n");
    va_list arg;
    va_start(arg, format);
    NSLogv(format, arg);
    va_end(arg);
}

int new_printf(const char *format, ...)
{
    int ret = 0;
    orig_printf("printf\n");
    va_list arg;
    va_start(arg, format);
    ret = vprintf(format, arg);
    va_end(arg);
    return ret;
}

- (void)viewDidLoad {
    [super viewDidLoad];
    
    // rebind `printf` 符号，让它指向到自定义的 `new_printf` 函数，orig_printf存放原来的printf的地址
    struct rebinding printf_rebinding = { "printf", new_printf, (void *)&orig_printf };
    // rebind `NSLog` 符号，让它指向到自定义的 `new_nslog` 函数, orig_nslog存放原来的printf的地址
    struct rebinding nslog_rebinding = { "NSLog", new_nslog, (void *)&orig_nslog };

    rebind_symbols((struct rebinding[2]){nslog_rebinding, printf_rebinding}, 2);
    
    printf("The answer to Life, the Universe and everything, is %d\n", 42);
    NSLog(@"The answer to Life, the Universe and everything, is %d\n", 42);
   
}
  //打印结果如下，从打印结果可以看出，当执行系统的printf时，先执行我们的函数
  I'm God\n
  The answer to Life, the Universe and everything, is 42
```

### 核心代码解析

```c

int rebind_symbols(struct rebinding rebindings[], size_t rebindings_nel) {
  int retval = prepend_rebindings(&_rebindings_head, rebindings, rebindings_nel);
  if (retval < 0) {
    return retval;
  }
  // If this was the first call, register callback for image additions (which is also invoked for
  // existing images, otherwise, just run on existing images
  if (!_rebindings_head->next) {
    //在dyld加载镜像时，会执行注册过的回调函数
    _dyld_register_func_for_add_image(_rebind_symbols_for_image);
  } else {
    //动态库的数量
    uint32_t c = _dyld_image_count();
    //_dyld_get_image_header:获取对应动态库header
    //_dyld_get_image_vmaddr_slide:获取对应动态库在内存中的slide，即为该动态库在内存中开始的虚拟地址
    for (uint32_t i = 0; i < c; i++) {
      _rebind_symbols_for_image(_dyld_get_image_header(i), _dyld_get_image_vmaddr_slide(i));
    }
  }
  return retval;
}


// 找到存储外部符号地址的Section的对应的Section_Header:
//  Section_Header(__DATA, __got)
//  Section_Header(__DATA, __nl_symbol_ptr)
//  Section_Header(__DATA, __la_symbol_ptr)

//header：动态库在内存中的虚拟地址
//slide: 即为该动态库在内存中的空间的虚拟地址的首地址
static void rebind_symbols_for_image(struct rebindings_entry *rebindings,
                                     const struct mach_header *header,
                                     intptr_t slide) {
  Dl_info info;
  if (dladdr(header, &info) == 0) {
    return;
  }

  segment_command_t *cur_seg_cmd;
  segment_command_t *linkedit_segment = NULL;
  struct symtab_command* symtab_cmd = NULL;
  struct dysymtab_command* dysymtab_cmd = NULL;

  //此时cur指向load commands
  uintptr_t cur = (uintptr_t)header + sizeof(mach_header_t);
  //遍历cmd，找到linkedit_segment, symtab_cmd, dysymtab_cmd
  for (uint i = 0; i < header->ncmds; i++, cur += cur_seg_cmd->cmdsize) {
    cur_seg_cmd = (segment_command_t *)cur;
    if (cur_seg_cmd->cmd == LC_SEGMENT_ARCH_DEPENDENT) {
      if (strcmp(cur_seg_cmd->segname, SEG_LINKEDIT) == 0) {
        //获得linkedit_segment
        linkedit_segment = cur_seg_cmd;
      }
    } else if (cur_seg_cmd->cmd == LC_SYMTAB) {
      //获得symtab_cmd
      symtab_cmd = (struct symtab_command*)cur_seg_cmd;
    } else if (cur_seg_cmd->cmd == LC_DYSYMTAB) {
      //获得dysymtab_cmd
      dysymtab_cmd = (struct dysymtab_command*)cur_seg_cmd;
    }
  }
  if (!symtab_cmd || !dysymtab_cmd || !linkedit_segment ||
      !dysymtab_cmd->nindirectsyms) {
    return;
  }
  // Find base symbol/string table addresses
  // 寻找符号表和字符串表的base address（在虚拟内存中的首地址）
  
  // slide:某个模块(image)的整体在虚拟内存中的偏移地址
    //    对于可执行文件，即为ASLR
    //    对于动态库，即为动态库的虚拟首地址
    
    // linkedit_segment->vmaddr -> 为linkedit_segment在对应mach-o文件中存储的虚拟地址, 其在加载到内存中后并不会改变为在内存中的u虚拟地址
    // linkedit_segment->fileoff-> 为linkedit_segment在对应mach-o文件中的偏移

    // 字符串表在内存中的虚拟地址的计算
    // 字符串表的在mach-o文件的虚拟地址 - linkedit_segment->vmaddr = symtab_cmd->stroff(字符串表在文件中的偏移量) - linkedit_segment->fileoff(linkedit_segment在文件中的偏移量)
    // 字符串表加载到内存中的虚拟地址 = 字符串表在mach-o文件的虚拟地址 + slide
    // 字符串表加载到内存中的虚拟地址 = slide + linkedit_segment->vmaddr - linkedit_segment->fileoff(linkedit_segment在文件中的偏移量) + symtab_cmd->stroff(字符串表在文件中的偏移量)

    // 符号表在内存中的虚拟地址的计算
    // 符号表的在mach-o文件的虚拟地址 - linkedit_segment->vmaddr = symtab_cmd->symoff(符号表在文件中的偏移量) - linkedit_segment->fileoff(linkedit_segment在文件中的偏移量)
    // 符号表加载到内存中的虚拟地址 = 符号表的虚拟地址在mach-o文件的虚拟地址 + slide
    // 符号表加载到内存中的虚拟地址 = slide + linkedit_segment->vmaddr - linkedit_segment->fileoff(linkedit_segment在文件中的偏移量) + symtab_cmd->symoff(符号表在文件中的偏移量)
  uintptr_t linkedit_base = (uintptr_t)slide + linkedit_segment->vmaddr - linkedit_segment->fileoff;
  
  //计算出符号表的虚拟内存地址
  nlist_t *symtab = (nlist_t *)(linkedit_base + symtab_cmd->symoff);
  //计算出字符串表的虚拟内存地址
  char *strtab = (char *)(linkedit_base + symtab_cmd->stroff);
  //计算出动态符号表的虚拟内存地址
  uint32_t *indirect_symtab = (uint32_t *)(linkedit_base + dysymtab_cmd->indirectsymoff);

  cur = (uintptr_t)header + sizeof(mach_header_t);
  //遍历cmd
  for (uint i = 0; i < header->ncmds; i++, cur += cur_seg_cmd->cmdsize) {
    cur_seg_cmd = (segment_command_t *)cur;
    if (cur_seg_cmd->cmd == LC_SEGMENT_ARCH_DEPENDENT) {
      if (strcmp(cur_seg_cmd->segname, SEG_DATA) != 0 &&
          strcmp(cur_seg_cmd->segname, SEG_DATA_CONST) != 0) {
        continue;
      }
      //LC_SEGMENT(__DATA)
      for (uint j = 0; j < cur_seg_cmd->nsects; j++) {
        //sect与上次相比每次偏移j*sizeof(struct section_t)个字节
        section_t *sect =
          (section_t *)(cur + sizeof(segment_command_t)) + j;
        if ((sect->flags & SECTION_TYPE) == S_LAZY_SYMBOL_POINTERS) {
          //处理S_LAZY_SYMBOL_POINTERS : Section_Header(__DATA, __la_symbol_ptr)
          perform_rebinding_with_section(rebindings, sect, slide, symtab, strtab, indirect_symtab);
        }
        if ((sect->flags & SECTION_TYPE) == S_NON_LAZY_SYMBOL_POINTERS) {
          //处理 S_NON_LAZY_SYMBOL_POINTERS: 
          //Section_Header(__DATA, __nl_symbol_ptr), Section_Header(__DATA, __got)
          perform_rebinding_with_section(rebindings, sect, slide, symtab, strtab, indirect_symtab);
        }
      }
    }
  }
}


```

```c
//找到对应符号的地址，并使用rebinding中replacement替换，并用replaced保存原来对应符号的地址
struct rebinding {
  const char *name;
  void *replacement;
  void **replaced;
};

static void perform_rebinding_with_section(struct rebindings_entry *rebindings,
                                           section_t *section,
                                           intptr_t slide,
                                           nlist_t *symtab,
                                           char *strtab,
                                           uint32_t *indirect_symtab) {
  const bool isDataConst = strcmp(section->segname, "__DATA_CONST") == 0;
    //indirect_symtab为动态符号表, indirect_symtab相当于 uint32_t的数组
    //indirect_symtab + section->reserved1
    //indirect_symbol_indices 相当于indirect_symtab从下标reserved1开始的子数组
  uint32_t *indirect_symbol_indices = indirect_symtab + section->reserved1;

  //indirect_symbol_bindings即为存放符号地址信息的表
  void **indirect_symbol_bindings = (void **)((uintptr_t)slide + section->addr);
  vm_prot_t oldProtection = VM_PROT_READ;
  if (isDataConst) {
    oldProtection = get_protection(rebindings);
    mprotect(indirect_symbol_bindings, section->size, PROT_READ | PROT_WRITE);
  }
  //section_t(section_header)对应的section（例如section(__DATA, __la_symbol_ptr)）是一个数组，每个位置存放1个地址
  //section->size / sizeof(void *) 即为求对应的的section数组共有几个元素
  for (uint i = 0; i < section->size / sizeof(void *); i++) {

    //indirect_symbol_indices即为对应section的动态符号表
    uint32_t symtab_index = indirect_symbol_indices[i];
    if (symtab_index == INDIRECT_SYMBOL_ABS || symtab_index == INDIRECT_SYMBOL_LOCAL ||
        symtab_index == (INDIRECT_SYMBOL_LOCAL   | INDIRECT_SYMBOL_ABS)) {
      continue;
    }
    //处理动态符号相关信息
    //strtab_offset 动态符号在字符串中的偏移
    uint32_t strtab_offset = symtab[symtab_index].n_un.n_strx;
    //symbol_name符号名
    char *symbol_name = strtab + strtab_offset;
    bool symbol_name_longer_than_1 = symbol_name[0] && symbol_name[1];
    struct rebindings_entry *cur = rebindings;
    while (cur) {
      for (uint j = 0; j < cur->rebindings_nel; j++) {
        if (symbol_name_longer_than_1 &&
            strcmp(&symbol_name[1], cur->rebindings[j].name) == 0) {
          if (cur->rebindings[j].replaced != NULL &&indirect_symbol_bindings[i] != cur->rebindings[j].replacement) {
             //保存符号原来的地址到replaced中
            *(cur->rebindings[j].replaced) = indirect_symbol_bindings[i];
          }
          //修改符号对应的地址为新传入的地址
          indirect_symbol_bindings[i] = cur->rebindings[j].replacement;
          goto symbol_loop;
        }
      }
      cur = cur->next;
    }
  symbol_loop:;
  }
  if (isDataConst) {
    int protection = 0;
    if (oldProtection & VM_PROT_READ) {
      protection |= PROT_READ;
    }
    if (oldProtection & VM_PROT_WRITE) {
      protection |= PROT_WRITE;
    }
    if (oldProtection & VM_PROT_EXECUTE) {
      protection |= PROT_EXEC;
    }
    mprotect(indirect_symbol_bindings, section->size, protection);
  }
}


```

### fishhook的使用

1. 通过fishhook，hook住objc_msgSend函数

   ```objc
   void smCallTraceStart() {
       _call_record_enabled = true;
       static dispatch_once_t onceToken;
       dispatch_once(&onceToken, ^{
           //创建同一个线程不同函数共享区的_thread_key
           pthread_key_create(&_thread_key, &release_thread_call_stack);
           //hook objc_msgSend
           fish_rebind_symbols((struct rebinding[6]){
               {"objc_msgSend", (void *)hook_Objc_msgSend, (void **)&orig_objc_msgSend},
           }, 1);
       });
   }
   ```

2. 在hook_objc_msgSend中去记住函数的调用的类，方法名，以及调用消耗的时间

   1. 先在栈中保护寄存器

      ```
      入栈参数，参数寄存器是 x0~ x7。对于 objc_msgSend 方法来说，x0 第一个参数是传入对象，x1 第二个参数是选择器 _cmd。syscall 的 number 会放到 x8 里。
      ```

   2. 调用before_objc_msgSend函数去记录调用的类， 方法名，以及调用的时间点

      ```
      使用 bl label进行函数跳转
      ```

   3. 恢复寄存器

   4. 执行原始的 objc_msgSend，保存返回值。

   5. 再次在栈中保护寄存器

   6. 调用after_objc_msgSend函数获得调用结束的时间点，然后计算出时间消耗，并生成一个记录(smCallRecord)，放到集合中

   7. 恢复寄存器

   ````assembly
   #define call(b, value) \
   __asm volatile ("stp x8, x9, [sp, #-16]!\n"); \
   __asm volatile ("mov x12, %0\n" :: "r"(value)); \
   __asm volatile ("ldp x8, x9, [sp], #16\n"); \
   __asm volatile (#b " x12\n");
   
   #define save() \
   __asm volatile ( \
   "stp x8, x9, [sp, #-16]!\n" \
   "stp x6, x7, [sp, #-16]!\n" \
   "stp x4, x5, [sp, #-16]!\n" \
   "stp x2, x3, [sp, #-16]!\n" \
   "stp x0, x1, [sp, #-16]!\n");
   
   #define load() \
   __asm volatile ( \
   "ldp x0, x1, [sp], #16\n" \
   "ldp x2, x3, [sp], #16\n" \
   "ldp x4, x5, [sp], #16\n" \
   "ldp x6, x7, [sp], #16\n" \
   "ldp x8, x9, [sp], #16\n" );
   
   #define link(b, value) \
   __asm volatile ("stp x8, lr, [sp, #-16]!\n"); \
   __asm volatile ("sub sp, sp, #16\n"); \
   call(b, value); \
   __asm volatile ("add sp, sp, #16\n"); \
   __asm volatile ("ldp x8, lr, [sp], #16\n");
   
   #define ret() __asm volatile ("ret\n");
   
   __attribute__((__naked__))
   static void hook_Objc_msgSend() {
       // Save parameters.
       save()
       
       __asm volatile ("mov x2, lr\n");
       __asm volatile ("mov x3, x4\n");
       
       // Call our before_objc_msgSend.
       call(blr, &before_objc_msgSend)
       
       // Load parameters.
       load()
       
       // Call through to the original objc_msgSend.
       call(blr, orig_objc_msgSend)
       
       // Save original objc_msgSend return value.
       save()
       
       // Call our after_objc_msgSend.
       call(blr, &after_objc_msgSend)
       
       // restore lr
       __asm volatile ("mov lr, x0\n");
       
       // Load original objc_msgSend return value.
       load()
       
       // return
       ret()
   }
   ````

3. 所有的记录都放在共享区， `[SMCallTrace save]`去打印记录集合

   ```
   pthread_getpecific和pthread_setspecific提供了在同一个线程中不同函数间共享数据即线程存储的一种方法
   ```

### 总结

1. rebind_symbols_for_image
   + 找到对应image的`symtab`，`strtab`,`dysymtab`, 以及`section_header(__DATA, __got)`, `section_header(__DATA, __nl_symbol_ptr)`, `section_header(__DATA, __la_symbol_ptr)`
2. perform_rebinding_with_section
   - 通过`section_header`和`symtab`, `dysymtab`, ` indirect_symtab`找到`section(__DATA, __got)`, `section(__DATA, __nl_symbol_ptr)`, `section(__DATA, __la_symbol_ptr)`中对应符号的地址，并用rebinding中replacement替换对应符号的地址，然后将真实的符号地址赋值给replaced

