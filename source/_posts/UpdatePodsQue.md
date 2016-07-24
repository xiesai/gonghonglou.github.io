title: 解决OS X 10.11之后CocoaPods的升级问题    
date: 2016-04-01 15:24:10    
category: iOS
tags:
---

### 问题描述：
OS X 10.11之后升级CocoaPods，执行命令：
    
    $ sudo gem update cocoapods           // 更新至最新版
    $ sudo gem update cocoapods --pre     // 或者 更新至测试版
     
报如下错误：
    
    ERROR: While executing gem ... (Errno::EPERM)
    Operation not permitted - /usr/bin/pod
        
### 解决方案一：
使用如下命令更新：
      
    $ sudo gem install -n /usr/local/bin cocoapods         // 更新至最新版
    $ sudo gem install -n /usr/local/bin cocoapods --pre    // 或者 更新至预览版
    
这种解决方法最开始的`sudo gem update cocoapods` 和 `sudo gem update cocoapods --pre`升级命令是不能用的，若想一切恢复正常请尝试解决方案二。

### 解决方案二：
推倒重来。。。   
安装Homebrew

    ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
    
安装Ruby：

    brew install ruby

升级CocoaPods，执行命令：
     
    $ sudo gem install cocoapods           // 更新至最新版
    $ sudo gem install cocoapods --pre     // 或者 更新至测试版
    
此后，一切OK～

### 后记
* 上一篇文章关于CocoaPods的详细介绍：[iOS的库依赖管理工具CocoaPods](http://gonghonglou.com/2016/04/01/CocoaPods)

* 转载请保留原文地址：[http://gonghonglou.com/2016/04/01/UpdatePodsQue](http://gonghonglou.com/2016/04/01/UpdatePodsQue)