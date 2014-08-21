安装完成之后的配置
===

我一直在使用windows，所以在刚开始使用centos的时候，会觉得很多地方不太舒服，当然，在桌面端，windows的用户体验应该是要比linux更好的，但是从研究来说，开源的linux肯定是要比windows更好的。为了使得使用linux时轻松一点，下面是安装了之后的一些要做的配置。

### 1.更改软件源
centos默认的软件源是官方的，其服务器在国外，直接访问的话非常慢。所以使用国内的源很重要。国内做的最好的应该是[中科大的源](http://mirrors.ustc.edu.cn/)，具体的使用方法其也说了的。

### 2.安装chrome浏览器
centos自带的是firefox浏览器，其版本也是较低的，我还是更加喜欢chrome浏览器，但是现在google不能被访问，所以可以在windows下面将chrome的RPM包下载下来安装。

`yum install xxx.rpm`就可以了。
