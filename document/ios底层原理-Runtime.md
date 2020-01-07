## Runtime

### Runtime简介

+ Objective-C是一门动态性比较强的编程语言，跟C、C++等语言有着很大的不同
+ Objective-C的动态性是由Runtime API来支撑的
+ Runtime API提供的接口基本都是C语言的，源码由C\C++\汇编语言编写

### isa简介

+ 要想学习Runtime，首先要了解它底层的一些常用数据结构，比如isa指针

+ 在arm64架构之前，isa就是一个普通的指针，存储着Class、Meta-Class对象的内存地址

+ 从arm64架构开始，对isa进行了优化，变成了一个共用体（union）结构，还使用位域来存储更多的信息

  ![](./images/runtime0.png)

  ![](./images/runtime1.png)

  ![](./images/runtime2.png)


### isa结构理解-通过位设置取值

- 从位取值 

  ```powershell
  例如:二进制数据 0b1010中取出某一位
  
  取出倒数第二位:0
    1010  值
  & 0001  掩码 1 << 0
   -------
    0000 
    
  取出倒数第二位:1
    1010 值
  & 0010 掩码 1 << 1
   -------
    0010 
    
  取出倒数第三位:0
    1010 值
  & 0100 掩码 1 << 2
   -------
    0000 
    
  取出倒数第四位:1
    1010 值
  & 1000 掩码 1 << 3
   -------
    1000
  ```

- 用位存值

  ```powershell
  例如:通过0b1010进行存值
  
  1. 向倒数第一位存值1
    1010 原值
  | 0001 掩码 1 << 0
   -------
    1011 结果
    
  2. 向倒数第一位存值0
    1010 原值
  & 1110 掩码 ~(1 << 0)
   -------
    1010 结果
  
  
  3. 向倒数第二位存值1
    1010 原值
  | 0010 掩码 1 << 1
   -------
    1010 结果
    
  4. 向倒数第二位存值0
    1010 原值
  & 1101 掩码 ~(1 << 1)
   -------
    1000 结果
  
  
  5. 向倒数第三位存值1
    1010 原值
  | 0100 掩码 1 << 2
   -------
    1110 结果
    
  6. 向倒数第三位存值0
    1010 原值
  & 1011 掩码 ~(1 << 2)
   -------
    1010 结果
    
    
  7. 向倒数第四位存值1
    1010 原值
  | 1000 掩码 1 << 3
   -------
    1010 结果
    
  8. 向倒数第四位存值0
    1010 原值
  & 0111 掩码 ~(1 << 3)
   -------
    0010 结果
  
  ```

+ 用位来存储布尔值

  ```objc
  
  @interface MJPerson : NSObject
  - (void)setTall:(BOOL)tall;
  - (void)setRich:(BOOL)rich;
  - (void)setHandsome:(BOOL)handsome;
  
  - (BOOL)isTall;
  - (BOOL)isRich;
  - (BOOL)isHandsome;
  
  @end
  ```

  ```objc
  #import "MJPerson.h"
  
  // &可以用来取出特定的位
  
  // 掩码，一般用来按位与(&)运算的
  //#define MJTallMask 1
  //#define MJRichMask 2
  //#define MJHandsomeMask 4
  
  //#define MJTallMask 0b00000001
  //#define MJRichMask 0b00000010
  //#define MJHandsomeMask 0b00000100
  
  #define MJTallMask (1<<0)
  #define MJRichMask (1<<1)
  #define MJHandsomeMask (1<<2)
  
  @interface MJPerson()
  {
      char _tallRichHansome;
  }
  @end
  
  @implementation MJPerson
  - (instancetype)init
  {
      if (self = [super init]) {
          _tallRichHansome = 0b00000100;
      }
      return self;
  }
  
  - (void)setTall:(BOOL)tall
  {
      if (tall) {
          _tallRichHansome |= MJTallMask;
      } else {
          _tallRichHansome &= ~MJTallMask;
      }
  }
  
  - (BOOL)isTall
  {
      return !!(_tallRichHansome & MJTallMask);
  }
  
  - (void)setRich:(BOOL)rich
  {
      if (rich) {
          _tallRichHansome |= MJRichMask;
      } else {
          _tallRichHansome &= ~MJRichMask;
      }
  }
  
  - (BOOL)isRich
  {
      return !!(_tallRichHansome & MJRichMask);
  }
  
  - (void)setHandsome:(BOOL)handsome
  {
      if (handsome) {
          _tallRichHansome |= MJHandsomeMask;
      } else {
          _tallRichHansome &= ~MJHandsomeMask;
      }
  }
  
  - (BOOL)isHandsome
  {
      return !!(_tallRichHansome & MJHandsomeMask);
  }
  
  @end
  
  ```

  ```objc
  int main(int argc, const char * argv[]) {
      @autoreleasepool {
          MJPerson *person = [[MJPerson alloc] init];
          person.rich = YES;
          person.tall = NO;
          person.handsome = NO;
          NSLog(@"tall:%d rich:%d hansome:%d", person.isTall, person.isRich, person.isHandsome);
      }
      return 0;
  }
  
  ```

