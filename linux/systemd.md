systemd的使用
========

在以前的版本中，都是使用sysvinit来控制开启启动项，以及后面的对各种服务的管理。init的PID为1，也就是系统启动的第一个进程。

在ubuntu中，其使用了upstart，这是一个事件启动的init程序。

现在在red hat系列中，包括rhel,cnetos,fedora，都是使用的systemd，其是fedora的一个项目。

## 基本概念
### systemd unit
在systemd中，将每一个要管理的东西叫做一个unit，一共有12个种类的unit，比如target，mount，automount，service。。。所有的这些unit都用一个文件来表示，其后缀就是其类型

### systemd unit 配置文件的语法
打开这些配置文件，我们会发现其和windows的ini文件的语法十分类似，实际上其是参考其语法来设计的。

对于所有的unit来说，[Unit]和[Install]都是存在的，但是对于不同的unit，其还会有相应的段，比如对于service，那么其还有[Service]段。

对于boolean的设置，在unit 文件中，如果是肯定的，那么1,on,true,yes都是等价的，如果是否定的，那么0,off,false,no都是等价的。

对于一个foo.service，同时可能还存在一个foo.service.wants文件夹，这个文件夹下面的
