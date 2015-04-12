angularjs
====

angularjs是google推出不久的一个前端的框架。

现在web开发的趋势是前后端分离。前端采用某些js框架，后端采用某些语言提供restful API，两者以json格式进行数据交互。

在前端的框架中，使用angularjs的越来越多。

## Module
Module是angularjs中的一个个的模块单元，因为在js中，只有一个全局的global，我们必须将要实现的功能装到一个个的模块单元中，这样才能分离实现。

每个Module中就是装了其他的一些元素，这些元素包括,`controller,service,filter,dierctive`等等的。

一个Module中包含两种block，一个是config，一个是run：

config block: provider进行注册和配置的过程。只有provider和constant可以被注入到config块中。这是为了阻止一个service在正确的配置之前就被实例化了

run block: 当injector创建完成了会运行，这个时候这个应用就启动了。只有constant和instance可以被注入到run中。这个是为了阻止在系统已经运行的情况下还对这个应用进行配置。

	angular.module('myModule', []).
		config(function(injectables) { // provider-injector
		  // This is an example of config block.
		  // You can have as many of these as you want.
		  // You can only inject Providers (not instances)
		  // into config blocks.
		}).
		run(function(injectables) { // instance-injector
		  // This is an example of a run block.
		  // You can have as many of these as you want.
		  // You can only inject instances (not Providers)
		  // into run blocks
		});

### config block
	angular.module('myModule', []).
	  value('a', 123).
	  factory('a', function() { return 123; }).
	  directive('directiveName', ...).
	  filter('filterName', ...);
	
	// is same as
	
	angular.module('myModule', []).
	  config(function($provide, $compileProvider, $filterProvider) {
	    $provide.value('a', 123);
	    $provide.factory('a', function() { return 123; });
	    $compileProvider.directive('directiveName', ...);
	    $filterProvider.register('filterName', ...);
	  });
上面给出的是一个典型的config block的代码。
angularjs为了方便，将常用的config函数从各个provider中暴露到`angular.module`中了。
所以上面的代码中上下是相同的。

### run block
run block是angular中和一般程序的main函数最相近的东西。run block中的代码用来启动应用。它会在所有的service都被配置并且injector被创建了之后才运行。run block中的代码不好进行unit test。

### 依赖
一个module可以依赖于其他的Module。表示被依赖的Module会先被载入。也就是被依赖的Module的config会被先执行，run block也是。每个module只会被载入一次，即使被依赖了多次。

### 异步载入
Module是用来管理$injector的配置的（也就是管理$injector可以将哪些东西给注入到需要的函数中），其对将一个js载入VM的过程完全没有影响。有其他的项目专门管理js载入过程的，比如说require.js，angular可以和这些project同时使用。在载入过程中，Module不会做任何的事情（当载入完成之后，config开始运行的时候才会做实际的事情），所以它们可以以任何的顺序被载入到VM中，那么像require.js这类库就可以进行并行的载入了。

### 创建和得到
要注意的是`angular.module('myModule', [])`总是会创建一个叫做`myModule`的module，如果这个module已经存在了，那么其会被覆盖。如果要得到这个module（也就是进行get操作），那么使用`angular.module('myModule')`

## 启动应用
	<!doctype html>
	<html  ng-app="xxxx">
	  <body>
	    ...
	    <script src="angular.js"></script>
	    <script src="myscript.js"></script>
	  </body>
	</html>
一个典型的angular应用应该如上，将angular.js放在`body`的最后面可以让月面上面的元素尽可能早的显示出来，因为不用等待js代码的载入。
将`ng-app`放到`html`元素上可以使得整个应用都在angular的作用下，典型的应用都是这样的。
`myscript.js`在angularjs之后，这样我们写的Module就可以被载入了。

### 自动启动应用
当浏览器将整个DOM树载入完成之后，其会发出`DOMContentLoaded`事件，angular监听到这个事件之后就会自动的启动。
然后angular会扫描DOM树，找到ng-app所在的位置，ng-app表明了一个应用的root，如果找了ng-app，那么其会做如下的事情：

* 载入ng-app表明的那个Module
* 为这个应用创建一个injector
* 以ng-app为根，编译相应的DOM树

![](concepts-startup.png)

上图给出了这个过程。可以看到，在浏览器载入完成DOM的时候，DOM是一个静态的。当angular编译完成了相应的DOM之后，这部分DOM就是动态的了，表示在angular中的操作会让DOM中的内容动态的变化。

**一般的，我们都应该使用自动启动应用的方式**

## data binding
数据同步是angular设计最出彩地方。

![](one.png)

