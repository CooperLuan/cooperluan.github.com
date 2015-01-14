---
layout: post
title: "Advanced Aggregate in MongoDB(1) MongoDB 聚合操作进阶(1)"
description: "一些 MongoDB 的 aggregate 操作技巧"
category: MongoDB
tags: [MongoDB, Aggregate]
---
{% include JB/setup %}

<style type="text/css">
    table {
        width: 100%;
    }
    table, table td, table th {
        border: 1px solid #ddd;
    }
</style>

先看一组数据

| id        | group     | visible |   count |
|-----------|-----------|---------|---------|
|1          | A         | 1       |     13  |
|2          | B         | 1       |    93  |
|3          | A         | 0       |    31  |
|4          | C         | 0       |   194  |

要对 `group`/`visible` 两个字段的 `count` 做 `sum` 操作，有两种方法

第一种是直接在 `_id` 中指定这两个字段

```javascript
db.sample.aggregate([
    {$group: {
        _id: {'group': '$group', 'visible': '$visible'},
        count: {$sum: '$count'}
    }}
])
```

这样出来的结果是

```javascript
[{
    "_id": {
        "group": "A",
        "visible": 1
    },
    "value": 13
}, {
    "_id": {
        "group": "A",
        "visible": 0,
    },
    "count": 31
}, {
    "_id": {
        "group": "B",
        "visible": 1,
    },
    "count": 93
}, {
    "_id": {
        "group": "C",
        "visible": 0,
    },
    "count": 194
}]
```

这时 `_id` 是一个 dict, 如果在一些场景下要求 `_id` 是 `string`, 而 `visible` 是 `boolean` 类型，就可以这样

```javascript
db.sample.aggregate([
    {$group: {
        _id: "$group",
        visibleCount: {$sum: {$cond: [{$eq: ["$visible", 1]}, "$count", 0]}},
        invisibleCount: {$sum: {$cond: [{$ne: ["$visible", 1]}, "$count", 0]}}
    }}
])
```

结果是这样的

```javascript
[{
    "_id": "A",
    "visibleCount": 13,
    "invisibleCount": 31
}, {
    "_id": "B",
    "visibleCount": 93,
    "invisibleCount": 0
}, {
    "_id": "C",
    "visibleCount": 0,
    "invisibleCount": 194
}]
```

测试代码在这里

```python
from pymongo import MongoClient
client = MongoClient('192.168.1.202')
db = client['test']

# 插入测试数据
collection = db['20150114']
data_columns = ['group', 'visible', 'count']
data_rows = [
    ('A', 1, 13),
    ('B', 1, 93),
    ('A', 0, 31),
    ('C', 0, 194),
]
collection.insert([dict(zip(data_columns, row)) for row in data_rows])

# 测试1
result = collection.aggregate([
    {'$group': {
        '_id': {'group': '$group', 'visible': '$visible'},
        'count': {'$sum': '$count'}
    }}
])['result']
print(result)

# 测试2
result2 = collection.aggregate([
    {'$group': {
        '_id': "$group",
        'visibleCount': {'$sum': {'$cond': [{'$eq': ["$visible", 1]}, "$count", 0]}},
        'invisibleCount': {'$sum': {'$cond': [{'$ne': ["$visible", 1]}, "$count", 0]}}
    }}
])['result']
print(result2)

# Cleanup
collection.drop()
```
