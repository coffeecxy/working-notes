spring的一些配置
========

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

## 2014年11月19日12:31:23
### `xxx-servlet.xml`和`@Configuration`
Spring mvc需要配置。
一般的，在servlet container提供的`web.xml`中，我们会定义一个`DispatcherServlet`的servlet,那么按照其名字就可以生成一个`xxx-servlet.xml`，这个文件用来配置Spring 的环境。

对于spring的配置，在最开始的时候，只可以使用xml文件，但是在后面的版本，因为java中引入了annotation，而且有些人不喜欢使用xml来配置，所以现在也有基于annotation来配置了。

比较常见的是同时使用xml文件和java code结合起来配置Spring。

如果要使用Java代码来配置spring，那么和spring的bean一眼，需要一个POJO的Java class.
其要使用`@Configuration` annotation，表明其实用来配置spring的，
同时还需要`@EnableWebMvc`，表明其会配置spring mvc。
```
@Configuration
@EnableWebMvc
public class WebConfig extends WebMvcConfigurerAdapter{

}
```
还要注意的是其extends了`WebMvcConfigurerAdapter`，这样是为了方便编码。

要注意的是，`@Configuration`的meta-annotation中有`@Component`，表示其也是一个bean，所以要让其起作用，那么需要在`xxx-servlet.xml`中添加。


	<context:component-scan base-package="com.springapp.mvc"/>

当然，这句话一般IDE会直接生成。

如果是用xml配置，那么等价的就是

	<mvc:annotation-driven />
	
虽然这样看起来要简单一些，但是真正要配置MVC的时候，会发现使用Java代码更加直观。
