
---
title: jekyll和github pages的使用

permlink: /use/jekyll
---




jekyll和github pages的使用
========

github被称作为程序员的facebook，现在很多的开源项目都会在github上面。

github在设计的时候，考虑到大家看到一个项目的时候，第一步不是要看到其代码，而是想看到一个项目的说明，所以给出了了github pages。同时，对于每一个开发者个人或者组织，也可以建立一个github pages主页。

基于此，我们可以在github上面免费的搭建一个个人主页。

github pages后端使用的是jekyll这个静态页面生成器。可以使用轻量级的markdown标记语言进行写作。

## 在windows上面安装ruby和jekyll
jekyll是一个ruby的包。在windows上面安装网上有一些教程。


首先，到[这个网站](http://rubyinstaller.org/downloads/)下载`ruby`和`devkit`的安装包。

然后安装`ruby`，安装的时候选择将其可执行文件的路径加入`PATH`。

然后运行`devkit`安装文件，其是一个自解压的文件，解压到`e:\devkit`

```bat
cd /d e:\devkit
ruby dk.rb init
```	

然后在`e:\devkit`有一个`config.yml`文件。修改其为
```yml
---
- d:/Ruby200-x64
```
然后运行
```bat
ruby dk.rb install
gem install jekyll
```	
这会安装jekyll的依赖项以及自身。
要等一定的时间就可以安装好了。


## 使用jekyll

在本地使用jekyll十分简单，直接使用
```
jekyll serve --watch
```
然后访问`localhost:4000`就可以了。

要特别注意，在windows下面，命令行默认使用的936，也就是GBK编码的，这样会导致很多命令有问题，在cmd中先使用

```
chcp 65001
```

转成utf-8的，这样就没有问题了。

