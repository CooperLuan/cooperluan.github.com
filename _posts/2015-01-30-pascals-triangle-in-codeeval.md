---
layout: post
title: "pascals triangle in codeeval 杨辉三角的生成"
description: ""
category: [python, codeeval, algorithm]
tags: [python, algorithm, codeeval]
---
{% include JB/setup %}

来自 codeeval 的题目 [PASCALS TRIANGLE](https://www.codeeval.com/open_challenges/66/)

要求给出 n 生成 n 行的杨辉三角形数组

## 思路

- 第一行初始化给定
- 从第二行开始有固定的规律
- 第一位和最后一位是固定的 1
- 中间的数值个数是上一行的数值个数 - 1
- 中间的数值可以用 ns[1:] + ns[:-1] 而成

那写出来就是这样的

```python
# 根据上一行 计算下一行的数值
def next_triangle_line(ns):
    return [1] + [i + j for i, j in zip(ns[1:], ns[:-1])] + [1]


# 生成整个三角形数组
def pascals_triangle(n):
    arrs = [[1]]
    for i in range(1, n):
        arrs.append(next_triangle_line(arrs[-1]))
    return arrs

```

## 待优化
