---
layout: post
title: "[摘抄] 怎么用 grep"
description: ""
category: [linux, Excerpt]
tags: [linux, Excerpt]
---
{% include JB/setup %}

本文摘抄自

> [grep 是什么？怎么用？](http://mp.weixin.qq.com/s?__biz=MzAxODI5ODMwOA==&mid=205263092&idx=1&sn=010770255305b9ca7927011626038da9&key=8ea74966bf01cfb631eeb005f985c34abe2ddb3f1387dcebcfc79b9833a422e62234892375eac9b9e7d411b9dd6cd238&ascene=0&uin=MjQwMjI0ODcyMA%3D%3D&devicetype=iMac+MacBookPro11%2C1+OSX+OSX+10.10.2+build(14C1510)&version=11020012&pass_ticket=3Y7tcdLU4jhvDpSWjwTWoxok2PMUXJp4zqdUiKOhjotZbTQarJGs53dZrQeB%2Bopf)

grep 用来搜索匹配输入中的行, 可以用 `man grep` 查看帮助

以下是一些 sample 用法, 加深记忆可以帮助快速的回头搜索和有效开发

- 匹配单词 `-w` 如 `man grep | grep -w "\\-w"` 找到文档中 `-w` 选项的说明
- 匹配单词开头 `\<` 如 `man grep | grep "\<word"`
- 匹配单词结尾 `\>` 如 `man grep | grep "ed\>"`
- 匹配行开头 `^`
- 匹配行结尾 `$`
- 显示匹配结果的上下文 `-C[num]` 如显示结果的上下两行  `man grep | grep -C2 -w "\\-w"`
- 指定显示下文或者上文 `-A[num]/-B[num]` 如 `man grep | grep -A2 -w "\\-w"`
- 用正则匹配数字 `grep -E "192.168.1.[0-9]{1,3}"`
- 匹配单词边界如 `grep -E "\b192.168.1.[0-9]{1,3}"` 更准确
- 显示不匹配的结果 `-v` 如 `grep -v "#"` 去掉注释
- 匹配多个单词 `grep -E "(socket|ITEM)"`
