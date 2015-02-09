---
layout: post
title: "Android App Network Sniffer Android 应用流量嗅探"
category: [Hack]
tags: [Hack, Android, Sniffer]
---
{% include JB/setup %}

> 嗅探 app 的流量能用来干嘛我想大伙都懂得, 这篇文章暂时不外传, 只用来记录过程
>
> 技术版权不属于我, 来源于同事分享

需要准备的东西

- android 模拟器 genymotion
- 中间人代理 mitmproxy

## 环境初始化

- [env](#env)
- [genymotion](#genymotion)
- [mitmproxy](#mitmproxy)
- [setup proxy](#setup-proxy)
- [set proxy for android](#set-proxy-for-android)

### env

- Ubuntu 12.04 X64
- Python2.7 (mitmproxy 只支持 py2)

### genymotion

下载 genymotion 后得到一个 `bin` 文件

```sh
chomod +x genymotion-2.3.1_x64.bin
sudo ./genymotion-2.3.1_x64.bin -d /opt
# start genymotion
./opt/genymotion/
```

然后下载安装一个系统

### mitmproxy

这个库有比较多的系统库依赖

```sh
sudo apt-get install build-essential autoconf libtool pkg-config python-opengl python-imaging python-pyrex python-pyside.qtopengl idle-python2.7 qt4-dev-tools qt4-designer libqtgui4 libqtcore4 libqt4-xml libqt4-test libqt4-script libqt4-network libqt4-dbus python-qt4 python-qt4-gl libgle3 python-dev libffi-dev libssl-dev libssl-doc
virtualenv -p /usr/bin/python2 .
bin/pip install netlib -i http://pypi.douban.com/simple
bin/pip install mitmproxy -i http://pypi.douban.com/simple
```

### setup proxy

```
mitmproxy -p 7766
```

### set proxy for android

- 进入 setting
- 进入 Wi-Fi
- 长按 WiredSSID -- Modify Network
- 修改代理地址/端口为 你的 IP/7766

## 嗅探

通过浏览器下载安装你要嗅探的 android 程序之后, 打开它做一些网络操作, 即可在 `mitmproxy` 的界面中看到一些 HTTP 记录, `Tab` 切换 `Request`/`Response`, 自带 `filter`

后面需要用嗅探做东西的时候再来一篇

## apk 文件的拆分工具

- apktools
- androguard
- dex2jar
