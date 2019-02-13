---
title: 如何在不同的电脑上优雅的写blog
date: 2019-02-12 19:35:58
tags: hexo
categories: 技术
---
许多使用hexo编写blog的人应该都会有这样的困惑，如何才能在不同的电脑中编写自己的blog呢？  
带着这个疑问，我开始搜索各种网上的资料，大多都是利用git的分支来实现，还有是通过私有仓库来同步，  
那说明git分支应该是比较主流的思路了，终于我找到了这篇文章，然后跟着这篇文章的思路，一步一步，  
实现了我家里的电脑与公司的电脑同时可以编辑，下面我们就着重跟着这个思路来实现，同时结合一下我碰到的问题。

1. 首先你的blog已经部署完成，可以正常使用，环境也都没有问题
2. 最好能在你调试的时候，两台电脑同时测试
3. 经过我的测试，还是linux系统比较适合，环境部署比较方便，推荐deepin或者ubuntu。  
  
下面我们就来说下具体的操作步骤：（*假设你有个人PC与工作PC，原有博客搭建在个人PC*）  
# 个人PC
## 在github重新建立远程仓库(repository)
1. 记录下你原来仓库的名称，然后将其删除（方法：进入仓库，点击setting，将页面拉直最底端，点击Delete this repository，输入密码将其删除），新建一个和愿挨名字一样的空仓库，此时只有一个空的master分支。
2. 将本地目录不要移动，复制一份备用，以免误操作导致内容改动。
## 本地初始化一个新的hexo项目
本地新建一个空目录，作为你新的blog目录，进入该目录，git bash here，初始化一个Hexo项目
```
hexo init
npm install
npm install hexo-deployer-git --save
```
然后用原有blog里的文件替换掉新初始化的这些文件source、scaffolds、themes、_config.yml。注意，这里把theme中所有主题里的.git/目录删除(.git为隐藏文件夹)，否则下次push到远程仓库hexo分之后jacman为空，这也是一个大坑。
## 将整个目录推送到master
```python
git init
//把新的blog目录下的所有文件推送到mater分支下
git remote add origin git@github:xxx/xxx.github.io.git   #仓库建立时会有显示，记得保存ssh链接
git add .    #添加所有文件
git commit -m "first add source code"   #对你需要上传的文件进行标注
git push origin master   #推送至master分支
```
这里可能需要有几个注意的地方
1. 先在git bash中测试一下ssh -T git@github.com是否建立成功
2. remote的方式建议使用ssh方式，否则后期会出现每次push的时候都需要输入账户和密码，比较麻烦
3. windows系统下如果无法无法push的话，可能需要添加Hosts，记住要添加带有ssl的，类似```151.101.185.194 github.global.ssl.fastly.net```，否则可能会导致网页访问github失败。
4. 如果remote地址写错了，可以通过修改blog/.git/目录下的config来调整url

其他某些具体问题，请大家自行百度下，以上是我遇到的问题。
## 在github上新建一个分支