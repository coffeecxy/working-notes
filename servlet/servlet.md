servlet
===

在JAVA EE中，servlet是最重要的一个标准，其要解决的问题就是C/S模型中的server端的处理。servlet API很好的定义了这个问题。

HTTP协议是当前使用的最广泛的一个C/S协议，servlet本身是通用的，HTTP servlet具体化了，使其适合处理HTTP协议。

### servlet-mapping & DefaultServlet
每一个servlet实际上都是一个类，其被实例化之后就可以用来对特定的URL进行服务，那么这个特定的URL就是通过`web.xml`中的`servlet-mapping`字段指名的
```
<servlet>
    <servlet-name>helloworld</servlet-name>
    <servlet-class>Sample.HelloWorld</servlet-class>
</servlet>
<servlet-mapping>
    <servlet-name>helloworld</servlet-name>
    <url-pattern>/hello</url-pattern>
</servlet-mapping>
```
比如上面的配置就指定了`Sample.HelloWorld`这个servlet可以服务于/hello这个URL。

在`<url-pattern>`中加入通配符`*`，那么就可以让一个servlet服务一定的URL。下面的四个匹配方式优先级不断降低。

* 使用直接匹配，比如`/hello.html`，`/images/chart.gif`，意思是只有这个URI可以被匹配到，其他所有的URI都不能被匹配到。
* 使用前缀匹配，比如`/lite/*`,` /catalog/item/*`，其规则是以`/`开头，然后以`/*`结尾，那么就可以匹配到URI中以该前缀开始的所有URI。使用`request.getPathInfo()`，可以得到`*`部分的值。
* 使用扩展名匹配，这个是用来匹配特定格式的文件的，比如`*.wm`和`*.jsp`，这个就是说，RUI中只要以这个格式结尾，那么匹配上了，这个就是JSP的默认匹配方式。
* 默认匹配，就是`/`,这个是用来进行静态文件服务的，比如css,js,img这些文件，其实际表示的是`/*`，也就是匹配任何URI，但是因为其是优先级最低的，所以当签名三个匹配都没有匹配上的时候才会被匹配，也就是所以前面处理不了的，都放到这儿来处理，其实际使用的servlet是`org.apache.catalina.servlets.DefaultServlet`，完成的事情就是进行静态文件的发送，如果给出的路径找不到静态文件，那么就404了。

### keep-alive connection
在HTTP1.1中，对于从client发出的request，其header中引入了`Connection`字段，我们现在看到的一般都是`Connection: keep-alive`。其要解决的问题是，当browser请求了一个html文件之后，这个HTML文件本身一般都包含了img等等需要再次请求的字段，在HTTP1.0中，对于每一次请求，client都必须新建一个socket到server，server也必须新建一个socket与之通信，当请求的文件发送完成之后，这个socket就会关闭了。当keep-alive之后，这个client之后要请求的资源也是通过这个socket进行发送。

那么就引入了一个问题，在这个socket上面会传送好几个资源，那么client怎么知道一个资源传送完了，然后可以进行下一次请求了。

有两个方法，一个是在response的header中加入`Content-Length: xxxx`这个字段。我们知道socket的传输时一个流，也就是client一定是先接收到status code,然后是header，然后才是body。所以client接收body的时候就知道body的大小了，那么接收完呢这么多数据之后就可以进行下一个资源的request了。这个方法适合那些对静态资源进行访问的情况，比如说一个html文件，图片，js代码,css代码，server可以知道这些文件的大小，当然也就可以进行设置了。

但是对于动态生成内容并返回的情况，比如在servlet的`doGet`返回中使用`response.getWriter()`,然后不断的通过这个writer将内容发送给client的情况，`doGet`函数返回前，是根本不知道到底会发送多少内容的。此时就需要在response的header中加入`Transfer-Encoding: chunked`而不是`Content-Length: xxxx`了。chunked是一种编码，这种编码将数据分为一个一个的chunk，当出现某一个特殊的chunk的时候，表示内容结束了。那么servlet将多少数据作为一个chunk呢？然后servlet在什么使用使用`Content-Length: xxxx`，什么使用使用`Transfer-Encoding: chunked`？