### 模仿系统的位运算

```objc
#import "ViewController.h"
typedef enum {
    MJOptionsNone = 0,    // 0b0000
    MJOptionsOne = 1<<0,   // 0b0001
    MJOptionsTwo = 1<<1,   // 0b0010
    MJOptionsThree = 1<<2, // 0b0100
    MJOptionsFour = 1<<3   // 0b1000
} MJOptions;

@interface ViewController ()
@end
@implementation ViewController

- (void)setOptions:(MJOptions)options
{
    if (options & MJOptionsOne) {
        NSLog(@"包含了MJOptionsOne");
    }
    
    if (options & MJOptionsTwo) {
        NSLog(@"包含了MJOptionsTwo");
    }
    
    if (options & MJOptionsThree) {
        NSLog(@"包含了MJOptionsThree");
    }
    
    if (options & MJOptionsFour) {
        NSLog(@"包含了MJOptionsFour");
    }
}

- (void)viewDidLoad {
    [super viewDidLoad];
    [self setOptions: MJOptionsOne | MJOptionsTwo | MJOptionsFour];
}


@end

```



### isa结构理解-位域的基本使用

```objc
@interface MJPerson()
{
    // 位域
    struct {
        char tall : 1; //最低位
        char rich : 1;
        char handsome : 1;
    } _tallRichHandsome;
}
@end
  
@implementation MJPerson

- (void)setTall:(BOOL)tall
{
    _tallRichHandsome.tall = tall;
}

- (BOOL)isTall
{
    return !!_tallRichHandsome.tall;
}

- (void)setRich:(BOOL)rich
{
    _tallRichHandsome.rich = rich;
}

- (BOOL)isRich
{
    return !!_tallRichHandsome.rich;
}

- (void)setHandsome:(BOOL)handsome
{
    _tallRichHandsome.handsome = handsome;
}

- (BOOL)isHandsome
{
    return !!_tallRichHandsome.handsome;
}

@end
```

- `struct _tallRichHandsome`采用了位域的技术，tall，rich，handsome占1位,整个结构体对齐只占一个字节

- `_tallRichHandsome.tall`值为0b0或0b1

  ```
  0b1被扩展为1个字节时，变为0b1111 1111为-1，所以通过`!!`得到真正的结果
  ```



### isa结构理解-共用体和位域

```objc
#define MJTallMask (1<<0)
#define MJRichMask (1<<1)
#define MJHandsomeMask (1<<2)
#define MJThinMask (1<<3)

@interface MJPerson()
{
    union {
        char bits;
        
        struct {
            char tall : 1;
            char rich : 1;
            char handsome : 1;
            char thin : 1;
        };
    } _tallRichHandsome;
}
@end

@implementation MJPerson

- (void)setTall:(BOOL)tall
{
    if (tall) {
        _tallRichHandsome.bits |= MJTallMask;
    } else {
        _tallRichHandsome.bits &= ~MJTallMask;
    }
}

- (BOOL)isTall
{
    return !!(_tallRichHandsome.bits & MJTallMask);
}

- (void)setRich:(BOOL)rich
{
    if (rich) {
        _tallRichHandsome.bits |= MJRichMask;
    } else {
        _tallRichHandsome.bits &= ~MJRichMask;
    }
}

- (BOOL)isRich
{
    return !!(_tallRichHandsome.bits & MJRichMask);
}

- (void)setHandsome:(BOOL)handsome
{
    if (handsome) {
        _tallRichHandsome.bits |= MJHandsomeMask;
    } else {
        _tallRichHandsome.bits &= ~MJHandsomeMask;
    }
}

- (BOOL)isHandsome
{
    return !!(_tallRichHandsome.bits & MJHandsomeMask);
}



- (void)setThin:(BOOL)thin
{
    if (thin) {
        _tallRichHandsome.bits |= MJThinMask;
    } else {
        _tallRichHandsome.bits &= ~MJThinMask;
    }
}

- (BOOL)isThin
{
    return !!(_tallRichHandsome.bits & MJThinMask);
}

@end

```

- 此时结构体位域仅仅是为了增加可读性，没有实际的意义。

- 通过这种结构可以在`bits`中存储更多的信息。

