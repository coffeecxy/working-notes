DIALY
====

### 2014年11月17日

#### maven
使用maven的时间还不是很长，有些问题还是要记一下。

    <dependency>
        <groupId>javax.servlet</groupId>
        <artifactId>servlet-api</artifactId>
        <version>2.5</version>
        <!--<scope>provided</scope>-->
    </dependency>
	<dependency>
	    <groupId>javax.servlet.jsp</groupId>
	    <artifactId>jsp-api</artifactId>
	    <version>2.1</version>
    	<scope>provided</scope>、
	</dependency>
		

使用spring的时候，肯定是要使用servlet api的，当IDEA使用模板生成一个spring mvc项目的时候，其是给出了上面的servlet的dependency的，但是调试的时候总是会有说在`HttpServletRequest`中没有`getRequestDispatcher`这个函数，但是在servlet api 3.1中，这个函数实际上是有的。

我使用的是tomcat 8，其自带了servlet-api的jar文件，所以只需要在dependency中加入`<scope>provided</scope>`,这样在生成war文件的时候，这两个文件就不会被打包到`lib`文件夹下面去，而是使用tomcat自己提供的servlet-api的jar文件。就不会出现这个问题了。

        <!--servlet-->
        <!--这两个jar文件在tomcat中是提供了的所以其scope要使用provide-->
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>servlet-api</artifactId>
            <version>2.5</version>
            <scope>provided</scope>
        </dependency>

        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.1</version>
            <scope>provided</scope>
        </dependency>
		
所以对于idea生成的mvc项目，一定要记住在`pom.xml`加上这句话。

#### @Controller
首先，看看@Controller的定义
```java
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Component
public @interface Controller {

	/**
	 * The value may indicate a suggestion for a logical component name,
	 * to be turned into a Spring bean in case of an autodetected component.
	 * @return the suggested component name, if any
	 */
	String value() default "";

}
```
其中之重要的是其有`@Component`这个annotation，annotation是可以继承的，当然，除了其value属性，所以@Controller就是一个@Component。也就是其会被扫描到，然后例化成一个bean.

当然，要让这个@Controller被扫描到，我们需要在相应的`DispatcherServlet`的配置文件中加上

    <context:component-scan base-package="com.springapp.mvc"/>
在IDEA生成的项目中，这句话是已经有了的。

#### @RequestMapping
@Controller和@RequestMapping一起就可以设计出restful的处理方式了。

@RequestMapping可以出现在type也就是class上面，表示这个controller要处理的一个事情,其后面跟上的值表示其要处理的uri，注意这儿给出的是`/greeting`，表示其可以匹配到的URI是`/greeting`,`/greeting.*`,`/greeting/`，也就是`/greeting`后面可以使用或者不使用后缀，或者是`/greeting/`本身。如果给出的是`/greeting/`，也就是在结尾处给出了`/`，那么只会匹配到`/greeting/`。所以一般情况下都不要给出末尾的`/`。
```java
@Controller
@RequestMapping("/greeting")
public class GreetingController {
	@RequestMapping(method = RequestMethod.GET)
	public String greeting(ModelMap model) {
		model.addAttribute("message", "Hello world!");
		return "hello";
	}

	@RequestMapping("/yes")
	public String yes(ModelMap model) {
		model.addAttribute("message", "Hello yes!");
		return "hello";
	}

}
```
当出现在method上面的时候，表示这个method是一个handdler，给出的值会起到一个narrow的作用，比如上面的`yes`函数就会匹配到`/greeting/yes`，`/greeting/yes.*`,`/greeting/yes/`，而`greeting`函数没有narrow的字符串，但是其指定了能匹配到的method，默认情况下，所有的method都是可以的，这儿`greeting`指定只有`GET`可以了。

#### URI template
在rfc中，有URI template，其语法就是将相应的template包含在`{}`中，比如`http://www.example.com/users/{userId}`那么其可以匹配到`/{userId}`,`/{userId}.*`，`/{userId}/`，真实给出的URI中的相应的值就是userId的值。

