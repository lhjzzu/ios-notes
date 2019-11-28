## 签名

### 重签名概述

+ 如果希望将破坏了签名的安装包，安装到非越狱手机上。需要对安装包进行重签名
+ 注意
  + 安装包中的可执行文件必须是经过脱壳的，重签名才会有效
  + .app包内部的所有动态库(.framework, .dylib), AppExtesion(PlugIns文件夹, 扩展名为appex), WatchApp(WatchApp)都需要脱壳， 然后再重签名
  + 现在通过pods安装的第三库等在最终.app中的Frameworks文件夹，Frameworks里面都是动态库都需要脱壳，以及重签名
+ 重签名打包后，安装到设备的过程中，可能需要经常查看设备的日志信息
  + 程序运行过程中: Window-> Device and Simulators -> View Device Logs, 通过查看crash文件定位运行中的问题所在
  + 程序安装过程中: Window-> Device and Simulators -> Open Console。搜索"install如果安装失败，可以用来查看失败原因。

### 重签名步骤

1.  准备一个embedded.mobileprovision文件(必须是付费证书产生的，device一定要匹配), 并放入app包中
   + 可以通过xcode自动生成,然后再编译后的App包中找到
   + 可以去开发者中心去创建，然后下载下来
   
2.  从embedded.mobileprovision文件中提取出entitlements.plist权限文件
   
   ```shell
   $ security cms -D -i  embedded.mobileprovision > temp.plist
   $ /usr/libexec/PlistBuddy -x -c 'Print :Entitlements' temp.plist > entitlements.plist
   ```
   
3. 查看可用证书

   ```shell
   $ security find-identity -v -p codesigning
   
     1) F9243741EE8F7C93BDBC7014C4DB153DC517CE57 "iPhone Developer: lhjzzu@163.com (572GZQJ63T)" (CSSMERR_TP_CERT_REVOKED)
     2) 69134CF2D655514BE57A24464558B5B2F9CC3C0D "iPhone Developer: lhj 刘 (4KMSA98VYA)"
     3) C0C80FBECA51C9496966D433AE1E3EAC0EA4944F "iPhone Distribution: Hangzhou DanYue Technology Co.Ltd. (9BK7Q35648)"
   
   ```

4. 对xx.app内部的已经脱壳的动态库, AppExtension, Watch的内容等进行签名

     ```shell
   $ codesign -fs 69134CF2D655514BE57A24464558B5B2F9CC3C0D xxx.dylib
   $ codesign -fs 69134CF2D655514BE57A24464558B5B2F9CC3C0D xxx.framewok
   $ codesign -fs 69134CF2D655514BE57A24464558B5B2F9CC3C0D xxx.appex
     ```

5.  对xxx.app包进行重签名

   ```shell
   $ codesign -fs 69134CF2D655514BE57A24464558B5B2F9CC3C0D --entitlements entitlements.plist xxx.app
   ```

### 重签名GUI工具

+ iOS App Signer
  + https://github.com/DanTheMan827/ios-app-signer
  + 可以对xx.app重签名打包成ipa
  + 需要在xxx.app中提供对应的embedded.mobileprovision文件(会自动提取entitlements.plist)
+ iReSign
  + https://github.com/maciekish/iReSign
  + 可以对ipa进行重签名
  + 需要提供entitlements.plist,  embedded.mobileprovision文件路径
+ 上面两个工具只能对xxx.app进行重签名，不能对内部的动态库等进行重签名

### * 简单的重签名过程

1. 建立一个oc项目Test,用真机编译过后，打开Test.app。

2. 将Test.app放到Payload文件夹中，压缩后，更改后缀名为.ipa,然后即可安装到手机上

3. 在开发者中心创建一个xxxx.mobileprovision文件,选择appId,证书，以及包含要安装的设备

4. 下载xxxx.mobileprovision并重命名为embedded.mobileprovision文件

5. 从embedded.mobileprovision获取entitlements.plist

   ````shell
   $ security cms -D -i  embedded.mobileprovision > temp.plist
   $ /usr/libexec/PlistBuddy -x -c 'Print :Entitlements' temp.plist > entitlements.plist
   ````

6. 将embedded.mobileprovision放到Test.app中

7. 通过命令行，查看可用证书

   ```shell
   $ security find-identity -v -p codesigning
   ```

8. 对Test.app进行重签名

   ```shell
   codesign -fs 证书ID --entitlements entitlements.plist Payload/Test.app
   ```

9. 压缩Payload文件夹后，更改后缀名为.ipa,然后即可安装到手机上

10. 如果使用macho-view修改可执行文件的内容后要进行重新签名

### 动态库注入到可执行文件

+ 可以使用insert_dylib库将动态库注入到Mach-o文件中

  1. 下载https://github.com/Tyilo/insert_dylib
  2. 选择scheme->release编译生成insert_dylib可执行文件,然后把insert_dylib放到`/usr/local/bin`目录下，即可全局使用insert_dylib命令

+ 用法

  + 有2个常用参数选项

    + --weak,即使动态库找不到也不会报错

      ```
      不加这一项时，这样如果动态库没有加载成功会直接crash,我们可以通过
      Window-> Device and Simulators -> View Device Logs查看crash文件
      ```

    + --all-yes, 后面所有的选择都为yes。

  ```shell
  $ insert_dylib @executable_path/tweak_landi.dylib ClassPlatform_Landi --all-yes --weak ClassPlatform_Landi
  
  这个时候可以用MachOView查看可执行文件,在Load Commands的最后一行，会发现新加入的动态库信息
  ```

