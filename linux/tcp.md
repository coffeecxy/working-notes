# tcp协议相关


在当前使用的最多的TCP/IP协议栈中,传输层的TCP协议十分重要.

## 协议运行过程
TCP的协议运行过程可以分成三个部分.

1. 第一个是连接建立过程,也就是在真正传输数据之前,TCP协议必须保证双方以及建立的tcp连接.
这是因为TCP是一个全双工的传输协议,而且其必须包含数据的传输的完全正确的,如果不进行任何
连接的协商,那么就不能保证得到这个效果.

1. 第二个是数据传送过程, 这个过程中双方都可以向对方传送数据,其为全双工的.

1. 第三个是连接中断过程, 当数据传送传送完毕之后,双方都会中断这个连接,这样才能节约系统
资源,因为TCP连接是由操作系统维护的,如果不及时的释放不需要的TCP连接资源,系统的性能会受到
极大的影响.

下面是操作系统中的一个TCP对象在建立连接的时候会处于的状态.

![](TCP_SYN.svg)

* LISTEN, (server)等待来自cient的TCP链接请求. 比如http服务器,其启动之后,就会在监听tcp
80端口的请求.

* SYN_SENT, (client), client发送了SYN包之后就会进入SYN_SENT状态,现在在等着server应答ACK.

* SYN_RECEIVED, (server)server接受到了client的SYN然后回复了ACK+SYN包之后,其就处于SYN_RECEIVED状态,现在等着client回复ACK.

* ESTABLISHED (client) 当server的ACK+SYN到达了client,那么`client-->server`的数据传输
通道就已经建立了,client发送ACK给server,发送了之后cient就进入ESTABLISHED状态.

* ESTABLISHED (server) 当client发送的ACK成功到达了server,那么`server-->client`的数据
传输通过就建立了,那么server也会进入ESTABLISHED状态.

也就是说,在三次握手成功之后,全双工的数据传输通道久都建立好了.

### 更改tcp连接建立的配置
在linux中,可以使用`sysctl`命令来配置和显示内核处理TCP连接的属性.

#### SYN_SENT

	tcp_syn_retries: integer = 6

client发起TCP连接的时候,会发送SYN包,如果这个SYN包发送不成功,也就是没有接收到ACK+SYN
那么其会再重新发送这么多次.

#### SYN_RECEIVED
对于这个属性,可以有如下的配置属性

    net.ipv4.tcp_synack_retries ： integer = 5

默认值为`5`. 当sever接收到一个SYN包之后,其要发送ACK+SYN包回去,但是这个包可能发送不成功,
那么server就需要不断的重发,这个属性就指定的重发的次数,5次对应着180s左右,如果重发次数用完
了都还是没有发送成功,那么这个TCP的连接就无法建立了.
一般都不会改变这个值,因为我们不想因为偶尔的丢包就导致TCP连接无法建立.

	tcp_syncookies: bool = 1
	tcp_max_syn_backlog: integer = 2048

假设一个client发送了一个SYN给server,然后自己就死掉了,那么后面server过来的SYN+ACK是
不能被其响应ACK的,这样server会重试上面的tcp_synack_retries次数然后丢弃这个连接建立
请求.

在server端,这些处于SYN_RECEIVED的tcp连接成为半连接,其会被存储在操作系统的半连接队列(syn bcklog queue)中,如果这些tcp半连接收到了client过来的ACK,那么其会被移出半连接队列.
如果同时有大量的处于SYN_RECEIVED的tcp半连接,那么半连接队列就可能会溢出,半连接队列的长度由tcp_max_syn_backlog这个值决定.
半连接队列溢出之后,后续的tcp连接请求就会被丢弃了,这个就是SYN flood攻击.

使用syn cookie技术,可以有效的防止syn flood攻击. 当启用了这个技术之后,如果半连接队列满了,
那么kernel不会直接丢弃TCP建立连接的SYN包,而是会根据这个SYN包计算出一个cookie值,然后发送
一个SYN+ACK包回去,再次收到client的ACK包的时候,会根据刚才的那个cookie值来检测这个TCP连接的
合法性,如果合法的话,就会将其变成established的.

### 为什么TCP需要三次握手来建立连接

TCP是一个全双工的,对服务提供保证的连接,为了达到这个目的,连接的双方在开始发送数据之前,
必须交换一些信息. 交换信息的过程也就是三次握手的过程. 下面是三次握手更加详细的解释.

* client发送一个SYN包,也就是发送的这个TCP包中没有包含任何的数据部分,只有TCP的头部,在头部中的
SYN被置位. 此时,这个包的source port为client系统随机分配的一个端口,一般都是1024以上的端口,dst port为80. sequence nubmer会被设置一个随机的32bit的值,比如为`A`,ACK number没有用.

