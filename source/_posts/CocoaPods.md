title: iOS的库依赖管理工具CocoaPods    
date: 2016-04-01 15:17:10    
category: iOS     
tags:
---


工欲善其事，必先利其器！在iOS开发中 [CocoaPods](https://github.com/CocoaPods/CocoaPods) 作为库依赖管理工具就是一把利器。

有了CocoaPods你再也不需要手动将需要的第三方库拖进你的工程、添加第三方库所依赖的framework、设置如`-fno-objc-arc`等编译参数、手动管理这些库的更新，当某个第三方库有依赖其他的库还要继续拖、拖、拖......

我们只需要将所需要的第三方库声明到一个名为`Podfile`文件中，然后执行`Pod install`命令就OK了，CocoaPods就会自动去下载我们所需要库，并为我的工程设置好相应的系统依赖和编译参数。

只能说，好用到哭～


## 安装CocoaPods
CocoaPods依赖于Ruby环境，刚好Mac下自带Ruby，使用Ruby的gem命令即可安装CocoaPods。            
为防止因gem太老而引发问题，建议执行如下命令先更新gem：

    $ sudo gem update --system
    
执行如下命令安装CocoaPods：

    $ sudo gem install cocoapods
    
如果执行上述命令后没有反应，那是因为Ruby的软件源 [https://rubygems.org](https://rubygems.org) 使用的是亚马逊的云服务而被墙了（GFW的伟大。。。）    
可以用淘宝的Ruby镜像来访问cocoapods，依次执行如下命令将官方的Ruby源替换成国内淘宝的源：

    $ gem sources --remove https://rubygems.org/
    $ gem sources -a https://ruby.taobao.org/
 
执行如下命令验证Ruby镜像的确是taobao的：

    $ gem sources -l
    
出现如下文字才说明上面的命令是执行成功的：

    *** CURRENT SOURCES ***

    https://ruby.taobao.org/
    
此时，再次执行gem命令安装CocoaPods：

    $ sudo gem install cocoapods
    $ pod setup
    
稍等片刻 即可安装成功。    
> 注： [`pod setup`](http://guides.cocoapods.org/terminal/commands.html#pod_setup) 是Cocoapods将它的信息下载到 `~/.cocoapods/repos` 目录下。即使在安装时不执行此命令，在初次执行 `pod install` 命令时，系统也会自动执行 `pod setup`
    
## 升级CocoaPods
升级CocoaPods非常简单，使用Ruby的gem命令：

    $ sudo gem update cocoapods           // 更新至最新版
    $ sudo gem update cocoapods --pre     // 或者 更新至预览版
    
当然执行如下命令也可以更新：

    $ sudo gem install cocoapods           // 更新至最新版
    $ sudo gem install cocoapods --pre     // 或者 更新至预览版
    
    
> 注：OS X 10.11之后升级CocoaPods会有问题。解决方案参见下一篇博客：[解决OS X 10.11之后CocoaPods的升级问题](http://gonghonglou.com/2016/04/01/UpdatePodsQue)
     
## 降级CocoaPods
有时我们需要降低CocoaPods版本来解决某些第三方库的兼容问题，例如[RestKit](https://github.com/RestKit/RestKit)不兼容CocoaPods的`0.39.0`版本，降级到`0.38.2`就OK了。
    
#### 移除RubyGems中的Cocoapods程序包
查看[gems](http://baike.baidu.com/view/1772268.htm)中本地程序包，执行如下命令：

    $ gem list
    
输出如下：

```    
*** LOCAL GEMS ***

activesupport (4.2.5.2)    
bigdecimal (1.2.8)    
claide (1.0.0.beta.1, 0.9.1)    
cocoapods (1.0.0.beta.4, 0.39.0)    
cocoapods-core (1.0.0.beta.4, 0.39.0)    
cocoapods-deintegrate (1.0.0.beta.1)    
cocoapods-downloader (1.0.0.beta.1, 0.9.3)    
cocoapods-plugins (1.0.0.beta.1, 0.4.2)    
cocoapods-search (1.0.0.beta.1, 0.1.0)    
cocoapods-stats (1.0.0.beta.3, 0.6.2)    
cocoapods-trunk (1.0.0.beta.2, 0.6.4)    
cocoapods-try (1.0.0.beta.2, 0.5.1)    
colored (1.2)    
did_you_mean (1.0.0)    
escape (0.0.4)    
fourflusher (0.3.0)    
fuzzy_match (2.0.4)    
i18n (0.7.0)    
io-console (0.4.5)    
json (1.8.3)    
minitest (5.8.3)    
molinillo (0.4.4)    
nap (1.1.0)    
net-telnet (0.1.1)    
netrc (0.7.8)     
power_assert (0.2.6)    
psych (2.0.17)    
rake (10.4.2)    
rdoc (4.2.1)    
rubygems-update (2.6.1)    
test-unit (3.1.5)    
thread_safe (0.3.5)    
tzinfo (1.2.2)    
xcodeproj (1.0.0.beta.3, 0.28.2)        
```

其中包含的CocoaPods版本：
 
    cocoapods (1.0.0.beta.4, 0.39.0)    

移除指定版本cocoapods如1.0.0.beta.4，执行如下命令：
   	
    $ sudo gem uninstall cocoapods -v 1.0.0.beta.4
    
成功删除则输出：

    Successfully uninstalled cocoapods-1.0.0.beta.4
    
还有一个0.39.0版本，移除程序包，执行如下命令：
   	
    $ sudo gem uninstall cocoapods -v 0.39.0
    
当移除最后一个版本时，询问：

    Remove executables:
	        pod, sandbox-pod

    in addition to the gem? [Yn]

按下回车键删除pod。查看CocoaPods组件的安装目录，执行命令`$ which pod `所得目录下的`pod`文件随即删除。    

#### 安装指定版本的Cocoapods程序包
安装指定版本的CocoaPods 如0.39.0，执行如下命令：

    $ sudo gem install cocoapods -v 0.39.0  

> 注：若不指定版本，即命令如`sudo gem install cocoapods`则默认安装最新版。    
    
安装成功后，执行命令查看版本号：

    $ pod --version
    
输出：

    0.39.0
  
## 使用CocoaPods

#### 搜索第三方库
为判断某第三方库（如AFNetworking）是否支持CocoaPods，执行如下命令来搜索：

    $ pod search AFNetworking
 
若如下图所示，则可用CocoaPods管理AFNetworking
![searchAFNetworking](http://7xn9bi.com1.z0.glb.clouddn.com/CocoaPods%2FsearchAFNetworking.png)

#### 创建`Podfile`的文件
CocoaPods就可以根据`Podfile`文件里的内容来帮你下载你所需要的库。[点击前往](https://guides.cocoapods.org/using/the-podfile.html)CocoaPods官方对`Podfile`文件的介绍。    
终端`cd`到你的项目所在目录下，创建`Podfile`文件：

    $ vim Podfile

按下`i`键进入输入状态，在`Podfile`文件里输入以下文字：

    platform :ios, '8.0'
    
    target 'Your_App_Name' do
    pod 'AFNetworking', '~> 3.0'
    end
    
按下`esc`键退出输入。然后保存退出，命令是`:wq`。你当然可以使用`vim`之外的编辑软件来编辑`Podfile`文件。
> 注：cocoapods-1.0.0.beta版本后规定`Podfile`文件必须如上所示格式（加上`target`）

> 当然，采用创建`Podfile`文件的另一种方式，终端`cd`到你的项目所在目录下执行命令 `pod init` 会自动生成格式，自己试一下你会喜欢的～


终端`cd`到你的项目所在目录下执行如下命令来利用CocoPods下载第三方库：

    $ pod install
    
如下图所示则下载成功：  
![pod install](http://7xn9bi.com1.z0.glb.clouddn.com/CocoaPods%2FpodInstall.png)


> 提示：[!] Please close any current Xcode sessions and use `Your_App_Name.xcworkspace` for this project from now on.

打开`Your_App_Name.xcworkspace` 工程之后会看到 `Pods` 文件，`AFNetwoking`已经成功导入项目了。 
   
你或许应当[点击前往](https://guides.cocoapods.org/using/pod-install-vs-update.html)CocoaPods官网查看对`pod install vs. pod update`的介绍。

> 注：当你 `clone` 别人的项目到本地后也需要终端`cd`到项目所在目录下执行命令 `$ pod install`


#### 关于`Podfile.lock`的文件
执行`pod install`之后，CocoaPods会生成一个名为`Podfile.lock`的文件。并锁定当前各依赖库的版本，之后如果多次执行`pod install`或者团队中的其它人check下来这份包含`Podfile.lock`文件的工程后再执行`pod install`命令时，获取下来的Pods依赖库的版本就和最开始用户获取到的版本一致。如果没有`Podfile.lock`文件，执行`pod install`命令会获取第三方库的最新版本，这就有可能造成同一个团队使用的依赖库版本不一致，这对团队协作的危害无疑是灾难性的！    
在这种情况下，如果团队想使用当前最新版本的依赖库，有两种方案可修改`Podfile.lock`的纪录：    

* 更改`Podfile`中各依赖库的版本
* 执行`pod update`命令

鉴于`Podfile.lock`文件对团队协作如此重要，我们应该将它添加到版本控制里。

[点击前往](http://guides.cocoapods.org/using/using-cocoapods.html#should-i-ignore-the-pods-directory-in-source-control)CocoaPods官网查看对`Podfile.lock`的介绍。


> 补充：有时执行`pod update`命令会特别慢，可以尝试使用如下命令：

>     $ pod update --verbose --no-repo-update
>`pod install`命令同理：

>     $ pod install --verbose --no-repo-update

## 发布自己的开源框架到CocoaPods
你需要如下图创建一个Framework来打造你自己的开源框架

![Framework](http://7xn9bi.com1.z0.glb.clouddn.com/CocoaPods%2FFramework.png)

发布自己的开源框架到CocoaPods同样需要一个类似`Podfile`的文件来告诉CocoaPods我们开源库的名称、版本、作者、描述、地址、所需的framework、依赖库等，这个文件叫`Your_Framework_Name.podspec`，开始创建这个文件

终端`cd`到工程目录下，执行如下命令：

    $ pod spec create Your_Framework_Name
    
这样在你的工程目录下会生成一个`Your_Framework_Name.podspec`文件，大概内容为：

```
Pod::Spec.new do |s|     
  s.name         = "Your_Framework_Name"    
  s.version      = "0.0.2"    
  s.summary      = "My framework"    
  s.description  = <<-DESC    
                    It's my framework.    
                   DESC    
  s.ios.deployment_target = "8.0"   
  s.source       = { :git => "https://github.com/gonghonglou/Your_Framework_Name.git",     
                     :tag => s.version }    

  s.source_files  = "Class/*.{h,m}"    
  s.public_header_files = ["Your_Framework_Name/Your_Framework_Name.h"]    
end
```

给框架打上 tag
 
    $ git tag 1.0.2
    $ git push origin --tags
    
检查 podspec 语法和项目是否正常编译，执行如下命令：

    $ pod spec lint Your_Framework_Name.podspec
    
确保没有任何 error 和 warning ，然后推送 podspec 到 CocoaPods 的主仓库就可以了

    $ pod trunk push Your_Framework_Name.podspec    // 提交到 CocoaPods 中心仓库


## 后记
* 小白出手，请多指教。如言有误，还望斧正！

* 转载请保留原文地址[http://gonghonglou.com/2016/04/01/CocoaPods](http://gonghonglou.com/2016/04/01/CocoaPods)

## 参考链接
* [Using CocoaPods](https://guides.cocoapods.org/using/index.html)
* [CocoaPods安装和使用教程](http://www.superqq.com/blog/2014/10/16/cocoapodsan-zhuang-he-shi-yong-jiao-cheng/) 
* [用CocoaPods做iOS程序的依赖管理](http://blog.devtang.com/2014/05/25/use-cocoapod-to-manage-ios-lib-dependency/) 
* [如何打造一个让人愉快的框架](https://onevcat.com/2016/01/create-framework/)

