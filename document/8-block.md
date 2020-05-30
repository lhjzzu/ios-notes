## block

### block底层结构

+ block本质上也是一个OC对象，它内部也有个isa指针

+ block是封装了函数调用以及函数调用环境的OC对象

+ block的底层结构如下图所示

  ![](./images/block0.png)

+ 查看block的底层结构

  1. 编译测试代码，生成底层结构

     ```objc
     int main(int argc, const char * argv[]) {
         @autoreleasepool {
             int age = 10;
             void (^block)(int, int) =  ^(int a , int b){
                 NSLog(@"this is a block! -- %d", age);
                 NSLog(@"this is a block!");
                 NSLog(@"this is a block!");
                 NSLog(@"this is a block!");
             };        
             block(10, 10);
         }
         return 0;
     }
     
     ```

     ```shell
     $ xcrun  -sdk  iphoneos  clang  -arch  arm64  -rewrite-objc main.m -o main-arm64.cpp
     ```

  2. 编译后,底层核心代码如下

     - block的结构如下

       ```objc
       struct __main_block_impl_0 {
         struct __block_impl impl;
         struct __main_block_desc_0* Desc;
         int age;
         __main_block_impl_0(void *fp, struct __main_block_desc_0 *desc, int _age, int flags=0) : age(_age) {
           impl.isa = &_NSConcreteStackBlock;
           impl.Flags = flags;
           impl.FuncPtr = fp;
           Desc = desc;
         }
       };
       ```

       - 包含`__block_impl`,  `__main_block_desc_0`, 变量`age`,以及构造函数

     - __block_impl结构如下

       ```objc
       struct __block_impl {
         void *isa;
         int Flags;
         int Reserved;
         void *FuncPtr;
       };
       ```

       - isa说明是一个OC对象
       - block要执行的代码被包装成一个函数，FuncPrt指针指向该函数。

     - __main_block_desc_0结构

       ```objc
       static struct __main_block_desc_0 {
         size_t reserved;
         size_t Block_size;
         void (*copy)(struct __main_block_impl_0*, struct __main_block_impl_0*);
         void (*dispose)(struct __main_block_impl_0*);
       } __main_block_desc_0_DATA = { 0, sizeof(struct __main_block_impl_0)};
       ```

       - 对block的描述，如果捕获的是基本数据类型，不涉及内存管理
       - 如果涉及到内存管理，它还包括copy，dispose函数

     - 此时block的结构可以表示为下面的结构

       ```objc
       struct __main_block_impl_0 {
         void *isa;
         int Flags;
         int Reserved;
         void *FuncPtr;
         size_t reserved;
         size_t Block_size;
         void (*copy)(void*, void*);
         void (*dispose)(void*)
         variables
       }
       ```

       

     - block要执行的代码，被包装成一个函数,其地址放在FuncPtr指针中

       ```objc
       static void __main_block_func_0(struct __main_block_impl_0 *__cself, int a, int b) {
         int age = __cself->age; // bound by copy
       
                   NSLog((NSString *)&__NSConstantStringImpl__var_folders_2r__m13fp2x2n9dvlr8d68yry500000gn_T_main_3f4c4a_mi_0, age);
                   NSLog((NSString *)&__NSConstantStringImpl__var_folders_2r__m13fp2x2n9dvlr8d68yry500000gn_T_main_3f4c4a_mi_1);
                   NSLog((NSString *)&__NSConstantStringImpl__var_folders_2r__m13fp2x2n9dvlr8d68yry500000gn_T_main_3f4c4a_mi_2);
                   NSLog((NSString *)&__NSConstantStringImpl__var_folders_2r__m13fp2x2n9dvlr8d68yry500000gn_T_main_3f4c4a_mi_3);
               }
       ```

     - 测试代码编译后对应的代码

       ```objc
       int main(int argc, const char * argv[]) {
           /* @autoreleasepool */ { __AtAutoreleasePool __autoreleasepool; 
               int age = 10;
               /*原代码
               void (*block)(int, int) = ((void (*)(int, int))&__main_block_impl_0((void *)__main_block_func_0, &__main_block_desc_0_DATA, age));
               ((void (*)(__block_impl *, int, int))((__block_impl *)block)->FuncPtr)((__block_impl *)block, 10, 10);
       
               */
                 //代码精简后
                 //1. 定义block
               void (*block)(int, int) = 
                 &__main_block_impl_0(
                 __main_block_func_0, 
                 &__main_block_desc_0_DATA,
                 age));
                                   
                 //2. 执行block
                 block->FuncPtr(block, 10, 10);
           }
           return 0;
       }
       ```

       + 调用block结构体`__main_block_impl_0`的构造函数，创建block
       + 通过block的函数指针来调用函数

