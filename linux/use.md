安装完成之后的配置
===

我一直在使用windows，所以在刚开始使用centos的时候，会觉得很多地方不太舒服，当然，在桌面端，windows的用户体验应该是要比linux更好的，但是从研究来说，开源的linux肯定是要比windows更好的。为了使得使用linux时轻松一点，下面是安装了之后的一些要做的配置。

### 1.更改软件源
centos默认的软件源是官方的，其服务器在国外，直接访问的话非常慢。所以使用国内的源很重要。国内做的最好的应该是[中科大的源](http://mirrors.ustc.edu.cn/)，具体的使用方法其也说了的。


### 2.添加中文输入法
linux在国内不受欢迎的另外一个原因就是国内的互联网公司太少开发其下的软件了，连最基本的输入法都很少，只有最近搜狗开发了一个版本，但是都是给ubuntu开发的，在centos中也使用不了。

centos自代了一个拼音输入法可以使用，但是在7中需要开启。

使用tweak tool >> typing >> show all installed input types, 打开这个选项.

在region and language >> input source 中找到 chinese(intelligent pinying),开启就可以了。

但是使用之后，会发现其还是不能和windows下面的各种输入法相提并论。

### 3.安装chrome浏览器
centos自带的是firefox浏览器，其版本也是较低的，我还是更加喜欢chrome浏览器，但是现在google不能被访问，所以可以在windows下面将chrome的RPM包下载下来安装。

`yum install xxx.rpm`就可以了。

