---
title: 启用hexo-next头像
date: 2019-02-06 16:00:40
tags: hexo
categories: 技术
---
hexo版本：v3.8.0  
在网上查找了很多资料，对于一个非常不熟悉代码的人来说，有点困难，虽然花了一点时间，  
最终还是修改成功了，  
具体操作方法：  
1. 将图片下载下来放置于相应主题下的/images文件夹下面（或者直接使用网络图片地址） 
2. 找到主题对应的_config.yml文件
3. 找到以下代码对应的位置  
Sidebar Avatar  
avatar:  
 url: /images/20120607173654_M8QeA.png  
 在url这里，将你的对应文件名替换上去就可以了  
 最后重新生成一下  
 hexo g  
 hexo s