- 共用体`union`

  ```objc
  1. 其大小为内部最大变量的大小
  2. 所有变量占据同一块内存
    
  union Date { 
    int year,
    char month;
    char day;
  }
  year,month,day占据同一块内存
  ```

###  isa结构理解-详解

![](./images/runtime3.png)

- nonpointer

  - 0，代表普通的指针，存储着Class、Meta-Class对象的内存地址
  - 1，代表优化过，使用位域存储更多的信息

- has_assoc

  - 是否有**设置过**关联对象，如果没有，释放时会更快

- has_cxx_dtor

  - 是否有C++的析构函数（.cxx_destruct），如果没有，释放时会更快

    ```objc
    //Destroys an instance without freeing memory. 
    //销毁对象时
    void *objc_destructInstance(id obj) 
    {
        if (obj) {
            // Read all of the flags at once for performance.
            bool cxx = obj->hasCxxDtor();
            bool assoc = obj->hasAssociatedObjects();
    
            // This order is important.
            //1. 有析构函数，要先调用析构函数
            if (cxx) object_cxxDestruct(obj);
            //2. 有关联对象，要先调用释放关联对象
            if (assoc) _object_remove_assocations(obj);
            //3. dealloc对象本身
            obj->clearDeallocating();
        }
    
        return obj;
    }
    ```

    

- shiftcls

  - 存储着Class、Meta-Class对象的内存地址信息

  - `bits & ISA_MASK`的到Class、Meta-Class的内存地址

    ```
    ISA_MASK为0x0000000ffffffff8ULL,其最后四位为1000。
    所以，Class、Meta-Class的内存地址的二进制位的最后三位一定为0。
    因此地址的十六表示的最后一位为0或0
    ```

    ![](./images/runtime4.png)

- magic

  - 用于在调试时分辨对象是否未完成初始化

- weakly_referenced

  - 是否有被弱引用指向过，如果没有，释放时会更快

- deallocating

  - 对象是否正在释放

- extra_rc

  - 里面存储的值是引用计数器减1

- has_sidetable_rc

  - 引用计数器是否过大无法存储在isa中
  - 如果为1，那么引用计数会存储在一个叫SideTable的类的属性中

### Class结构

![](./images/runtime14.png)

- class_rw_t里面的methods、properties、protocols是二维数组，是可读可写的，包含了类的初始内容、分类的内容

  ![](./images/runtime6.png)

- class_ro_t里面的baseMethodList、baseProtocols、ivars、baseProperties是一维数组，是只读的，包含了类的初始内容

  ![](./images/runtime7.png)



### method_array_t

![](./images/runtime11.png)

+ list_array_tt是一个模板类
+ method_array_t继承list_array_tt，是一个二维数组，内部放着method_list_t, method_list_t中放着method_t

### property_array_t

![](./images/runtime12.png)

+ property_array_t继承list_array_tt，是一个二维数组，内部放着property_list_t, property_list_t中放着property_t



### protocol_array_t

![](./images/runtime13.png)

+ protocol_array_t继承list_array_tt，是一个二维数组，内部放着protocol_array_t, protocol_array_t中放着property_ref_t
+ property_ref_t是指向protocol_t的指针

### Method结构

+ method_t是对方法\函数的封装

  ![](./images/runtime8.png)

+ IMP代表函数的具体实现

  ```objc
  typedef id _Nullable (*IMP)(id _Nonull, SEL _Nonull,...)
  ```

+ SEL代表方法\函数名，一般叫做选择器，底层结构跟char *类似

  - 可以通过@selector()和sel_registerName()获得

  - 可以通过sel_getName()和NSStringFromSelector()转成字符串

  - 不同类中相同名字的方法，所对应的方法选择器是相同的

    ```objc
    typedef struct objc_selector *SEL;
    ```

+ types是包含了函数返回值，以及参数类型编码的字符串

### Type Encoding

+ iOS中提供了一个叫做@encode的指令，可以将具体的类型表示成字符串编码

  ![](./images/runtime9.png)

+ 为了简单`MJClassInfo`将方法变成了一维数组

  ![](./images/runtime10.png)

  ```objc
  方法 -(int)test:(int)age height:(float)height的类型编码为i24@0:8i16f20
  objc_msgSend(person,@selector(test:height:),age,height)
  i表示返回值为int类型
  24表示所有参数的字节总数,self + _cmd + age + height = 8 + 8 + 4 + 4 = 24
  @是id的类型编码
  0是代表self参数的起始偏移
  :是SEL的类型编码
  8是代表_cmd参数的起始偏移
  i是int的类型编码
  16是代表age参数的起始偏移
  f是float的类型编码
  20是代表height参数的起始偏移
    
  @encode(int)可获取类型编码
  ```

