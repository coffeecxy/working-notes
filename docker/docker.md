# docker启动流程


docker设计的时候,将其client和server的代码是写在了一起的,通过`-d`参数来辨别是要开启一个daemon,还是只是运行一个命令.

## client的方式

如果是运行一个命令的话,其过程就会比较直接简单,因为其就像我们运行`ls $HOME`这个一般的命令一样.

## daemon的方式

docker中最重要的部分是在docker daemon的实现中,也就是加上了`-d`选项之后docker的运行流程中.

首先会有一个Engine被建立,然后会分出两个routine,一个routine去建立一个docker Daemon,另外一个routine去建立一个serve api.

### serveapi

serveapi是docker daemon中负责和docker client通信的部分. 为了满足大部分的通信环境,在传输层协议的使用上,docker支持现在基本会使用到的unix socket,tcp,fd.
