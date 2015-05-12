# socket

需要使用socket进行通信.

## socket

    #include <sys/types.h>          /* See NOTES */
    #include <sys/socket.h>

    int socket(int domain, int type, int protocol);
新建一个通信需要的点.

domain参数表示通信使用的网络层的协议. 常用的是如下的:

- AF_UNIX,AF_LOCAL 本地通信,使用unix socket.
- AF_INET,AF_INET6 ipv4/v6协议.

type一般表示传输层的协议.

- SOCK_STREAM 提供一个有序的,可靠的,双向的,面向连接的传输. 一般就是TCP的意思.
- SOCK_DGRAM 支持数据传输(无连接的,不可靠的连接),一般即使UDP的意思.


protocol指定这个socket要使用的protocol. 一般的,一个socket只会有一个protocol,所以一般只需要给出0就是了.

SOCK_STREAM是全双工的比特流. 也就是不会有传输数据的边界记录.(也就是数据可以一直传送). 一个stream socket必须在真正传输数据之前建立连接. 使用`connect`来建立连接. 一旦建立了连接,可以使用`read,write`或者是`send,recv`来传输数据. 当数据传输完成之后,需要使用close来关闭连接.

SOCK_STREAM的socket可以保证数据的传输是不会丢失或者是重复的. 如果在有缓冲区的一方的一块数据不能在一段时间内传输,那么这个连接就会被认为是dead了. 当SO_KEEPALIVE被指定了,那么会使用协议相关的方式来检测另外一方是不是还alive.

SOCK_DGRAM和SOCK_RAW用来进行数据包(datagram)发送. 使用的函数是sendto和recvfrom.

fcntl函数,使用F_SETOWN可以用来设置在接收到SIGURG和SIGPIPE时需要进行的处理函数.

socket的操作会被socket的options控制. 使用setsockopt和getsockopt来设置和得到options.


## bind

    int bind(int sockfd, const struct sockaddr *addr,
                    socklen_t addrlen);
当一个socket使用socket函数创建之后,其会存在于相应address family中,但是还没有一个地址赋值到这个socket上面. bind将addr指定地址给到sockfd对应的socket上面. addrlen表示第二个参数的字节长度.

一般的,这个函数完成的事情是`给一个socket赋值一个名字`.

对于SOCK_STREAM类型的socket,在其`accept`链接之前,需要先给其bind一个本地的地址.

对一个socket进行名字绑定需要遵循的规则对于不同的地址family是不同的. 对AF_INET,要看ip(7),对AF_INET6,就是ipv6(7), AF_UNIX为unix(7).

addr参数的数据类型给家地址类型不同是不同的. sockaddr的数据类型是这样定义的.

    struct sockaddr {
      sa_family_t		sa_family;	/* address family, AF_xxx	*/
      char			sa_data[14];	/* 14 bytes of protocol address	*/
    };

这个数据结构的`唯一目的`是提供一个类型转换的效果,这样可以保证编译的时候没有warning(不同的指针类型). 典型的使用方法如下.

	struct sockaddr_un my_addr;
	memset(&my_addr, 0, sizeof(struct sockaddr_un));
	/* Clear structure */
	my_addr.sun_family = AF_UNIX;
	strncpy(my_addr.sun_path, MY_SOCK_PATH,
		sizeof(my_addr.sun_path) - 1);

	if (bind(sfd, (struct sockaddr *) &my_addr,
		sizeof(struct sockaddr_un)) == -1)
		handle_error("bind");

可以看到,真正传入的addr的数据类型是sockaddr_un,进行了强制类型转换才传到bind的.

因为第二个参数的类型其实不是固定的,所以第三个参数指定addr占用的字节数.

## listen
    int listen(int sockfd, int backlog);
将sockfd标明的socket标注为一个passive socket,也就是在调用了accept之后才能接收进来的requst的socket. (有些socket是不用listen的,bind之后直接就可以接收request了.)

注意sockfd指向的socket是属于SOCK_STREAM和SOCK_SEQPACKET的.

backlog定义了这个socket等待连接的队列的长度,
