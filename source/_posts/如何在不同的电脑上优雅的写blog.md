---
title: 如何在不同的电脑上优雅的写blog
date: 2019-02-12 19:35:58
tags: hexo
categories: 技术
---
<font face="微软雅黑">
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
新建一个分支hexo(名字可以自定义)，这时候hexo分支和master分支的内容是一样的，都是源文件。  
这时需要把hexo设为默认分支，这样的话，在另外一台机器上克隆下来的就是hexo分支了，并且以后所有的源文件操作都是在hexo分支下完成。  
![设置默认分支.png](https://i.loli.net/2019/02/17/5c694226b2928.png)  
- 为什么需要这个额外的分支呢？

因为hexo d只把静态网页文件部署到master分支上，所以你换了另外一台电脑，就无法pull下来继续写博客了。  
有了hexo分支的话，就可以把hexo分支中的源文件(配置文件、主题样式等)pull下来，再hexo g的话就可以生成一模一样的静态文件了  
## 部署博客
还是和以前一样  
`hexo g -d` #生成博客并上传至master分支  
这时候到github查看两个分支的内容，hexo分支里是源文件，master里是静态文件  
***注意：根目录下的_config.yml配置文件中branch一定要填master，否则hexo d就会部署到hexo分支下。***
## 关联到远程hexo分支
在本地新建一个hexo分支并与远程hexo分支关联：  
```python
git checkout -b hexo
git pull origin hexo
```
另外别忘了，如果有修改的话，要推送到hexo分支上去：  
```python
git add . 
git commit -m  ""
git push origin hexo
```
这样才能在另外一台机器上，将源文件完全pull下来，保持同步
## 另一台PC
个人PC上的工作已经完成了，下面讲一下如果你换到了另外一台电脑上，应该如何操作。  
## 将博客项目克隆下来
`git clone git@github.com:xxx/xxx.github.io.git`  
***注意这里一定要用ssh的方式，而不要用https的方式，前面已经说过原因了  
克隆下来的仓库和你在个人PC中的目录是一模一样的，所以可以在这基础上继续写博客了。  
但是由于.gitignore文件中过滤了node_modules\，所以克隆下来的目录里没有node_modules\，  
这是hexo所需要的组件，所以要在该目录中重新安装hexo，但不需要hexo init。
```python
npm install hexo
npm install
npm install hexo-deployer-git --save
```
## 新建一篇文章进行测试
`hexo new "work PC test"`  
## 推送到hexo分支
```
git add .
git commit -m "add work PC test"
git push origin hexo
```
##部署到master分支  
`hexo g -d `  
**这样你在另一台电脑上写的博客就能正常的发布到你的网站上了**
***
## 日常操作
如果上面的过程都操作无误的话，你就可以在任何能联网的电脑上写博客啦。一般写博客的流程是下面这样。  
## 写博客前
不管你本地的仓库是否是最新的，都先pull一下，将git上的源码文件拉取下来，以防万一：  
`git pull origin hexo`  
## 写博客
`hexo new "title"`  
 然后打开source/_posts/title.md，撰写博文。  
## 写完博客
将源码推送到hexo分支上：  
```
git add .
git commit -m "add xxx"
git push origin hexo
```
然后将其部署到master分支上  
`hexo g -d`  
整个流程就是这样的
</font>