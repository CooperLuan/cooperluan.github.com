---
layout: post
title: "Pandas Snippets & Tips"
description: "Pandas 使用小技巧"
category: python
tags: [python, pandas, snippet]
---
{% include JB/setup %}

## Indexes

- 像 RDB 一样 `left join` 两个 `dataset`
- 在多级索引的 `dataset` 中使用索引

### 像 RDB 一样 join 两个 dataset

```python
import pandas as pd
import numpy as np
import string
df1 = pd.DataFrame({
    'id': list(string.ascii_letters[:10]),
    'sales': np.random.rand(10),
})
df2 = pd.DataFrame({
    'id': list(string.ascii_letters[3:13]),
    'count': np.random.rand(10),
})
df3 = pd.merge(df1, df2, on='id')
df4 = pd.merge(df1, df2, on='id', how='left')
```

输出

```python
In [49]: df = pd.merge(df1, df2, on='id')

In [50]: df
Out[50]: 
  id     sales     count
0  d  0.875390  0.902091
1  e  0.193723  0.546562
2  f  0.250351  0.858821
3  g  0.203284  0.867194
4  h  0.473823  0.991851
5  i  0.602875  0.435074
6  j  0.320997  0.193434

In [52]: pd.merge(df1, df2, on='id', how='left')
Out[52]: 
  id     sales     count
0  a  0.051131       NaN
1  b  0.571569       NaN
2  c  0.846158       NaN
3  d  0.875390  0.902091
4  e  0.193723  0.546562
5  f  0.250351  0.858821
6  g  0.203284  0.867194
7  h  0.473823  0.991851
8  i  0.602875  0.435074
9  j  0.320997  0.193434
```

## 在多级索引的 dataset 中使用索引

```python
df = pd.DataFrame({
    'A': list('aaadeff'),
    'B': list('abbgeaz'),
    'Val': np.random.rand(7),
})
df = df.set_index(['A', 'B'])
idx = pd.IndexSlice
df.loc[idx[:,['g']], :]
```

输出

```python
Out[1]: 
          Val
A B          
d g  0.257651
```
