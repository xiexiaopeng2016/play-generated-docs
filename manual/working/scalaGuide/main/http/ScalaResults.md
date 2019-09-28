<!--- Copyright (C) 2009-2019 Lightbend Inc. <https://www.lightbend.com> -->
# 处理结果

## 更改默认的`Content-Type`

结果内容类型将从您在响应正文指定的Scala值中自动推断出来。

例如：

@[content-type_text](code/ScalaResults.scala)

会自动将`Content-Type`头部设置为`text/plain`，同时：

@[content-type_xml](code/ScalaResults.scala)

会将Content-Type头部设置为`application/xml`。

> **提示:** 这是通过`play.api.http.ContentTypeOf`类型类完成的。

这是非常有用的，但是有时您想要更改它。只需在结果上使用`as(newContentType)`方法即可创建新的类似结果, 其使用一个不同的`Content-Type`头部：

@[content-type_html](code/ScalaResults.scala)

如果使用以下方式，将会更好：

@[content-type_defined_html](code/ScalaResults.scala)

> **注意:** 使用`HTML`代替`"text/html"`的好处是，将自动为您处理字符集，并将实际的Content-Type标头设置为`text/html; charset=utf-8`。我们[[待会再看|ScalaResults#Changing-the-charset-for-text-based-HTTP-responses]]。

## 处理HTTP标头

您还可以添加(或更新)任何HTTP头部到结果中：

@[set-headers](code/ScalaResults.scala)

请注意，设置HTTP头部将自动覆盖先前的值，如果在原始结果中存在的话。

## 设置和丢弃Cookie

Cookies只是HTTP头部的一种特殊形式，但我们提供了一组帮助程序，以使其变得更容易。

您可以使用以下方法轻松地将Cookie添加到HTTP响应中：

@[set-cookies](code/ScalaResults.scala)

另外，要丢弃先前存储在Web浏览器中的Cookie，请执行以下操作：

@[discarding-cookies](code/ScalaResults.scala)

您还可以在同一响应中设置和删除部分Cookie：

@[setting-discarding-cookies](code/ScalaResults.scala)

## 更改基于文本的HTTP响应的字符集

对于基于文本的HTTP响应，正确处理字符集非常重要。Play会自动为您处理并默认使用`utf-8`(请参阅[为什么使用utf-8](http://www.w3.org/International/questions/qa-choosing-encodings#useunicode))。

字符集用于将文本响应转换为相应的字节以通过网络套接字发送，并使用适当的`;charset=xxx`扩展去更新`Content-Type`标头。

字符集通过`play.api.mvc.Codec`类型类自动处理。只需在当前作用域中导入一个`play.api.mvc.Codec`隐式实例，即可更改所有操作将使用的字符集：

@[full-application-set-myCustomCharset](code/ScalaResults.scala)

在这里，由于作用域中存在一个隐式字符集值，因此，将XML消息转换为`ISO-8859-1`编码的`Ok(...)`方法和生成`text/html; charset=iso-8859-1`Content-Type头部都将使用该值。

现在，如果您想知道`HTML`方法的工作原理，这里是它的定义方式：

@[Source-Code-HTML](code/ScalaResults.scala)

如果需要以通用方式处理字符集，则可以在API中执行相同的操作。
