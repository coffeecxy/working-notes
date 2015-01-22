DispatcherServlet
=====

DispatcherServlet是spring mvc中最重要的一个类了。

## 使用DispatcherServlet

在/WEB-INF/web.xml中，添加如下的内容来使用DispatcherServlet。

```xml
<!-- 使用spring mvc，需要载入DispatcherServlet -->
<servlet>
    <!-- 名字为zoom,会使用zoom-servlet.xml作为其配置文件 -->
    <servlet-name>zoom</servlet-name>
    <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
    <!-- 一般来说，我们都希望DispatcherServlet直接启动 -->
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
    <servlet-name>zoom</servlet-name>
    <url-pattern>/</url-pattern> #1
</servlet-mapping>
```

在上面的代码中，最重要的是`#1`标注的这行。


在Servlet标准中，对于`url-pattern`的优先级有如下的规定：



1. Map exact URL
1. Map wildcard paths
1. Map extensions
1. Map to the default servlet

上面的匹配优先级从高到低。

第一个精确匹配就是路强里面没有*，比如/test/cxy.html这种。直观的，这样的匹配当然应该有最高的优先级。

第二个叫做wildcard匹配，比如`/abc/*`，那么在/abc/后面的任何路径都会被匹配上。

第三个为后缀匹配，也就是给出的是`*.xxx`这种类型，那么当访问的为任何以`xxx`结尾的路径都能匹配上。

第四个为匹配到默认的DefaultServlet，其匹配的路径是`/`。

注意，在一般的servlet container中，都会有一个DefaultServlet，其匹配到默认匹配。
然后有一个用于处理jsp代码的后缀匹配。


	<servlet-mapping>
	  <servlet-name>default</servlet-name>
	  <url-pattern>/</url-pattern>
	</servlet-mapping>
	<!-- The mappings for the JSP servlet -->
	<servlet-mapping>
	  <servlet-name>jsp</servlet-name>
	  <url-pattern>*.jsp</url-pattern>
	  <url-pattern>*.jspx</url-pattern>
	</servlet-mapping>

可以看出，这两个匹配的优先级都是比较低的。

这儿最容易让人误解的是`/*`和`/`谁的优先级高。从上面的介绍可以看出，`/*`是更高的，而且其是wildcard匹配，所以如果有一个servlet匹配到`/*`，那么后面的后缀匹配和默认匹配都用不上了。

