# systemd

在以前的各种linux发行版中，基本都是使用`sysvinit`来作为系统的init程序。sysvinit是基于runlevel的一个init程序,这种策略在一起是没有问题的,但是现在系统越来越复杂,所以就有更加先进的init程序出现了.

ubuntu中,前几个版本使用的是upstart,这个init程序是基于事件的,其功能是比sysvinit更加强大的.在我使用的ubuntu 14.04中,还是使用的upstart.

fedora社区中的程序员开发出了一个比upstart更加适合于现代系统的init程序,其为systemd.在debian 7中,其已经使用systemd了,ubuntu社区经过讨论之后,决定在下一个lts版本,也就是16.04中才使用systemd,在14.04中仍然使用upstart.

所以,尽早的熟悉systemd的使用方法,可以在系统升级到16.04之后尽快的熟悉其init的使用.

## 基本概念
### systemd unit
在systemd中，将每一个要管理的东西叫做一个unit，一共有12个种类的unit，比如target，mount，automount，service。。。所有的这些unit都用一个文件来表示，其后缀就是其类型

### systemd unit 配置文件的语法
打开这些配置文件，我们会发现其和windows的ini文件的语法十分类似，实际上其是参考其语法来设计的。

对于所有的unit来说，[Unit]和[Install]都是存在的，但是对于不同的unit，其还会有相应的段，比如对于service，那么其还有[Service]段。

对于boolean的设置，在unit 文件中，如果是肯定的，那么1,on,true,yes都是等价的，如果是否定的，那么0,off,false,no都是等价的。

对于一个foo.service，同时可能还存在一个foo.service.wants文件夹，这个文件夹下面的
