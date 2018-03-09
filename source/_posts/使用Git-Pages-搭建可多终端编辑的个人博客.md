---
title: 使用Git Pages 搭建可多终端编辑的个人博客
date: 2018-03-09 16:57:57
tags: GitPages, Hexo
categories: 技术
---
## 使用Git Pages 搭建可多终端编辑的个人博客
[TOC]
Hexo作为一个优秀的静态博客框架，与传统的博客网站不同的是，Hexo的博客源文件是保存在本地的，Hexo是博客生成工具，将博客源文件，与页面展示分离，分开管理，使用Markdown语法编辑博客，然后使用Hexo 提供的 `hexo generate`	和 `hexo deploy`	命令将markdown文件生成相应的html文件。
* 并不想在自己电脑上搭建服务
* 也不想去购买云服务
* 也不想用一些博客网站提供的个人博客功能，一来广告多，二来受人制约
**所以最好的办法是使用GitHub提供的GitHub-Pages服务来免费搭建自己的博客。**
### 1.	提出问题
查看文件目录：

![Alt text](./hexo.png)

我们通过`Hexo deploy`	将文件发布到GitHub-Pages的时候[此处实现方式省略]，会将public目录（HTML源文件）自动push到远程仓库的master分支。但是这个对于多终端博客同步来说没有任何意义，因为我们每次进行`Hexo generate`	的时候，都会根据source目录下的markdown源文件重新生成HTML文件，所以当我们在当前目录使用git同步的时候，master分支下会出现博客目录下的所有文件，而我们真正应该关心的是source目录下的markdown源文件，要维护的也是这部分文件的同步。所以问题的关键是：**如何同步source目录下的源文件，以及配置文件_config.yml， scanffolds目录，themes目录。**

>**生成HTML页的目录：**
![Alt text](./1.png)
###2.解决问题
首先在GitHub上创建一个同名的远程库，例如上图中我的repo名为：[username].github.io，使用这个库的master分支来管理public目录下的内容，（实际是.deploy_git中的内容，也就是说博客根目录git 初始化完毕后，里面的内容不需要使用git push 来提交到master分支，因为`hexo d`这条指令已经包含了相关的操作，>具体请百度或google查看hexo github 配置，而且如果在本地博客根目录下使用`git pull`	会把master分支下的public目录中的内容都拉到本地，导致文件错乱）然后另新建一个名为source的分支来管理博客根目录下的无需发布，但需要多终端配置的内容（包括source文件夹下的.md文件
.gitignore文件中的内容说明的是当执行git push的时候忽略的文件，其他文件都要上传到source分支下进行同步。）
![Alt text](./4.png)
到这里，解决问题的思路就清晰了：
*	master分支来管理发布的博客内容
*	source分支来管理Hexo博客的工作空间
###3.具体步骤：
1. 在blog根目录初始化仓库，并切换到source分支，关联远程仓库，并push到远程仓库的source分支，这里会忽略 ignore掉的文件夹。
```
cd blog
git init //在当前目录下初始化git仓库
git checkout -b source //新建一个source分支，并切换到该分之下
git add . //添加blog目录下的所有文件夹（.gitignore声明的文件除外）
git commit -m 'write some message'
git remote add origin git@github.com:username/username.github.io.git //如果已经配置了源，这一条可以忽略
git push origin source //将本地更新push到远程库
```
>至此，远程库中source分支下，已经有了我们本地的Hexo配置以及blog的源文件：.md文件


2. 将远程内容同步到另外一台机器 B主机
在另外一台电脑上，先把node环境配好，安装hexo。同时找个位置建立一个Blog的根目录文件夹。
``` 
npm install hexo-cli -g
```
安装好Hexo后，注意不要进行初始化，因为这样又会生成Hexo的新的配置文件，而我们所需的内容都在source分支上管理，所以这里直接进行如下操作：
```
git clone -b source git@github.com:username/username.github.io.git
cd username.github.io // packeage.json在clone下来的项目目录中
npm install //根据package.json来下载依赖包，这也是为什么我们不需要同步node_modules中的内容的原因!

```
>这时我们已经有了远程的.md文件，也就是博客的源文件，到这里，我们A端，git服务器端，以及B端，已经有相同的工作空间与blog源文件内容了。（B端还没有Hexo g，所以没有public目录，没有Hexo d所以没有deploy文件，）

3.	我们可以开始在B终端写博客了
```
hexo new "about hexo sync"
hexo generate
hexo deploy
git add .
git commit -m "add blog"
git push origin source
```
4.	记住：以后git push的时候都要在source分支下，并且向source分支push，不需要在master分支做任何操作）。

5.	B端上传博客并发布后，回到A端，同样再到source分支下进行同步。
```
git checkout source
git pull

```
6. 然后在A端同样写博客：
```
hexo new "about hexo sync again"
hexo generate
hexo deploy
git add .
git commit -m "add blog"
git push origin source

```
7. 查看个人主页就能看到相应的变化，然后A端新增的内容同样再回到B端在source分之下进行git pull。
8. 至此多终端同步工作完美实现。







