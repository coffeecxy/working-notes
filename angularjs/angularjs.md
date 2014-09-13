angular
====
## service
### angularjs中的service到底是什么？
service就是一个javascript object
### 那为什么各个service可以做的事情如此的不一样？
js object是一中相当自由的数据结构，如果object中只有一个property，而这个property就是一个数，一个字符串，那么这个service就简单的提供了这些信息；而如果是一个function，那么这个service就可以做一些事情了。而且object可以有不止一个property，那么这儿service也就可以同时做几件相互有关的事情了。

### service, service factory function, service provider function有什么关系？
首先，service provider function是一个constructor function，也就是用new来调用的function，其会返回一个object。

这个Object就是angular中的provider（也就是会被注入到.config中的东西）。

angularjs规定了provider中必须有$get这个property，而这个property对应的是一个function，这个function就是factor function，调用这个factory function要返回一个object（这儿要注意的是，调用这个function，可能使用new，也可能不使用，也就是说factory function可能是constructor，也可能是一般的function），返回的这个object就是service了。


### $provide又是什么东西？
$provide是AUTO这个module里面提供的一个service，AUTO这个module会在每个APP中自动的被调用。

$provide这个service就会register到$injector中了。它里面包含了几个简化我们将自定义的service注册到$injector的method（注意到service就是一个object，这些method就是其property）。

### $provide里面的method有哪些？
#### `provider(name, provider)`
这个向`$injector`注册一个新的provider function，当我们要使用相应的service的时候，$injector负责生成它。

其中的`name`是一个字符串，注意这个provider会被angular加上Provider，也就是你给的是`aa`，那么会变成`aaProvider`。而provider参数是一个构造函数，要注意其返回的对象里面必须有$get这个property，然后这个property必须是一个函数，注意这个函数是可以有参数的，而且一般这些参数都是其他的service，然后这个函数必须返回一个object，这个object就是service。

#### `value(name, value)`
对于上面的provider函数，如果factory function的$get function（$get对于的function）是没有参数的（如果有参数的话，一般就是其他的service）,而且provider function里面除了$get 以外就没有其他的property了（其他的property一般就是用来对service进行配置的），**那么factory function返回的就是一个不依赖于其他的service，而且不会被配置的简单的service**,称之为value service（意思就是说，这个service就是一个简单的value）。
此时我们就可以用value函数来注册这个service。name参数为service的名字，value就是这个service的值，可以是number,string,function,array,object（也就是js中的所有类型的对象都可以）。

#### `factory(name, $getFn)`
当例化provider function返回对象里面就只有一个$get property的时候，就可以使用factory了，**也就是我们要的service是不用配置的但是可能依赖于其他的service。**

name参数是service的名字，$getFn参数就是$get 对应的function，也就是factory function，里面会返回一个对象(注意到js中什么东西都是对象，也就是里面什么都可以返回)，返回的东西就是我们的service了。


#### `service(name, constructor)`
和factory类似，只是在factory中，调用$getFn会返回一个service。

在service中，constructor参数是一个constructor function，**也就是要使用new来例化得到一个service。**



