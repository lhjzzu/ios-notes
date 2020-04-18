## dyld浅析

### dyld加载流程

1. dyld的**自举**（slideOfMainExecutable、rebaseDyld 完成自身的**重定位**）

2. mach_init

3. 设置堆栈保护: __guard_setup

4.  获取应用的slide（appsSlide） 调用dyld的main函数

   1. 配置环境

   2. 加载动态库(共享缓存)

   3. 实例化主程序

   4. 插入动态库：（越狱中编写插件就是修改这个配置让自己写的库被加载，这个配置也只有root用户才有权限修改，本来是苹果给自己预留插入动态库用的）

   5.  重定位完所有需要重定位的库，然后初始化主程序

      1. 经过一系列初始化函数的调用，到notifySingle函数

         - 通过断点调试发现此函数的回调是load_images这个函数
         - load_images里执行call_load_methods函数
           - 循环调用各个类的 load 方法

      2. 然后调用了 doModInitFunctions 函数

         - 内部会调用全局C++对象的构造函数（带__attribute__((constructor))的c函数）

      3. 返回主程序的入口函数，进入主程序的main函数, 历经千辛万苦，我们抵达了最熟悉的main函数

         ```c
         int main(int argc, char * argv[]) {
             @autoreleasepool {
                 return UIApplicationMain(argc, argv, nil, NSStringFromClass([AppDelegate class]));
             }
         }
         ```

### 参考

+  https://juejin.im/post/5c727262e51d457139116208
+ https://juejin.im/post/5e12ce8a5188253a6647c900#heading-14