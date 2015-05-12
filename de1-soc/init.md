# init

init程序对于Linux系统十分重要,在现在的桌面发型版中,一般都使用upstart或者是systemd提供的init. 但是在嵌入式的Linux中,还是使用的以前的sysvinit. systemvinit本身比较简单.

## /etc/inittab
inittab描述了在系统启动过程中需要启动的进程,也描述了正常运行的时候在不同runlevel中需要运行的进程. sysvinit使用runlevel来管理进程的启动.

runlevel有0-6和A,B,C. inittab文件中每一行是一个记录,格式为

    id:runlevels:action:process

* id是包含1-4个字符的名称.用来标示inittab中的一个记录.
* runlevels,表示后面的action要作用的runlevel.
* action表示要采取的动作,在后面列出.
* process表示要运行的进程.

runlevels中可以包含多个运行级别.比如123表示后面指定的动作在123运行级别中都要启动.对于`sysinit,boot,bootwait`这三个动作的runlevel是被忽略的.

当运行级别改变的时候(通过init命令),所有正在运行的不属于新的运行级别的进程都会被关闭,首先会发送sigterm给他们,然后发送sigkill(就是一定要关闭了.)

有效的action.

 `respawn` 只要指定的进程关闭了,那么就会被重新启动.比如getty进程.

    1:2345:respawn:/sbin/getty 38400 tty1

`wait` 指定的进程会在系统进入指定的runlevel之后被启动. init进程也会等待这个进程的结束(使用waitpid系统调用).

    l0:0:wait:/etc/init.d/rc 0
    l1:1:wait:/etc/init.d/rc 1
    l2:2:wait:/etc/init.d/rc 2
    l3:3:wait:/etc/init.d/rc 3
    l4:4:wait:/etc/init.d/rc 4
    l5:5:wait:/etc/init.d/rc 5
    l6:6:wait:/etc/init.d/rc 6

`once` 指定的进程会在系统进入指定的runlevel之后运行一次.

`boot` 指定的进程在系统启动的过程中会被运行.

`bootwait` 指定的进程在系统启动过程中被运行,init会等待其结束.

`ondemand` 在ondemand上的进程会在指定的ondemand运行级别被调用的是运行.但是,系统不会进行运行级别的切换.a,b,c就是ondemand运行级别. (不同于1-6这几个运行级别,在运行`init [0-6]`的时候,系统会进行运行级别的切换,`init [a-c]不会进行运行级别的切换,只会导致定义在这几个运行级别的程序被运行,所以一般用得不多`)

`initdefault` 指定系统内核启动之后应该进入的运行级别,一般都是5,如果没有指定,那么其会让在console中输入一个运行级别.后面指定的process会被忽略掉.

    id:5:initdefault:

`sysinit` 指定的进程会在系统启动中被运行.指定的进程会在`boot`和`bootwait`之前运行.指定的runlevel会被忽略(因为现在系统还在启动自己的内核,内核与运行级别是没有关系的,所以不会理会给出的运行级别.)

    si::sysinit:/etc/init.d/rcS

## init

init是所以进程的父亲. 其主要的任务就是按照/etc/inittab中描述的内容来创建出系统启动之后需要运行的进程.这个文件中一般都会有一个调用/sbin/getty的一行.

runlevel 0,1,6是系统保留的,0用来关闭系统,1进入当用户模式,6用来重新启动系统.S不是直接被使用的,而是在inittab中用来表示进入到了单用户模式的.s和S是一样的.

在系统内核启动完成之后,也就是Init进程启动起来之后,其会去看/etc/inittab这个文件,首先去看里面是不是有`initdefault`这个entry.其决定了系统初始要进入的运行级别. 如果文件中没有这个entry,那么就要从console输入一个了.

运行级别S,s会让系统进入到单用户模式,也就是系统维护模式.

当进入到多用户模式(2345)的时候,init会先运行`boot`和`bootwait`中的内容来允许在用户登陆之前让文件系统挂载. 然后才会运行匹配这个runlevel中的其他entry.

在运行一个新的进程的时候,init会先查看/etc/initscript文件,如果存在的话,会使用这个文件来运行进程.

---
init生成了所有的指定的进程之后,其就会进入等待状态.其会等待一个子孙进程死亡,一个powerfail信号,或者是使用`init 1-6`来改变当前的运行级别. 当上面三个条件中的一个发生的时候,init会重新检查inittab的内容.
我们随时都可以向inittab中添加内容.但是这些添加的内容还是要等到上面三个条件发生之后才会被读取. 如果想要添加的内容马上被读取,那么可以使用`init q`来主动唤醒init来重新读取inittab中的内容.

当init改变运行级别的时候,其会发送一个SIGTERM到所有的在新的级别中没有被定义的进程. 在5秒之后,会发送SIGKILL. 注意init会假设这些进程还在init创建它们的时候所在的进程组中,如果一些进程在运行的时候改变了自己的进程组,那么他们收不到这两个信号,就需要特别的处理.

## telinit
telinit是init的一个链接.其接受一个字符的参数,让init进程执行特定的任务.

* 0,1,2,3,4,5,6 让init进入指定的运行级别
* a,b,c 让init去查看/etc/inittab文件的内容,其只会处理运行级别是a,b,c的,而且不会变化运行级别.
* Q,q 让init重新读取/etc/inittab,用来立即刷新init.
* S,s 让init进入当用户模式.

telinit 可以使用`-t`选项来指定在发送sigterm和sigkill中要等待的时间.默认值是5s.

init程序使用pid来检测其是init进程还是telinit进程.init进程的pid总是1,所以我们也可以使用init而不是telinit来完成上面的功能.

---