###cache_t

+ Class内部结构中有个方法缓存（cache_t），用散列表（哈希表）来缓存曾经调用过的方法，可以提高方法的查找速度

  ![](./images/runtime15.png)

  - _buckets是一个散列表, 存储着所有的方法缓存。
  - 等于散列表的长度 -1 
  - _occupied是散列表中已经缓存的方法的数量

### 散列表缓存

- _buckets本质上是一个数组，其内部存放着bucket_t结构体

- bucket_t的_key是方法选择器SEL, _imp是函数的内存地址

- 简单的散列表存储图解

  ![](./images/runtime16.png)

  ```
  1. 对于person对象有personTest, personTest2方法
  2. 那么调用personTest方法后要将personTest存储到_buckets中
  3. 在没有缓存时， _buckets是一个具有默认长度的数组，里面存储的都为NULL
  4. @selector(personTest)&_mask的值作为索引，将对应的方法存储到buckets中
  ```

+ 源码解析:`  objc-cache.mm - > bucket_t * cache_t::find(cache_key_t k, id receiver)`

  + 查找方法缓存 bucket_t

    ![](./images/runtime17.png)

  + 通过hash获取存储位置

    ![](./images/runtime18.png)

  + 获取存储位置，发生hash碰撞时的处理

    ![](./images/runtime19.png)

  + 存储容量不够时，扩容

    ![](./images/runtime20.png)

    ![](./images/runtime21.png)

  + 扩容时，会把原来的缓存直接清空

    ![](./images/runtime22.png)

+ 测试代码

  ```objective-c
  @interface MJPerson : NSObject
  - (void)personTest;
  @end
  
  @implementation MJPerson
  - (void)personTest{}
  @end
  
  @interface MJStudent : MJPerson
  - (void)studentTest;
  @end
  
  @implementation MJStudent
  - (void)studentTest{}
  @end
  
  @interface MJGoodStudent : MJStudent
  - (void)goodStudentTest;
  @end
  
  @implementation MJGoodStudent
  - (void)goodStudentTest{}
  @end
  
  int main(int argc, const char * argv[]) {
      @autoreleasepool {
  
          MJGoodStudent *goodStudent = [[MJGoodStudent alloc] init];
          mj_objc_class *goodStudentClass = (__bridge mj_objc_class *)[MJGoodStudent class];
  
          [goodStudent goodStudentTest];
          [goodStudent studentTest];
  //        [goodStudent personTest];
  //        [goodStudent goodStudentTest];
  //        [goodStudent studentTest];
          cache_t cache = goodStudentClass->cache;
          bucket_t *buckets = cache._buckets;
          for (int i = 0; i <= cache._mask; i++) {
              bucket_t bucket = buckets[i];
              NSLog(@" *****  %s %p", bucket._key, bucket._imp);
          }
          
          NSLog(@"123");
      }
      return 0;
  }
  ```

  ![](./images/runtime23.png)

  ![](./images/runtime24.png)

  ![](./images/runtime25.png)

  ![](./images/runtime26.png)

### objc_msgSend流程

+ OC中的方法调用，其实都是转换为objc_msgSend函数的调用

+ objc_msgSend的执行流程可以分为3大阶段

  - 消息发送
  - 动态方法解析
  - 消息转发

+ 源码阅读顺序

  - objc-msg-arm64.s

    - ENTRY _objc_msgSend
    - b.le	LNilOrTagged
    - CacheLookup NORMAL
    - .macro CacheLookup
    - .macro CheckMiss
    - STATIC_ENTRY  __objc_msgSend_uncached
    - .macro MethodTableLookup
    - __class_lookupMethodAndLoadCache3

  - objc-runtime-new.mm

    - _class_lookupMethodAndLoadCache3
    - lookUpImpOrForward
    - getMethodNoSuper_nolock、search_method_list、log_and_fill_cache
    - cache_getImp、log_and_fill_cache、getMethodNoSuper_nolock、log_and_fill_cache
    - _class_resolveInstanceMethod
    - _objc_msgForward_impcache

  - objc-msg-arm64.s

    - STATIC_ENTRY __objc_msgForward_impcache
    - ENTRY __objc_msgForward

  - Core Foundation

    - `__forwarding__`（不开源）

    

#### 消息发送

![](./images/runtime27.png)

+ receiver通过isa指针找到receiverClass
+ receiverClass通过superclass指针找到superClass
+ 如果是从class_rw_t中查找方法
  - 已经排序的，二分查找
  - 没有排序的，遍历查找

####  动态方法解析

![](./images/runtime28.png)

+ 开发者可以实现以下方法，来动态添加方法实现
  - +resolveInstanceMethod:
  - +resolveClassMethod:
