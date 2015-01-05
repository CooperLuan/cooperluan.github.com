---
layout: post
title: "play with pubnub"
description: ""
category: python
tags: [python, pubnub, pubsub]
---
{% include JB/setup %}

浏览 Python Weekly 文章时看到有一个消息订阅/消费服务商 [PubNub](http://www.pubnub.com/), 并且提供一个不存储历史数据的免费版本, 试用看一下这个服务怎么样

## 服务简介

- 提供终身免费版本
- 无消息存储服务
- 每天限量 1 百万条
- [收费] 查看订阅用户的订阅相关数据
- [收费] 存储历史数据
- [收费] 流控制 在单连接中管理上千个频道
- [收费] 不同设备间的数据同步
- [收费] 手机消息推送
- [收费] 实时分析
- [收费] 访问控制

## 怎么用

注册免费账户获取 `Publish Key`/`Subscribe Key`

### SDK

官方提供许多主流语言 sdk, 我找了一个 [python SDK](https://github.com/pubnub/python) 但是发现它在 `Pubsub.py` 文件中莫名其妙的加了 `python **` 代码，所以 fork 了一份到 [我的 Github](https://github.com/CooperLuan/pubnub)

可以通过

{% highlight sh %}
pip install git+git://github.com/CooperLuan/pubnub.git
{% endhighlight %}

来安装

### 使用

完整的 demo 代码在 [我的 gist](https://gist.github.com/CooperLuan/21490fdfb706d5e7fdb1) 可以看到

- 包引用
   {% highlight python %}
   from Pubnub import Pubnub
   {% endhighlight %}
- 初始化
   {% highlight python %}
   PUB_KEY = 'YOUR PUB KEY'
SUB_KEY = 'YOUR SUB KEY'
pubnub = Pubnub(publish_key=PUB_KEY, subscribe_key=SUB_KEY)
   {% endhighlight %}
- 测试
   {% highlight python %}
def _time_callback(response):
   print('time callback', response)
pubnub.time(callback=_time_callback)
# time callback [14204378053277459]
   {% endhighlight %}
   服务器会返回一个时间戳

- 订阅
   {% highlight python %}
import threading
def _sub_callback(message, channel):
    print('sub callback', channel, message)


def _sub_error(message):
    print('sub error', message)

threading.Thread(
    target=pubnub.subscribe,
    kwargs=dict(channels="demo", callback=_sub_callback, error=_sub_error),
).start()
   {% endhighlight %}
- 发布
   {% highlight python %}
def _pub_callback(message):
    print(message)
channel = 'demo'
for c in ['purple', 'white', 'black', 'yellow', 'green']:
    message = {'color': c}
    pubnub.publish(
        channel, message, callback=_pub_callback, error=_pub_callback)

   {% endhighlight %}
- 历史
    {% highlight python %}
def _his_callback(*args, **kwargs):
    print('his callback', args, kwargs)


def _his_error(*args, **kwargs):
    print('his err', args, kwargs)

pubnub.history(
    channel="demo", count=100, callback=_his_callback, error=_his_error)

    {% endhighlight %}
- 输出
   {% highlight python %}
[1, 'Sent', '14204378053584782']
[1, 'Sent', '14204378053603093']
[1, 'Sent', '14204378053639586']
[1, 'Sent', '14204378053777784']
sub callback demo {'color': 'white'}
sub callback demo {'color': 'purple'}
sub callback demo {'color': 'yellow'}
sub callback demo {'color': 'green'}
his callback ([['Storage is not enabled for this subscribe key. Please contact help@pubnub.com'], 0, 0],)
{}
[1, 'Sent', '14204378062380378']
sub callback demo {'color': 'black'}
   {% endhighlight %}

## 小结

- 从注册到最终实现非常简单
- sdk 不适用 python3 在 python3 普及率比较高的今天不知为何还会出现这种情况
- pub/sub 的延迟也不稳定/略高 从 0.1 - 1s 都有
