---
layout: post
title: "[译] parsing raw data 怎样优雅的清理原始数据"
description: "文章读后总结"
category: [data, translation]
tags: [data, translation]
---
{% include JB/setup %}

原文: [Parsing Raw Data](http://pgbovine.net/parsing-raw-data.htm)

清理数据是一件随时可以开始做的事情，但是走过的坑多了之后还是可以总结一些方法的

## 用 assert

用 `assert` 判断每行/每个字段是否是你想要的格式, 作者说 `ASSERT ALL THE THINGS`, 直到全部通过

这里你写出了第一阶段的 `filter` 让它通过 `assert` 满足基本处理要求

作者说, assert 可能会让程序崩溃很多次, 但是一旦 `assert` 通过, 这就是你想要的数据了

我的经历是, `assert` 通过后可以减少很多 `datetime/int` 等转换错误

## 不要悄悄的过滤掉任何一条记录

- 每条被过滤掉的数据都需要有 `log.warning`
- 带上 `counter` 这样可以回头检查数据
- 如果过滤了 `35%` 的数据 那就要小心了

## 用 `counter` 记录可枚举值

- 用 `set` / `counter` 记录可枚举值
- 出现新的枚举值后 用 `log.wanring` 打印出来
- 根据新出现的值完善 `assert`
- 最后打印出这个 `set` / `counter` 看是否有异常值

## 让程序可以从 `dataset` 的中间开始

每次从头开始是个很愚蠢的方法，费时费神

- 打印出当前正在处理的行内容
- 让程序可以接收一个 `lineno` 参数, 程序从 `lineno` 处开始处理
- 所有中间处理完成后，重新跑一次，防止新的 `assert` 无法通过之前已经通过的数据

## 现在小的数据集上测试

拿样本下手, 样本 OK 后再跑所有数据

## 将 `STDOUT` `STDERR` 输出到文件

同样为了追溯出错原因, 同时 `STDOUT`/`STDERR` 应该分开

## 将 `Raw Data` 和 `Cleaned Data` 分开保存

OF COUSE

## 再检查一次

这部分数据就是你将要开始分析的数据, 如果除了任何差错, 将会影响到 `data analysis` 的判断


## 代码例子

```py
"""
simple parsing raw data functions
"""
import re
import logging
import sys
from collections import Counter, defaultdict

log_stdout = None
log_stderr = None
cols_counter = defaultdict(Counter)
COLUMNS = ['A', 'B', 'C', 'D']


def setup_log():
    """
    setup STDOUT/STDERR to different files
    """
    global log_stdout, log_stderr
    formatter = logging.Formatter(
        '%(asctime)-15s - %(name)s - %(levelname)s - %(message)s')

    log_stdout = logging.getLogger('STDOUT')
    log_stdout.setLevel(logging.INFO)
    handler = logging.FileHandler('20150129_std.log', mode='w')
    handler.setFormatter(formatter)
    log_stdout.addHandler(handler)

    log_stderr = logging.getLogger('STDERR')
    log_stderr.setLevel(logging.INFO)
    handler = logging.FileHandler('20150129_err.log', mode='w')
    handler.setFormatter(formatter)
    log_stderr.addHandler(handler)


def parse_args():
    sys.args.extend([None] * 3)
    lineno = sys.args[1]
    return {
        'lineno': lineno,
    }


def yield_lines(start=0, stop=None):
    """
    @param start : start lineno
    @param stop : stop lineno
    """
    with open('......') as f:
        for i, line in enumerate(f.read()):
            if i >= start:
                yield i, line
            if stop is not None and stop >= i:
                break


def process_line(i, line):
    global cols_counter
    cols = line.split('\t')

    # filter here
    if cols[0] == '-1':
        log_stderr.wanring('LINENO %s SKIP cols[0] can not be -1', i)

    # counter here
    if cols[2].lower() not in cols_counter['C']:
        log_stderr.wanring('LINENO %s C new value %s', i, cols[2])
    cols_counter['C'][cols[2].lower()] += 1

    # assert here
    assert cols[0].isdigit()
    assert len(cols[1]) == 3
    assert cols[2].lower() in ['A', 'B', 'O', 'AB']
    assert re.match(r'\d{4}-\d{2}-\d{2}', cols[3])


def save_line(line):
    pass


def main():
    """
    parsing data set from LINENO
    """
    kwargs = parse_args()
    for i, line in yield_lines(start=kwargs.pop('lineno', 0)):
        log_stdout.info('process line %s %s', i, line)
        process_line(i, line)
        save_line(line)


def main_for_sample():
    """
    parsing sample data
    """
    for i, line in yield_lines(stop=100):
        log_stdout.info('process line %s %s', i, line)
        process_line(i, line)
        save_line(line)


if __name__ == '__main__':
    setup_log()
    main()

```