+ insert_dylib的本质是往Mach-O文件的Load Command 中添加一个`LC_LOAD_DYLIB`或`LC_LOAD_WEAK_DYLIB`

+ 可以通过otool查看Mach-O的动态库依赖信息

   ```shell
  $ otool -L Mach-O文件
   ```

### 更改动态库所依赖的其他动态库的地址

+ 可以使用install_name_tool修改Mach-O中动态库的加载地址

   ```
  $ install_name_tool -change 旧地址 新地址 Mach-O文件
   ```

+ 通过Theos开发的动态库插件

  + 默认都依赖于/Library/Frameworks/CydiaSubstrate.framework/CydiaSubstrate
  + 如果要将动态库插件打包到ipa中，也需要将CydiaSubstrate打包到ipa中，并且修改CydiaSubstrate的加载地址
  + 如要让可执行文件加载开发的动态库， 还要让可执行文件依赖开发的动态库。

+ 2个常用环境变量
  + @executable_path代表可执行文件所在目录
  + @loader_path代表动态库所在的目录

### * 对AppStore应用进行重签名, 安装到非越狱手机

1. 下载"兰迪少儿英语"到越狱的手机

2. 利用frida-ios-dump进行砸壳

   ```shell
   + frida-ios-dump不但可以将.app进行解密, 而且可以将app内部的动态库等都进行解密
   + 配置frida-ios-dump环境
   + 启动兰迪少儿英语
   
   在mac上执行下面的命令，在当前路径得到解密的"兰迪少儿英语.ipa"文件
   $ dump.py 兰迪少儿英语
   ```

3. 使用tweak开发动态库插件 tweak_landi

   ```objective-c
   1. 在Tweak.x中写如下代码
   %hook LERNNavigationController
   - (void)viewWillAppear:(BOOL)animated {
   
      %orig;
       dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
           
           UIAlertView *alertView = [[UIAlertView alloc] initWithTitle:@"测试" message:@"message" delegate:nil cancelButtonTitle:@"取消" otherButtonTitles:nil, nil];
           [alertView show];
       });
   }
   %end
   
   2. 编译安装到手机中，测试弹窗是否正常
   ```

4. 获取tweak_landi.dylib,CydiaSubstrate

   ```
   文件所在目录
   1. tweak_landi.dylib: /Library/MobileSubstrate/DynamicLibraries/tweak_landi.dylib
   2. CydiaSubstrate: /Library/Frameworks/CydiaSubstrate.framework/CydiaSubstrate
   ```

5. 解压"兰迪少儿英语.ipa",并将tweak_landi.dylib,CydiaSubstrate放入到Payload/ClassPlatform_Landi.app中

6. 将动态库路径注入到ClassPlatform_Landi文件中

   ```shell
   $ insert_dylib @executable_path/tweak_landi.dylib Payload/ClassPlatform_Landi.app/ClassPlatform_Landi --all-yes --weak Payload/ClassPlatform_Landi.app/ClassPlatform_Landi
   
   查看注入结果
   $ otool -L  Payload/ClassPlatform_Landi.app/ClassPlatform_Landi
   ....
   ....
   @executable_path/tweak_landi.dylib (compatibility version 0.0.0, current version 0.0.0)
   ```

7. 修改tweak_landi.dylib依赖的CydiaSubstrate路径

   ```shell
   1. 修改前，查看依赖
   $ otool -L Payload/ClassPlatform_Landi.app/tweak_landi.dylib
   ....
   ....
   /Library/Frameworks/CydiaSubstrate.framework/CydiaSubstrate
   
   2. 通过install_name_tool修改依赖地址
   $ install_name_tool -change /Library/Frameworks/CydiaSubstrate.framework/CydiaSubstrate @loader_path/CydiaSubstrate Payload/ClassPlatform_Landi.app/tweak_landi.dylib
   
   3. 修改前，查看依赖
   $ otool -L Payload/ClassPlatform_Landi.app/tweak_landi.dylib
   ....
   ....
   @loader_path/CydiaSubstrate
   ```

   

8. 通过命令行，查看可用证书,并对动态库进行重签名

   ```shell
   1 查看可用证书
   $ security find-identity -v -p codesigning
   
   2. 对插件动态库进行重签名
   $ codesign -fs 证书ID Payload/ClassPlatform_Landi.app/tweak_landi.dylib
   $ codesign -fs 证书ID Payload/ClassPlatform_Landi.app/CydiaSubstrate
   
   3. 对应用自身所使用的动态库进行重签名
   $ codesign -fs 证书ID Payload/ClassPlatform_Landi.app/Frameworks/xxx.framework
   ....
   ....
   $ codesign -fs 证书ID Payload/ClassPlatform_Landi.app/Frameworks/xxx.dylib
   
   4. 如果存在PlugIns和Watch，也要对其中的文件进行重签名
   ```

9. 对ClassPlatform_Landi.app本身进行重签名

   ```shell
   1. 获取embedded.mobileprovision文件
   2. 通过embedded.mobileprovision获取entitlements.plist文件
       $ security cms -D -i  embedded.mobileprovision > temp.plist
       $ /usr/libexec/PlistBuddy -x -c 'Print :Entitlements' temp.plist > entitlements.plist  
   3. 将embedded.mobileprovision放到ClassPlatform_Landi.app中
   4. 重签名ClassPlatform_Landi.app
      $ codesign -fs 证书ID --entitlements entitlements.plist Payload/ClassPlatform_Landi.app 
   ```

10. 压缩Payload文件夹，然后进行重命名为landi.ipa，  即可安装到非越狱的手机上。