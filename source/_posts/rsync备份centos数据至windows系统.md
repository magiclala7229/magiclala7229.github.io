---
title: rsync备份centos数据至windows系统
date: 2019-02-17 16:00:33
tags: rsync
categories: 技术
---
实验目的：每天晚上将centos服务器端的的文件备份到windows端，实现定时无差异备份  
实验准备：1. 服务器端系统centos6  
　　　　　2. IP 192.168.92.139  
　　　　　3. 客户端系统windows7  
# 一、 服务器端配置 
## 1. 关闭lelinux
编辑config文件，注释掉所有只留下SELINUX，随后重启服务器  
`vim /etc/selinux/config`  
`SELINUX=disabled`  
## 2. 安装rsync服务端并进行配置
`yum install -y rsync`  
## 3. 在目录下新建配置文档  
/etc目录下新建/rsyncd目录，然后在/rsyncd新建三个文件分别为rsyncd.conf、rsyncd.motd、rsyncd.passwd
```
touch rsyncd.conf
touch rsyncd.motd
touch rsyncd.passwd
```
- 复制以下内容到rsyncd.conf中
```
# Distributed under the terms of the GNU General Public License v2
# Minimal configuration file for rsync daemon
# See rsync(1) and rsyncd.conf(5) man pages for help
# This line is required by the /etc/init.d/rsyncd script
#告诉进程写到 /var/run/rsyncd.pid 文件中
pid file = /var/run/rsyncd.pid 
lock file =/var/run/rsyncd.lock

#日志文件
log file =/var/log/rsyncd.log
log format = %t %a %m %f %b
syslog facility = local3

#指定运行端口，默认是873
port = 8877     

#指定服务器IP地址
address = 192.168.92.139

#服务器端传输文件时，要发哪个用户和用户组来执行，默认是nobody
uid = root
gid = root

#如果"use chroot"指定为yes，那么rsync在传输文件以前首先chroot到path参数所指定的目录下。这样做的原因是实现额外的安全防护，但是缺点是需要以root权限，并且不能备份指向外部的符号连接所指向的目录文件。默认情况下chroot值为yes
use chroot = no

#客户端最多连接数
max connections = 5
motd file = /etc/rsyncd/rsyncd.motd
timeout = 300

#同步模块，模块名称可以自定义，但是相对的后面也要进行修改
[resources] 
#指定需要备份的文件目录所在路径
path =  /home/resources

#list 意思是把rsync 服务器上提供同步数据的目录在服务器上模块是否显示列出来。默认是yes 。如果你不想列出来，就no ；如果是no是比较安全的，至少别人不知道你的服务器上提供了哪些目录。你自己知道就行了
list=no

#指定在 rsync 服务器上运行 delete 操作时是否忽略 I/O 错误。一般来说 rsync 在出现 I/O 错误时将将跳过 –delete 操作，以防止因为暂时的资源不足或其它 I/O 错误导致的严重问题。
ignore errors

#如果为yes，表示只读本地文件就无法同步到服务器
read only = no

#允许连接的ip，在演示中使用的云服务器就直接写*，表示无限制 如果要规定ip或者ip段（192.168.92.0/24）需要进行其他配置
hosts allow=*
#hosts allow=192.168.92.0/24
#hosts deny=*

#auth users 是必须在服务器上存在的真实的系统用户，如果你想用多个用户，那就以,号隔开
auth users = root

#密码存在rsyncd.passwd文件里
secrets file = /etc/rsyncd/rsyncd.passwd
```
- 编辑rsyncd.passwd，你可以自定义密码  
`root:123456`
- 更改rysncd.passwd权限  
`chmod 600 /etc/rsyncd/rsyncd.passwd`
- 编辑rsyncd.motd内容，也可以不编辑，因为这个只是显示连接之后的内容  
`++++++++++++++++++++`  
`welcome`  
`++++++++++++++++++++`
## 4. 启动rsync服务
`rsync --daemon --config=/etc/rsyncd/rsyncd.conf`  
- 添加开机启动服务，在/etc/rc.d/rc.local下添加  
`rsync --daemon --config=/etc/rsyncd/rsyncd.conf`  
*如果没有rc.local，就新建一下*
## 5. 检查服务是否正常启动
进程命令: ps -ef | grep rsync  
端口命令: netstat -lntup | grep xxx（你自定义的端口号或者是默认端口号873）
## 6. 在防火墙中添加相应端口放行
如果没有关闭iptables，则需要对自定端口进行放行处理，这里的自定义端口为8877  
```
iptables -A INPUT -p tcp -m state --state NEW  -m tcp --dport 8877 -j ACCEPT
service iptables save
```
## 7. 测试是否可以访问数据模块
`rsync --port=8877 --list-only  root@10.1.4.44::resources`  
输入命令后会显示rsyncd.motd的界面（如过编辑过的话），然后会要求输入密码，则表示成功

