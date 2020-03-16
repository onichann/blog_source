---
title: SonarQube7.4安装和使用
date: 2020-03-12 21:42:21
tags:
  - SonarQube  
categories: [SonarQube]  
---
近期比较关注代码的检测，之前由于用的findbugs，因此没有可视化的界面
所以便搜索了一些代码检测管理平台，于是便搜索到了这一款平台，接下来就开始搭建吧
<!--more-->

*文章和[SonarQube7.4安装和使用](https://www.jianshu.com/p/dd4a4bc59fc3 "点击我") 相同，因为原篇也是本人所写，移植过来而已*

## 前期准备：
- jdk 1.8.0._131
- maven 3.5.3
- mysql 5.7

## 开始搭建
1. 软件下载
登陆网址 https://www.sonarqube.org/downloads/
直接下载最新的社区版即可
<!--![1](https://upload-images.jianshu.io/upload_images/15054472-0756f23a355627ba.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)-->
{% asset_img 1.png 第一张 %}


2. 配置数据源
打开配置文件：..\sonarqube-7.4\conf\sonar.properties
增加配置：以下是mysql的配置
``` 
#----- DEPRECATED 
#----- MySQL >=5.6 && <8.0
# Support of MySQL is dropped in Data Center Editions and deprecated in all other editions
# Only InnoDB storage engine is supported (not msyISAM).
# Only the bundled driver is supported. It can not be changed.
#sonar.jdbc.url=jdbc:mysql://localhost:3306/sonar?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false
sonar.jdbc.url=jdbc:mysql://127.0.0.1:3306/sonarqube?useUnicode=true&characterEncoding=utf8&rewriteBatchedStatements=true&useConfigs=maxPerformance&useSSL=false
sonar.jdbc.username=root
sonar.jdbc.password=root
sonar.sorceEncoding=UTF-8
sonar.login=admin
sonar.password=admin
```

3. 启动
..\sonarqube-7.4\bin\windows-x86-64\StartSonar.bat

> **原先配置的数据源是Oracle，启动的时候提示连接oracle的jar包不存在，将连接oracle的jar放到路径**
..\sonarqube-7.4\extensions\jdbc-driver\oracle\ojdbc14-10.2.0.1.0.jar

> 如果启动的时候 **提示“另一程序正在使用此文件”**
由于之前启动的sonar进程未关闭，有冲突；打开资源管理器（ctrl+shift+esc），杀掉java进程重新启动，问题解决

再次启动的时候，启动过会儿窗口会自动关闭，那是因为报错了，打开log文件..\sonarqube-7.4\logs\sonar.log，发现提示**“远程主机强迫关闭了一个现有的连接”**，如下所示：
<!--![2](https://upload-images.jianshu.io/upload_images/15054472-f9694698c7b0adcb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)-->
{% asset_img 2.png 第二张 %}
然后打开web.log，发现提示如下：
<!--![3](https://upload-images.jianshu.io/upload_images/15054472-d47453d640058945.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)-->
{% asset_img 3.png 第三张 %}
**这是因为连接的oracle的jar的版本过低**

还有官方要求oracle的版本：
```
#----- Oracle 11g/12c
# The Oracle JDBC driver must be copied into the directory extensions/jdbc-driver/oracle/.
# Only the thin client is supported, and only the versions 11.2.x or 12.2.x must be used. See
# https://jira.sonarsource.com/browse/SONAR-9758 for more details.
# If you need to set the schema, please refer to http://jira.sonarsource.com/browse/SONAR-5000
#sonar.jdbc.url=jdbc:oracle:thin:@localhost:1521/XE
```
想想算了，不想去找jar包了于是便升级mysql版本到mysql5.7，然后mysql的连接上面已经提供了
配置好重新启动，由于第一次需要创建表，所以可能有点慢

4. 登陆系统
访问http://localhost:9000
初始用户名 密码 admin  admin
          
**登陆系统后按照如下步骤下载安装 中文汉化包**
<!--![4](https://upload-images.jianshu.io/upload_images/15054472-935872cd87aa4894.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)-->
{% asset_img 4.png 第四张 %}

安装好插件重启sonar，登录后如下（已经创建了一个项目，首次登录后界面有些许差异）
<!--![5](https://upload-images.jianshu.io/upload_images/15054472-072268e6fd0a16b2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)-->
{% asset_img 5.png 第五张 %}

接下来创建第一个项目
<!--![6](https://upload-images.jianshu.io/upload_images/15054472-d1d9057b318c2aa2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)-->
{% asset_img 6.png 第六张 %}

将 
```
mvn sonar:sonar \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=0006282044b5a5098d23d999c93f9c11eef16025
```
复制到maven项目的根目录下启动检查即可

检查后可以在SonarQube平台里看到一些代码的问题，如下：
<!--![7](https://upload-images.jianshu.io/upload_images/15054472-bd19ff05a0aa007f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)-->
<!--![8](https://upload-images.jianshu.io/upload_images/15054472-12723e23e24250d6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)-->
<!--![9](https://upload-images.jianshu.io/upload_images/15054472-b0d9b7c8be686c24.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)-->
{% asset_img 7.png 第七张 %}
{% asset_img 8.png 第八张 %}
{% asset_img 9.png 第九张 %}

5. Windows下重启sonar：（***这一步不清楚有没有更好的方法，如有请指出***）
- 首先关闭SonarQube.bat窗口
- 再Ctrl+Shift+Esc调出windows资源管理器
- 在进程中关闭所有java.exe进程
- 然后重新进入.\sonarqube-7.4\bin\windows-x86-64\，运行StartSonar.bat文件

## IDEA集成SonarLint
SonarLint 是一个插件，可以集成到开发工具里，有以下功能
- 当打开java文件时可自动分析静态文件，也可以手动对整个项目做分析；
- 可连接到SonarQube同步分析规则、质量规则与自定义设置；

由于鹅主只使用IDEA，接下来就说明下IDEA如何集成
1. 首先配置maven的settings.xml文件，目的是为了将结果同步到SonarQube平台上
配置代码如下：
```
<profile>
            <id>sonar</id>
            <activation>
                <activeByDefault>true</activeByDefault>
            </activation>
            <properties>
                <sonar.jdbc.url>
                    jdbc:mysql://127.0.0.1:3306/sonarqube
                </sonar.jdbc.url>
                <sonar.jdbc.driver>com.mysql.jdbc.Driver</sonar.jdbc.driver>
                <sonar.jdbc.username>root</sonar.jdbc.username>
                <sonar.jdbc.password>root</sonar.jdbc.password>
                <sonar.host.url>http://127.0.0.1:9000</sonar.host.url>
                <!-- your_sonar_host是你的服务器地址，如果你的服务在本机则使用localhost -->
            </properties>
        </profile>
```
2. 安装插件sonarLint
<!--![10](https://upload-images.jianshu.io/upload_images/15054472-a1fd62534f948625.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)-->
{% asset_img 10.png 第十张 %}
3. 配置SonarQube servers
<!--![11](https://upload-images.jianshu.io/upload_images/15054472-68267a4b3ed2e559.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)-->
{% asset_img 11.png 第十一张 %}
> 如果勾选 Automatically tigger analysis ，将会自动对打开的文件进行分析
4. 绑定上一步骤创建的，以及选择项目对应的SonarQube的项目
***这一步是由于上面步骤已经在SonarQube里分析了一次所以这里可以选择SonarQube project,不清楚有没有其他方法，如果有的话请指出***
<!--![12](https://upload-images.jianshu.io/upload_images/15054472-bd0127833f835003.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)-->
{% asset_img 12.png 第十二张 %}
5. 执行命令 ，即可将项目进行分析，以及将分析的结果同步到SonarQube平台上，**如果只是用插件进行项目分析的话，测试发现是不会同步到平台上的**，***不清楚是不是哪里步骤有问题...***
```
mvn clean install 
mvn sonar:sonar
```

**至此整个过程讲解结束，如果有疑问或者指点的话欢迎留言(`・ω・´)**
