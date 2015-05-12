# pthread

在我们的应用中,在板子上的程序要使用多线程的方式,一个线程负责和主机通信,另外一个线程完成计算.

我们使用pthread来做.

## pthread_create

     int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
                          void *(*start_routine) (void *), void *arg);

pthread_create在调用的进程中创建一个新的线程.start_routine是新线程要运行的函数,arg是这个函数需要的参数.

新线程会在下面的情况下退出:

1.  在线程中调用 pthread_exit,其可以返回一个void*的值,这个值会被调用pthread_join的线程得到.
2.  在线程中使用了return来返回,相当于调用了pthread_exit,返回值也会被调用pthread_join的线程得到.
3.  被取消了,pthread_cacal.
4.  进程中的任何一个线程调用了exit函数,或者是main线程调用了return函数.也就是进程整个就要退出了.这样所有的线程都会推出.

`attr`参数是在创建线程时需要使用的到的参数,如果其为NULL,那么使用默认的配置来创建线程.

在pthread_create函数成功返回之前,新的线程的tid会被放到`thread`中,这个参数就是后面调用pthread_xxx中表明这个线程的参数.

新的线程继承了创建线程的信号屏蔽位(pthread_sigmask). 这个线程的待处理信号队列为空.
新的线程继承创建线程的浮点运算环境.

---
调用成功的时候,返回0(Linux中的函数的一般规则). 如果发生了错误,返回一个错误号,thread中的值是不定的.(不能使用).

---
使用pthread_self可以得到一个线程的tid. 除非使用了实时的配置,那么在调用pthread_create之后,创建线程和新的线程谁先执行是不确定的.

一个线程可以是joinable的或者是detached. 如果一个线程是joinable的,那么其他的线程(一般是主线程)可以调用pthread_join来等待这个线程结束并得到这个线程的返回值. 一个joinable线程在结束后,其占用的资源`不会马上返回给系统`,而是要等到其被另一个线程join之后这些资源才会返回(如果没有特殊原因,要将一个thread join了).

对于一个detached线程,其结束之后,使用的资源会自动返回给系统,不能对一个detached线程进行jion操作与期得到其退出时返回的值.(所以,一般都是使用joinable的线程). detached的线程在一些daemon程序中会使用,因为在这些情况下,线程退出的状态是不重要的.

默认的,一个新的线程被创建为`joinable`.如果要让一个线程是detached的,那么需要更改attr的值.

## pthread_exit

    void pthread_exit(void *retval);

调用pthread_exit的线程会退出. 其返回值会进入到retval中. 返回值会被调用了pthread_join的线程得到.

使用pthread_cleanup_push加入的handler会被弹出并执行. 如果这个线程有自己特有的数据,那么在运行完了clean up函数之后,这些数据会被释放.

当一个线程退出的时候,属于进程的数据(mutex,semaphores,file descriptor)这些是不会被释放的. 通过atexit注册的函数是不会被执行的.

当进程的最后一个线程退出后,这个进程就退出了. 进程的资源也就会被释放了.

---
在非main thread中的start函数使用return语句,那么也就是一个隐式的pthread_exit调用.参数就是return的值.

> 为了让其他的线程继续运行,那么main thread需要使用pthread_exit来结束而不是exit或者是return.

retval不应该指向一个在当前线程的stack中存在的数据,因为在pthread_exit之后,这个stack会被清除. 一般的,使用malloc来分配这个返回值.


## pthread_join

	int pthread_join(pthread_t thread, void **retval);
调用pthread_join的线程会等待thread线程结束. 如果那个线程已经结束了,那么pthread_join会立即返回. thread指向的线程必须是joinable的.

如果retval不是空的,那么pthread_join会将*retval指向的值填充为结束进程调用pthread_exit返回的值. 如果thread指向的线程被cancel了,那么PTHREAD_CANCELED 被放到*retval中.

---
如果pthread_join成功返回了,那么调用线程可以保证target thread是退出了的.

将一个已经被jion的线程进行jion操作,那么结果是不定的.

如果对一个joinable的线程进行join操作失败了,那么那个线程将变成zombie thread. `不要这样做`,因为如果有很多的zombie thread,每个zombie thread都会占用一定的系统资源,那么当zombie
thread足够多的时候,系统就不能新建线程和进程了.