在我用spring mvc的时候，我最开始就将DispatcherServlet给映射到了`/*`上面，导致后面我想访问一个jsp文件的时候，总是有错误。
给出的Log如下的。

	15:18:32,464 DEBUG [JstlView]:166 - Forwarding to resource [/WEB-INF/jsp/products.jsp] in InternalResourceView 'products'
	在这个地方，因为DispatcherServlet被配置为要处理/*上面的请求，所以
	#1 15:18:32,464 DEBUG [DispatcherServlet]:845 - DispatcherServlet with name 'zoom' processing GET request for [/WEB-INF/jsp/products.jsp] 
	15:18:32,464 DEBUG [RequestMappingHandlerMapping]:297 - Looking up handler method for path /WEB-INF/jsp/products.jsp
	15:18:32,464 DEBUG [RequestMappingHandlerMapping]:305 - Did not find handler method for [/WEB-INF/jsp/products.jsp]
	15:18:32,465  WARN [PageNotFound]:1120 - No mapping found for HTTP request with URI [/WEB-INF/jsp/products.jsp] in DispatcherServlet with name 'zoom'
	15:18:32,465 DEBUG [DispatcherServlet]:996 - Successfully completed request
	15:18:32,466 DEBUG [DispatcherServlet]:996 - Successfully completed request
	
在上面的log中，我先访问的是/test，然后其被Forwarding到/WEB-INF/jsp/products.jsp，到这儿都是没有错误的。
但是`#1`标注的地方就开始有错误了，因为我们想要得到的结果是上面配置的jsp那个servlet因为后缀匹配匹配到了，然后来处理这个jsp文件。
但是发现又是DispatcherServlet来处理了。
原因就是现在我们配置的DispatcherServlet匹配的`/*`是wildcard的匹配，其优先级比jsp servlet的后缀匹配高，所以DispatcherServlet就来处理了，然后其又处理不了，就只能返回404了。

当将DispatcherServlet的配置为`/`之后，其优先级最低，所以jsp servlet就能处理了，最后就能得到先要的月面了。

## 在使用web mvc的时候同时要使用DefualtServlet
如果直接使用上面的DispatcherServlet的配置，那么tomcat自带的DefaultServlet就使用不了了，因为其的url-pattern被覆盖了。

DefaultServlet本身是用来进行静态的文件输出的，如果我们还想用DefaultServlet来进行静态文件的输出，那么可以进行如下的配置。

	@Configuration
	@EnableWebMvc
	public class WebConfig extends WebMvcConfigurerAdapter {
	
	    @Override
	    public void configureDefaultServletHandling(DefaultServletHandlerConfigurer configurer) {
	        configurer.enable(); #1
	    }
	
	
	    @Override
	    public void configureViewResolvers(ViewResolverRegistry registry) {
	        registry.jsp("/WEB-INF/jsp/",".jsp"); #2
	    }
	}

`#1`就表示在DispatcherServlet映射到`/`上，导致defaultServlet的配置被覆盖的情况下，将其打开。

`#2`就是配置来进行Jsp的输出的。

## webMvc config中的viewResolver
我们知道DispatcherServlet本身的默认配置是只有一个`InternalResourceViewResolver`，那么在使用webMvc config之后，这个默认配置是不是也有呢。

对web mvc使用默认的配置

	@Configuration
	@EnableWebMvc
	public class WebConfig extends WebMvcConfigurerAdapter {
	
然后在xxx-servlet.xml中没有配置任何的ViewResolver，DispatcherServlet.initViewResolvers函数运行后的viewResolvers的值
	
![](mvcResolver.png)

可以看出，默认配置下,webMvc给出了一个ViewResolverComposite,ViewResolverComposite是实现了ViewResolver接口的，其本身是一个集合，里面包含了一个InternalResourceViewResolver。

所以默认的配置下，webMvc对ViewResolver的配置也是只有一个InternalResourceViewResolver。

    @Override
    public void configureViewResolvers(ViewResolverRegistry registry) {


        InternalResourceViewResolver internalResourceViewResolver = new InternalResourceViewResolver();
        internalResourceViewResolver.setPrefix("/WEB-INF/jsp/");
        internalResourceViewResolver.setSuffix(".jsp");
        registry.viewResolver(internalResourceViewResolver);

        ResourceBundleViewResolver bundleViewResolver = new ResourceBundleViewResolver();
        registry.viewResolver(bundleViewResolver);


    }

如果我们使用configureViewResolvers函数在webmvc中配置了如上的两个ViewResolver，

	<bean id="resolver1" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/jsp/"/>
        <property name="suffix" value=".jsp"/>
        <property name="order" value="2"/>
    </bean>

    <bean id="resolver2" class="org.springframework.web.servlet.view.ResourceBundleViewResolver">
        <property name="order" value="1"/>
    </bean>

然后在xxx-servlet.xml中配置了如上的两个ViewResolver。

![](mvcResolver2.png)

此时的viewResovler是如上的。可以看到上层一共有三个，两个是我们在xxx-servlet.xml中配置的，最后一个是webMvc生成的。
注意它们的排序是按照其order从小到大排列的。
而在这个viewResovlerComposite里面，其包含了在configureViewResolvers中配置的两个viewResolver。这个viewResovlerComposite的默认的order是最大的，所以其会排在最后一个。

更加需要注意的是，在viewResovlerComposite里面，这些viewResolver的排列顺序是**按照它们被加入的顺序的**,就是说，谁先加入，谁就会被先考察。

所以，在使用了webMvc配置的时候，我们最好还是将所有需要的viewResolver放到xxx-servlet.xml中，这样可以手动的指定它们的order，如果全部放在viewResovlerComposite里面,那么需要注意它们的放置顺序。

## ContentNegotiatingViewResolver

在spring中，一般来说，要完成一个功能，都是有很多的方法的，找到一种适合于自己的方法就可以了。

在viewResovler中一个比较特殊的是ContentNegotiatingViewResolver,其核心的思想是，对于同一个资源