+ 调试查看block的底层结构

  ![](./images/block1.png)

  ![](./images/block2.png)

  - 查看block的底层结构，确实与上面分析一致
  - FuncPtr指针确实存储的是blokc要执行的函数的地址

### auto局部变量

+ auto用来修饰局部变量，会被block捕获，是值传递。不能在外面修改block内部捕获的变量的值。

+ 捕获基本变量测试代码

  ```objective-c
  int main(int argc, const char * argv[]) {
      @autoreleasepool {
          //auto int age = 10;
          //不能在外面修改block内部捕获的变量的值
          int age = 10;
          void (^block)(void) =  ^{
              //age = 20 
              //错误，不能在内部修改变量
              //Variable is not assignable (missing __block type specifier)
              NSLog(@"age is %d", age);
          };
          age = 20;
          block();
      }
      return 0;
  }
  
  //打印结果
  age is 10
  ```

+ 查看底层实现原理

  ![](./images/block3.png)

  ![](./images/block4.png)

  ![](./images/block5.png)

+ 关系图

  ![](./images/block6.png)

+ 捕获对象指针变量测试代码

  ```objc
  @implementation MJPerson
  
  - (void)test
  {
      void (^block)(void) = ^{
          NSLog(@"-------%d", [self name]);
      };
      block();
  }
  
  - (instancetype)initWithName:(NSString *)name
  {
      if (self = [super init]) {
          self.name = name;
      }
      return self;
  }
  @end
  
  //可以在外部修改name的值，但是self本身并没有改变
  ```

+ 查看底层实现原理

  ![](./images/block11.png)

  - block里面访问`_name`时，不是捕获`_name`而是捕获`self`

### static局部变量

+ static修饰的局部变量，会被block捕获，是指针传递。能够在外部修改block内部捕获的变量的值。

+ 测试代码

  ```objc
  int main(int argc, const char * argv[]) {
      @autoreleasepool {
          //能够在外部修改block内部捕获的变量的值
          static int age = 10;
          void (^block)(void) =  ^{
              // 可以在内部修改 age = 30;
              NSLog(@"age is %d", age);
          };
          age = 20;
          block();
      }
      return 0;
  }
  
  //打印结果为
  age is 20
  ```

+ 查看底层实现原理

  ![](./images/block7.png)

  ![](./images/block8.png)

  ![](./images/block9.png)

  

###  全局变量

+ 全局变量不会被block捕获，在block内部是直接访问全局变量

+ 测试代码

  ```objc
  int age = 10;
  int height = 10;
  int main(int argc, const char * argv[]) {
      @autoreleasepool {
      
          void (^block)(void) =  ^{
              NSLog(@"age is %d, height is %d", age, height);
          };
          age = 20;
          height = 20;
          block();
      }
      return 0;
  }
  
  //打印结果
  age is 20, height is 20
  ```

+ 查看底层实现原理

  ![](./images/block10.png)

  

  

### block的变量捕获小结

+ 为了保证block内部能够正常访问外部的变量，block有个变量捕获机制

  ![](./images/block12.png)

