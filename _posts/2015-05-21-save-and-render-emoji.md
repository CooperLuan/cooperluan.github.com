---
layout: post
title: "保存和渲染 emoji"
description: ""
category: [Web]
tags: [Web, Emoji, MySQL]
---
{% include JB/setup %}

在 MySQL 中保存原生 emoji 表情时，会提示非法字符无法保存

原因

- 在 MySQL 中 utf8 字符集每个字符最多可以有 3 个字节码
- 补充字符如 emoji 每个字符占用 4 个字节码
- utf8mb4 可以用来保存扩展字符集

```python
In [1]: emoji = '😄'

In [2]: emoji.encode()
Out[2]: b'\xf0\x9f\x98\x84'
```

## 方法

```sql
alter table emoji change column text text varchar(100) CHARACTER SET utf8mb4 NOT NULL;
```

## 在 Web 页面中显示 emoji

从 utf8mb4 字符集列中取出的 emoji 是  utf8/unicode 编码

- 😄 \U0001f604 --> &#x1f604;

## 参考

- [The utf8mb4 Character Set (4-Byte UTF-8 Unicode Encoding)](https://dev.mysql.com/doc/refman/5.5/en/charset-unicode-utf8mb4.html)
- [Emoji Unicode Tables](http://apps.timwhitlock.info/emoji/tables/unicode)
- [How to display emoji char in HTML](http://stackoverflow.com/a/10580580/1820305)
- [Charbase](http://www.charbase.com/e056-unicode-invalid-character)
