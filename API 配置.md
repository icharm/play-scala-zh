## API 配置

Play 使用 [Typesafe config library](https://github.com/lightbend/config)，但是Play提供了优雅的包装类` Configuration`，支持更多Scala高级特性。 如果你对Typesafe配置不是很熟悉，可以先阅读这篇文章[configuration file syntax and features](https://www.playframework.com/documentation/2.7.x/ConfigFile)。



####访问配置(Accessing the configuration)

通常，你可以通过依赖注入([Dependency Injection](https://www.playframework.com/documentation/2.7.x/ScalaDependencyInjection))获得`Configuration`对象，或者简单的在组件间传递`Configuration`实例。

```scala
class MyController @Inject()(config: Configuration, c: ControllerComponents) extends AbstractController(c) {
  def getFoo = Action {
    Ok(config.get[String]("foo"))
  }
}
```

`get`方法是最常用的，用来获取配置文件中单个配置值。

```scala
// foo = bar
config.get[String]("foo")

// bar = 8
config.get[Int]("bar")

// baz = true
config.get[Boolean]("baz")

// listOfFoos = ["bar", "baz"]
config.get[Seq[String]]("listOfFoos")
```

`get`方法有一个隐式参数`ConfigLoader`，但是对于最常见的类型`String`、 `Int`，甚至于`Seq[String]`， 已经提供了定义好的[对应的加载器](https://www.playframework.com/documentation/2.7.x/api/scala/play/api/ConfigLoader$.html).

`Configuration`还支持验证一组有效值。

```scala
config.getAndValidate[String]("foo", Set("bar", "baz"))
```



####配置加载器(ConfigLoader)

通过自定义[`ConfigLoader`](https://www.playframework.com/documentation/2.7.x/api/scala/play/api/ConfigLoader.html)，可以很轻松地将配置转换为自定义类型。

这在Play内部广泛使用，可以为配置使用提供更多的类型安全特性。例如：

```scala
case class AppConfig(title: String, baseUri: URI)
object AppConfig {

  implicit val configLoader: ConfigLoader[AppConfig] = new ConfigLoader[AppConfig] {
    def load(rootConfig: Config, path: String): AppConfig = {
      val config = rootConfig.getConfig(path)
      AppConfig(
        title = config.getString("title"),
        baseUri = new URI(config.getString("baseUri"))
      )
    }
  }
}
```

然后可以像上面那样使用`config.get`：

```scala
// app.config = {
//   title = "My App
//   baseUri = "https://example.com/"
// }
config.get[AppConfig]("app.config")
```



####可选的配置键(Optional configuration keys)

Play的`Cofiguration`支持获取可选的配置键，通过使用`getOptional[A]`方法。和`get[A]`类似，但是如果键名不存在会返回`None`。我们建议您不要使用此方法，而是在配置文件中将可选键设置为`null`并使用`get [Option [A]]`。但是，为了方便您需要与以非标准方式使用配置信息，我们提供此方法。



下一章：HTTP编程