## Results 使用

### 修改默认的`Content-Type`

Result响应体的内容类型是从Scala值自动推断出来的，例如：

```scala
val textResult = Ok("Hello World!")	
```

上面会自动设置`Content-Type`为`text/plain`，因此：

```scala
val xmlResult = Ok(<message>Hello World!</message>)
```

会设置`Content-Type`为`Application/xml`。

Tip: 这项工作由`play.api.http.ContentTypeOf`来完成。

这是非常方便的，但是有时你会想改变它，只需调用`as(newContentType)`方法就可以创建一个想要的的Result:

```scala
val htmlResult = Ok(<h1>Hello World!</h1>).as("text/html")
```

如下，可能会更好:

```scala
val htmlResult2 = Ok(<h1>Hello World!</h1>).as(HTML)
```

Note: 使用`HTML`代替`text/html`的好处是，`HTML`会自动填充`text/html; charset=utf-8`的Content-Type, 下文会继续提到这点。

### 修改HTTP头信息

你可以添加或者更新result的任何HTTP头信息:

```scala
val result = Ok("Hello World!").withHeaders(CACHE_CONTROL -> "max-age=3600", ETAG -> "xx")
```

Note: 如果更新一个已存在的头信息，会覆盖原来的头信息。

### 设置和丢弃Cookies

Cookies是一组特殊的HTTP头，Play提供专门的方法来更方便的修改它们。

添加Cookie到HTTP响应:

```scala
val result = Ok("Hello world")
  .withCookies(Cookie("theme", "blue"))
  .bakeCookies()
```

丢弃一个储存在Web浏览器上的Cookie：

```scala
val result2 = result.discardingCookies(DiscardingCookie("theme"))
```

同样可以在同一个result上同时设置和丢弃cookie：

```scala
val result3 = result.withCookies(Cookie("theme", "blue")).discardingCookies(DiscardingCookie("skin"))
```

### 设置HTTP响应编码

对于基于文本的HTTP响应，使用正确的编码处理是非常重要的。Play默认使用`utf-8`来处理。([为什么使用utf-8](https://www.w3.org/International/questions/qa-choosing-encodings#useunicode))

编码用于将响应文本转换为通过网络发送的字节，并使用正确的带有` ;charset=xxx`的`Content-Type`头，以便于客户端正确的解析内容。

编码会经由`play.api.mvc.Codec`类自动处理，只需在当前作用域中导入`play.api.mvc.Codec`的隐式实例，即可让所有操作使用的该字符集：

```scala
class Application @Inject()(cc: ControllerComponents) extends AbstractController(cc) {

  implicit val myCustomCharset = Codec.javaSupported("iso-8859-1")

  def index = Action {
    Ok(<h1>Hello World!</h1>).as(HTML)
  }

}
```

如上，在当前作用域下有一个隐式的字符集，这将会使得`Ok(...)`方法将XML内容转成`ISO-8859-1`的Content-Type头信息。

现在，如果你还在好奇`HTML`方法是怎么工作的，下面是它的定义：

```scala
def HTML(implicit codec: Codec) = {
  "text/html; charset=" + codec.charset
}
```

同样，在你的API里，可以类似的处理字符集问题。