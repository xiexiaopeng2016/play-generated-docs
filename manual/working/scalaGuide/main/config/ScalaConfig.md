<!--- Copyright (C) 2009-2019 Lightbend Inc. <https://www.lightbend.com> -->
# Scala配置API

Play使用[Typesafe配置库](https://github.com/typesafehub/config)，但Play还提供了一个不错的Scala包装器，名为[`Configuration`](api/scala/play/api/Configuration.html)，该包装器具有更高级的Scala功能。如果您不熟悉Typesafe配置，则可能还需要阅读[[配置文件语法和功能|ConfigFile]]文档。

## 访问配置

通常，您将通过[[依赖注入|ScalaDependencyInjection]]获得`Configuration`对象，或者简单地将`Configuration`的实例传递给组件：

@[inject-config](code/ScalaConfig.scala)

该`get`方法是最常用的方法。用于获取配置文件的一个路径中的单个值。

@[config-get](code/ScalaConfig.scala)

它接受一个隐式`ConfigLoader`，但对于大多数常见类型，如`String`, `Int`,甚至`Seq[String]`，这是您期望的[已定义的加载器](api/scala/play/api/ConfigLoader$.html)。

`Configuration`还支持针对一组有效值进行验证：

@[config-validate](code/ScalaConfig.scala)

### ConfigLoader

通过定义自己的[`ConfigLoader`](api/scala/play/api/ConfigLoader.html)，您可以轻松地将配置转换为自定义类型。这在Play内部广泛使用，是带来更多安全类型用于配置的好方法。例如：

@[config-loader-example](code/ScalaConfig.scala)

然后，您可以像上面那样使用`config.get`：

@[config-loader-get](code/ScalaConfig.scala)

### 可选配置键

Play的`Configuration`支持使用`getOptional[A]`法获取可选的配置键。它的工作方式与`get[A]`类似，但是如果键不存在，它将返回`None`。
我们建议在配置文件中将可选键设置为`null`，并使用`get[Option[A]]`，而不是使用此方法。但是，如果您需要与以非标准方式使用配置的库进行接口连接，我们将为您提供方便的此方法。
