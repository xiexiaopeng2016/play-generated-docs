<!--- Copyright (C) 2009-2019 Lightbend Inc. <https://www.lightbend.com> -->
# HTTP 路由

## 内建的HTTP路由器

路由器是负责将每个传入的HTTP请求转换为动作的组件。

MVC框架将HTTP请求视为一个事件。此事件包含两个主要信息：

- 请求路径(例如`/clients/1542`, `/photos/list`)，包括查询字符串
- HTTP方法(例如`GET`, `POST`, …)。

路由在已编译的`conf/routes`文件中定义。这意味着您将直接在浏览器中看到路由错误：

[[images/routesError.png]]

## 依赖注入

Play的默认路由生成器会创建一个路由器类，该类在带@Inject注释的构造函数中接受控制器实例。这意味着该类适合与依赖项注入一起使用，也可以使用构造函数手动实例化。

在Play 2.7.0之前，Play支持一个静态路由生成器，该生成器允许将控制器定义为`object`而不是`class`。由于Play不再依赖静态状态，因此不再支持该功能。如果您希望使用自己的静态状态，则仍可以在控制器中使用`class`。

## 路由文件语法

`conf/routes`是路由器使用的配置文件。该文件列出了应用程序所需的所有路由。每个路由都包含一个HTTP方法和URI模式，它们都与对`Action`生成器的调用相关联。

让我们看看路由定义是什么样的：

@[clients-show](code/scalaguide.http.routing.routes)

每个路由均以HTTP方法开头，随后是URI模式。最后一个元素是调用定义。

您也可以使用`#`字符将注释添加到路由文件。

@[clients-show-comment](code/scalaguide.http.routing.routes)

您可以告诉路由文件使用一个指定前缀后面的路由器，即"->"紧随的前缀:

```
->      /api                        api.MyRouter
```

此功能特别有用， 当与[[字符串内插路由DSL|ScalaSirdRouter]](也称为SIRD路由)结合使用时，或者与使用多个路由文件进行路由的[[子项目|sbtSubProjects]]一起使用时。

也可以通过在路径前面加上以开头`+`的行来应用修饰符。这可以更改某些Play组件的行为。这样的修饰符之一就是"nocsrf"修饰符，它绕过了[[CSRF过滤器|ScalaCsrf]]:

@[nocsrf](code/scalaguide.http.routing.routes)

## HTTP方法

HTTP方法可以是任何HTTP支持的有效方法(`GET`, `PATCH`, `POST`, `PUT`, `DELETE`, `HEAD`)。

## URI模式

URI模式定义了路由的请求路径。请求路径的一部分可以是动态的。

### 静态路径

例如，要完全匹配传入的`GET /clients/all`请求，可以定义以下路由：

@[static-path](code/scalaguide.http.routing.routes)

### 动态部分

如果要定义通过ID检索顾客的路由，则需要添加动态部份：

@[clients-show](code/scalaguide.http.routing.routes)

> **注意:** 一个URI模式可能包含多个动态部分。

动态部分的默认匹配策略由正则表达式`[^/]+`定义，这意味着定义为`:id`的任何动态部分都将完全匹配一个URI路径片段。与其他模式类型不同，路径片段在传递给控制器​​之前会先在路径中自动进行URI解码，然后在反向路径中进行编码。

### 跨多个`/`的动态部分

如果希望动态部分捕获多个以正斜杠分隔的URI路径片段，则可以使用`*id`语法(也称为通配符模式)来定义动态部分，该语法使用`.*`正则表达式：

@[spanning-path](code/scalaguide.http.routing.routes)

在这里，一个类似`GET /files/images/logo.png`的请求，`name`动态部分将捕获`images/logo.png`值。

请注意，跨越几个动态部分/不会由路由器解码或由反向路由器编码。您有责任像对任何用户输入一样，验证原始URI段。反向路由器仅执行字符串连接，因此您需要确保生成的路径有效，并且不包含例如多个前导斜线或非ASCII字符。

Note that *dynamic parts spanning several `/` are not decoded by the router or encoded by the reverse router*. It is your responsibility to validate the raw URI segment as you would for any user input. The reverse router simply does a string concatenation, so you will need to make sure the resulting path is valid, and does not, for example, contain multiple leading slashes or non-ASCII characters.

### 具有自定义正则表达式的动态部分

您还可以使用以下`$id<regex>`语法为动态部分定义自己的正则表达式：

@[regex-path](code/scalaguide.http.routing.routes)

就像通配符路由一样，该参数*不会由路由器解码或由反向路由器编码*。您负责验证输入，以确保在这种情况下有意义。

## 调用动作生成器方法

路由定义的最后一部分是调用。这部分必须定义一个有效调用，方法返回`play.api.mvc.Action`值，该方法通常是控制器动作方法。

如果该方法未定义任何参数，则只需提供完全限定的方法名称：

@[home-page](code/scalaguide.http.routing.routes)

如果动作方法定义了一些参数，所有这些参数值将从请求URI中搜索，从URI路径本身或查询字符串中提取。

@[page](code/scalaguide.http.routing.routes)

或者:

@[page](code/scalaguide.http.routing.query.routes)

与其对应的，这是定义在`controllers.Application`控制器中的`show`方法：