+ 为什么auto局部变量是值捕获，而static变量是地址捕获?

  - 普通auto局部变量和static局部变量在作用域外面无法直接被访问
  - 普通的auto修饰的基本变量出了作用域后就被释放了。而static局部变量出了作用域后，虽然无法拿到变量名去直接访问。但是其仍然在内存中。

### block的类型

+ block有3种类型，可以通过调用class方法或者isa指针查看具体类型，最终都是继承自NSBlock类型

  - `__NSGlobalBlock__ （ _NSConcreteGlobalBlock ）`
  - `__NSStackBlock__ （ _NSConcreteStackBlock ）`
  - `__NSMallocBlock__ （ _NSConcreteMallocBlock ）`

   ![](./images/block13.png)

- 获取block类型

  ```objc
   // __NSGlobalBlock__ : __NSGlobalBlock : NSBlock : NSObject
  void (^block)(void) = ^{
     NSLog(@"Hello");
  }; 
  NSLog(@"%@", [block class]);
  NSLog(@"%@", [[block class] superclass]);
  NSLog(@"%@", [[[block class] superclass] superclass]);
  NSLog(@"%@", [[[[block class] superclass] superclass] superclass]);
  
  //__NSGlobalBlock__
  //__NSGlobalBlock
  //NSBlock
  //NSObject
  ```

+ `__NSGlobalBlock__`

  - 没有访问auto变量的都是全局block

    ```objc
    //情形1
    void test() {
      void (^block1)(void) = ^{
                NSLog(@"Hello");
      };
    }
    
    //情形2
    int a = 10;//全局变量
    void test() {
      void (^block2)(void) = ^{
                NSLog(@"Hello %d", a);
    };
    }
    
    //情形3
    void test() {
      static int a = 10;
      void (^block3)(void) = ^{
                NSLog(@"Hello %d", a);
      };
    }
    ```

+ `__NSStackBlock__`

  - 访问了auto变量

    ```objc
    //情形1
    void test() {
      int a = 10;//auto局部变量
      void (^block)(void) = ^{
                NSLog(@"Hello %d", a);
      };
    }
    
    //在MRC下，打印block的类型为__NSStackBlock__
    //在ARC下，打印block的类型为__NSMallocBlock__,是因为arc自动帮我们调用了copy函数。将block复制到了堆上
    ```

    

  - 在mrc下测试代码如下,为什么a会被释放， a不是block里面的变量吗 ??? 此时block没有被释放

    ![](./images/block16.png)

    ![](./images/block17.png)

+ `__NSMallocBlock__`

  - `__NSStackBlock__`调用了copy

    ```objc
    //MRC下
    void test() {
      int a = 10;//auto局部变量
      void (^block)(void) = [^{
                NSLog(@"Hello %d", a);
      } copy];
    }
    
    //在MRC下，__NSStackBlock__类型的block调用copy函数，会将block复制到堆上
    ```

+ 类型归纳如下

  ![](./images/block14.png)

+ 每一种类型的block调用copy后的结果如下所示

  ![](./images/block15.png)

### block的copy

+ 在ARC环境下，编译器会根据情况自动将栈上的block复制到堆上，比如以下情况
  - block作为函数返回值时
  - 将block赋值给__strong指针时
  - block作为Cocoa API中方法名含有usingBlock的方法参数时
  - block作为GCD API的方法参数时
+ MRC下block属性的建议写法
  - @property (copy, nonatomic) void (^block)(void);
+ ARC下block属性的建议写法
  - @property (strong, nonatomic) void (^block)(void);
  - @property (copy, nonatomic) void (^block)(void);

### 对象类型的auto变量

>  当block内部访问了对象类型的auto变量时

- 如果block是在栈上，将不会对auto变量产生强引用

  ![](./images/block19.png)

- 如果block被拷贝到堆上

  - 会调用block内部的copy函数
  - copy函数内部会调用_Block_object_assign函数
  - _Block_object_assign函数会根据auto变量的修饰符（__strong、__weak、__unsafe_unretained）做出相应的操作，形成强引用（retain）或者弱引用

