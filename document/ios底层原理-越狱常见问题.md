## 越狱常见问题

> 操作系统是ios 10.1.1

### 1  SSH无法连接问题

+ 参考: https://blog.csdn.net/dianshanglian/article/details/62422627

### 2  SCP不能使用的问题

+ 参考: https://www.jianshu.com/p/c64073f2ce0b

```
$ cd /usr/bin
$ ldid -S scp
$ chmod 777 scp
//进入/usr/lib下面
$ cd /usr/lib/
//增加可执行权限
$ chmod 755 libcrypto.0.9.8.dylib
$ chmod 755 libcrypto.dylib
资源的下载我在百度网盘的软件目录也存了一份。
```

### 3 Cycript不可用

+ 默认已经安装，从Cydia中找到，更改，再次安装即可

### 4 安装RevealLoader

+ 在Cydia中搜索到的不能安装,在工具中提供了一个RevealLoader.deb, 将它放到手机的/var/root/Media/Cydia/AutoInstall，然后重启
+ mac的Reveal的help找到ios Library,找到RevealServer。然后放到手机的/Library/RHRevealLoader/目录， RHRevealLoader文件夹不存在的话，手动创建

###  5 hopper-disassebler破解下载

+ 参考: https://www.waitsun.com/hopper-disassembler-4-2-0.html

###  6 用dumpdecrypted脱壳失败

+ 错误信息

  ```
  + dyld: could not load inserted library 'dumpdecrypted.dylib' because no suitable image found. Did find: 
  dumpdecrypted.dylib: required code signature missing for 'dumpdecrypted.dylib' 
  ```

+  解决方案

  ```
  https://www.cnblogs.com/wangyaoguo/p/9084939.html
  ## 列出可签名证书
  security find-identity -v -p codesigning 
  ## 为dumpecrypted.dylib签名
  codesign --force --verify --verbose --sign "iPhone Developer: xxx xxxx (xxxxxxxxxx)" dumpdecrypted.dylib
  ```

### 7 Theos插件安装到根目录下，不生效

+ 参考: https://blog.csdn.net/wj610671226/article/details/81209785

+ 解决方案

  ```
   $THEOS/即为theos工程所在的路径
   打开 $THEOS/makefiles/package/deb.mk
  1、brew install dpkg (一定记得要更新一下dpkg)
  2、修改 $THEOS/theos/makefiles/package/deb.mk文件内容
  $(ECHO_NOTHING)COPYFILE_DISABLE=1 $(FAKEROOT) -r $(_THEOS_PLATFORM_DPKG_DEB) -Z$(_THEOS_PLATFORM_DPKG_DEB_COMPRESSION) -z$(THEOS_PLATFORM_DEB_COMPRESSION_LEVEL) -b "$(THEOS_STAGING_DIR)" "$(_THEOS_DEB_PACKAGE_FILENAME)"$(ECHO_END)
  上面这一句修改为下面这一句代码就可以搞定
  $(ECHO_NOTHING)COPYFILE_DISABLE=1 $(FAKEROOT) -r dpkg-deb -Zgzip -b "$(THEOS_STAGING_DIR)" "$(_THEOS_DEB_PACKAGE_FILENAME)" $(STDERR_NULL_REDIRECT)$(ECHO_END)
  ```

###  8 在tweak文件中hook某个类，不能使用self

+ 参考: http://iosre.com/t/hook-self/12864

  ```
  1. 加上类的声明 @Someclass;
  2. 加上要调用的方法的声明
             
  @interface  Someclass
  //声明要调用的方法
  -(void)test;
  @end
  ```

  