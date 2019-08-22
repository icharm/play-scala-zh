## Actions, Controllers和Results

### 什么是Action?

Play应用收的大多数请求会被`Action`处理。

`play.api.mvc.Action`基于`(play.api.mvc.Request => play.api.mvc.Result)`函数处理请求，然后生成结果发送给客户端。

```scala
def echo = Action { request =>
  Ok("Got request [" + request + "]")
}
```

一个`Action`返回一个`play.api.mvc.Result`对象，表示要发送给web客户端的HTTP响应，在上例中`Ok`会构造一个响应体为**text/plain**的**200 OK**响应。

### 编写Action

在任何扩展`BaseController`的控制器中，`Action`是默认的构建器。此构建器包含一些用于创建action对象的帮助程序。

一个最简单的`Action`：

```scala
Action {
  Ok("Hello world")
}
```

这是创建Action的最简单的方式，但是我们没有使用到传入的请求。此Action不是处理HTTP请求的通常样子。

因此，还有其他的Action构建器，它将函数`Request => Result`作为参数：

```scala
Action { request =>
  Ok("Got request [" + request + "]")
}
```

将`request`参数标记为`implicit`通常很有用，因为它可以被其他需要的API隐式使用：

```scala
def action = Action { implicit request =>
  anotherMethod("Some para value")
  Ok("Got request [" + request + "]")
}

def anotherMethod(p: String)(implicit request: Request[_]) = {
  // 可以在这里做任何基于request的事情
}
```

创建Action方法的最后一种方式是指定一个额外的`BodyParser`参数：

```scala
Action(parse.json) { implicit request =>
  Ok("Got request [" + request + "]")
}
```

Body解析器将在本手册的后面部分介绍。现在，只需要知道创建Action方法的其他方式使用默认的**Any content body parser**。

### 控制器其实是action生成器

Play中的控制器只不过是一个生成Action方法的对象。控制器通常被定义为可以依赖注入( [Dependency Injection](https://www.playframework.com/documentation/2.7.x/ScalaDependencyInjection))的类。

<!--注意：在未来的Play版本中不支持将控制器定义为`object`。建议使用`class`。-->

定义action生成器最简单的用例是返回一个不带参数的Action对象：

```scala
package controllers

  import javax.inject.Inject

  import play.api.mvc._

  class Application @Inject()(cc: ControllerComponents) extends AbstractController(cc) {

    def index = Action {
      Ok("It works!")
    }

  }
```

当然，action生成器方法可以有参数传入，`Action`闭包可以捕获这些参数:

```scala
def hello(name: String) = Action {
  Ok("Hello " + name)
}
```

### 简单的Results

现在我们只关注简单的Result：一个HTTP响应result，携带着响应码、头信息和响应体发送给Web客户端。

这些results由`play.api.mvc.Result`定义：

```scala
import play.api.http.HttpEntity

def index = Action {
  Result(
    header = ResponseHeader(200, Map.empty),
    body = HttpEntity.Strict(ByteString("Hello world!"), Some("text/plain"))
  )
}
```

Play提供了一些助手用来创建常见的响应results, 就像上面的`Ok` result一样：

```scala
def index = Action {
  Ok("Hello world!")
}
```

这产生与以前完全相同的result。以下是创建各种result的几个示例：

```scala
val ok           = Ok("Hello world!")
val notFound     = NotFound
val pageNotFound = NotFound(<h1>Page not found</h1>)
val badRequest   = BadRequest(views.html.form(formWithErrors))
val oops         = InternalServerError("Oops")
val anyStatus    = Status(488)("Strange response type")
```

所有的助手代码都可以在`play.api.mvc.Results` trait和companion object 里找到。

### 重定向也是简单results

将浏览器定向到一个新的URL是另外一种简单Result形式，尽管这些Result不携带响应体。

Play提供一些助手方法快速穿件重定向Results:

```scala
def index = Action {
  Redirect("/user/home")
}
```

默认使用 `303 SEE_OTHER`作为响应类型，如果你需要的话可以设置特定的状态码：

```scala
def index = Action {
  Redirect("/user/home", MOVED_PERMANENTLY)
}
```

### `TODO`虚拟页面

你可以创建一个定义为`TODO`的空`Action`实现，结果会返回一个标准的'Not implemented yet'的页面:

```
def index(name: String) = TODO
```



原文 [Actions, Controllers and Results](https://www.playframework.com/documentation/2.7.x/ScalaActions)