- 如果block从堆上移除

  - 会调用block内部的dispose函数
  - dispose函数内部会调用_Block_object_dispose函数
  - _Block_object_dispose函数会自动释放引用的auto变量（release）

- copy函数，dispose函数调用时机

  ![](./images/block18.png)

- 当copy到堆上，对象被__strong修饰时

  ```shell
  $ xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc -fobjc-arc -fobjc-runtime=ios-8.0.0 main.m
  ```

  ![](./images/block20.png)

  ![](./images/block21.png)

+ 当copy到堆上，对象被__weak修饰时

  ![](./images/block22.png)

  ![](./images/block23.png)


### __weak编译问题解决

+ 在使用clang转换OC为C++代码时，可能会遇到以下问题
  - cannot create __weak reference in file using manual reference
+ 解决方案：支持ARC、指定运行时系统版本，比如
  - xcrun -sdk iphoneos clang -arch arm64 -rewrite-objc -fobjc-arc -fobjc-runtime=ios-8.0.0 main.m

### __block的本质

+ __block可以用于解决block内部无法修改auto变量值的问题

+ __block不能修饰全局变量、静态变量（static）

+ 编译器会将__block变量包装成一个对象

+ 用__block变量包装一个基本变量时

  ![](./images/block27.png)

  ![](./images/block24.png)

  ![](./images/block25.png)

  ![](./images/block26.png)

  - 包装成`__Block_byref_age_0`对象，将该对象地址传递给block
  - block结构体copy到堆上时，调用copy函数，`__Block_byref_age_0`对象也在堆上copy了一份。此时无论栈上，还是堆上的`__Block_byref_age_0`对象的`__forwarding`指针都指向堆上的 `__Block_byref_age_0`对象
  - `__Block_object_assgin`和`__Block_object_dispose`对`__Block_byref_age_0`对象进行内存管理

+ 用__block变量包装一个对象时

  ![](./images/block31.png)

  ![](./images/block28.png)

  ![](./images/block29.png)

  ![](./images/block30.png)

  ![](./images/block32.png)

  

### __block细节

+ __block修饰后的变量，在block内部和外部访问时，到底访问的是被修饰的变量，不是修饰后包装的对象。

  ![](./images/block33.png)

  ![](./images/block34.png)

  ![](./images/block35.png)

  

### __block内存管理

+ 当block在栈上时，并不会对__block变量产生强引用

+ 当block被copy到堆时

  - 会调用block内部的copy函数

  - copy函数内部会调用_Block_object_assign函数

  - _Block_object_assign函数会对__block变量形成强引用（retain）

    ![](./images/block36.png)

+ 当block从堆中移除时

  - 会调用block内部的dispose函数

  - dispose函数内部会调用_Block_object_dispose函数

  - _Block_object_dispose函数会自动释放引用的__block变量（release）

    ![](./images/block37.png)

### __forwarding指针

![](./images/block38.png)



### 对象类型的auto变量、__block变量

+ 当block在栈上时，对它们都不会产生强引用
+ 当block拷贝到堆上时，都会通过copy函数来处理它们
  - __block变量（假设变量名叫做a）
    - `_Block_object_assign((void*)&dst->a, (void*)src->a, 8/*BLOCK_FIELD_IS_BYREF*/);`
  - 对象类型的auto变量（假设变量名叫做p）
    - `_Block_object_assign((void*)&dst->p, (void*)src->p, 3/*BLOCK_FIELD_IS_OBJECT*/);`
+ 当block从堆上移除时，都会通过dispose函数来释放它们
  - __block变量（假设变量名叫做a）
    - `_Block_object_dispose((void*)src->a, 8/*BLOCK_FIELD_IS_BYREF*/);`
  - 对象类型的auto变量（假设变量名叫做p）
    - `_Block_object_dispose((void*)src->p, 3/*BLOCK_FIELD_IS_OBJECT*/);`



### 被__block修饰的对象类型