```java
	@RequestMapping("/{userId}")
	public String yes(@PathVariable String userId,ModelMap model) {
		model.addAttribute("message", "Hello yes!");
		model.addAttribute("userId",userId);
		return "hello";
	}
```

使用`@PathVariable`可以将URI中的这个值给提取出来，变成参数的传入值。比如上面的代码，访问的uri是`/greeting/abcde`的话，那么`yes`函数的`userId`就是`abcde`，当然，其也可以不是String类型的，但是一定要是可以被正确的转换成相应的类型，如果上面使用的是`int userId`，而给出的是`abcd`，那么会抛出`TypeMismatchException`,在tomcat中，其会返回一个404，给出的信息是`The request sent by the client was syntactically incorrect.`

注意这个地方使用的函数的变量的名字和uri template的名字是一样的，这样会比较方便。

##### params和headers
这两个@RequestMapping的属性是用来narrow匹配的，就是如果路径匹配上了，但是要求client发送过来的request必须有相应的paramter或者是header，那么如果发过来的request是没有的话，也会导致匹配不上。

#### method中可以出现的参数
##### @RequestParam 
@RequestParam可以从request的URL中提取出其parameter的值。

	@RequestMapping(value = "/param")
	public String useParam(@RequestParam(value = "p1",defaultValue = "234") int p1,ModelMap model) {
		model.addAttribute("message", "Hello yes!");
		model.addAttribute("p1", p1);
		return "hello";
	}

如上，如果给出的是`/param?p1=12`，那么输入参数`p1`就是`12`，默认情况下，必须给出`p1`这个参数，如果使用了`defaultValue`，那么就可以不给出这个param，其会使用给出的默认值。

##### @RequestBody
没有搞清楚是怎么回事。
**？？？？？**

##### @ResponseBody
这个annotation是用在method上面的，其表示这个method返回的值是直接变成返回给浏览器的response的body部分。

    @RequestMapping(value = "/resbody")
    @ResponseBody
    public String useResBody() {
        return "just the response body";
    }

当访问`/resbody`的时候，就会直接返回`just the response body`为其body。

##### @ResponseBody

##### @ModelAttribute
当@ModelAttribute作用在一个method上面的时候，表示这个method不会处理request，而是在**request**运行之前，都会被运行，其会向`ModelMap`中加入一些属性，这样当request handdler执行的时候，`ModelMap`中就总是有相应的属性了。

    @ModelAttribute
    public void addAccount(@RequestParam String name,
                           @RequestParam int age,
                           ModelMap modelMap) {
        modelMap.addAttribute("name", name);
        modelMap.addAttribute("age", age);
    }

    @RequestMapping("/model")
    public String modeluse() {
        return "hello";
    }
	
比如上面的代码，当访问`/model`的时候，addAccount会先执行，其向`ModelMap`填充如一些数据，然后`modeluse`才会被执行，在`hello.jsp`文件中，我们可以使用`${name}`和`${age}`.

当`@ModelAttribute`用在method的param前面的时候，表示这个参数将从`modelMap`中提取出来，具体的提取方法比较复杂。

##### @SessionAttributes
`@SessionAttributes`只可以用在controller type上面，表示指定的attribute会在session中被保存，比如
`@SessionAttributes("pet")`那么pet这个attribute会被保存在session中，在下一次request的时候也可以被使用。

##### @CookieValue
    @RequestMapping("/session")
    public String useSession(@CookieValue("JSESSIONID") String sessionId,
                             ModelMap modelMap) {
        modelMap.addAttribute("sessionId", sessionId);
        return "hello";
    }
使用`@CookieValue`可以从request的cookie 头部得到cookie的值，其和@RequestParam是一样的，如果转换的时候不是使用string，那么一点要保证其有效性。

#####  @RequestHeader
和上面的`@CookieValue`是一样的，从request header中提取出需要的字段。

##### 支持Last-Modified Response
使用缓存可以很好的减轻服务器的负担，当使用spring mvc的时候，对于一个method，可以使用`request.checkNotModified(nowTime)`来看看要返回的内容是不是变化了，如果其返回的是`true`，表示内容没有变化，返回一个Null,那么就不会让view resolver来生成显示的内容的body了。这样就可以很快的返回。

    @RequestMapping("/lastmodi")
    public String useSession(WebRequest request,
                             ModelMap modelMap) {

        if(request.checkNotModified(nowTime)) {
            return null;
        }
        return "hello";
    }
	
