
docker中的网络设置
===

docker中可以开启很多的容器,这些容器就相当于使用virtualBox的时候运行的很多个虚拟机.
使用virtualBox的时候,我们可以对虚拟机的网络进行各种各样的设置,在docker中,也可以实现其中的大部分功能.

下图是一个我应用docker的时候处于的网络环境.

![docker的典型网络环境](docker-net.svg)


在我们的本机上面,有一个物理的网卡,其为`eth0`,其IP为`192.168.0.128/24`,这是因为我们的主机在一个局域网内.实际上,我们可以认为我们的主机在公网中,也就是有一个公网的IP.

docker0这个这个interface是一个虚拟的,因为物理硬件上面并没有这个.在virtualBox中,其也会使用一个虚拟的网络设备.这个网络设备的IP为`172.17.42.1/16`.可以看到,其和eth0并不在一个网段内,但是因为它们都在同一个主机上面,所以它们之间的包是可以forward的.意思是linux主机可以看成是一个router,其会根据自己的路由表来将包在这两个网络接口之间交换.



    Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
    default         192.168.0.1     0.0.0.0         UG    0      0        0 eth0
    172.17.0.0      *               255.255.0.0     U     0      0        0 docker0
    192.168.0.0     *               255.255.255.0   U     1      0        0 eth0

上面是当前主机的路由表.表示目的地是`172.17.0.0/16`网段的包都转发到docker0去,目的地是`192.168.0.0/24`的包都转发到eth0去,其他的包也转发到eth0.也就是主机当前处于两个LAN中.如果要去到公网,通过192那个LAN的gateway.

    Destination     Gateway         Genmask         Flags Metric Ref    Use Iface
    default         172.17.42.1     0.0.0.0         UG    0      0        0 eth0
    172.17.0.0      *               255.255.0.0     U     0      0        0 eth0
上面是任何一个容器的路由表,表示去到当前LAN中的主机的包都从eth0走,其他网段的也走eth0.

下面是主机发送的包可能的路径
* 如果一个包从主机出去到公网,比如ping百度的首页的包,那么其目的地址不是后面指定的两个网段,所以从eth0出去,然后通过gateway到公网. 这个ping响应回来时,其目的地址是本机的IP`192.168.0.128`,所以不会再经过路由了,直接去到主机.
* 如果主机发送一个包到192这个LAN中的一个主机去,比如ping那个主机,那么第三条会被匹配上,这个包从eth0出去,因为这两个主机处于同一个LAN内,所以不会通过这LAN的gateway.
* 如果主机发送一个包到一个docker容器,比如主机ping container  1这个容器,那么这个包的目的地址满足第二条,这个包会被转发到docker0去,回来的包的目的地址是docker0的地址,直接被使用了.

下面是一个容器中的包可能的路径

* 如果容器之间相互通信,比如container 1 ping container 2,那么这个包的源地址是172.17.0.2,目的地址是172.17.0.3,匹配第二条,从eth0进去,



当我们使用`docker run`运行了几个容器之后,每一个容器中都会有一个eth网络设备. 在我们的列子中,有三个容器在运行,可以看到,这三个容器的网络地址都是在同一个网段内的,而且和docker0也是在同一个网段中的. 通过查看它们的route,会发现它们的gateway就是docker0.


    cxy@cxy-X9DAi:~/soft/clion-1.0$ sudo iptables-save
    # Generated by iptables-save v1.4.21 on Sat Apr 11 13:32:34 2015
    *mangle
    :PREROUTING ACCEPT [252924:246361857]
    :INPUT ACCEPT [252814:246304941]
    :FORWARD ACCEPT [28:3302]
    :OUTPUT ACCEPT [167440:15225315]
    :POSTROUTING ACCEPT [167804:15266869]
    COMMIT
    # Completed on Sat Apr 11 13:32:34 2015
    # Generated by iptables-save v1.4.21 on Sat Apr 11 13:32:34 2015
    *nat
    :PREROUTING ACCEPT [1509:317646]
    :INPUT ACCEPT [1493:310172]
    :OUTPUT ACCEPT [6034:382667]
    :POSTROUTING ACCEPT [6034:382667]
    :DOCKER - [0:0]
    -A PREROUTING -m addrtype --dst-type LOCAL -j DOCKER
    -A OUTPUT ! -d 127.0.0.0/8 -m addrtype --dst-type LOCAL -j DOCKER
    -A POSTROUTING -s 172.17.0.0/16 ! -o docker0 -j MASQUERADE
    COMMIT
    # Completed on Sat Apr 11 13:32:34 2015
    # Generated by iptables-save v1.4.21 on Sat Apr 11 13:32:34 2015
    *filter
    :INPUT ACCEPT [53437:44686502]
    :FORWARD ACCEPT [0:0]
    :OUTPUT ACCEPT [41199:4084835]
    :DOCKER - [0:0]
    -A FORWARD -o docker0 -j DOCKER
    -A FORWARD -o docker0 -m conntrack --ctstate RELATED,ESTABLISHED -j ACCEPT
    -A FORWARD -i docker0 ! -o docker0 -j ACCEPT
    -A FORWARD -i docker0 -o docker0 -j ACCEPT
    COMMIT
    # Completed on Sat Apr 11 13:32:34 2015


