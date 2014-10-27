servlet
===

在JAVA EE中，servlet是最重要的一个标准，其要解决的问题就是C/S模型中的server端的处理。servlet API很好的定义了这个问题。

HTTP协议是当前使用的最广泛的一个C/S协议，servlet本身是通用的，HTTP servlet具体化了，使其适合处理HTTP协议。


### keep-alive connection
在HTTP1.1中，对于从client发出的request，其header中引入了`Connection`字段，我们现在看到的一般都是`Connection: keep-alive`。其要解决的问题是，当browser请求了一个html文件之后，这个HTML文件本身一般都包含了img等等需要再次请求的字段，在HTTP1.0中，对于每一次请求，client都必须新建一个socket到server，server也必须新建一个socket与之通信，当请求的文件发送完成之后，这个socket就会关闭了。当keep-alive之后，这个client之后要请求的资源也是通过这个socket进行发送。

那么就引入了一个问题，在这个socket上面会传送好几个资源，那么client怎么知道一个资源传送完了，然后可以进行下一次请求了。

有两个方法，一个是在response的header中加入`Content-Length: xxxx`这个字段。我们知道socket的传输时一个流，也就是client一定是先接收到status code,然后是header，然后才是body。所以client接收body的时候就知道body的大小了，那么接收完呢这么多数据之后就可以进行下一个资源的request了。这个方法适合那些对静态资源进行访问的情况，比如说一个html文件，图片，js代码,css代码，server可以知道这些文件的大小，当然也就可以进行设置了。

但是对于动态生成内容并返回的情况，比如在servlet的`doGet`返回中使用`response.getWriter()`,然后不断的通过这个writer将内容发送给client的情况，`doGet`函数返回前，是根本不知道到底会发送多少内容的。此时就需要在response的header中加入`Transfer-Encoding: chunked`而不是`Content-Length: xxxx`了。chunked是一种编码，这种编码将数据分为一个一个的chunk，当出现某一个特殊的chunk的时候，表示内容结束了。那么servlet将多少数据作为一个chunk呢？然后servlet在什么使用使用`Content-Length: xxxx`，什么使用使用`Transfer-Encoding: chunked`？

servlet发送数据的时候使用了一个buffer。`response.getBufferSize()`返回这个buffer的大小，首先，所有要发送的数据都被写入到这个buffer中，如果在`doGet`函数执行完了，写入buffer中的数据的大小<=buffer的大小（也就是buffer装下了所有的要发送的数据），那么servlet就会设置为`Content-Length: xxxx`，因为其知道要发送的内容的大小，然后再将buffer中的所有数据发送出去。如果在`doGet`执行中，buffer就被填满了，那么servlet就不知道要发送的内容到底有多大了，那么就会使用`Transfer-Encoding: chunked`，然后buffer中的数据被发送，然后buffer清空，不断的再写数据到buffer中。
