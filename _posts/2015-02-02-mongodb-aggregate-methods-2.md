---
layout: post
title: "MongoDB Aggregate Methods(2) MonoDB 的 3 种聚合函数"
description: "MongoDB 进阶"
category: [MongoDB]
tags: [mongodb, aggregate]
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

这里有 MongoDB 官方文档中所有的[函数列表](http://docs.mongodb.org/manual/reference/method/js-collection/)

其中用来做聚合操作的几个函数是

- `aggregate(pipeline,options)` 指定 group 的 keys, 通过操作符 `$push/$addToSet/$sum` 等实现简单的 reduce, 不支持函数/自定义变量
- `group({ key, reduce, initial [, keyf] [, cond] [, finalize] })` 支持函数(`keyf`) `mapReduce` 的阉割版本
- `mapReduce` 终极大杀器
- `count(query)` too young too simple
- `distinct(field,query)`

最后两个操作很简单, 这里主要比较前面 3 个操作

- [样本数据](#样本数据)
- [aggregate](#aggregate)
- [group](#group)
- [mapReduce](#mapReduce)
- [总结](#总结)
- [样本数据生成](#样本数据生成)

## 样本数据

- created 时间戳(ms)
- group 组别 [A-Z]
- category 目录 C[1-10]
- count
- title 26 个字母随机组成

```javascript
{'group': 'E', 'created': 1402764223433, 'count': 63, 'datetime': datetime.datetime(2014, 6, 15, 0, 43, 43, 433000), 'title': 'KNBVHICLGOUESDFYTJWRXMPZQA', 'category': 'C8'}
{'group': 'I', 'created': 1413086660063, 'count': 93, 'datetime': datetime.datetime(2014, 10, 12, 12, 4, 20, 62000), 'title': 'UJZXCIHBFKELPYVWGAQRMNOSDT', 'category': 'C10'}
{'group': 'H', 'created': 1440750343730, 'count': 41, 'datetime': datetime.datetime(2015, 8, 28, 16, 25, 43, 730000), 'title': 'HSDKQPFMNBJEGXYZARITCVOWLU', 'category': 'C1'}
{'group': 'S', 'created': 1437710373637, 'count': 14, 'datetime': datetime.datetime(2015, 7, 24, 11, 59, 33, 637000), 'title': 'ZFJITUDHMBRLGKNEWSCPXVOYQA', 'category': 'C10'}
{'group': 'Z', 'created': 1428307315436, 'count': 78, 'datetime': datetime.datetime(2015, 4, 6, 16, 1, 55, 436000), 'title': 'FUYXWOQZPTKMSCNGDHEJVBILAR', 'category': 'C5'}
{'group': 'R', 'created': 1402809274964, 'count': 74, 'datetime': datetime.datetime(2014, 6, 15, 13, 14, 34, 963000), 'title': 'JKZCMPIAONDTWRQLHGUEVSFXYB', 'category': 'C9'}
{'group': 'Y', 'created': 1400571321919, 'count': 66, 'datetime': datetime.datetime(2014, 5, 20, 15, 35, 21, 918000), 'title': 'KSPGZJDMHNUALCEWBYXVIQOTFR', 'category': 'C2'}
{'group': 'L', 'created': 1416562128208, 'count': 5, 'datetime': datetime.datetime(2014, 11, 21, 17, 28, 48, 207000), 'title': 'ZHDLBRMNPXEVAQKJYSITCFGWUO', 'category': 'C1'}
{'group': 'E', 'created': 1414057884852, 'count': 12, 'datetime': datetime.datetime(2014, 10, 23, 17, 51, 24, 851000), 'title': 'EBZKXDTOQSCYJAGFPVIHNRULMW', 'category': 'C3'}
{'group': 'L', 'created': 1418879346846, 'count': 67, 'datetime': datetime.datetime(2014, 12, 18, 13, 9, 6, 845000), 'title': 'DZMQRXYVHOJFUGENCASTLWBPKI', 'category': 'C3'}
```

## aggregate

接受两个参数 `pipeline`/`options`, `pipeline` 是 array, 相同的 operator 可以多次使用

pipeline 支持的方法

- `$geoNear` 不予测试
- `$group` 指定 group 的 `_id`(key/keys) 和基于操作符(`$push`/`$sum`/...) 的累加运算
- `$limit` 限制输出
- `$match` 输入过滤条件
- `$out` 将输出结果保存到 `collection`
- `$project` 修改数据流中的文档结构
- `$redact` 是 `$project`/`$match` 功能的合并
- `$skip`
- `$sort` 对结果排序
- `$unwind` 拆解数据

`$group` 允许用的累加操作符 `$addToSet`/`$avg`/`$first`/`$last`/`$max`/`$min`/`$push`/`$sum`  
不被允许的累加操作符 `$each`...  
`$group` 操作默认最多可以用 100MB RAM, 增加 `allowDiskUse` 可以让 `$group` 操作更多的数据

下面是一个揉进全部特性的用法

```js
db.data.aggregate([
    {$match: {}},
    {$skip: 10}, // 跳过 collection 的前 10 行
    // 选择需要的字段
    {$project: {group: 1, datetime: 1, category: 1, count: 1}},
    // 如果不选择 {count: 1} 最后的结果中 count_all/count_avg = 0
    // {$project: {group: 1, datetime: 1, category: 1}},
    {$redact: { // redact 简单用法 过滤 group != 'A' 的行
        $cond: [{$eq: ["$group", "A"]}, "$$DESCEND", "$$PRUNE"]
    }},
    {$group: {
        _id: {year: {$year: "$datetime"}, month: {$month: "$datetime"}, day: {$dayOfMonth: "$datetime"}},
        group_unique: {$addToSet: "$group"},
        category_first: {$first: "$category"},
        category_last: {$last: "$category"},
        count_all: {$sum: "$count"},
        count_avg: {$avg: "$count"},
        rows: {$sum: 1}
    }},
    // 拆分 group_unique 如果开启这个选项, 会导致 _id 重复而无法写入 out 指定的 collection, 除非再 $group 一次
    // {$unwind: "$group_unique"},
    // 只保留这两个字段
    {$project: {group_unique: 1, rows: 1}},
    // 结果按照 _id 排序
    {$sort: {"_id": 1}},
    // 只保留 50 条结果
    // {$limit: 50},
    // 结果另存
    {$out: "data_agg_out"},
], {
    explain: true,
    allowDiskUse: true,
    cursor: {batchSize: 0}
})
db.data_agg_out.find()
db.data_agg_out.aggregate([
    {$group: {
        _id: null,
        rows: {$sum: '$rows'}
    }}
])
db.data_agg_out.drop()
```

- `$match` 聚合前数据筛选
- `$skip` 跳过聚合前数据集的 n 行, 如果 `{$skip: 10}`, 最后 `rows = 5000000 - 10`
- `$project` 之选择需要的字段, 除了 `_id` 之外其他的字段的值只能为 1
- `$redact` 看了文档不明其实际使用场景, 这里只是简单筛选聚合前的数据
- `$group` 指定各字段的累加方法
- `$unwind` 拆分 array 字段的值, 这样会导致 `_id` 重复
- `$project` 可重复使用多次 最后用来过滤想要存储的字段
- `$out` 如果 `$group`/`$project`/`$redact` 的 `_id` 没有重复就不会报错
- 以上方法中 `$project`/`$redact`/`$group`/`$unwind` 可以使用多次

## group

`group` 比 `aggregate` 好的一个地方是 `map/reduce` 都支持用 `function` 定义, 下面是支持的选项

- `ns` 如果用 `db.runCommand({group: {}})` 方式调用, 需要 `ns` 指定 collection
- `cond` 聚合前筛选
- `key` 聚合的 key
- `initial` 初始化 累加 结果
- `$reduce` 接受 `(curr, result)` 参数, 将 `curr` 累加到 `result`
- `keyf` 代替 `key` 用函数生成聚合用的主键
- `finalize` 结果处理

需要保证输出结果小于 16MB 因为 `group` 没有提供转存选项

```js
db.data.group({
    cond: {'group': 'A'},
    // key: {'group': 1, 'category': 1},
    keyf: function(doc) {
        var dt = new Date(doc.created);
        // or
        // var dt = doc.datetime;
        return {
            year: doc.datetime.getFullYear(),
            month: doc.datetime.getMonth() + 1,
            day: doc.datetime.getDate()
        }
    },
    initial: {count: 0, category: []},
    $reduce: function(curr, result) {
        result.count += curr.count;
        if (result.category.indexOf(curr.category) == -1) {
            result.category.push(curr.category);
        }
    },
    finalize: function(result) {
        result.category = result.category.join();
    }
})
```

如果要求聚合大量数据, 就需要用到 `mapReduce`

## mapReduce

先看看它支持的特性/选项

- `query` 聚合前筛选
- `sort` 对聚合前的数据排序 用来优化 reduce
- `limit` 限制进入 map 的数据
- `map`(function) emit(key, value) 在函数中指定聚合的 K/V
- `reduce`(function) 参数 `(key, values)` `key` 在 map 中定义了, `values` 是在这个 K 下的所有 V 数组
- `finalize` 处理最后结果
- `out` 结果转存 可以选择另外一个 db
- `scope` 设置全局变量
- `jdMode`(false) 是否(默认是)把 map/reduce 中间结果转为 BSON 格式, BSON 格式可以利用磁盘空间, 这样就可以处理大规模的数据集
- `verbose`(true) 详细信息

如果设 `jsMode` 为 true 不进行 BSON 转换, 可以优化 reduce 的执行速度, 但是由于内存限制最大在 emit 数量小于 500,000 时使用


写 mapReduce 时需要注意

- emit 返回的 value 必须和 reduce 返回的 value 结构一致
- `reduce` 函数必须幂等
- 详见 [Troubleshoot the Reduce Function](http://docs.mongodb.org/manual/tutorial/troubleshoot-reduce-function/)

```js
db.data.mapReduce(function() {
    // var d = new Date(this.created);
    var d = this.datetime;
    var key = {
        year: d.getFullYear(),
        month: d.getMonth() + 1,
        day: d.getDate(),
    };
    var value = {
        count: this.count,
        rows: 1,
        groups: [this.group],
    }
    emit(key, value);
}, function(key, vals) {
    var reducedVal = {
        count: 0,
        groups: [],
        rows: 0,
    };
    for(var i = 0; i < vals.length; i++) {
        var v = vals[i];
        reducedVal.count += v.count;
        reducedVal.rows += v.rows;
        for(var j = 0; j < v.groups.length; j ++) {
            if (reducedVal.groups.indexOf(v.groups[j]) == -1) {
                reducedVal.groups.push(v.groups[j]);
            }
        }
    }
    return reducedVal;
}, {
    query: {},
    sort: {datetime: 1},    // 需要索引 否则结果返回空
    limit: 50000,
    finalize: function(key, reducedVal) {
        reducedVal.avg = reducedVal.count / reducedVal.rows;
        return reducedVal;
    },
    out: {
        inline: 1,
        // replace: "",
        // merge: "",
        // reduce: "",
    },
    scope: {},
    jsMode: true
})
```

## 总结

| method    | allowDiskUse   |   out |  function  |
|-----------|-----------|---------|-------------- |
|aggregate  | true      |     pipeline/collection |  false |
|group      | false     | pipeline   | true       |
|mapReduce  | jsMode    | pipeline/collection | true |


- `aggregate` 基于累加操作的的聚合 可以重复利用 `$project`/`$group` 一层一层聚合数据, 可以用于大量数据(单输出结果小于 16MB)  不可用于分片数据
- `mapReduce` 可以处理超大数据集 需要严格遵守 mapReduce 中的结构一致/幂等 写法,   可增量输出/合并, 见 `out` options
- `group` RDB 中的 `group by` 简单需求可用(只有 inline 输出)  会产生 `read lock`
- [StackOverflow 中关于三者比较的解答](http://stackoverflow.com/questions/12337319/mongodb-aggregation-comparison-group-group-and-mapreduce)

## 清理

```python
# cleanup
client.drop_database(db)
```

## 样本数据生成

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
        'datetime': datetime.fromtimestamp(t / 1000),
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