在spring中，还可以实现`LastModified`这个interface，但是具体怎么做我还不清楚。**？？？？？**


##### @JsonView
这个使用了jackson的json处理，Java有很多的处理json的库，但是jackson是用的很多的，这儿需要对jackson有一定的了解，**？？？？**

### 2014年11月18日

#### view和view resolver
作为MVC中的view，在spring中实现的时候有两个重要的interface，`View`和`ViewResolver`，`ViewResolver`将给出的`view`的名字（string）解析成一个view，而`view`负责使用一种view technology生成输出到浏览器的内容。

每个controller的handler都会显式或者隐式的返回一个view name，比如返回一个string，就是显式的返回的，如果什么都没有返回，那么就会用函数名作为view name。view resolver会负责将

##### redirect
有些时候处理一个request的时候，我们会需要将其进行redirect，在spring mvc中，可以使用将返回的view name加上redirect的前缀，当viewresolver看到这个前缀的时候，其不会将其解析成一个view，而是马上返回一个`302 Found`，然后在response header中给出一个Location字段。

    @RequestMapping("/redirect")
    public String useRedict() {

		//return "redirect:/main/greeting";
       return "redirect:http://www.baidu.com";
    }
	
##### ContentNegotiatingViewResolver
可以完成对同一个资源，用不同的view来返回，比如用html,pdf,excel之类的，其处理的都是同一个model。
这个view resolver本身并不是用来resolve一个view name的，但是其有`private List<ViewResolver> viewResolvers;`，其根据这些viewResolvers选择一个和需要的返回类型最相近的view来处理。
同时，其还有`private List<View> defaultViews;`，如果找不到一个viewResolver可以匹配到，那么就会使用这个默认的view来处理。

具体的使用方法要到时再看。

##### multipart (file upload)
要进行文件上传我们又两种方法，一个是使用form的file标签，然后使用POST和multipart/form-data的方式来传送文件。

	<form action="/main/upload" method="post" enctype="multipart/form-data">
	    <input type="text" name="name" value="fileName"/> <br/>
	    <input id="fileUpload" type="file" name="file"/> <br/>
	    <input type="submit" value="上传1"/>
	</form>

form的enctype属性表示在请求被提交之前，浏览器需要对form的内容进行编码，可以使用的编码有如下：
* application/x-www-form-urlencoded 默认的编码方式，其就是进行url编码，就是将表单中的空格变成+号，其他的URL中的字符也进行编码。这个需要接收端对其进行相应的decode。在编码完成之后，实际传输的request的body为a=b&c=d...，其中的a,b,c,d都是编码之后的。如果我们要传输的就是简单的字符串，那么就是使用这种编码。
* text/plian 对表单中的值不进行任何编码，输入的是什么东西，就将什么东西放到body中，每个input占用一行，如下。

```
name=fileName
file=asdfasf
```
这个编码方式使用的很少，因为如果要使用这个编码方式的话，那么我们应该是要使用默认的url编码方式。

* multipart/form-data 这种编码方式实际上也是不会进行任何编码，其主要是对于要上传文件的时候使用。对于所有非file type的input，其直接将其中的内容放到如下的编码格式中。

```
Content-Type: multipart/form-data; boundary=---------------------------994144521677
Content-Length: 258

-----------------------------994144521677
Content-Disposition: form-data; name="name"

fileName
-----------------------------994144521677
Content-Disposition: form-data; name="file"

sdafddff 我们可以
-----------------------------994144521677--

```

上面的结果是从firefox的firebug中得到的，在这个POST的body中，首先指出其为一个`multipart/form-data`，同时指名了其boundary，然后给出了其length，而后面的数据中就有两个部分，每个部分用上面指定的bounary来进行区分，最后有一个结束。
还可以看到的是，每个part没有指名使用的编码格式，实际上其使用的是text/plian，也就是都没有编码。

