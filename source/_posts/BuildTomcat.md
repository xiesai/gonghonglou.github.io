title: Mac上搭建Apache Tomcat ＋ MySQL    
date: 2016-04-01 16:27:06    
category: Technology
tags:
---
因连接公司项目的测试服务器各种不便所以搭了个本地服务器Apache Tomcat，数据库用的MySQL。写下过程一来总结，二来分享，希望能帮到诸位一二，来介绍需要下载用到的工具吧。

### Java环境     
点击下载：[jdk-7u79-macosx-x64.dmg](http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html)

下载之后一路点击安装即可

### MySQL数据库        
点击下载：[mysql-5.6.29-osx10.8-x86_64](https://dev.mysql.com/downloads/file/?id=461482) 

下载之后同样一路点击安装即可，默认的安装路径为 `/usr/local/mysql-5.6.29-osx10.8-x86_64`

此时，在系统偏好设置的最下方如下图所示：

![](http://7xn9bi.com1.z0.glb.clouddn.com/apachetomcat/mysql.png) 

点击MySQL，然后点击 `Start MySQL Server` 开启数据库服务


### 阿帕奇服务器
点击下载：[apache-tomcat-7.0.68.zip](http://tomcat.apache.org/download-70.cgi)
 
下载解压后放到任一目录中，我选择的是MySQL的默认安装目录下 `/usr/local/`

终端 `cd` 到阿帕奇服务器所在目录的 `bin` 目录下，执行如下命令：

    $ ./startup.sh        // 开启服务器
    $ ./shutdown.sh       // 关闭服务器
       
新下载的tomcat在启动时会报错：

	./startup.sh: Permission denied
	
这是因为权限问题，执行如下命令即可解决：

	$ chmod u+x *.sh	//表示对当前目录下的*.sh文件的所有者增加可执行权限

再次执行开启服务器命令即可。成功开启服务器后，浏览器打开 [http://localhost:8080](http://localhost:8080) 出现如下界面表示开启成功：

![](http://7xn9bi.com1.z0.glb.clouddn.com/apachetomcat/localhost.png)

如上图点击`Manager App`登录需要账号密码，点击`取消`会有解释。    
即打开`/usr/local/apache-tomcat-7.0.68/conf`目录下的`tomcat-users.xml`文件，在下方添加配置用户名密码如：

    <role rolename="manager-gui"/>
    <user username="tomcat_name" password="tomcat_password" roles="manager-gui"/>


### Sequel Pro    
点击下载mysql的图形化工具：[sequel-pro-1.1.1.dmg](http://sequelpro.com) 

安装成功后打开，设置如下：

![](http://7xn9bi.com1.z0.glb.clouddn.com/apachetomcat/sequel.png)

`Name`随意，MySQL初始用户名为`root` 密码为空，因为初始并没有数据库所以`Database`也为空。

点击`Connect`进入后

1：左上角选择`Add Database...`，填入一个名字如`troy`，点击`add`

2：`Sequel Pro`菜单栏选择`Database`-->`User Accounts...`,给 root 用户添加用户名密码，点击`Apply`

这样在下次登录时就要把`Password`和`Database`填上再连接

### 添加数据
将自己的`.sql`文件用`Sequel Pro`打开并运行，这样在数据库中就有了数据

将自己的`.war`的`jar`包添加到`/usr/local/apache-tomcat-7.0.68/webapps`目录下，这样就有了执行程序

终端 `cd` 到阿帕奇服务器所在目录的 `bin` 目录下，开启服务器可执行如下任一命令：

    $ ./startup.sh         // 单纯开启服务器
      
    $ ./catalina.sh run    // 开启服务器，控制台可打印服务器数据
       
到这里本地服务器就开启了

执行如下命令，查看服务器在局域网中的地址：

    $ ifconfig

这样你的手机等设备也可以连接到本地服务器了

### 总结一下
1.  jdk-7u79-macosx-x64.dmg    
     java环境

2.  mysql-5.6.29-osx10.9-x86_64.dmg    
     mysql数据库

3.  apache-tomcat-7.0.68.zip    
     阿帕奇服务器

4.  sequel-pro-1.1.1.dmg    
     mysql的图形化工具

5.  troy_2016-03-25.sql    
     sql语句，放在 Sequel Pro中执行，生成数据）
     
6.  rest.war    
     运行程序，放在 apache-tomcat-7.0.68/webapps 目录下
     
7. 开启服务器

8. 愉快的玩耍

### 后记
转载请保留原文地址：[http://gonghonglou.com/2016/04/01/BuildTomcat](http://gonghonglou.com/2016/04/01/BuildTomcat)