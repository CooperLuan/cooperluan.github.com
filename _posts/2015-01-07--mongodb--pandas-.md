---
layout: post
title: "用 MongoDB.mapReduce + Pandas 做简单聚合统计"
description: "用 MongoDB.mapReduce + Pandas 做简单聚合统计"
category: [python, statistics]
tags: [mongodb, pandas, python, mapReduce]
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

在对时间序列数据做各种维度的统计操作时，考虑到操作的数据集可能比较大，需要先用 `MongoDB` 做 `mapReduce` 操作，
因为如果数据集超过内存, MongoDB 可以开启 `allowDiskUse`
而 pandas 是全内存操作，适合处理中间聚合/统计结果

## 数据集样本

| id        | created   | group     |   count |
|-----------|-----------|-----------|---------|
|1          |1420069372983 | A      |     13  |
|2          |1420073372174 | B      |     93  |
|3          |1420072572983 | A      |     31  |
|4          |1419908285069 | C      |    194  |
|...        |....          | ..     |     ....|

假设大概有 100w 条数据

## 用 Mongo.mapReduce 做聚合操作

将数据按照 `created` 时间戳所在月份 + `group` 两个 key 进行聚合，对 `count` 求汇总

具体做法是在 `mapReduce` 中

- map 将记录中的 `created` 转换为 `%Y-%m` 格式字符串作为 key, 将需要汇总的字段加入 `value` 中
- reduce 对 `value` 进行累加即可
- 详细信息请参考 MongoDB 官方 mapReduce 示例

```javascript
var mapFunction = function() {
    // 将时间戳格式化为 %y-%m 格式字符串
    var d = new Date(this.created);
    var month = "" + (d.getMonth() + 1);
    var pad = "00";
    var d_str = d.getFullYear() + '-' + pad.substring(0, pad.length - month.length) + month;
    var key = {
        'group': this.group,
        'ym': d_str
    };
    var value = this.count;
    emit(key, value);
};
var reduceFunction = function(key, val) {
    var reduceVal = 0;
    for (var idx = 0; idx < val.length; idx++) {
        var item = val[idx];
        reduceVal += item;
    }
    return reduceVal;
};
db.sample.mapReduce(
    mapFunction,
    reduceFunction, {
        out: "sample_stats",
    }
);
```

在 `mongo` 中u运行次代码后，即可通过 `db.sample_stats.find()` 查看聚合结果

```javascript
/* 0 */
{
    "_id" : {
        "group" : "jd",
        "ym" : "2010-10"
    },
    "value" : 40
}

/* 1 */
{
    "_id" : {
        "group" : "jd",
        "ym" : "2010-11"
    },
    "value" : 445
}
```

## 在 pandas 做转置/输出操作

```python
from pymongo import MongoClient
import pandas as pd

client = MongoClient()
db = client['sample']
collection = db.sample_stats
```

将 mongo 中的每个 Document 组成 list(dict), 并生成 `pd.DataFrame` 对象(一张内存表, 详见 pandas.DataFrame 文档)

```python
rows = [
    dict(list(row['_id'].items()) + list({'count': row['value']}.items()))
    for row in collection.find()]
df = pd.DataFrame(rows)
```

查看 df 对象的前 10 行

```python
In [11]: df.head()
Out[11]: 
   count  group       ym
0     40     jd  2010-10
1    445     jd  2010-11
2    635     jd  2010-12
3   1177     jd  2011-01
4    850     jd  2011-02
```

最后将数据输出为 Excel, 这里要用到 `pivot` 操作，自定义转置后的 `index`/`columns`/`values` 即可

这里用月份作为列头，`group` 作为表格列, 填充 `count` 值

- 可以通过 `fillna` 来填充缺失值
- 用 `pandas` 强大的索引方式选择你要的时间范围 `df[:'2014-12']`
- 通过 `to_excel` 导出数据，注意 `encoding=GB18030`

```python
df2 = df.pivot(index='ym', columns='group', values='count')
df2 = df2.fillna(0)[:'2014-12']
df2.to_excel('sample_stats.xlsx', encoding='GB18030')
```

## 相关文档

- [MongoDB.mapReduce](http://docs.mongodb.org/manual/tutorial/map-reduce-examples/)
- [pandas](http://pandas.pydata.org/)
- [pandas.DataFrame](http://docs.mongodb.org/manual/tutorial/map-reduce-examples/)
- [pandas.DataFrame.pivot](http://pandas.pydata.org/pandas-docs/stable/reshaping.html)
