---
title: "对象存储vs块存储"
date: 2023-09-07T13:13:09+08:00
lastmod: 2023-010-08T11:18:19+08:00
draft: false
tags: ["technology", "object-storage"]
categories: ["object-storage"]
author: "大白猫"
---

## 什么是块存储？

块存储（Block Storage）是最古老、最简单的数字存储形式。在块存储中，数据被存储在固定大小的“块”中，每个block通常只存储完整文件的一部分。应用程序进行SCSI调用来查找block的地址，然后将它们组织起来形成完整的文件。

由于数据是零散的，因此地址是block的唯一标识——没有与blcok关联的元数据。当应用程序和存储空间都位于本地时，这种结构会带来更快的性能，但如果二者距离较远，就会导致较高的延迟。

块存储提供的粒度控制使其非常适合需要高性能的应用程序，例如事务或数据库应用程序。

## 块存储和对象存储的区别

{{< figure src="/image/object-storage-vs-block-storage.jpg" title="Object Storage vs Block Storage">}}

对象存储的出现时间比块存储晚得多。在对象存储中，数据与可定制的元metadata、唯一标识符捆绑在一起，从而构成object。每个object都存储在平铺的地址空间中，没有数量限制，因此更容易横向扩展。

metadata是对象存储的一个关键优势——它们可以更好地识别和区分数据。我们可以将object视为一种self-describing storage：它们具有由创建者或应用程序分配的描述性标签。因此我们可以轻松地搜索指定object，哪怕数据本身不易被检索（例如图像、媒体或数据集）。易于检索和无限规模使对象存储成为非结构化数据存储的理想选择。

以上就是关于对象存储与块存储之间差异的简要概括。块存储在企业中有很多应用，但对象存储最适合处理爆炸性增长的非结构化数据。



Original: [Cloudian_Object_v_Block_Storage](https://cloudian.com/wp-content/uploads/2017/05/Cloudian_Object-v-Block-Storage.pdf)