+ 动态解析过后，会重新走“消息发送”的流程
  
- “从receiverClass的cache中查找方法”这一步开始执行
  
+ 实例方法和类方法动态解析测试

  ```objc
  #import "MJPerson.h"
  #import <objc/runtime.h>
  
  @interface MJPerson : NSObject
  - (void)test;
  + (void)test;
  @end
    
  @implementation MJPerson
  
  void c_other(id self, SEL _cmd)
  {
      NSLog(@"c_other - %@ - %@", self, NSStringFromSelector(_cmd));
  }
  - (void)other
  {
      NSLog(@"%s", __func__);
  }
  
  //动态添加实例方法
  + (BOOL)resolveInstanceMethod:(SEL)sel
  {
      if (sel == @selector(test)) {
          // 获取其他方法
          Method method = class_getInstanceMethod(self, @selector(other));
          // 动态添加test方法的实现
           class_addMethod(self, sel,
                          method_getImplementation(method),
                          method_getTypeEncoding(method));
           class_addMethod(self, sel,
                          (IMP)c_other,
                          "v@:"); // "v16@0:8"
  
          // 返回YES代表有动态添加方法
          return YES;
      }
      return [super resolveInstanceMethod:sel];
  }
  
  //动态添加类方法
  + (BOOL)resolveClassMethod:(SEL)sel
  {
      if (sel == @selector(test)) {
          // 第一个参数是object_getClass(self)
          class_addMethod(object_getClass(self), sel, (IMP)c_other, "v16@0:8");
          return YES;
      }
      return [super resolveClassMethod:sel];
  }
  
  @end
  
  ```

+ 查看method_t结构

  ![](./images/runtime29.png)

#### 消息转发

![](./images/runtime30.png)

+ 开发者可以在forwardInvocation:方法中自定义任何逻辑
+ 以上方法都有对象方法、类方法2个版本（前面可以是加号+，也可以是减号-

+ 其他对象来处理这个消息

  ```objc
  @implementation MJPerson
  
  //处理实例方法
  - (id)forwardingTargetForSelector:(SEL)aSelector
  {
      if (aSelector == @selector(test)) {
          // objc_msgSend([[MJCat alloc] init], aSelector)
          // 返回一个能够处理该消息的对象,
          return [[MJCat alloc] init];
      }
      return [super forwardingTargetForSelector:aSelector];
  }
  
  // 方法签名：返回值类型、参数类型
  - (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector
  {
      if (aSelector == @selector(test)) {
          return [NSMethodSignature signatureWithObjCTypes:"v16@0:8"];
      }
      return [super methodSignatureForSelector:aSelector];
  }
  
  // NSInvocation封装了一个方法调用，包括：方法调用者、方法名、方法参数
  //    anInvocation.target 方法调用者
  //    anInvocation.selector 方法名
  //    [anInvocation getArgument:NULL atIndex:0]
  - (void)forwardInvocation:(NSInvocation *)anInvocation
  {
  //    anInvocation.target = [[MJCat alloc] init];
  //    [anInvocation invoke];
      [anInvocation invokeWithTarget:[[MJCat alloc] init]];
  }
  
  //处理类方法
  + (id)forwardingTargetForSelector:(SEL)aSelector
  {
      // objc_msgSend([[MJCat alloc] init], @selector(test))
      // [[[MJCat alloc] init] test]
      //即使是类方法，也可以返回一个实例对象处理这个消息
      if (aSelector == @selector(test)) return [[MJCat alloc] init];
  
      return [super forwardingTargetForSelector:aSelector];
  }
  
  + (NSMethodSignature *)methodSignatureForSelector:(SEL)aSelector
  {
    if (aSelector == @selector(test)) return [NSMethodSignature signatureWithObjCTypes:"v@:"];
     
      return [super methodSignatureForSelector:aSelector];
  }
  
  + (void)forwardInvocation:(NSInvocation *)anInvocation
  {
      NSLog(@"1123");
  }
  @end
  
  ```

#### 源码解析