上图是一个典型的从model->view的单向数据绑定的示意图，在大部分的mvc中，都是这样做的。比如在后端使用的spring mvc中，就是将model中所有的数据交给view，让view去进行render的操作。 在后端中，使用这种方式并没有什么问题，因为后端处理的就是一个request/response的模式，其本身就是一个单向的过程。 controller接受request，产生modele，传给view，然后从response返回。

但是在前端，也就是浏览器中，用户即把浏览器作为输入，比如用户会点击，输入之类的，同时也会把浏览器作为输出。如果还是使用这种单向绑定的方式，我们需要手写很多的js代码来同步model和view中的数据。  比如我们使用JQuery库的时候，这个问题就会十分的明显。



![](two.png)

在angular中，使用了上图所示的双向绑定的模式，在dom树被compile之后，其就变成了一个动态的dom树了。 任何在view中的改变会被映射到model中，同样的，在model中的改变会导致dom树的更新，也就是在view中反应出来。

## scope
scope是angular中一个比较复杂的概念，因为其复杂性，对于不喜欢angular的人来说，批评最多的也是angular中的scope。

scope就是angular中的Model。它是angular expression的运行环境。scope自身会组成一个层次结构。

### scope的特性
scope提供了API，`$watch`来观察model的变化，也即是model中的值是不是改变了。

### Scope as Data-Model
scope是controller和view中间的胶水。在template(也就是html，angular直接是用html作为自己的template)的linking阶段，相应的dierctive会`$watch` scope。当scope有变化的时候，`$watch`让directive知道其变化了，这样这个directive就可以更新自己的DOM来自动响应这个变化了。

controller和dierctive都指向了同一个scope，这就让controller和directive以及dom都分开了。

	html代码：
	<div ng-controller="MyController">
	  Your name:
	    <input type="text" ng-model="username">
	    <button ng-click='sayHello()'>greet</button>
	  <hr>
	  {{greeting}}
	</div>
	
	js代码：
	angular.module('scopeExample', [])
	.controller('MyController', ['$scope', function($scope) {
	  $scope.username = 'World';
	
	  $scope.sayHello = function() {
	    $scope.greeting = 'Hello ' + $scope.username + '!';
	  };
	}]);
	
如上的代码中，`ng-controller`会创建一个scope,下面的`ng-model,ng-click,{{greeting}}`这三个都是directive，它们都指向了这个scope。
然后在相应的js代码中，`$scope`也表示的是这个scope。


### Scope Hierarchies
每一个angular应用都只有一个rootScope，但是可以有许多的child scope。

一个应用会有多个的scope，因为一些dierctive会创建新的child scope。当新的scope创建的时候，它们会被添加为parent scope的child scope。

当angular执行`{{name}}`的时候，其首先会看这个表达式对应的scope的`name` property。如果这个property没有被发现，那么其会搜索相应的parent scope直到root scope。

	<div class="show-scope-demo">
	  <div ng-controller="GreetController">
	    Hello {{name}}!
	  </div>
	  <div ng-controller="ListController">
	    <ol>
	      <li ng-repeat="name in names">{{name}} from {{department}}</li>
	    </ol>
	  </div>
	</div>
	
	angular.module('scopeExample', [])
	.controller('GreetController', ['$scope', '$rootScope', function($scope, $rootScope) {
	  $scope.name = 'World';
	  $rootScope.department = 'Angular';
	}])
	.controller('ListController', ['$scope', function($scope) {
	  $scope.names = ['Igor', 'Misko', 'Vojta'];
	}]);
	
![](scope.png)

从上面可以看出,`ng-app,ng-controller,ng-repeat`都会产生一个scope。注意到那些产生scope的element angular都会给其生成一个ng-scope的类。
注意$rootScope是放在ng-app所在的element上面的。

注意$scope本身是作为一个data property放在相应的DOM元素的属性里面的。 这个是js的一个性质了，在js中可以将完全不相关的东西放到一个object中去。
	
## template
在angular中，template就是使用html写的，只是其中包含了一些angular规定的元素和属性。
这些是可能被使用的angular element和attribute:

* directive 是一个element或者是attribute作用在当前已经存在的DOM上的
* markup 就是使用`{{}}`包含的expression
* filter 就是写在`{{xxx|fiter}}`的。
* form control 验证用户的输入

	<html ng-app>
	 <!-- Body tag augmented with ngController directive  -->
	 <body ng-controller="MyController">
	   <input ng-model="foo" value="bar">
	   <!-- Button tag with ng-click directive, and
	          string expression 'buttonText'
	          wrapped in "{{ }}" markup -->
	   <button ng-click="changeFoo()">{{buttonText}}</button>
	   <script src="angular.js">
	 </body>
	</html>
	
