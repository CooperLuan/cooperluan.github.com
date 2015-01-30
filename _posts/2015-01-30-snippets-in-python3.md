---
layout: post
title: "snippets in python3"
description: ""
category: [python]
tags: [python, snippet]
---
{% include JB/setup %}

## index

- [sort list by zh-CN word](#sort-list-by-zh-cn-word)

## sort list by zh-CN word

按照中文拼音顺序

```python
import functools
import locale
locale.setlocale(locale.LC_COLLATE, 'zh_CN.UTF8')

names = ['曾国藩', '李鸿章', '左宗棠', 'Facebook', 'Google', 'UK', '鼻祖', 'Start-Up', 'Quora']
sorted(names, key=functools.cmp_to_key(locale.strcoll))
```

Output

```python
['Facebook', 'Google', 'Quora', 'Start-Up', 'UK', '鼻祖', '曾国藩', '李鸿章', '左宗棠']
```

ORZ 如果混如英文单词的话，它会排在最前面, 如果要中英文混排, 最好把中文转换为拼音, github 上有第三方库