# 二、 客户端配置
- 下载cwRsync，进行安装
- 管理员身份运行cmd
- 切换目录：`cd C:\Program Files (x86)\cwRsync\bin`
- 运行命令：`rsync --port=8877 -vzrtopg --progress --delete root@192.168.92.139::resources /cygdrive/e/data`
- 如果成功则在程序路径下C:\Program Files (x86)\cwRsync\bin分别添加两个文件
1. 第一个password.txt，输入/rsyncd.passwd时的密码，保存
2. 第二个新建一个批处理文件（名字可以自定义，这里我们就用resources.bat）,复制以下内容
```
@echo off
echo.
echo 开始同步数据，请稍等...
echo.
rsync --port=8877 -vzrtopg --progress --delete root@192.168.92.139::resources /cygdrive/e/data  < password.txt
echo.
echo 数据同步完成
echo.
```
- 这段命令的意思简单说下  
- –port=8877 #端口  
- root #执行数据同步的用户  
- 192.168.92.139 #服务器地址  
- resources #模块名称  
- /cygdrive/d/data 表示本地的同步文件夹/e/data 为同步文件夹  

1. 开始-设置-控制面板-任务计划
2. 打开添加任务计划，下一步
3. 浏览，选择打开C:\Program Files (x86)\cwRsync\bin目录里面的resources.bat
4. 执行这个任务，选择每天，下一步
5. 起始时间：3:00
6. 运行这个任务：每天，下一步
7. 输入Windows系统管理员的登录密码，下一步
8. 完成  

>排错必备思想：  
>1、部署流程步骤熟练  
>2、rsync原理理解  
>3、学会看日志，rsync命令行输出，日志文件/var/log/rsyncd.log  

>rsync服务端排错思路:  
>1、查看rsync服务配置文件路径是否正确，正确的默认路径为：/etc/rsyncd.conf  
>2、查看配置文件里host allow，host deny，允许的ip网段是否是允许客户端访问的ip网段。  
>3、查看配置文件中path参数里的路径是否存在，权限是否正确（正常应为配置文件中的UID参数对应的属主和组）。  
>4、查看rsync服务是否启动（是否有监听端口）。查看进程命令为（ps -ef|grep rsync）端口是存在（netstat -lntup|grep 873）  
>5、查看iptables防火墙和selinux是否开启允许rsync服务通过，也可考虑关闭。  
>6、查看服务端rsync配置的密码文件是否为600的权限，密码文件格式是否正确，正确格式 用户名:密码 ，文件路径和配置文件里的secrect files参数对应。  
>7、如果是推送数据，要查看下，配置rsyncd.conf文件中用户是否对模块下目录有可读写的权限。  

>rsync客户端排错思路：  
>1、查看客户端rsync配置的密码文件是否为600的权限，密码文件格式是否正确，注意：仅需要有密码，并且和服务端的密码一致。  
>2、用telnet连接rsync服务器ip地址873端口，查看服务是否启动（可测试服务端防火墙是否阻挡）。   
telnet 192.168.92.139  
>3、客户端执行命令时：   
rsync -avzP rsync_backup@192.168.92.139::home/resources /resources/ –password-file=/etc/rsync.password  
此命令的细节要记清楚，尤其是192.168.92.139::home/resources/处的双冒号和其后的backup为配置文件中定义的模块名称。  
可以自我模拟错误，增长经验。

`具体rsync资料可以参照：`https://blog.csdn.net/mr_rsq/article/details/79272189