servlet发送数据的时候使用了一个buffer。`response.getBufferSize()`返回这个buffer的大小，首先，所有要发送的数据都被写入到这个buffer中，如果在`doGet`函数执行完了，写入buffer中的数据的大小<=buffer的大小（也就是buffer装下了所有的要发送的数据），那么servlet就会设置为`Content-Length: xxxx`，因为其知道要发送的内容的大小，然后再将buffer中的所有数据发送出去。如果在`doGet`执行中，buffer就被填满了，那么servlet就不知道要发送的内容到底有多大了，那么就会使用`Transfer-Encoding: chunked`，然后buffer中的数据被发送，然后buffer清空，不断的再写数据到buffer中。

### status code
response的第一个字段是status code,表示对于这个request做出的反应，默认的，也就是使用的是最多的是`HTTP/1.1 200 OK`,表示对于request需要的资源，servlet可以正确的给出，那么后面的header和body就是这个资源的信息了。

也有的情况是这个资源servlet没有，或者是其他的很多错误类型。此时就有两种办法来返回这个错误了。

比如对于404 NOT FOUND，首先使用`response.setStatus(HttpServletResponse.SC_NOT_FOUND)`来设置返回的status code，然后使用`response.getWriter()`来将我们自己设计的not found页面的内容返回。

第二种办法就是直接使用`response.sendError(HttpServletResponse.SC_NOT_FOUND)`，其会设置status code为404，同时发送一个预先设计好的页面的内容。

### response headers
response headers里面的内容有很多用处，header里面会出现的字段和status code很有关系。比如status code是200,那么就一般会出现`Content-Length`字段。还有一些特殊的字段。

#### `Location`
经常会有这种情况，当我们访问一个网址的时候，其会`自动`跳转到另外一个网址去。比如我们访问google中国的时候，自动会跳转到google香港去。

response header中的`Location`字段就是用来完成这个事情的,当使用`response.sendRedirect("http://www.baidu.com")`的时候，函数的是

```
HTTP/1.1 302 Found
Server: Apache-Coyote/1.1
Location: http://www.baidu.com
Content-Type: text/plain;charset=ISO-8859-1
Content-Length: 0
Date: Mon, 27 Oct 2014 12:10:04 GMT
```
当浏览器接收到这个字段之后，浏览器就会直接去访问`http://www.baidu.com`这个URL。

#### `Refresh`
还有一个有趣的现象，有些网站，比如赛事的文字直播网站，其会给出一个选项，让我们选择多久自动刷新一下页面。

我们知道现在我们可以使用js代码来完成这件事情。但是在服务器端也可以完成这个事情，就是通过在response的header中设置`Refresh`字段，那么浏览器就会在指定的时间之后再请求同一个url.

还有一个就是访问一个网址的时候，其在一定的时间之后去到另外一个网址，其也是通过`Refresh`字段实现的，当然，现在我们更加倾向于用js实现这个功能。

## client pull & server push
在需要不断地实时更新内容的页面中，有两种方法可以完成（除去使用js代码的）

一个就是client pull，原理是让browser不断地刷新同一个页面，就是上面提到的`Refresh`字段。这个方法的一个不好的地方就是每次browser刷新一下页面，都会有一个新的socket建立。

第二个方法是server push，就是让server 返回的时候在头中加入
`Content-Type:"multipart/x-mixed-replace; boundary=End"`，意思是返回的不是一个页面，而是很多个页面，每个页面用指定的boundary来分开，这儿我们使用的boundary是`End`。replace表示后面的页面会不断的替换前面的页面。
```

--End
Content-type: text/plain

3...

--End
Content-type: text/plain

2...

--End
Content-type: text/plain

1...

--End
Content-type: text/plain

0...

--End
--End--

```
上面就是一个使用server push返回的`body`，可以看到每一个页面都是由这个页面的头部开始，然后有一个空行，然后是这个页面的body，然后有一个`--End`。所有页面都完了之后，就是`--End--`，表示这个server push完了。