```
Content-Type: multipart/form-data; boundary=---------------------------36551657111622
Content-Length: 104854

-----------------------------36551657111622
Content-Disposition: form-data; name="name"

fileName
-----------------------------36551657111622
Content-Disposition: form-data; name="file"; filename="Contig.zip"
Content-Type: application/octet-stream

Contig.zip's content(it's very long)
-----------------------------36551657111622--
```
上面表示的是如果我们的表单里面有一个是使用的file type，那么那个part会有一个Content-Type字段，表示该文件的编码，比如上面的zip文件，其就是使用的octet-stream编码，如果是图片的话，又会有其他的编码。

在服务器端，需要如下的代码

    @RequestMapping
    public String handleFormUpload(@RequestParam("name") String name,
                                   @RequestParam("file") MultipartFile file) {
        if (!file.isEmpty()) {
            try {
                FileCopyUtils.copy(file.getBytes(), new File("d:/tmp/" + file.getOriginalFilename()));
            } catch (IOException e) {
            }
            return "upload/success";
        }
        return "upload/fail";
    }

注意这儿使用了`@RequestParam`，其中的MultipartFile是spring自动转换过来的。

### 处理Exception
当使用一个controller的method来处理一个request的时候，很有可能在处理的时候会有exception抛出。
这个exception会一直向上抛出直到有代码catch了它，如果到最后都没有catch到，那么就会到达servlet container那儿，在我使用的tomcat中，其就会返回一个500回去。

```java
@Controller
@RequestMapping("/exce")
public class MyExceptionController {

    @RequestMapping
    public String doEx(@RequestParam boolean thr) throws IOException {
        if (thr) {
            throw new IOException("this is an exception I manually throw");
        }
        return "excep/ok";
    }

//    @ExceptionHandler(IOException.class)
    public String handleTheException(IOException ex) {

        return "excep/ex";
    }
}
```

比如对于上面的代码，当使用`/exec?thr=true`来访问的时候，其会抛出异常，但是没有代码将这个异常捕获，所以tomcat会返回一个`HTTP Status 500`的页面。

在spring中，有几种方式来处理异常，其中最简单直接的就是使用`@ExceptionHandler`这个annotation来标注一个method。

对上面的代码，如果将注释去掉，那么这个controller中的所有的使用`@RequestMapping`标注的方法抛出的`IOException`异常都会在这个函数中被捕获。@ExceptionHandler的value表示这个method会处理的所有的exception的类型，如果这个地方没有给出，那么所有在method参数中给出的类型都会被捕获。
注意这儿我们没有使用try/catch结构来写这个代码，这样就可以把逻辑处理和异常处理给分开了。
对于`handleTheException`函数，其接收的输入和输出参数与使用`@RequestMapping`函数可以使用输入和输出参数是类似的。
在这个代码中，返回了一个String，那么就是view的name，其会被viewResolver解释成一个JSP文件的名字。

### 打开spring输出的DEBUG信息
为了输出spring在做的每一步的详细，需要使用`log4j`，首先在`pom.xml`文件中添加

    <!--log4j-->
    <dependency>
        <groupId>log4j</groupId>
        <artifactId>log4j</artifactId>
        <version>1.2.17</version>
    </dependency>
	
这样IDEA就可以直接从maven中下载apache log4j包了。

然后需要配置log4j

新建`\src\main\resources\log4j.properties`文件，向文件中写入如下的内容，

	log4j.rootLogger=DEBUG, A1
	log4j.appender.A1=org.apache.log4j.ConsoleAppender
	log4j.appender.A1.layout=org.apache.log4j.PatternLayout
	log4j.appender.A1.layout.ConversionPattern=%-4r %-5p [%t] %37c %3x - %m%n

上面的内容使用了`properties`文件的语法，具体的信息要看log4j的文档。
需要特别注意的是第一句`log4j.rootLogger=DEBUG, A1`，其表示要输出的信息的级别，默认的是`INFO`，就是在IDEA中可以看到的信息前面都有`INFO`,这里将其改成了`DEBUG`，所以就可以看到所有的DEBUG信息了。
