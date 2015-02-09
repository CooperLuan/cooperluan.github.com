---
layout: post
title: "[译] Docker For Pythonistas 写给 Pythonistas 的 Docker 指南"
description: ""
category: [cloud technologies]
tags: [cloud technologies, docker]
---
{% include JB/setup %}

原文 [Docker For Pythonistas](http://kalnitsky.org/2015/02/03/docker-for-pythonistas/)

> 译者注: 原文作者从方便 Pythoner 测试的角度入手 Docker, 开发了一个多版本 python 的 image 供下载使用
> 
> tutorial 类型的文章以后不翻译了, 直接看原文并上手的成本很低

我从一年前开始熟悉 docker, 那时候我们正打算用 docker 做 [Fuel for OpenStack](https://wiki.openstack.org/wiki/Fuel) 的升级.
从那时起我学到了关于 Docker 的很多东西: 瓶颈/使用陷阱/bugs, 我甚至知道怎样在 CentOS 6.5 上用老 Linux 内核跑最新的 1.4.1 版本的 docker

但在这里我不打算写 docker 的使用案例/机制/瓶颈, 但是我会关注它的日常工作流程/我为什么觉得它有用/为什么它对大多数 Pythonistas 来说是有用的

开始前先问两个问题

- 开发者最常见的工作是什么
- 开发者第二常见的工作是什么

很显然, 第一个问题的答案是写代码, 第二个答案是跑测试确保整个系统没有 bug, 但是这个是有点难度的.
许多 Python 开发者在同时使用多个版本的 python 编译器, py2 的多个版本或者 Py2/Py3 混合,
所以需要在多个编译器上测试通过, 但是如果你的 Linux 发行版没有其中某一个版本呢?

这里就需要 docker

Docker 是一个创建/传输/运行容器的平台, 换句话说它允许你事先准备一些软件镜像, 然后 push 到 `docker registry`, 随时可以 pull 下来在任何地方运行

当时我想在我的机器上用 python2.6 测试但是发现 Debian Jessie 上没有这个版本, 
于是我花了些时间写了一个 [pythonistas docker image](https://github.com/ikalnitsky/pythonista), 这个 image 中包含了所有流行的 python 解释器版本,
任何人都可以用这个 image 并用它来做测试, 你可以自由修改 image -- 这个并不难

只需要 1 条命令就可以在 container 中跑测试, 不需要任何配置

```bash
sudo docker pull ikalnitsky/pythonista
sudo docker run -v /path/to/src/:/src -w /src ikalnitsky/pythonista tox
```

第一条命令用来下载 docker image, 第二条运行 container

这样可以节省时间/精力, 因为你不需要在任何机器上编译多个版本的 Python 解释器了
