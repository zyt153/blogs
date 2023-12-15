---
title: "Customer case - content length is zero"
date: 2023-10-08T16:15:32+08:00
draft: false
tags: ["technology", "pr"]
categories: ["pr"]
author: "大白猫"
---

这是一个今年上半年非常令我头大的PR，一方面是产生错误的原因不好找，另一方面是和客户的沟通比较费劲。最终我在一次客户会议中解决了问题，当时觉得很有成就感。

## 问题描述

客户在尝试获取object storage中的一个json文件时出现**FixedLengthOverflowException**：
```java
[task] ERROR org.xnio.channels.FixedLengthOverflowException: null
at io.undertow.conduits.AbstractFixedLengthStreamSinkConduit.write(AbstractFixedLengthStreamSinkConduit.java:100)
at org.xnio.conduits.Conduits.writeFinalBasic(Conduits.java:132)
at io.undertow.conduits.AbstractFixedLengthStreamSinkConduit.writeFinal(AbstractFixedLengthStreamSinkConduit.java:175)
at org.xnio.conduits.ConduitStreamSinkChannel.writeFinal(ConduitStreamSinkChannel.java:104)
at io.undertow.channels.DetachableStreamSinkChannel.writeFinal(DetachableStreamSinkChannel.java:195)
at io.undertow.server.HttpServerExchange$WriteDispatchChannel.writeFinal(HttpServerExchange.java:2171)
at io.undertow.servlet.spec.ServletOutputStreamImpl.writeBufferBlocking(ServletOutputStreamImpl.java:582)
at io.undertow.servlet.spec.ServletOutputStreamImpl.close(ServletOutputStreamImpl.java:617)
at io.undertow.servlet.spec.ServletOutputStreamImpl.updateWritten(ServletOutputStreamImpl.java:373)
at io.undertow.servlet.spec.ServletOutputStreamImpl.write(ServletOutputStreamImpl.java:155)
......
```

但这个异常仅出现在这个json文件内容较多时，一旦json文件长度超过了某个限制，就无法获取到内容。

进一步检查log中的request和response发现：

* 一旦json文件长度超过了某个限制，返回的response就会被gzip压缩，此时**response header中的content-length为0**，client读不到content-length，进而无法确定内容长度，导致**FixedLengthOverflowException**。
* 在request header中发现`Cookie：BIGipServer...`

## 问题分析

从现象可以猜测是压缩导致了content-length被清空，google中的某些帖子也提出了相似的问题（由于时间有点久远，我不记得当时看了哪些帖子，但确实现学现卖了解了不少），大意是说由于response被压缩，http server无法确定实际的内容长度，于是干脆舍弃content-length。但我们的平台只负责调用AWS client并获得结果，并没有直接处理http响应的unzip。难道AWS client不能处理压缩请求吗？因此，我们的后续处理是：

1. 找对应的object storage平台的工程师确认他们是否会对s3请求的response进行压缩，得到的答复是不会；
2. 找客户确认是不是在我们的平台与object storage平台之间配置了中间件，这个中间件会导致response被压缩，客户说没有。但后续一系列沟通发现的事实是，s3请求的response会先经过一个F5 load balancer（也就是header中的Cookie：BIGipServer...）。

与此同时，我检查了code，发现在AWS client configuration中有一个配置叫`useGzip`，AWS在注释中对这个参数的描述是：

```java
// Optional whether to use gzip decompression when receiving HTTP responses.
```

这个参数在AWS client中的默认值是`false`，但我们自己的代码中不知道为啥配置了`true`，大概是想要支持gzip解压。

进一步往里找发现，AWS http client是调用了Apache http client，相关的代码逻辑在`/com/amazonaws/http/apache/client/impl/ApacheHttpClientFactory.java`中。在处理gzip的逻辑处，有这样一个注释：

```java
// By default http client enables Gzip compression. So we disable it
// here.
// Apache HTTP client removes Content-Length, Content-Encoding and
// Content-MD5 headers when Gzip compression is enabled. Currently
// this doesn't affect S3 or Glacier which exposes these headers.
//
if (!(settings.useGzip())) {
    builder.disableContentCompression();
}
```

也就是说，如果我们enable gzip compression，Apache http client就会移除content-length这个header，因此AWS client的做法是默认将gzip开关设为false（但他们后续自己做了解压？因为即使设了`gzip=false`，zipped response也可以被正常处理）。

为了验证gzip压缩的影响，我用nginx模拟可以压缩数据的中间件进行了验证，基本可以确定content-length=0的原因就是来自于gzip。

## 解决方案

解决方案说来也简单，*就是将`useGzip`再次改回`false`*。感谢AWS工程师在注释中的说明，really useful comments。
