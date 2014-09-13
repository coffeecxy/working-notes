mongod的配置
========

mongod是mongodb的最主要的组成部分，其接受来自各种client（mongodb有很多的driver，可以是各种语言写成的client）的连接，然后管理数据库，接受各种命令。

mongod是一个命令行的可执行程序，通过

	mongod -h

可以得到其可以接受的所有参数。

实际上使用的时候，我们会使用一个yaml格式的配置文件来指名mongod启动时候的参数。

	d:\MongoDB\bin\mongod.exe --config d:\MongoDB\mongod.yaml

yaml格式是一种和JSON类似的格式的文件，其可以很方便的被人和计算机读出来，其能表示的数据结构也很丰富，和JSON能表示的差不多。而且大部分语言都有相应的处理yaml给事文件的API可以调用，所以很多时候其会被用来作为配置文件使用（另外使用很广泛的配置文件格式有json,ini,xml，具体会使用哪种格式的文件就看开发者的偏好了）

mongod可以配置的属性被分成了很多个组。下面一个一个的说。

### systemLog
systemLog用来配置mongod的log的属性，就是当mongod运行的时候，其会对正在进行的事情进行输出。

`systemLog.verbosity`

integer，表示verbosity的，数字越大，那么输出的东西越多，越能够定位问题，但是也会有很多太过繁琐的东西输出。

我一般不配置这个属性，此时就输出默认的东西。

`systemLog.quiet`

boolean，设置为true的时候，mongod会被限制输出尽量少的log。

不推荐设置，因为这样对于除错很不方便。

`systemLog.path`

string,设置log输出到一个文件中而不是默认的输出到stdout，这样我们可以打开这个文件来查看具体的log

`systemLog.destination`

string，为syslog或者是file,如果是file，那么log输出到`systemLog.path`指定的地方，如果是syslog，那么就会输出到系统的log里面。

一般都使用file，配合着`systemLog.path`。

### processManagement

`processManagement.fork`

boolean，让mongod运行为一个daemon。

默认为false，此时mongod运行为一个一般的进程，命令行会停止响应，直到按下ctrl+c强制停止mongod。

在windows下面，这个配置是**不行的**，会直接说processManagement.fork是unrecognized的。
>在windows下面，就使用前面提到过的安装service的方式来以daemon的方式运行mongod。


>在linux下面，这个应该是可以的，但是我们可以使用systemd(centos)或者是upstart(ubnutu)来启动mongod这儿daemon。

### net
`net.port`

integer,mongod监听的端口，默认是27017.

mongod使用的是TCP连接。

`net.bindIp`

string，mongod监听的interface，也就是监听从哪些IP过来的连接。默认是all interface(0.0.0.0)，也就是所有的IP过来的连接都监听。

在deb和rpm安装的时候，其会有一个默认的配置文件，在这个配置文件中，bindIp的值为127.0.0.1.

监听127.0.0.1的意思是mongod数据库和我们运行的应用在同一台主机上面。

如果监听所有IP，那么一定要保证防火墙有正确的配置，不然就会有危险。

`net.maxIncomingConnections`

integer,默认是1000000，表示最多同时能够连接的数量。

`net.wireObjectCheck`

boolean，默认为true，表示在写入之前，都会做检查，保证写入的数据是有效的，不会破坏存储在硬盘上面的数据。

`net.http.enabled`

boolean,默认为false。就是不打开http连接。client连接过来的时候，默认是连接到本机的20107端口，使用的协议是mongo,通过TCP。

但是我们也可以使用HTTP来连接到mongod，此时也是通过的TCP。

？？？ 相应的测试还需要一台电脑，所以就还没有做。

一般情况下，都不要打开HTTP连接，因为这样会加大风险。

`net.http.RESTInterfaceEnabled`

boolean,默认是false。时候打开mongod的HTTP连接的RESTful API。当打开了这个API的时候，可以使用RESTful的方式来通过HTTP操作数据库。

---
当打开了HTTP连接的时候，mongod会监听27017(port)+1000=28017 HTTP端口，所以使用浏览器打开这个端口可以看到一个页面。

要正常的通过HTTP操作，那么RESTInterfaceEnabled要设置为true。

`net.unixDomainSocket.enabled`

boolean,默认为false，和上面的http.enabled类似，表示是不是可以使用unix socket来连接，当mongod运行在unix上面的时候才可以使用。

`net.ipv6`

boolean,默认为false。默认情况下，是不允许通过ipv6网络连接到mongod的。

### storage
关于存储在磁盘的数据的设置。

`storage.dbPath`

设置数据存储的地方。比如我的设置

```yaml
storage:
 dbPath: E:\mongodb\data
 directoryPerDB: true
```
这样数据就会存在E:\mongodb\data下面。

`storage.directoryPerDB`

默认为false，mongodb在存储的时候，默认是所有的database都存储在`storage.dbPath`，当`storage.directoryPerDB`为true的是，每一个database都会存储在相应的子目录下面。

![](datapath.png)

上面的图中，说明当前mongo中有db1,db2两个我们新建的database。

admin，local是系统建立的database。

journal是在开启了journal功能的时候mongod建立的一个文件夹。

每一个database存储的内容的目录如下。

![](datahe.png)

`storage.nsSize`

integer，默认为16

从上图可以看到，每个database都有一个16M大小的.ns文件，这个是这个database的namespace文件。其大小有这个参数决定，最大为2047.

`storage.journal.enabled`

boolean，在64bit上面是默认开启的，32bit时是关闭的。

当开启了journal功能的时候，能保证在任何情况下，数据都是有效的，可以恢复的。开启之后再dbPath下面会有一个journal文件夹。

`storage.journal.commitIntervalMs`

integer，默认为100或者30

这个值设置的是在多少ms内，mongod必须进行一次journal操作。所以这个值设置得越低，那么这个数据库就越不容易丢失数据。范围是2-300,但是如果设置的太低了，那么系统大部分的时间都会去进行journal操作了，performance会受到影响。