1. objc_msgSend: x0是receiver, x1是_cmd

   ```objc
     .data
   	.align 3
   	.globl _objc_debug_taggedpointer_classes
   _objc_debug_taggedpointer_classes:
   	.fill 16, 8, 0
   	.globl _objc_debug_taggedpointer_ext_classes
   _objc_debug_taggedpointer_ext_classes:
   	.fill 256, 8, 0
   
   ENTRY _objc_msgSend  //入口  
   	UNWIND _objc_msgSend, NoFrame
   	MESSENGER_START
     //1. x0寄存器存放receiver,  比较x0寄存器和0的差值
   	cmp	x0, #0		
     //2. 如果x0寄存器的值<=0，那么跳转到LNilOrTagged,返回nil结束
   	b.le	LNilOrTagged		
     //3. x0存放receiver, 取receiver的前8个字节(isa)放到x13寄存器
   	ldr	x13, [x0]		
     //4. isa & ISA_MASK = class or meta-class地址, x16为class或meta-class
   	and	x16, x13, #ISA_MASK	
   LGetIsaDone:
     //6. 从缓存中查找imp
   	CacheLookup NORMAL		
   
   LNilOrTagged:
   	b.eq	LReturnZero		// nil check
   
   	// tagged
   	mov	x10, #0xf000000000000000
   	cmp	x0, x10
   	b.hs	LExtTag
   	adrp	x10, _objc_debug_taggedpointer_classes@PAGE
   	add	x10, x10, _objc_debug_taggedpointer_classes@PAGEOFF
   	ubfx	x11, x0, #60, #4
   	ldr	x16, [x10, x11, LSL #3]
     //5. 跳转LGetIsaDone
   	b	LGetIsaDone    
   
   LExtTag:
   	// ext tagged
   	adrp	x10, _objc_debug_taggedpointer_ext_classes@PAGE
   	add	x10, x10, _objc_debug_taggedpointer_ext_classes@PAGEOFF
   	ubfx	x11, x0, #52, #8
   	ldr	x16, [x10, x11, LSL #3]
   	b	LGetIsaDone
   	
   LReturnZero:
   	// x0 is already zero
   	mov	x1, #0
   	movi	d0, #0
   	movi	d1, #0
   	movi	d2, #0
   	movi	d3, #0
   	MESSENGER_END_NIL
   	ret
   	
   	END_ENTRY _objc_msgSend  //结束
   ```

