<!--- Copyright (C) 2009-2019 Lightbend Inc. <https://www.lightbend.com> -->
# 动作，控制器和结果

## 什么是动作？

Play应用收到的大多数请求都由`Action`来处理。

一个`play.api.mvc.Action`基本上是一个`(play.api.mvc.Request => play.api.mvc.Result)`函数，它处理一个请求并生成要发送到客户端的结果。

@[echo-action](code/ScalaActions.scala)

一个动作返回一个`play.api.mvc.Result`值，该值表示要发送到Web客户端的HTTP响应。在此示例中，`Ok`构造一个**200 OK**响应，其包含一个**text/plain**响应主体。



## 构建动作

在任何扩展`BaseController`的控制器中，该`Action`值是默认的动作构建器。该动作构建器包含多个用于创建`Action`的帮助器。

第一个最简单的构建器只是把一个返回`Result`的表达式块作为参数：

@[zero-arg-action](code/ScalaActions.scala)

这是创建Action的最简单方法，但是我们没有获得传入请求的引用。在动作调用中访问HTTP请求通常很有用。

因此，还有另一个Action构建器以一个`Request => Result`函数作为参数：

@[request-action](code/ScalaActions.scala)

将`request`参数标记为`implicit`通常很有用，以便其他需要它的API可以隐式的使用它：

@[implicit-request-action](code/ScalaActions.scala)

如果您已将代码分解为方法，则可以传递Action中的隐式请求：

@[implicit-request-action-with-more-methods](code/ScalaActions.scala)

创建Action值的最后一种方法是指定一个附加的`BodyParser`参数：

@[json-parser-action](code/ScalaActions.scala)

正文解析器将在本手册的后面部分介绍。现在，您只需要知道创建Action值的其他方法使用默认的**Any content body parser**解析器即可。

## 控制器是动作生成器

Play中的控制器不过是一个生成`Action`值的对象。控制器通常被定义为利用[[依赖注入|ScalaDependencyInjection]]的类。

注意：请记住，将来的Play版本将不支持将控制器定义为对象。建议使用类。

> **注意:** 请记住，将来的Play版本将不支持将控制器定义为对象。建议使用类。

定义动作生成器的最简单用例是一个没有参数的方法，该方法返回`Action`值：

@[full-controller](code/ScalaActions.scala)

当然，动作生成器方法可以具有参数，并且这些参数可以由`Action`闭包捕获：

@[parameter-action](code/ScalaActions.scala)

## 简单的结果

现在，我们只对简单的结果感兴趣：一个带有状态码的HTTP结果，一组HTTP头部和要发送到Web客户端的正文。

这些结果定义为`play.api.mvc.Result`：

@[simple-result-action](code/ScalaActions.scala)

当然，有几个助手可以用来创建常见结果，例如上面示例中的`Ok`结果：

@[ok-result-action](code/ScalaActions.scala)

这将产生与以前完全相同的结果。

以下是创建各种结果的几个示例：

@[other-results](code/ScalaActions.scala)

所有这些帮助程序都可以在`play.api.mvc.Results`特质和伴生对象中找到。

## 重定向也是简单的结果

将浏览器重定向到新的URL只是另一种简单的结果。但是，这种结果类型不包含响应主体。

有几种可用于创建重定向结果的助手：

@[redirect-action](code/ScalaActions.scala)

默认是使用`303 SEE_OTHER`响应类型，但是如果需要，您还可以设置更具体的状态代码：

@[moved-permanently-action](code/ScalaActions.scala)

## `TODO` 虚拟页面

您可以使用定义为`TODO`的实现空`Action`：结果是标准的“尚未实现”结果页：

@[todo-action](code/ScalaActions.scala)
