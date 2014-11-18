JSP
=====

JSP,全名为`javaserver script`,其受启发与`.net`的asp,现在jsp,asp,php这三种脚本，语言是用的比较多的。

### JSP就是一种特殊的servlet
Java处理JSP的方式和php不一样，php本身就是一种编程语言，它是一种动态的脚本语言，比如以插件的方式嵌入到Apache服务器的方式，那么这个插件就会一行一行的运行php脚本。

JSP本质上是一种特殊的servlet，其工作的方式是，JSP文件会被先编译成一个Java文件，Java文件中是一个servlet的类，然后这个Java文件被编译成class文件，最后再运行。

所以要理解JSP是怎样工作的，就需要搞清楚JSP中的代码是怎么一一对应的变成生成的Java代码的。

### 转换规则
通过
```java
File tmpDir = (File) getServletContext().getAttribute(ServletContext.TEMPDIR);
```
我们可以看到一个JSP文件生成的Java文件的存放的路径，从这个Java文件和我们的JSP文件，可以看出它们的对应关系。

生成的servlet代码就像一个`GenericServlet`一样，其中有`public void _jspInit()`,`public void _jspDestroy()`,
`public void _jspService(final HttpServletRequest request, final HttpServletResponse response) throws IOException, ServletException` 这三个函数。对应着就是`init,destory,service`三个函数。

对应文件中所有的没有用下面介绍的特殊标记包围起来的代码，就是直接输出的，也就是使用`out.print()`输出的。当然，其中的comment除外。

### Expressions and Declarations

` <%= getName(request) %>`的是expression，里面包含的是一个合法的Java 表达式，在生成的Java中，其变成了` out.print( getName(request) );`，其中的`out`是`out = pageContext.getOut();`，其和`PrintWriter out = response.getWriter();`是类似的。

所以一定要记住在`<%=  %>`中的语句不能以`;`结束，因为这个语句会被放在`out.print()`函数中。

`<% %>`中将的代码会出现在上面的`service`函数里面，这个就是一些逻辑处理的东西了。

`<%! %>`中将的代码会出现在上面说的三个函数外面，所有可以用来定义方法，定义instance variable之类的。

### Directives
这种语法：`<%@ directiveName attribName="attribValue" %>`表示是一个Directive，其会影整个页面的生成，一般在JSP代码的中出现的第一个符号就是`page`这个directive。

## JavaBean


在上面的代码中，我们使用的方式是将要表现的内容和我们的处理逻辑写在了一个文件中，而实际上更好的写法是让具体的逻辑实现和输出的内容分开，这个就是JavaBean提出的意义。

JavaBean本质上就是一个Java的object，这个object有一些property，使用JavaBean就是操作这些property的值来实现我们的逻辑。

有三个tag来完成这个功能。

#### jsp:useBean

`<jsp:useBean id="hello" class="Bean.HelloBean" scope="page|request|session|application"/>`

就是根据一个class是实例化一个JavaBean出来，class表示其要使用的类的名字。

scope表示这个bean被存储的地方，page表示就是一个局部变量，request表示使用`request.setAttribute();`来存储这个bean，session表示使用`request.getSession().setAttribute();`来存储。application表示使用`getServletContext().setAttribute();`来存储。

id表示`xx.setAttribute()`的第一个参数，也就是key。

当我们将存入的bean读回来的时候，其默认的方法类型是Object，所以需要使用class来指名其类型，这样才可以进行类型装换。

`<jsp:useBean id="hello" class="Bean.HelloBean"/>`
生成的代码如下
```Java
Bean.HelloBean hello = null;
hello = (Bean.HelloBean) _jspx_page_context.getAttribute("hello", javax.servlet.jsp.PageContext.PAGE_SCOPE);
if (hello == null){
	hello = new Bean.HelloBean();
	_jspx_page_context.setAttribute("hello", hello, javax.servlet.jsp.PageContext.PAGE_SCOPE);
}
```
如果相应的scope中本来就已经有了指定的那个bean了，那么就使用那个，不然的话，新建一个bean。

#### `jsp:setProperty`
`<jsp:setProperty name="beanName" property="*" />`
用来指定bean的一个property的值。其是使用request中的parameter来设置其值的。`beanName`和`jsp:useBean`的`id`是一个意思，就是要设置property的bean的名字，`property`指定要被设置的property。

比如下面的`HelloBean`这个JavaBean。
```java
public class HelloBean {
    private String name="world";
    private int size = 9;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getSize() {
        return size;
    }

    public void setSize(int size) {
        this.size = size;
    }
}
```
其有两个property，`name`和`size`，然后提供了其getter和setter。

当`request`的parameter中包含了name和size的时候，那么其给出的值就会被`setName`和`setSize`给作为参数，bean的相应的property就会被更新了。

`<jsp:setProperty name="hello" property="*" />`会生成如下代码：
```java
org.apache.jasper.runtime.JspRuntimeLibrary.introspect(_jspx_page_context.findAttribute("hello"), request);
```

当然，`jsp:setProperty`也可以指定bean要更新的property，而不是所有的property。
比如
`<jsp:setProperty name="hello" property="name" />`就只更新hello的name property。

#### `jsp:getProperty`
`jsp:getProperty`就是调用bean的getter来得到相应的property。

比如
`<jsp:getProperty name="hello" property="name"/>`
会生成
```java
out.write(org.apache.jasper.runtime.JspRuntimeLibrary.toString((((Bean.HelloBean)_jspx_page_context.findAttribute("hello")).getName())));
```
也就是得到property之后会写到输出的html文件中去。