上面的代码就给出了一个angular的template。 这个template会被编译生成一个动态的DOM树。

## $http
$http是用来进行rest api访问的时候需要使用的。

### 用法
访问rest api的时候，一般都是使用get和post方法。

基本的GET使用方法

	// Simple GET request example :
	$http.get('/someUrl').
	  success(function(data, status, headers, config) {
	    // this callback will be called asynchronously
	    // when the response is available
	  }).
	  error(function(data, status, headers, config) {
	    // called asynchronously if an error occurs
	    // or server returns response with an error status.
	  });

基本的POST使用方法
	
	// Simple POST request example (passing data) :
	$http.post('/someUrl', {msg:'hello word!'}).
	  success(function(data, status, headers, config) {
	    // this callback will be called asynchronously
	    // when the response is available
	  }).
	  error(function(data, status, headers, config) {
	    // called asynchronously if an error occurs
	    // or server returns response with an error status.
	  });
	
$http本身是一个service，其函数原型为

	$http(config);
	
`config`是一个object，里面包含了要发送的http request的所有的信息。

`config`这个object可以有如下的属性。

* method - {string} 表示HTTP method。比如GET,POST之类的。 这个是必须给出的。
* url - {string} 指定要访问的url。 这个必须给出
* params - {object.<string|object>} 一般都是给出一个object，生成的是`?key1=value1&key2=value2`。 每一个value都会被进行url编码。
* data -{object|string} 放在request body中的数据，如果是string，那么直接发送，如果是object，会变成JSON格式的数据。如果想要得到url编码的params类似的数据，需要进行配置。
* headers -{object} 这个object中的各个属性可以是string或者是function，如果是function，而且其返回的数据为null，那么这个header就不会被设置。 一般情况下，就使用string就可以了。

* transformRequest - `{function(data, headersGetter)|Array.<function(data, headersGetter)>}`
* transformResponse - `{function(data, headersGetter, status)|Array.<function(data, headersGetter, status)>}` 这两个函数是用来进行request的body和response的body进行转换的函数。 如果不给出来的话，就会使用默认提供的。

其返回值是一个HttpPromise。

对于上面的两个函数，其是为了方便给出的来的，也就是$http的特列。

返回的`HttpPromise`是一个Object，其中包含了一个标准的`then`方法和http中特定的`success`和`error`方法。`then`方法需要两个参数，一个success和一个error callback函数，它们都是用来处理得到response的。而`success`和`error`方法就更简单了，它们也都需要一个函数，它们分别用来处理返回的response是正确和错误的情况。 这个函数的参数是返回的Http response被分解之后，有如下的参数。

* data - {string|object} 这个是http response的body部分，根据body中的内容不同，会是一个string或者是一个Object
* status - {number} 这个是http response的status code,就是我们常见的200,404之类的
* headers - {function([headerName])} 是一个函数，是http response的header部分的getter函数，只要给这个函数输入想要的header name，那么相应的值就会被返回
* config - {object} 传送给$http函数使用的config object。 这个参数我们一般不会使用，因为这个本来是$http的输入参数
* statusText -{string} http response的status text。

>需要注意的是，这几个参数必须按照上面列出的顺序出现。比如我们不想用返回的data，只需要返回的status，那么我们也需要把data写在前面。


### Transforming Requests and Responses
$http发送的request和接收到的response都可以被转换，使用的函数是配置的时候使用的`transformRequest`和`transformResponse`。这个属性可以使用一个函数` (function(data, headersGetter, status)) `，也可以是这种函数的一个数组，如果是一个数组的话，就构成了一个转换链。

#### 默认的转换函数
`$httpProvider`和`$http`都提供了默认的对request和response进行转换的函数，如果一个request没有自己显示的给出，那么就会使用默认的了。

默认的request转换函数会做如下的事情：

* 如果给出的`data`属性是一个Object，那么其会被转换成一个JSON

默认的response转换函数：

* 如果看到了`XSRF`前缀，那么会删除它?
* 如果发现返回了一个JSON，那么会将这个json变成一个object。

>上面的request中的object变成JSON和response中的JSON变成object是我们比较需要的特性。

#### 覆盖默认的转换函数
如果觉得默认的转换函数不能满足我们的要求，那么可以使用



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


## Module
在ja中，所有的东西如果没有加上一个命名空间的话，那么就会全部被放在global中，所以angular提出了一个module的东西，使用module再配合$injector就可以把我们的东西整理的很好了。


## $window
$window一个service。 其本质是浏览器的window的一个ref。 在js中，window这个object本身是全局的，这就导致要对其进行测试的话会很有问题。 在angular代码中，我们总是使用$window这个service来表示，其本身是可以被注入的，这个对于测试十分友好。
