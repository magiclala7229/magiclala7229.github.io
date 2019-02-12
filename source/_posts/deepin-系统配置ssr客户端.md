---
title: deepin系统 配置ssr客户端
date: 2019-02-12 12:00:37
tags: ssr
categories: 技术
---
1. 使用root用户登录(su -),运行一下命令：
```
wget http://www.djangoz.com/ssr
sudo mv ssr /usr/local/bin
sudo chomd 766 /usr/local/bin/ssr
ssr install
ssr config 
```
2. 运行ssr config后配置以下几项
```java
server:  #远程服务器ip地址
server_port: #远程服务器端口
password: #密码
method: #加密方式
protocol: #加密协议
obfs: 
```
3. 然后保存并退出
4. 该脚本运行需要运行git命令，所以如果没有安装过的话，先安装一下
```
sudo apt-get install git
```
5. 加入开机启动
由于新版的deepin系统，使用了systemd作为启动器，默认不包含rc.local文件，此时请在/etc目录下以管理员权限创建一个名为rc.local的春文本文件，并写入如下内容：
```
#!/bin/bash
#rc.local config file created by use
把需要开机启动的命令写在这里
exit0
```
6. 保存后，赋予该文件可执行权限：```sudo chmod +x /etc/rc.local```。下次重启时，systemd就会自动执行rc.local里面的命令了
7. 开启ssr与关闭ssr  
```sudo ssr start```  
```sudo ssr stop ```
8. 同时你还需要设置系统代理
![deepin ssr系统代理.png](https://i.loli.net/2019/02/12/5c6286c6b453d.png)
9. 这样你就可以访问国外网站了
10. 当你不需要访问国外网站时，可以直接关闭系统代理。