在phthread库中,没有类似于`waitpid(-1,&status,0)`这样的函数,也就是`将所有的已经结束的线程join`. **如果设计中出现了要使用这种函数的情况,那么需要重新进行设计**.

一个进程中的所有线程都是对等的: 一个进程中的线程相互join.

> 这个函数一个比较难理解的地方是其输入参数是一个二重指针.`void **retval`.
我们使用二重指针,要完成的事情就是让一个指针指向另外一个内存.
pthread_exit返回的值会被放在这个进程的heap区中. `*retval=xxx`就会达到得到返回值的效果.(需要仔细的想一下)


## pthread_detach

    int pthread_detach(pthread_t thread);

将thread线程变成detached的. 当一个detached线程结束的时候,其占用的系统资源会自动的返回到系统中,而不需要其他的线程去显示的jion它.

如果尝试对一个已经detached的数据再次detached,会有不定的结果.

---
只要一个线程是detached的了,那么其不可以被jion,也不可以变回joinable的了.

一个新的线程可以被创建为detached的.(通过设置attr).

detached仅仅是设置当一个线程结束的时候操作系统的反应.(joinable不会马上释放所有的资源,而是要等待被jion,detached就会马上释放所有的资源). 如果这个进程退出了,使用exit,或者是main thread使用了return,那么这个线程还是会被退出.

对于一个线程,pthread_join或者是pthread_detach必须有一个被调用,不然这个线程占用的资源不会被成功的释放.

## pthread_cancel

	int pthread_cancel(pthread_t thread);

发送一个取消的请求到thread指定的线程. 要求其结束.

## 操作cleanup handler

    void pthread_cleanup_push(void (*routine)(void *),
                             void *arg);
    void pthread_cleanup_pop(int execute);

这两个函数操作当前线程的clean up handlers. clean up handler是在一个线程cancel的时候自动运行的函数. 其函数原型在push中给出了. 一个典型的使用clean up的是将一个mutex解锁.

push将一个routine加入到这个thread的clean up handlers的最上面. 当这个routine被运行的时候,arg会作为其参数.


pop会从这个handlers中取出最上面的一个,如果execute不是0,那么相应的routine才会被执行.

下面的情况会导致一个routine被pop出来执行.
1. 当一个线程被其他的线程cancel的时候.
2. 当一个线程自己调用pthread_exit的时候.(如果是从这个线程的start fun中调用return,那么这些handler不会被执行).
3. 当一个线程主动调用pthread_cleanup_pop,并且其参数不是0的时候.

---
posix中要求pthread_cleanup_push和pthread_cleanup_pop都使用macro的方式来实现. 而且分别包含`{`和`}`.所以对这两个函数的调用必须是成对匹配的.



## pthread_self,pthread_equal

    pthread_t pthread_self(void);
    int pthread_equal(pthread_t t1, pthread_t t2);

返回当前线程的tid. 其和使用pthread_create返回的thread值是一样的. 这个调用总是会成功.

---
posix允许实现的时候自由的选择对pthread_t的实现方式,可以使用一个整形,也可使用一个结构体. 所以,不能简单的使用C提供的`==`来判断两个pthread_t是不是相等. 而是要使用phthread_equal.

tid应该被认为是一个不透明的数据,任何直接使用tid的代码都是认为不可移植的.

tid只在一个进程中是唯一的. 当一个线程被jion或者datach之后,其tid又可以被使用了.

通过pthread_self得到的值和`gettid`得到的值不是同一个东西.

## pthread_yield

    int pthread_yield(void);
当前线程出让时间片,让其他的线程来执行.

---
在Linux中,这个函数调用的是sched_yield.

在Linux中,线程是轻量级的进程,也就是共享了资源的进程. 对于时间片的使用,最终是调度的线程.这个函数就让当前线程主动出让自己的时间片.

## pthread_sigmask

     int pthread_sigmask(int how, const sigset_t *set, sigset_t *oldset);

这个函数和sigpromask是一样的,主要是因为posix中需要这个函数,所以才被定义了. 具体的man要看sigpromask.

##   pthread_sigqueue
    int pthread_sigqueue(pthread_t thread, int sig,
                    const union sigval value);

和sigqueue做的事情是类似的,但是,sigqueue是先一个进程发送信号.而pthread_sigqueue是向当前进程中的(另外一个)线程发送一个信号.

thread是接收信号的线程的tid. sig是要发送的信号. value是和信号一同发送的数据.
