http package
===

golang作为一门21世纪的语言，那么对于网络的支持肯定是很好的。
在http协议上，golang提供的标准包中就有`net/http`这个包。
这个包中提供了强大的HTTP协议的支持。
我们完全可以在不使用第三方的库的情况下，直接基于这个包做出一个web应用。

### func CanonicalHeaderKey

	func CanonicalHeaderKey(s string) string
对于http request和http response，我们都希望其header的key是Canonical的，也就是规范的，也就是单词都是使用第一个字母大写，单词的分割使用`-`，这个函数就可以将一个表示header的key的string `s`变成这个样子。

### func DetectContentType

	func DetectContentType(data []byte) string
对于Http message中的body部分，我们可以使用特定的算法知道其content type，这个算法只需要用到其中的最多512个字节的数据。
注意其输入参数是`[]byte`，也就是一个比特流，如果成功的判断出来了，那么返回判断出来的mime，比如说`text/html`，如果没有能够判断出来，那么会返回`appplication/octet-stream`，因为body中的肯定是一个比特流的。

### func Error

	func Error(w ResponseWriter, error string, code int)
对于那些需要返回一个错误的request，那么可以直接使用这个函数，注意其需要的参数是`ResponseWriter`.

### func Handle

	func Handle(pattern string, handler Handler)
向`DefaultServeMux`中添加一项。`DefaultServeMux`是一个MUX，也就是一个分发器，或者说是router，`pattern`表示要匹配到的路径，`handler`为添加的处理函数。

### func HandleFunc

	func HandleFunc(pattern string, handler func(ResponseWriter, *Request))
和上面的功能完全相同，但是加入的是一个函数，而上面加入的是`Handler`


### func ListenAndServe

	func ListenAndServe(addr string, handler Handler) error
>这个函数是这个包中的核心函数

监听TCP过来的到`addr`的连接，然后使用`handler`来处理这个链接，`handler`一般都会给`nil`，这样就会使用`DefaultServeMux`来进行请求的分发。
如果不使用`DefaultServeMux`，那个访问任何路径都会得到完全相同的结果，或者必须要在`handler`内部实现一个MUX。

### func ParseHTTPVersion

	func ParseHTTPVersion(vers string) (major, minor int, ok bool)
将输入的`vers`表示的HTTP版本解析开。	
对于`HTTP/1.0`，那么其会返回`(1, 0, true)`。

### func ParseTime

	func ParseTime(text string) (t time.Time, err error)
对于` HTTP/1.1`中定义的三种时间格式`TimeFormat, time.RFC850, and time.ANSIC`，这个函数可以将输入的`text`进行转换，转换出来的是一个golang中使用的时间表示。