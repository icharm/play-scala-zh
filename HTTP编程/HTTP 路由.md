## HTTP 路由

### 内置HTTP路由器

路由器是负责将每个传入的HTTP请求转换为Action的组件。

HTTP请求被MVC框架视为事件。此事件包含两个主要信息：

-  请求路径(e.g. `/client/1542`, `/photos/list`)，包括查询参数。
- HTTP方法(e.g. `GET`, `POST`, …)。

路由在conf / routes文件中定义，该文件会被编译。这意味着可以直接在浏览器中看到路径错误：

![img](https://www.playframework.com/documentation/2.7.x/resources/manual/working/scalaGuide/main/http/images/routesError.png)

### 依赖注入

Play的默认路由生成器创建一个路由器类，该类接受`@Inject-annotated`构造函数的控制器实例。这意味着该类既适用于依赖注入，也可以使用构造函数手动实例化。

在Play 2.7.0之前，Play支持一个静态路由生成器，它允许将控制器定义为对象而不是类。这不再受支持，因为Play不再支持静态依赖。如果您希望使用自己的静态，您仍然可以在类控制器中执行此操作。

### 路由文件语法

`conf/routes`是路由器的配置文件，这个文件中列出应用程序所需的全部路由信息。每个路由都包含HTTP方法和URI模式，两者都与对`Action`生成器的调用相关联。

让我们看看路由定义是什么样的：

```scala
GET   /clients/:id          controllers.Clients.show(id: Long)
```

每个路由都以HTTP方法开头，后跟URI模式。最后一个元素是调用定义。

还可以使用＃字符为路径文件添加注释。

```scala
# Display a client.
GET   /clients/:id          controllers.Clients.show(id: Long)
```

您可以通过使用“ - >”后跟给定前缀来告知路由文件使用特定前缀下的其他路由器：

```scala
->      /api                        api.MyRouter
```