+ 当__block变量在栈上时，不会对指向的对象产生强引用

+ 当__block变量被copy到堆时

  - 会调用__block变量内部的copy函数

  - copy函数内部会调用_Block_object_assign函数

  - _Block_object_assign函数会根据所指向对象的修饰符（`__strong`、`__weak`、`__unsafe_unretained`）做出相应的操作，形成强引用（retain）或者弱引用（注意：这里仅限于ARC时会retain，MRC时不会retain）

    ```objective-c
    __block __strong NSObject *object0 = [[NSObject alloc] init];
    __block __weak NSObject *object1 = [[NSObject alloc] init];
    __block __unsafe_unretained NSObject2 *object = [[NSObject alloc] init];
    ```

    

+ 如果__block变量从堆上移除

  - 会调用__block变量内部的dispose函数
  - dispose函数内部会调用_Block_object_dispose函数
  - _Block_object_dispose函数会自动释放指向的对象（release）

### 循环引用问题

+ 循环引用示例

  ![](./images/block39.png)

+ 原因分析

  ![](./images/block40.png)

  - person对象持有block对象， 而block对象内部又持有person对象
  - 这就造成相互引用无法释放

### 解决循环引用问题 - ARC

+ 用`__weak`、`__unsafe_unretained`解决

  ![](./images/block41.png)

  - __weak：不会产生强引用，指向的对象销毁时，会自动让指针置为nil
  -  __unsafe_unretained：不会产生强引用，不安全，指向的对象销毁时，指针存储的地址值不变

+ 用__block解决（必须要调用block）

  ![](./images/block42.png)

  



### 解决循环引用问题 - MRC

- 用__unsafe_unretained解决

  ![](./images/block43.png)

- 用__block解决

  ![](./images/block44.png)

  - MRC下,block修饰的对象不会被retain

### __strong作用

```objc
 __weak typeof(self) weakSelf = self;
    self.block = ^{
        __strong typeof(weakSelf) strongSelf = weakSelf;
        NSLog(@"age is %d", strongSelf->_age);
    };

```

- 通过__strong可以暂时保住self的命

### 总结

1. block本质上是封装了函数调用及调用环境的OC对象

   ```objc
   struct __main_block_impl_0 {
     void *isa;
     int Flags;
     int Reserved;
     void *FuncPtr;
     size_t reserved;
     size_t Block_size;
     void (*copy)(void*, void*);
     void (*dispose)(void*)
     variables
   }
   ```

2. 变量捕获

   1. 捕获auto变量
      - 值传递
      - 不能在block内部修改变量本身
      - 在外部的修改变量不影响内部的变量
   2. 捕获static变量
      - 地址传递
      - 可以在block内部修改变量本身
      - 可以在外部的修改变量
   3. 捕获__block修饰的变量
      - 被包装成一个OC对象
      - 变量本身被值传递到这个OC对象内部
      - 可以在block内部修改变量本身
      - 可以在外部的修改变量

3. 不捕获全局变量

4. block在mrc下被copy修饰，在arc下被strong和copy修饰， 最终被copy到堆上

5. 通过__weak修饰self，解决循环引用问题

   - block内部被copy到堆上时，会根据被捕获的对象的修饰符(`__weak`, `__strong`)，对对象进行内存管理

### 面试题

+ block的原理是怎样的？本质是什么？

  - 封装了函数调用以及调用环境的OC对象

+ __block的作用是什么？有什么使用注意点？

+ block的属性修饰词为什么是copy？使用block有哪些使用注意？

  - block一旦没有进行copy操作，就不会在堆上
  - 使用注意：循环引用问题

+ block在修改NSMutableArray，需不需要添加__block？

  - 不需要添加__block，因为修改的不是数组的指针，而是往里面添加元素

    ```objc
      MJPerson *person = [[MJPerson alloc] init];
      NSMutableArray *array = [NSMutableArray array];
      self.block = ^{
            //array = nil;
            [array addObject:@1];
        };
    ```