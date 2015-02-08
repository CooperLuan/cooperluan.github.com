---
layout: post
title: "matrix roation in codeeval 矩阵转置"
description: "codeeval 中的题目"
category: [Python, Codeeval, Algorithm]
tags: [Python, Codeeval, Algorithm]
---
{% include JB/setup %}

题目来自 codeeval 中的 easy 级别 [MATRIX ROTATION](https://www.codeeval.com/open_challenges/178/)

要顺时针翻转一个 `n * n` 矩阵, 有肉眼观察到的变化就是列变行, 第一列 --> 第一行并且由于是顺时针翻转, 依次将每列转为行之后还需要翻转每行的元素顺序

> 这是肉身解法, 算法不适用于翻转 180/270 的情况, 写这个只是为了证明肉身解法也挺方便的

```python
import sys
import math


def rotate_matrix(line):
    """
    依次遍历列, 然后翻转行元素次序
    """
    n = int(math.sqrt(len(line)))
    for i in range(n):
        for j in reversed(range(n)):
            yield line[j * n + i]

matrix_lines = """a b c d
a b c d e f g h i j k l m n o p
a b c d e f g h i"""
for line in matrix_lines.splitlines():
    print(' '.join(rotate_matrix(line.split())))
```

输出

```python
c a d b
m i e a n j f b o k g c p l h d
g d a h e b i f c
```
