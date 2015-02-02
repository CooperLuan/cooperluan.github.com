---
layout: post
title: "Advanced MongoDB Tips(2)"
description: "MongoDB 进阶"
category: [MongoDB]
tags: [mongodb, aggregate]
---
{% include JB/setup %}

这里有 MongoDB 官方文档中所有的[函数列表](http://docs.mongodb.org/manual/reference/method/js-collection/)

其中用来做聚合操作的几个函数是

- `aggregate(pipeline,options)` 指定 group 的 keys, 通过操作符 `$push/$addToSet/$sum` 等实现简单的 reduce, 不支持函数/自定义变量
- `group({ key, reduce, initial [, keyf] [, cond] [, finalize] })` 支持函数(`keyf`) `mapReduce` 的阉割版本
- `mapReduce` 和 `group` 类似, map/reduce 均可指定函数 只支持输出到 collection

  ```javascript
mapReduce(
    <map>,
    <reduce>, {
        out: <collection>,
        query: <document>,
        sort: <document>,
        limit: <number>,
        finalize: <function>,
        scope: <document>,
        jsMode: <boolean>,
        verbose: <boolean>,
})
  ```

- `count(query)` too young too simple
- `distinct(field,query)`

最后两个操作很简单, 这里主要比较前面 3 个操作

- [样本数据](#样本数据)
- [aggregate](#aggregate)
- [group](#group)
- [mapReduce](#mapReduce)
- [总结](#总结)

## 样本数据

这里用 python3 + pandas 生成 500w 条样本数据

```python
from datetime import date, datetime
import string
import pandas as pd
import numpy as np
from pymongo import MongoClient
from bson.objectid import ObjectId
client = MongoClient()
db = client[str(ObjectId())]

# fill data
N = 10000 * 500
dr = pd.date_range('2014-01-01', '2015-12-31', freq='M')
t1, t2 = dr.to_period()[0].start_time.timestamp(), dr.to_period()[-1].end_time.timestamp()
docs = []
bulk_size = 10000
letters = string.ascii_letters[26:]
for t in np.random.randint(t1 * 1000, t2 * 1000, N):
    docs.append({
        'created': int(t),
        'title': ''.join(np.random.permutation(list(letters))),
        'group': np.random.choice(list(letters)),
        'category': np.random.choice(['C%s' % (i + 1) for i in range(10)]),
        'count': int(np.random.randint(0, 100)),
    })
    if len(docs) >= bulk_size:
        db.data.insert(docs)
        docs = []
if docs:
    db.data.insert(docs)
for row in db.data.find(fields={'_id': 0}).limit(10):
    print(row)
```

样本输出

```javascript
{'count': 40, 'created': 1447162389256, 'group': 'H'}
{'count': 0, 'created': 1405358432462, 'group': 'F'}
{'count': 73, 'created': 1451207973496, 'group': 'G'}
{'count': 43, 'created': 1393096507018, 'group': 'C'}
{'count': 19, 'created': 1422668640892, 'group': 'H'}
{'count': 78, 'created': 1429706831821, 'group': 'G'}
{'count': 94, 'created': 1414204805996, 'group': 'H'}
{'count': 47, 'created': 1430891728820, 'group': 'C'}
{'count': 79, 'created': 1423033031540, 'group': 'A'}
{'count': 12, 'created': 1441172750750, 'group': 'I'}
```

## aggregate



## group



## mapReduce



## 总结



## 清理

```python
# cleanup
client.drop_database(db)
```
