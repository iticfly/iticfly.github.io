---
layout: post
title: 111将 Mozilla 源码里的 winEmbed 工程移植到 VC111
categories: Gecko
description: Mozilla 源码里有一个简单的内嵌 Gecko 的示例工程 winEmbed，但是无法直接在 VC 中使用，这是将它移植到 VC 中的方法。
keywords: winEmbed, Mozilla
---

最近在学习怎么将 Gecko 嵌入到自己的应用程序中，下载了一份比较早一点的源码在对照官方文档痛苦地推进——网上相关资料确实相当缺乏，难道大家都各种 webkit 去了？我的计划是先弄清怎么用，让程序跑起来，然后再根据官方文档结构说明去定制，削减掉不需要的部分，折腾这个移植就花了我不少时间，果断觉得应该跟大家分享之。废话不说,直接上过程。

