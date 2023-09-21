---
title: "对象存储vs块存储"
date: 2023-09-07T13:13:09+08:00
lastmod: 2023-09-08T16:02:19+08:00
draft: false
tags: ["technology", "object-storage"]
categories: ["object-storage"]
author: "大白猫"
---

## 什么是块存储？

块存储（Block Storage）是最古老、最简单的数字存储形式。在块存储中，数据被存储在固定大小的“块”中，每个block通常只存储完整文件的一部分。应用程序进行SCSI调用来查找block的地址，然后将它们组织起来形成完整的文件。