* 这个包进入到server之后,操作系统会看到这个TCP包中SYN被置位了,知道这是一个发起TCP连接的请求,
所以会在系统中新建一个用于处理TCP的对象,然后发送SYN+ACK包,这个包中也是没有数据的,
其中ACK被置位,ack number为`A+1`,SYN也被置位,seq number也是一个随机的值,比如为`B`. src port为80
和dst port为client端的大于1024的那个值.

* 这个包进入到client. 首先,client看到其ACK,发现其ACK NUMBER为`A+1`,知道从 client-->server这半边的
数据发送是可以的了. 然后client会看到这个包中的SYN被置位了,表示server也想向client发送数据, client构造
一个ACK包,其中的ack number为`B+1`. 同时,client如果有数据要发送的话,也可以将数据放在包的后面,
这样seq number就需要根据数据的大小来设置了.

可以看出,为了达到可靠的全双工连接,三次握手中一个很重要的事情就是双方交换各自的seq number,
这样可以保证后面的数据是按照流的方式来跑的. 就是不会出现重复和错位的情况.

### SYN攻击原理

基于TCP连接建立的3次握手过程,可以对这个服务器实施SYN包攻击.
让很多台client(或者是同一台client)同时向一个server发起TCP连接请求,也就是发送一个SYN包,服务器
收到这个SYN包之后,必须建立一个TCP对象,然后返回一个ACK+SYN包. 这些client在接收到这些ACK+SYN包之后
并不回应ACK,而是直接丢弃,TCP有自动重传功能,所以在一定的超时之后,server又会发送这个ACK+SYN包过来,但是
client还是丢弃.

如果同时发起SYN请求的client足够多,那么server就会创建很多的tcp对象,而且会不断的发送ACK+SYN包,导致系统的
内存和网络一直都很拥堵,根本就无法服务其他正常的client的请求.

---

在数据传输完成之后,TCP连接要被关闭,不同于tcp连接建立的时候,必须要client发起建立请求,在
tcp连接关闭的时候,client和sever都可以发起关闭请求,所以,如下图,首先发送FIN包的那端为
initiator,后发送fin包的那端为reciever.


![](TCP_CLOSE.svg)

* FIN_WATI_1, 请求关闭的那端发送了FIN包之后,其就从ESTABLISHED进入FIN_WATI_1,表示
等待另一端的ACK. 这一端被叫做active close,因为是其先发起的关闭连接请求.

* CLOSE_WAIT, 另一端接收到这个FIN包之后,会发送一个ACK回去,发送成功之后,其就进入到CLOSE_WAIT
状态. 这一端叫做passive close,因为其是被动的要关闭连接.

* FIN_WATI_2 初始端接收到ACK之后,进入到FIN_WATI_2状态. 这样初始端到接收端的数据传输通道就被关闭了.

* LAST_ACK 接收端发送一个FIN之后,其进入到LAST_ACK状态,等待初始端的ACK

* TIME_WAIT 初始端收到FIN,然后发送一个ACK回去,其进入到TIME_WATI状态.

* 接收端成功收到ACK后,其进入CLOSED状态.

* 初始端在TIME_WATI状态下会设置一个超时值,在这个时间过后,其就会进入CLOSED状态.

#### TIME_WAIT

初始端在发送了最后一个ACK,就是对接收端的FIN的回应,之后,会进入TIME_WAIT状态,在等待2个MSL
之后,进入closed状态.

MSL为Maximum Segment Life,表示一个IP数据包子啊网络中能存在的最大时间,超过这个时间的IP数据包
将从网络中消失,在linux中,其为2分钟.

TIME_WAIT的持续时间是2个MSL,所以为4分钟.

#### CLOSE_WATI
CLOSE_WAIT发生在被动关闭的那一方. 也就是主动关闭方发送的FIN被ACK之后,被动关闭方就会进入到
CLOSE_WAIT状态. 正常情况下,CLOSE_WAIT状态只会存在一会儿,因为被动关闭方马上就会发送自己的FIN
到主动关闭方而进入到LAST_ACK状态. 如果系统中出现大量的CLOSE_WAIT状态的连接,问题一般都是我们
自己编写的程序有问题.

也就是我们的程序在接收到client的关闭请求之后,并没有调用关闭socket的函数,这样server就不会
发送自己的FIN到client去.
这样的结果就是我们的server端有大量的处于CLOSE_WAIT的连接,对应的client端有大量的处于FIN_WAIT_2的状态.





