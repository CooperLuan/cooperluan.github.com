---
layout: post
title: "SWIG Related Links"
description: "SWIG 相关的一些资料"
category: Python
tags: [python, swig, c]
---
{% include JB/setup %}

在 [instream](https://github.com/CooperLuan/instream) 项目中，需要文本分词，除了 [jieba](https://github.com/fxsjy/jieba) 外，还增加了用 `ICTCLAS` 分词的方法

`ICTCLAS` 在 zip 文件中提供了 python sample, 但是默认只能在 python2 下工作，我 clone 了一份带吗在这里 [ICTCLAS-Python](https://github.com/CooperLuan/ICTCLAS2015-python)

在试图让它能在 python3 下工作的过程中看到一篇文章 [NLPIR（ICTCLAS 2013）分词工具Python封装](http://blog.yidooo.net/archives/nlpir-python-version.html)，
文中作者用了 `SWIG` 这个工具，这是一个为 `C/C++` 程序提供高级语言(python/ruby/java/perl等)接口

考虑到时间可能比较长，先留些资料，等之后需要调用第三方 C 库时再看

- [SWIG Offical Website](http://www.swig.org/)
- [SWIG Offical Chinese Tutorial](http://www.swig.org/translations/chinese/tutorial.html)
- [开发人员 SWIG 快速入门](http://www.ibm.com/developerworks/cn/aix/library/au-swig/)
- [NLPIR（ICTCLAS 2013）分词工具Python封装](http://blog.yidooo.net/archives/nlpir-python-version.html)