2. CacheLoopUp:从缓存中查找

   ```objc
   .macro CacheLookup
     
     //一些结构:
     //objc_class {  Class isa; Class superclass; cache_t cache; ... }
     //cache_t { bucket_t *_buckets;  mask_t _mask; mask_t _occupied; }
     //bucket_t {cache_key_t _key; IMP _imp;}
     
   	// 1. x1 = SEL, x16 = isa , #CACHE = 0x10
     // 2. 因此 [x16, #CACHE]即为cache
     // 3. 将cache存放到x10, x11中,由cache_t结构可知
     //    3.1 x10: _buckets , x11低32位:_mask, x11高32位:_occupied
   	ldp	x10, x11, [x16, #CACHE]	// x10 = buckets, x11 = occupied|mask
     // 4. w1: SEL  w11:mask  w12 = _cmd & mask
   	and	w12, w1, w11
     // 5. x12 = buckets + ((_cmd & mask)<<4)
     //   buckets为缓存数组的第一个下标的地址
     //   _cmd & mask计算出的该SEL在缓存数组buckets中的位置，所以要获得该位置的地址需要偏移
     //   (_cmd & mask)*sizeof(bucket_t)个字节 
     //   (_cmd & mask)*sizeof(bucket_t) = (_cmd & mask) * 16 == ((_cmd & mask)<<4)
     //   因此 此时x12的值为该方法对应的缓存地址
   	add	x12, x10, x12, LSL #4	
     // 6. 通过x12取出bucket,并将_key放到x9中, 将_IMP放到x17中
   	ldp	x9, x17, [x12]		// {x9, x17} = *bucket
     // 7. 比较x9, x1的差值
   1:	cmp	x9, x1
     // 8. 如果x9不等于x1, 则跳转到到2
   	b.ne	2f			//     scan 
     // 9. 如果x9等于x1，找到缓存,跳转到CacheHit
   	CacheHit $0			// call or return imp
   	
   2:	// not hit: x12 = not-hit bucket
     //10. 没有找到缓存
   	CheckMiss $0			// miss if bucket->sel == 0
   	cmp	x12, x10		// wrap if bucket == buckets
   	b.eq	3f
   	ldp	x9, x17, [x12, #-16]!	// {x9, x17} = *--bucket
   	b	1b			// loop
   
   3:	// wrap: x12 = first bucket, w11 = mask
   	add	x12, x12, w11, UXTW #4	// x12 = buckets+(mask<<4)
   
   	// Clone scanning loop to miss instead of hang when cache is corrupt.
   	// The slow path may detect any corruption and halt later.
   
   	ldp	x9, x17, [x12]		// {x9, x17} = *bucket
   1:	cmp	x9, x1			// if (bucket->sel != _cmd)
   	b.ne	2f			//     scan more
   	CacheHit $0			// call or return imp
   	
   2:	// not hit: x12 = not-hit bucket
   	CheckMiss $0			// miss if bucket->sel == 0
   	cmp	x12, x10		// wrap if bucket == buckets
   	b.eq	3f
   	ldp	x9, x17, [x12, #-16]!	// {x9, x17} = *--bucket
   	b	1b			// loop
   
   3:	// double wrap
   	JumpMiss $0
   	
   .endmacro
   ```

   ```objc
   //找到缓存
   .macro CacheHit
   .if $0 == NORMAL
   	MESSENGER_END_FAST
   	//直接调用实现 x17:imp
   	br	x17			// call imp
   .elseif $0 == GETIMP
   	mov	x0, x17			// return imp
   	ret
   .elseif $0 == LOOKUP
   	ret				// return imp via x17
   .else
   .abort oops
   .endif
   .endmacro
   
   //没有找到缓存
   .macro CheckMiss
   	// miss if bucket->sel == 0
   .if $0 == GETIMP
   	cbz	x9, LGetImpMiss
   .elseif $0 == NORMAL
     //调用__objc_msgSend_uncached方法
   	cbz	x9, __objc_msgSend_uncached
   .elseif $0 == LOOKUP
   	cbz	x9, __objc_msgLookup_uncached
   .else
   .abort oops
   .endif
   .endmacro
   ```

   ```objc
   STATIC_ENTRY __objc_msgSend_uncached
   	UNWIND __objc_msgSend_uncached, FrameWithNoSaves
   
   	// THIS IS NOT A CALLABLE C FUNCTION
   	// Out-of-band x16 is the class to search
     
   	//调用MethodTableLookup
   	MethodTableLookup
   	br	x17
   END_ENTRY __objc_msgSend_uncached
   ```

   ```objc
   
   .macro MethodTableLookup
   	
   	// push frame
   	stp	fp, lr, [sp, #-16]!
   	mov	fp, sp
   
   	// save parameter registers: x0..x8, q0..q7
   	//寄存器保护
   	sub	sp, sp, #(10*8 + 8*16)
   	stp	q0, q1, [sp, #(0*16)]
   	stp	q2, q3, [sp, #(2*16)]
   	stp	q4, q5, [sp, #(4*16)]
   	stp	q6, q7, [sp, #(6*16)]
   	stp	x0, x1, [sp, #(8*16+0*8)]
   	stp	x2, x3, [sp, #(8*16+2*8)]
   	stp	x4, x5, [sp, #(8*16+4*8)]
   	stp	x6, x7, [sp, #(8*16+6*8)]
   	str	x8,     [sp, #(8*16+8*8)]
   
   	// receiver and selector already in x0 and x1
   	mov	x2, x16
   	// 跳转_class_lookupMethodAndLoadCache3函数
   	bl	__class_lookupMethodAndLoadCache3
   
   	// imp in x0
   	mov	x17, x0
   	
   	// restore registers and return
   	//寄存器恢复
   	ldp	q0, q1, [sp, #(0*16)]
   	ldp	q2, q3, [sp, #(2*16)]
   	ldp	q4, q5, [sp, #(4*16)]
   	ldp	q6, q7, [sp, #(6*16)]
   	ldp	x0, x1, [sp, #(8*16+0*8)]
   	ldp	x2, x3, [sp, #(8*16+2*8)]
   	ldp	x4, x5, [sp, #(8*16+4*8)]
   	ldp	x6, x7, [sp, #(8*16+6*8)]
   	ldr	x8,     [sp, #(8*16+8*8)]
   
   	mov	sp, fp
   	ldp	fp, lr, [sp], #16
   
   .endmacro
   ```

3. _class_lookupMethodAndLoadCache3函数

   ```objc
   IMP _class_lookupMethodAndLoadCache3(id obj, SEL sel, Class cls)
   {
       return lookUpImpOrForward(cls, sel, obj, 
                                 YES/*initialize*/, NO/*cache*/, YES/*resolver*/);
   }
   ```

