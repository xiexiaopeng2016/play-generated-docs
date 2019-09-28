<!--- Copyright (C) 2009-2019 Lightbend Inc. <https://www.lightbend.com> -->
# 会话和Flash作用域

## 它在Paly中的差异

如果必须跨多个HTTP请求保留数据，则可以将它们保存在Session或Flash作用域中。存储在Session内的数据在整个用户会话期间可用，并存储在Flash作用域内的数据**只**可用于下一个请求。

重要的是要了解Session和Flash数据不是由服务器存储的，而是使用cookie机制添加到每个后续HTTP请求中的。这意味着数据大小非常有限(最大为4 KB)，并且您只能存储字符串值。cookie的默认名称是`PLAY_SESSION`。可以通过在application.conf中配置`play.http.session.cookieName`键来更改。

> 如果更改了cookie的名称，则可以使用[[设置和丢弃cookie|ScalaResults]]中提到的相同方法来丢弃早期的cookie。

当然，Cookie值是用密钥签名的，因此客户端无法修改Cookie数据(否则它将无效)。

Play的Session不打算用作缓存。如果您需要缓存与特定Session相关的数据，则可以使用Play内置的缓存机制，并在用户Session中存储唯一ID，以使它们与特定用户相关。

## Session配置

请参阅[[配置会话Cookie|SettingsSession]]，以获取有关如何在`application.conf`中配置会话Cookie参数的更多信息。

### Session超时/过期

默认情况下，Session没有技术上的超时。当用户关闭Web浏览器时，它会过期。如果一个特定应用程序需要功能上的超时，则可以通过配置`application.conf`中的`play.http.session.maxAge`键来设置会话Cookie的最长期限，并且也将设置`play.http.session.jwt.expiresAfter`为相同的值。该`maxAge`属性将cookie从浏览器中删除，并且将在cookie中设置JWT`exp`声明，并在给定的持续时间后使其无效。请参阅[[阅配置会话Cookie|SettingsSession]] for more information，以获取更多信息。

## 在会话中存储数据

由于Session只是一个Cookie，因它也是一个HTTP头部。您可以使用与处理其他结果属性相同的方式来处理会话数据：

@[store-session](code/ScalaSessionFlash.scala)

请注意，这将替换整个Session。如果需要将元素添加到现有Session中，只需将元素添加到传入Session中，然后将其指定为新Session：

@[add-session](code/ScalaSessionFlash.scala)

您可以使用相同的方法从传入会话中删除任何值：

@[remove-session](code/ScalaSessionFlash.scala)

## 读取会话值

您可以从HTTP请求中检索传入的会话：

@[index-retrieve-incoming-session](code/ScalaSessionFlash.scala)

## 丢弃整个会话

有一个特殊的操作会丢弃整个会话：

@[discarding-session](code/ScalaSessionFlash.scala)

## Flash作用域

Flash作用域的工作与Session完全相同，但有两个区别：

- 数据仅为一个请求保留
- Flash cookie没有签名，因此用户可以对其进行修改。

> **重要说明:** Flash作用域仅应用于在简单的非Ajax应用程序上传输成功/错误消息。由于数据仅为下一个请求保留，并且由于在复杂的Web应用程序中不能保证请求的顺序，因此Flash作用域受竞争条件的影响。

以下是一些使用Flash作用域的示例：

@[using-flash](code/ScalaSessionFlash.scala)

要在您的视图中检索Flash作用域值，请添加一个隐式Flash参数：

@[flash-template](code/scalaguide/http/scalasessionflash/views/index.scala.html)

在您的Action中指定一个`implicit request =>`，如下所示：

@[flash-implicit-request](code/ScalaSessionFlash.scala)

将基于隐式请求将一个隐式Flash提供给视图。

如果引发了'_找不到参数flash的隐式值: play.api.mvc.Flash_'错误，则这是因为您的Action在作用域内没有隐式请求。