@[show-page-action](code/ScalaRouting.scala)

### 参数类型

对于`String`类型的参数，指定参数类型是可选的。如果要让Play将传入的参数转换为特定的Scala类型，则可以显式指定参数类型:

@[clients-show](code/scalaguide.http.routing.routes)

并在`controllers.Clients`控制器中相应的`show`方法定义上执行相同的操作：

@[show-client-action](code/ScalaRouting.scala)

Play支持以下参数类型:

- String
- Int
- Long
- Double
- Float
- Boolean
- UUID
- 其他支持类型的AnyVal包装器

如果您有其他类型并想要实现它，可以看一看[[Request Binders|ScalaRequestBinders]]

### 具有固定值的参数

有时，您需要为参数使用固定值：

@[page](code/scalaguide.http.routing.fixed.routes)

### 具有默认值的参数

您还可以提供一个默认值，如果传入请求中未找到任何值，则将使用该默认值：

@[clients](code/scalaguide.http.routing.defaultvalue.routes)

### 可选参数

您还可以指定一个不需要在所有请求中都出现的可选参数：

@[optional](code/scalaguide.http.routing.routes)

## 路由优先级

多条路由可以匹配相同的请求。如果存在冲突，则使用第一条路由(按声明顺序)。

## 反向路由

路由器还可以用于从Scala调用中生成URL。这样就可以将所有URI模式集中在一个配置文件中，因此您可以在重构应用程序时更加放心。

对于路由文件中使用的每个控制器，路由器将在`routes`程序包中生成一个'反向控制器'，具有相同的动作方法，相同的签名，但返回一个`play.api.mvc.Call`而不是`play.api.mvc.Action`。

在`play.api.mvc.Call`定义了一个HTTP调用，并同时提供了HTTP方法和URI。

例如，如果您创建一个这样的控制器：

@[reverse-controller](code/ScalaRouting.scala)

如果将其映射到`conf/routes`文件中：

@[route](code/scalaguide.http.routing.reverse.routes)

然后，您可以将URL反向转换为动作方法`hello`， 通过使用`controllers.routes.Application`反向控制器：

@[reverse-router](code/ScalaRouting.scala)

注意：每个控制器程序包都有一个`routes`子程序包。因此，`controllers.Application.hello`动作可以通过`controllers.routes.Application.hello`来反转动作(只要在路由文件中没有与生成的路径匹配的其他路由)。

反转动作方法的工作原理非常简单：它将您的参数替换为路由模式。在路径片段(`:foo`)的情况下，值在替换之前已编码。对于正则表达式和通配符模式，该字符串以原始形式替换，因为该值可能跨越多个段。确保根据需要转义这些组件, 将这些组件传递到反向路径时，并避免传递未经验证的用户输入。

## 相对路由

在某些情况下，返回相对路径而不是绝对路径可能会很有用。`play.mvc.Call`返回的路由始终是绝对的(它们以开头`/`)，这可能会导致问题，当通过HTTP代理，负载平衡器和API网关重写对Web应用程序的请求时。使用相对路线会有用的一些示例包括：

* 在Web网关后面托管一个应用程序，该应用程序为所有路由添加`conf/routes`文件中未配置的前缀，并将您的应用程序以不期望的路由为根。
* 动态渲染样式表时，您需要资产链接是相对的，因为它们最终可能由CDN从不同的URL提供服务。

为了能够生成相对路由，您需要知道相对于目标路由的内容(起始路由)。可以从当前`RequestHeader`中检索开始路由。因此，要生成相对路由，需要将当前`RequestHeader`或起始路由作为`String`参数传递。

例如，给定的控制器端点如下：

@[relative-controller](code/scalaguide/http/routing/relative/controllers/Relative.scala)

> **注意:** 通过声明一个`implicit request`，将当前请求隐式传递给视图模板。

如果将其映射到`conf/routes`文件中：

@[relative-hello](code/scalaguide.http.routing.relative.routes)

然后，您可以像前面一样使用反向路由器定义相对路由，并包括一个附加的对`relative`的调用:

@[relative-hello-view](code/scalaguide/http/routing/relative/views/hello.scala.html)

> **注意:** 从控制器传递`Request`被转换为`RequestHeader`, 并在视图参数中被标记为`implicit`。然后将其隐式传递给对`relative`的调用

请求`/foo/bar/hello`时生成的HTML将如下所示：

@[relative-hello-html](code/scalaguide/http/routing/relative/views/hello.html)

## 默认控制器

Play包括一个[`Default`控制器](api/scala/controllers/Default.html)，它提供了一些有用的动作。这些可以直接从路由文件中调用：

@[defaultcontroller](code/scalaguide.http.routing.defaultcontroller.routes)

在此示例中，`GET /about`重定向到外部网站，但是也可以重定向到另一个动作(例如在上面的示例中的`/posts`)。

## 自定义路由

Play提供了一个用于定义嵌入式路由器的DSL，称为*字符串插值路由DSL*，简称SIRD。这种DSL有很多用途，包括嵌入轻量级的Play服务器，为常规的Play应用程序提供自定义或更高级的路由功能，以及用于测试的模拟REST服务。

请参阅[[字符串插值路由DSL|ScalaSirdRouter]]
