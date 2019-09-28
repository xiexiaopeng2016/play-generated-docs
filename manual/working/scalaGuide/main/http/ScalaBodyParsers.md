<!--- Copyright (C) 2009-2019 Lightbend Inc. <https://www.lightbend.com> -->
# 主体解析器

## 什么是主体解析器？

HTTP请求是头部，后跟主体。头部通常很小 - 它可以安全地缓冲在内存中，因此在Play中使用[`RequestHeader`](api/scala/play/api/mvc/RequestHeader.html)类对它进行建模。但是，主体可能非常长，因此不会在内存中进行缓冲，而是将其建模为流。但是，许多请求主体有效负载很小，可以在内存中建模，因此要将主体流映射到内存中的对象，Play提供了一种[`BodyParser`](api/scala/play/api/mvc/BodyParser.html)抽象。

由于Play是一个异步框架，因此传统`InputStream`方法不能用于读取请求主体 - 输入流正在阻塞，当您调用`read`时，调用它的线程必须等待数据可用。取而代之的，Play使用一个异步流库[Reactive Streams](http://www.reactive-streams.org/)。Akka Streams是[Reactive Streams](http://www.reactive-streams.org/)的实现，该SPI允许许多异步流API无缝地协同工作，因此传统的基于`InputStream`技术的不适合与Play一起使用，Akka Streams和Reactive Streams周围的整个异步库生态系统将为您提供与您需要的一切。

## 有关动作的更多信息

之前我们说过一个`Action`是一个`Request => Result`函数。这并非完全正确。让我们对`Action`特质进行更精确的了解：

@[action](code/ScalaBodyParsers.scala)

首先我们看到有一个通用类型`A`，然后一个动作必须定义一个`BodyParser[A]`。`Request[A]`被定义为：

@[request](code/ScalaBodyParsers.scala)

`A`类型是请求主体的类型。我们可以使用任何Scala类型作为请求体，`String`, `NodeSeq`, `Array[Byte]`, `JsonValue`, 或 `java.io.File`，只要我们有一个主体解析器能够处理它。

概括而言，`Action[A]`使用`BodyParser[A]`检索HTTP请求中的`A`类型的值，并构建一个`Request[A]`的对象传递给动作代码。

## 使用内置的主体解析器

大多数典型的网络应用都不需要使用自定义的主体解析器，它们可以简单的与Play的内置正文解析器一起工作。其中包括JSON，XML，表单的解析器，以及将纯文本主体作为String处理，将字节主体作为ByteString处理。

### 默认的主体解析器

如果您未明确选择主体解析器，则使用默认的主体解析器。将查看传入的`Content-Type`头部，并相应地解析主体。例如，一个`application/json`类型的`Content-Type`解析为`JsValue`，而`application/x-www-form-urlencoded`类型的`Content-Type`解析为`Map[String, Seq[String]]`。

默认的主体解析器生成[`AnyContent`](api/scala/play/api/mvc/AnyContent.html)类型的主体。所支持的各种类型`AnyContent`都可以通过as方法访问，比如`asJson`。该方法返回一个`Option`body类型：

@[access-json-body](code/ScalaBodyParsers.scala)

以下是默认主体解析器支持的类型的映射：

- **text/plain**: `String`, 可通过`asText`访问。
- **application/json**: [`JsValue`](https://static.javadoc.io/com.typesafe.play/play-json_2.12/2.6.9/play/api/libs/json/JsValue.html), 可通过`asJson`访问。
- **application/xml**, **text/xml** 或 **application/XXX+xml**: `scala.xml.NodeSeq`, 可通过`asXml`访问。
- **application/x-www-form-urlencoded**: `Map[String, Seq[String]]`, 可通过`asFormUrlEncoded`访问。
- **multipart/form-data**: [`MultipartFormData`](api/scala/play/api/mvc/MultipartFormData.html), 可通过`asMultipartFormData`访问。
- 其他任何内容类型: [`RawBuffer`](api/scala/play/api/mvc/RawBuffer.html), 可通过`asRaw`访问。

默认的主体解析器会在解析之前确定请求是否有主体。根据HTTP规范，存在`Content-Length`或`Transfer-Encoding`头部就表示存在主体，因此只有存在其中一个头部，或在`FakeRequest`时显式设置了非空主体，解析器才解析。

如果您想在所有情况下都尝试解析正文，则可以使用下文所述的anyContent正文解析器。

If you would like to try to parse a body in all cases, you can use the `anyContent` body parser, described [below](#Choosing-an-explicit-body-parser).

## 选择一个明确的主体解析器

如果要显式选择主体解析器，可以通过将主体解析器传递给`Action`[`apply`](api/scala/play/api/mvc/ActionBuilder.html#apply[A]\(bodyParser:play.api.mvc.BodyParser[A]\)\(block:R[A]=%3Eplay.api.mvc.Result\):play.api.mvc.Action[A])或[`async`](api/scala/play/api/mvc/ActionBuilder.html#async[A]\(bodyParser:play.api.mvc.BodyParser[A]\)\(block:R[A]=%3Escala.concurrent.Future[play.api.mvc.Result]\):play.api.mvc.Action[A])方法来完成。

Play提供了许多开箱即用的解析器，可以通过[`PlayBodyParsers`](api/scala/play/api/mvc/PlayBodyParsers.html)特质来使用，它们可以被注入到控制器中。

例如，定义一个需要json主体的动作(如上例所示)：

@[body-parser-json](code/ScalaBodyParsers.scala)

请注意，这次主体的类型是`JsValue`，因此可以更轻松地与主体一起工作，因为它不再是一个`Option`。之所以不是`Option`，是因为json主体解析器将验证请求具有`application/json`的`Content-Type`，并在请求不符合预期的情况下发回`415 不支持的媒体类型`响应。因此，我们无需再次检查动作代码。

当然，这意味着客户端必须表现良好，将正确的`Content-Type`头部与他们的请求一起发送。
如果您想放松一点，可以改用`tolerantJson`，它将忽略`Content-Type`并尝试将正文解析为json，无论如何：

@[body-parser-tolerantJson](code/ScalaBodyParsers.scala)

这是另一个示例，它将请求主体存储在文件中：

@[body-parser-file](code/ScalaBodyParsers.scala)

### 组合正文解析器

在前面的示例中，所有请求主体都存储在同一文件中。这有点问题，不是吗？让我们编写另一个自定义主体解析器，它从请求Session中提取用户名，从而为每个用户提供一个唯一的文件：

@[body-parser-combining](code/ScalaBodyParsers.scala)

> **注意:** 在这里，我们并不是实际在编写自己的BodyParser，而是将现有的组合起来。这通常就足够了，并且应该涵盖大多数用例。从头开始编写`BodyParser`将在高级主题部分中介绍。

### 最大内容长度

基于文本的主体解析器(例如**text**, **json**, **xml** 或 **formUrlEncoded**)使用最大内容长度，因为它们必须将所有内容加载到内存中。默认情况下，他们解析的最大内容长度为100KB。可以通过在`application.conf`中指定`play.http.parser.maxMemoryBuffer`属性来覆盖它：

    play.http.parser.maxMemoryBuffer=128K

对于在磁盘上缓存内容的解析器，例如raw解析器或`multipart/form-data`，使用`play.http.parser.maxDiskBuffer`属性指定最大内容长度，默认为10MB。该`multipart/form-data`解析器还强制了数据字段的总文本最大长度属性。

您还可以覆盖给定动作的默认最大长度：

@[body-parser-limit-text](code/ScalaBodyParsers.scala)

您还可以使用`maxLength`包装任何主体解析器：

@[body-parser-limit-file](code/ScalaBodyParsers.scala)

## 编写自定义的正文解析器

可以通过实现[`BodyParser`](api/scala/play/api/mvc/BodyParser.html)特质来创建自定义的主体解析器。此特质只是一个函数：

@[body-parser](code/ScalaBodyParsers.scala)

首先，此功能的签名可能有些令人生畏，所以让我们对其进行分解。

该函数需要一个[`RequestHeader`](api/scala/play/api/mvc/RequestHeader.html)。这可用于检查有关请求的信息-最常见的是，它用于获取`Content-Type`，以便可以正确解析主体。

这个函数的返回类型是[`Accumulator`](api/scala/play/api/libs/streams/Accumulator.html)。累加器(accumulator)是[Akka Streams](https://doc.akka.io/docs/akka/2.5/stream/index.html?language=scala)周围的一层薄的[`Sink`](https://doc.akka.io/api/akka/2.5/index.html#akka.stream.scaladsl.Sink)。一个累加器(accumulator)异步地将元素流(streams of elements)累加成一个结果，可以通过传入Akka Streams [`Source`](https://doc.akka.io/api/akka/2.5/index.html#akka.stream.scaladsl.Source)来运行它，这将返回一个`Future`, 在累加器完成后将被兑现的值。它本质上是和`Sink[E, Future[A]]`一样的东西。事实上，它只不过是这种类型的一个包装器，但最大的区别在于`Accumulator`提供了方便的方法，例如`map`, `mapFuture`, `recover`等。与结果的工作就好像它是一个承诺，其中Sink要求将所有此类操作包装在`mapMaterializedValue`调用中。

该累加器`apply`方法返回[`ByteString`](https://doc.akka.io/api/akka/2.5/akka/util/ByteString.html)类型的消耗元素 - 这些都是本质上是字节数组，但是与`byte[]`不同的是, `ByteString`是不可变的，并且许多操作，如切片和附加在发生恒定的时间。

累加器的返回类型是`Either[Result, A]` - 它要么将返回一个`Result`或将返回一个`A`类型的主体。通常在出现错误的情况下返回结果，例如，如果主体解析失败，`Content-Type`与主体解析器接受的类型不匹配，或者内存缓冲区溢出。当主体解析器返回结果时，这将缩短动作的处理 - 主体解析器结果将立即返回，并且该动作将永远不会被调用。

### 重定向主体到其他地方

编写主体解析器的一个常见用例是，当您实际上不想解析主体时，而是想将其流转到其他地方。为此，您可以定义一个自定义主体解析器：

@[forward-body](code/ScalaBodyParsers.scala)

### 使用Akka Streams自定义解析

在极少数情况下，可能有必要使用Akka Streams编写自定义解析器。在大多数情况下，首先缓冲主体到`ByteString`就足够了，这通常会提供一种更简单的解析方法，因为您可以在主体上使用命令式方法和随机访问。

但是，如果这不可行，例如，当您需要解析的主体太长而无法容纳在内存中时，那么您可能需要编写一个自定义主体解析器。

关于如何使用Akka Streams的完整描述不在本文档的范围之内 - 最好的起点是阅读[Akka Streams文档](https://doc.akka.io/docs/akka/2.5/stream/index.html?language=scala)。不管怎样，以下显示了CSV解析器，它基于Akka Streams cookbook的[从字节串流解析行](https://doc.akka.io/docs/akka/2.5/stream/stream-cookbook.html?language=scala#parsing-lines-from-a-stream-of-bytestrings)文档构建：

@[csv](code/ScalaBodyParsers.scala)
