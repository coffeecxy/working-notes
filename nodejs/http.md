http
====
http这个module是nodejs最重要的一个Module，我们使用nodejs大部分就是要使用这个module的。

## http.STATUS_CODES
这是一个object，里面罗列常用的http status code和他们对应的status text。

## http.createServer([requestListener])
返回一个web server object。 类型为`http.Server`。

[requestListener]表示这个requestListener，一个回调函数，可以给出，也可以不给出。如果给出了的话，这个函数就会被加到这个server的`request` event的listener中去。

这个函数是我们用得最多的。 调用这个函数之后，会返回一个Http server，后面我们对这个server进行设置。

## Class: http.Server
上面的`createServer`返回的就是这个类型的object。

其实际上是一个` EventEmitter `，会发送出如下的一些event。

### Event: 'request'
`function (request, response) { }`
这个为其监听函数的原型。

每次这个server收到了来自客户端的请求的时候，这个事件就会触发。 
监听函数中的request是`http.IncomingMessage `，response是` http.ServerResponse`

### Event: 'connection'
`function (socket) { }`

当一个新的TCP连接在这个server上面建立的时候，会发出这个事件。

`socket`的类型是`net.Socket`。 一般的，用户不会监听这个事件，而是会监听上面的`request`事件。

### Event: 'close'
`function () { }`

当这个server关闭的时候，其会发出这个事件。

### server.listen(port, [hostname], [backlog], [callback])
让这个sever开始监听特定的端口。 server创建之后，其不是马上就开始监听了，而是调用了这个函数之后才开始监听。

port是必须给出来的。 

hostname是可选的，如果不给出的话，那么从任何IP进来的连接都可以，因为一个主机上面可能有好几个网络接口，它们都有自己的IP，如果指定IP的话，那么只有那个IP进来的才可以，一般的，都不需要指定。
如果在Linux上面，而且不使用http，而是使用unix socket，那么把hostname设置为这个Unix socket的文件名。

backlog表示的在等待连接的连接的数量，这个一般也不用设置。

callback是一个函数，如果给出了，其会被添加为`net.Server`的`listening`的CB。 这个TCP上面的事情，我们也不会去用。

### server.listen(path, [callback])
启动一个unix socket server，然后监听指定的`path`的路径。

这个是用在unix socket上面的。

### server.close([callback])
关闭这个server，这样这个server就不能再接收连接请求了。

### server.setTimeout(msecs, callback)
### server.timeout = xxx
设置连接的socket的timeout时间。 如果一个socket的timeout发生了，那么会发送一个`timeout`事件到这个server上面，其参数为这个socket。

如果在Server上面有一个`timeout`的listener，那么其就会被调用了。

默认的，Server将timeout的时间设置为2分钟，然后server会将这个超时的socket删除了。 但是，如果自己在server上面加了一个`timeout`的listener，那么就要自己处理这个事情了。

这个地方要处理的场景是，现在一般的浏览器在发送请求的时候，都会给出

	Connection: keep-alive
	
就是说其会在后面使用当前建立的这个socket/tcp来请求后面需要的东西。 nodejs也是同意这样做的，但是其认为不能无限的保留这个tcp链接，所以默认在2分钟的时候就会关闭这个链接了。 一般的，我们就使用这个默认就可以了。

## Class: http.ServerResponse
这个object只会在Server内部自己创建，用户是创建不了的。 它会被传给server的`request`的第二个参数。

### Event: 'close'
`function () { }`

表示下面使用的TCP链接在` response.end()`调用之前就断了。

比如用户使用浏览器访问了这个网址，server生成了这个response来返回相应的数据，在这个response还没有处理完成的时候，用户把浏览器给关闭了，那么下面的tcp链接就会断了，此时这个事件就会触发了。


### Event: 'finish'
`function () { }`

当response被发送了之后，就会发出这个事件。 更加准确的说，这个事件会在response的header和body都被推送给操作系统之后就发出。 并不是说client一定就成功的接收到了这个response。

这个事件发出之后，response就不会再发出其他的事件了。



总的来说，Node.js本身提供的这个`http` module本身是没有提供什么功能的，也就是说，其提供的功能就是完成了对HTTP协议的支持。 其提供的功能和java Servlet api提供的是类似的。 如果直接基于`http`写一个web应用，会发现十分的麻烦。 所以在node.js中就有很多的web 框架。

现在用得最多的是express框架。 express框架本身是建立在`http`上面的，而且其设计思想是顺着http的设计模式。 结果就是里面会使用到大量的回调函数，如果不喜欢回调函数，会发现代码看起来很麻烦。

基于这个原因，现在express的原班人马在开发一个新的框架，叫做koa
