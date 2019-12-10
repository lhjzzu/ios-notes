## 越狱环境搭建

>  越狱的系统是10.1.1， 不完美越狱

### 越狱工具

+ 使用爱思助手进行越狱
+ 越狱成功之后在手机上出现一个Cydia

### Cydia安装软件

#### 1 步骤

1. 添加软件源
2. 进行搜索要安装的插件
3. 如果安装完出现"重启SpringBord"，那么点击重启

#### 2 安装必要的插件

##### 2.1 安装Apple File Conduit "2"

+ 可以访问整个iOS设备的文件系统
2. 类似的补丁还有：afc2、afc2add

+ 软件源 
  1. http://apt.saurik.com
  2. http://apt.25pp.com

##### 2.2 安装 AppSync Unified

+ 可以绕过系统验证，随意安装、运行破解的ipa安装包
+ 软件源
  1. http://apt.25pp.com

##### 2.3 安装 iFile

+ 可以在iPhone上自由访问iOS文件系统
+ 类似的还有Filza File Manager、File Browser

+ 软件源
  1. http://apt.thebigboss.org/repofiles/cydia

##### 2.4 PP助手

+ 可以利用PP助手自由安装海量APP
+ 软件源
  1. http://apt.25pp.com/

### Mac必备

+ iFunBox
  1. 管理文件系统

+ PP助手
  1. 自由安装海量APP
  2. 卸载APP
  3. 备份APP为ipa安装包（iOS9开始，不再支持备份APP）

### 安装包

+ 通常情况下
  1. 通过Cydia安装的安装包是deb格式的（结合软件包管理工具apt）
  2. 通过PP助手安装的安装包是ipa格式的
+ 如果通过Cydia源安装deb失败
  1. 可以先从网上下载deb格式的安装包
  2. 然后将deb安装包放到/var/root/Media/Cydia/AutoInstall
  3. 重启手机，Cydia就会自动安装deb

### 用代码判断是否越狱

```objective-c
+ (BOOL)re_isJailbreak {
   return [[NSFileManager defaultManager] fileExistsAtPath:@"/Applications/Cydia.app"];
}
```

### Mac效率工具

+ Alfred

  + 便捷搜索

  + 工作流

+ XtraFinder

  + 增强型Finder

+ iTerm2

  + 完爆Terminal的命令行工具

+ Go2Shell

  + 从Finder快速定位到命令行工具