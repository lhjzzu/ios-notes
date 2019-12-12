## Mach-O文件


### Mach-O

+ Mach-O是Mach object的缩写，是Mac\iOS上用于存储程序、库的标准格式

+ 属于Mach-O格式的文件类型有

  ![](./images/mach-o文件0.png)

+ 可以在xnu源码中，查看到Mach-O格式的详细定义（https://opensource.apple.com/tarballs/xnu/）

  + EXTERNAL_HEADERS/mach-o/fat.h
  + EXTERNAL_HEADERS/mach-o/loader.h

### 常见的Mach-O文件类型

+ MH_OBJECT
  - 目标文件（.o）
  - 静态库文件(.a），静态库其实就是N个.o合并在一起
+ MH_EXECUTE：可执行文件
  - .app/xx
+ MH_DYLIB：动态库文件
  - .dylib
  - .framework/xx
+ MH_DYLINKER: 动态链接编辑器
  - /usr/lib/dyld
+ MH_DSYM：存储着二进制文件符号信息的文件
  - .dSYM/Contents/Resources/DWARF/xx（常用于分析APP的崩溃信息）  

### 在Xcode中查看target的Mach-O类型

![](./images/mach-o文件1.png)



### Universal Binary（通用二进制文件）

+ 通用二进制文件
  - 同时适用于多种架构的二进制文件
  - 包含了多种不同架构的独立的二进制文件
+ 因为需要储存多种架构的代码，通用二进制文件通常比单一平台二进制的程序要大
+ 由于两种架构有共同的一些资源，所以并不会达到单一版本的两倍之多
+ 由于执行过程中，只调用一部分代码，运行起来也不需要额外的内存
+ 因为文件比原来的要大，也被称为“胖二进制文件”（Fat Binary）

### xcode支持多架构

![](./images/mach-o文件2.png)

+ 生成的文件的架构，即为Architechtures和Valid Architechtures的交集
+ `$(ARCHS_STANDARD)`即为默认支持的架构

### 窥探Mach-O的结构

+ 命令行工具

  + file：查看Mach-O的文件类型

    ```shell
    $ file  TestApp
    
    TestApp: Mach-O universal binary with 2 architectures: [arm_v7:Mach-O executable arm_v7] [arm64:Mach-O 64-bit executable arm64]
    TestApp (for architecture armv7):	Mach-O executable arm_v7
    TestApp (for architecture arm64):	Mach-O 64-bit executable arm64
    ```

  + otool：查看Mach-O特定部分和段的内容

    ```shell
      -f print the fat headers
    	-a print the archive header
    	-h print the mach header
    	-l print the load commands
    	-L print shared libraries used
    	-D print shared library id name
    	-t print the text section (disassemble with -v)
    	-p <routine name>  start dissassemble from routine name
    	-s <segname> <sectname> print contents of section
    	-d print the data section
    	-o print the Objective-C segment
    	-r print the relocation entries
    	-S print the table of contents of a library (obsolete)
    	-T print the table of contents of a dynamic shared library (obsolete)
    	-M print the module table of a dynamic shared library (obsolete)
    	-R print the reference table of a dynamic shared library (obsolete)
    	-I print the indirect symbol table
    	-H print the two-level hints table (obsolete)
    	-G print the data in code table
    	-v print verbosely (symbolically) when possible
    	-V print disassembled operands symbolically
    	-c print argument strings of a core file
    	-X print no leading addresses or headers
    	-m don't use archive(member) syntax
    	-B force Thumb disassembly (ARM objects only)
    	-q use llvm's disassembler (the default)
    	-Q use otool(1)'s disassembler
    	-mcpu=arg use `arg' as the cpu for disassembly
    	-j print opcode bytes
    	-P print the info plist section as strings
    	-C print linker optimization hints
    	--version print the version of /Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/otool
    ```
1. 打印fat header : `$  otool -f 文件路径`
  
   ```shell
       $ otool -f TestApp
       
       Fat headers
       fat_magic 0xcafebabe
       nfat_arch 2
       architecture 0
           cputype 12
           cpusubtype 9
           capabilities 0x0
           offset 16384
           size 533456
           align 2^14 (16384)
       architecture 1
           cputype 16777228
           cpusubtype 0
           capabilities 0x0
           offset 557056
           size 623712
           align 2^14 (16384)
   ```
   
2. 打印mach header：` $ otool -h 文件路径`
  
   ```shell
       $   otool -h TestApp
       Mach header
             magic cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags
        0xfeedface      12          9  0x00           2    26       3156 0x00200085
       Mach header
             magic cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags
        0xfeedfacf 16777228          0  0x00           2    26       3736 0x00200085
   ```
   
3. 打印load commands: `$ otool -l 文件路径`
  
   ```shell
       $ otool -l TestApp
       
       TestApp (architecture armv7):
       Mach header
             magic cputype cpusubtype  caps    filetype ncmds sizeofcmds      flags
        0xfeedface      12          9  0x00           2    26       3156 0x00200085
       Load command 0
             cmd LC_SEGMENT
         cmdsize 56
         segname __PAGEZERO
          vmaddr 0x00000000
         ...
         ...
         Load command 24
             cmd LC_DATA_IN_CODE
         cmdsize 16
         dataoff 340376
        datasize 16
       Load command 25
             cmd LC_CODE_SIGNATURE
         cmdsize 16
         dataoff 596848
        datasize 26864
        
        $ otool -l TestApp | grep crypt
       
          cryptoff 16384
           cryptsize 196608
             cryptid 0
            cryptoff 16384
           cryptsize 229376
             cryptid 0
        ## 查看是否加密
   ```
   
4. 打印依赖的动态库: ` $ otool -L 文件路径`
  
   ```shell
       $ otool -L TestApp
       
       TestApp (architecture armv7):
       	/System/Library/Frameworks/CoreGraphics.framework/CoreGraphics (compatibility version 64.0.0, current version 1251.12.0)
       	....
       	/System/Library/Frameworks/CoreFoundation.framework/CoreFoundation (compatibility version 150.0.0, current version 1570.15.0)
       TestApp (architecture arm64):
       	/System/Library/Frameworks/CoreGraphics.framework/CoreGraphics (compatibility version 64.0.0, current version 1251.12.0)
       	.....
       	/System/Library/Frameworks/CoreFoundation.framework/CoreFoundation (compatibility version 150.0.0, current version 1570.15.0)
   ```
   
   
   
+ lipo：常用于多架构Mach-O文件的处理
  
  + 查看架构信息：lipo  -info  文件路径
  
    ```shell
      $ lipo -info  TestApp
      Architectures in the fat file: TestApp are: armv7 arm64
    ```
  
  + 导出某种特定架构：lipo  文件路径  -thin  架构类型  -output  输出文件路径
  
    ```shell
      $ lipo TestApp -thin armv7 -output TestApp_armv7
      $ lipo TestApp -thin arm64 -output TestApp_arm64
    ```
  
  + 合并多种架构：lipo  -create 文件路径1  文件路径2  -output  输出文件路径
  
    ```shell
      $ lipo -create TestApp_armv7 TestApp_arm64 -output TestApp_2
    ```
  
+ GUI工具
  
  - MachOView（https://github.com/gdbinit/MachOView）

### Mach-O的基本结构

+ 官方描述:  https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/MachOTopics/0-Introduction/introduction.html

+ 一个Mach-O文件包含3个主要区域

  - Header: 文件类型、目标架构类型等

  - Load commands: 描述文件在虚拟内存中的逻辑结构、布局

  - Raw segment data: 在Load commands中定义的Segment的原始数据

    ![](./images/mach-o文件3.png)



### dyld和Mach-O

+ dyld用于加载以下类型的Mach-O文件

  - MH_EXECUTE
  - MH_DYLIB
  - MH_BUNDLE

+ APP的可执行文件、动态库都是由dyld负责加载的

+ 在dyld的原码的dyld.cpp文件中

  ```c++
  loadPhase6方法中有下面这段代码
   // only MH_BUNDLE, MH_DYLIB, and some MH_EXECUTE can be dynamically loaded
  const mach_header* mh = (mach_header*)firstPages;
  		switch ( mh->filetype ) {
  			case MH_EXECUTE:
  			case MH_DYLIB:
  			case MH_BUNDLE:
  				break;
  			default:
  				throw "mach-o, but wrong filetype";
  }
  
  ```

+ dyld是`MH_DYLINKER`类型的Mach-O文件





 