使用server push的话，就只有一个socket连接，但是servlet会运行得比较久。

>需要注意的是，`Content-Type:"multipart/x-mixed-replace; boundary=End"`现在在chrome浏览器中是不支持的，而且在IE浏览器中，其一直都是不支持的，或者是支持得不好的，我完成上面的结果都是在firefox中做的。

### 使用cookie来进行session管理
为了保持session，有很多中办法，但是好些办法因为其实现起来的复杂性或者是安全性的问题，都没有被使用了。cookie技术现在是使用的最广泛的一个session管理技术。

session有两种分类，一个叫做`temporary session`，一个叫做`persistent session`。

`temporary session`就是说在一次对一个网址进行访问的过程中，对于每个页面的请求服务器都能记住这个client。但是在下一次对这个网址进行访问的时候，浏览器就不知道这个client了。

`persistent session`就是说，只要client访问过一个网址，这个网站就记住了这个用户了。

在servlet中`public Cookie(String name, String value)`可以创建一个cookie。我们知道一个cookie有好些属性，这些属性可以使用`Cookie.setXxx`来进行设置，然后使用`public void HttpServletResponse.addCookie(Cookie cookie)`来将这个cookie写到`response`的header中，这样浏览器收到这个response之后，就会将这个cookie存储或者更新了。

cookie的属性中比较重要的有
* MaxAge,表示这个cookie浏览器将存储的时间。有三种可能的值，默认值是-1，表示是一个session cookie，就是在浏览器关闭的时候这些cookie会被删除。0，表示浏览器要将这个cookie删除了。正数，表示浏览器可以保存这个cookie的时间，以秒为单位，比如我们想保持1个星期，那么就需要设置值为`24*60*60*7`。
* Domain，默认情况下，只有当浏览器再次访问`设置cookie的host`的时候，其才会返回这个cookie，但是有时候我们想突破这个限制。第一个字符必须是`.`，而且至少应该有两个`.`，比如当访问`www.foo.com`的时候其设置了一个cookie，将其Domain设置为`.foo.com`，那么当访问www.foo.com和mail.foo.com也就是任何xxx.foo.com的时候，这个cookie会被返回。但是访问www.abc.foo.com的时候，这个cookie就不会被返回了。这个属性可以扩展一个cookie使用的范围，但是不能无限扩展。特别的，当我们访问一个host的时候，比如`www.foo.bar`,不能设置另外的domain的cookie，比如`.bar.com`。
* Path，默认情况下，只有当再次访问设置cookie的URI或者更久具体的URI的时候，这个cookie才会返回，比如，当访问`servlet/Foo`的时候设置了一个cookie，那么只有在访问`/servlet/*`的时候这个cookie才会被返回，而访问`/`或者是`/aa`之类的就不会被返回。将Path设置为`/`，那么就可以保证所有URI都会返回这个cookie。

### HttpSession类
使用前面介绍的cookie，可以实现一个功能，servlet给每个client一个唯一的sessionid，这个sessionid对应着一块servlet中的内存，在这个内存中可以存储这个会话中的所有信息。这个sessionid用cookie（或者是其他的方式）传到client去，那么以后浏览器每次request都给出这个sessionid，那么这个session就被确定了。

servlet中实现了`HttpSession`这个类来实现上面的功能。然后会使用`JSESSIONID`这个cookie来进行交互，这个可以从浏览器的调试窗口中看到。

注意`HttpSession`在内存中存放的时间是有限的，意思是servlet接收到了一个request，那么其就开始计时，如果计时到了一个规定的值，client还没有发送第二个request过来，那么这个session就会从内存中清除了。因为server的资源有限，如果所有的session都保存的话，那么server肯定会受不了的。

`HttpSession`本身只能用于一次会话中的保存，有三种情况可以使得这个session无效了，也就是不存在在内存中了，也就是client使用那个给定的`JSESSIONID`无法再访问到这个内从了。
* 第一个就是浏览器关闭了，因为给出的这个`JSESSIONID`这个cookie是一个session cookie，当浏览器再次启动并访问该页面的时候，`JSESSIONID`这个cookie