4. lookUpImpOrForward:消息发送，动态解析，消息转发

   ```c
   IMP lookUpImpOrForward(Class cls, SEL sel, id inst, 
                          bool initialize, bool cache, bool resolver)
   {
       IMP imp = nil;
       bool triedResolver = NO;
   
       runtimeLock.assertUnlocked();
   
       // Optimistic cache lookup
       if (cache) {
           //从缓存中查找方法
           imp = cache_getImp(cls, sel);
           if (imp) return imp;
       }
       runtimeLock.read();
   
       if (!cls->isRealized()) {
           // Drop the read-lock and acquire the write-lock.
           // realizeClass() checks isRealized() again to prevent
           // a race while the lock is down.
           runtimeLock.unlockRead();
           runtimeLock.write();
   
           realizeClass(cls);
   
           runtimeLock.unlockWrite();
           runtimeLock.read();
       }
   
       if (initialize  &&  !cls->isInitialized()) {
           runtimeLock.unlockRead();
           _class_initialize (_class_getNonMetaClass(cls, inst));
           runtimeLock.read();
           // If sel == initialize, _class_initialize will send +initialize and 
           // then the messenger will send +initialize again after this 
           // procedure finishes. Of course, if this is not being called 
           // from the messenger then it won't happen. 2778172
       }
   
       
    retry:    
       runtimeLock.assertReading();
       // Try this class's cache.
       // 从缓存中查找方法
       imp = cache_getImp(cls, sel);
       // 如果找到imp，返回imp
       if (imp) goto done;
   
       // Try this class's method lists.
       // 尝试从类的方法列表中查找方法
       {
           Method meth = getMethodNoSuper_nolock(cls, sel);
           if (meth) {
               //如果查到方法，则将该方法缓存下来
               log_and_fill_cache(cls, meth->imp, sel, inst, cls);
               imp = meth->imp;
               goto done;
           }
       }
   
       // Try superclass caches and method lists.
       // 根据superclass循环从父类中查找
       {
           unsigned attempts = unreasonableClassCount();
           for (Class curClass = cls->superclass;
                curClass != nil;
                curClass = curClass->superclass)
           {
               // Halt if there is a cycle in the superclass chain.
               if (--attempts == 0) {
                   _objc_fatal("Memory corruption in class list.");
               }
               
               // Superclass cache.
               // 先从父类的缓存查找
               imp = cache_getImp(curClass, sel);
               if (imp) {
                   if (imp != (IMP)_objc_msgForward_impcache) {
                       // Found the method in a superclass. Cache it in this class.
                       //查找到方法时，缓存到cls中
                       log_and_fill_cache(cls, imp, sel, inst, curClass);
                       goto done;
                   }
                   else {
                       // Found a forward:: entry in a superclass.
                       // Stop searching, but don't cache yet; call method 
                       // resolver for this class first.
                       break;
                   }
               }
               
               // Superclass method list.
               // 从父类的方法列表中查找
               Method meth = getMethodNoSuper_nolock(curClass, sel);
               if (meth) {
                   //查找到方法时，缓存到cls中
                   log_and_fill_cache(cls, meth->imp, sel, inst, curClass);
                   imp = meth->imp;
                   goto done;
               }
           }
       }
   
       // No implementation found. Try method resolver once.
       // 如果没有找到实现，那么尝试一次动态方法解析，并重新走消息转发机制
       if (resolver  &&  !triedResolver) {
           runtimeLock.unlockRead();
           _class_resolveMethod(cls, sel, inst);
           runtimeLock.read();
           // Don't cache the result; we don't hold the lock so it may have 
           // changed already. Re-do the search from scratch instead.
           triedResolver = YES;
           //重新走消息转发机制
           goto retry;
       }
   
       // No implementation found, and method resolver didn't help. 
       // Use forwarding.
       // 进入消息转发流程
       imp = (IMP)_objc_msgForward_impcache;
       //缓存到cls中
       cache_fill(cls, sel, imp, inst);
    done:
       runtimeLock.unlockRead();
   
       return imp;
   }
   
   ```
   


109-Runtime21-objc_msgSend04-动态方法解析01.vep

110-Runtime21-objc_msgSend04-动态方法解析02.vep

111-Runtime21-objc_msgSend04-动态方法解析03.vepz

day14

112-Runtime24-objc_msgSend07-消息转发01.vep

113-Runtime25-objc_msgSend08-消息转发02.vep

114-Runtime26-objc_msgSend09-消息转发03.vep

115-Runtime27-objc_msgSend10-消息转发04.vep

116-Runtime28-objc_msgSend11-消息转发05.vep

117-Runtime29-objc_msgSend12-总结

118-Runtime30-super01

119-Runtime31-super02

120-Runtime32-答疑.vep

day15

121-Runtime33-class面试题01.vep

122-Runtime34-class面试题02.vep

123-Runtime35-super面试题01.vep

124-Runtime36-super面试题02.vep

125-Runtime37-super面试题03.vep

126-Runtime38-super面试题04.vep

127-Runtime39-super面试题05.vep

128-Runtime40-答疑.vep

day16

129-Runtime41-LLVM的中间代码.vep

130-Runtime42-API01-类.vep

131-Runtime43-API02-成员变量01.vep

132-Runtime44-API02-成员变量02.vep

133-Runtime45-API02-成员变量03.vep

134-Runtime46-API03-方法01.vep

135-Runtime47-总结.ve

day17

136-Runtime48-API03-方法02.vep

137-Runtime49-API03-方法03.vep