---
layout: post
title: "读 我为什么从python转向go"
description: "文章摘抄"
category: [Python, Excerpt, Go]
tags: [Python, Excerpt, Go]
---
{% include JB/setup %}

> [我为什么从python转向go](http://www.jianshu.com/p/afa14e631930)

## 为什么放弃 Python

### 动态语言

> 出现 int + string 这样的运行时错误

> 允许同名函数的存在

在静态语言中，这些都可以在编译的时候发现

### 性能

> GIL 导致无法充分多核资源，进程间的通讯开销需要考虑

> 无状态的分布式处理使用多进程很方面，譬如处理 Http 请求，我们在 nginx 后段挂载了 200 多个 django server 处理 HTTP 请求，但是这么多进程导致机器整体负载偏高
> 
> 一些超大量请求，Python 仍然处理不过来，虽然可以使用 openresty(高频次请求用 Lua 实现)，但是这样导致需要用两种开发语言写两份不同的代码

### 同步网络模型

django 的同步阻塞，只能通过多开 django 进程来解决

### 异步网络模型

tornado 网络模型是异步的，但是 Mysql 驱动都不支持异步

### 开发运维部署

python 项目在 docker 以前的部署非常麻烦，相对来说静态编译语言只需要扔一个二进制文件，非常方面

### 代码失控

> Python 灵活简单，太简单导致很多同学无法对代码进行深层次的思考、对构架进行细致的考量，新的需求可以快速实现，导致项目代码失控，当然也是因为我们没有很好的 review 机制/项目规范
> 
> 一个程序员没有经过良好的编程训练，很容易写出烂的代码，因为太自由了
> 
> 并不是说无法用 Python 做大项目的开发，豆瓣/Dropbox 都做到了，但是我们的项目失控了


随着项目复杂度的增大，流量压力的增大，Python 并不是一个很好的选择

## 为什么选择 go

- 静态语言 强类型
- gofmt 大家的代码风格都一样
- 天生的并行支持 因为 goroutine 以及 channel
- 性能 单机部署了一个 go 的进程，就干了以前 200 个 Python 进程干的事情，CPU/MEM 占用更低
- 运维部署 直接部署二进制文件
- 开发效率 堪比 Python

值得吐槽的地方

- error 约定的最后一个返回值是 error, [error-are-values](https://blog.golang.org/errors-are-values) 是一个不错的解决方案
- 包管理
- GC 该用对象池/内存池的时候一定要用，虽然代码丑，但是性能上去了
- 泛型

## 实际项目中的一些 Go Tips

- godep 包管理
- IO deadline 没懂
- GC
- Go gob

