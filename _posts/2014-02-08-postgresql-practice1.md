---
layout: post
title: "PostgreSQL Practice(1)"
description: "PostgreSQL 使用"
category: postgresql
tags: [python, postgresql]
---
{% include JB/setup %}

1. 需求

   用官方 API 保存个人微博的 `timeline` 到数据库，鉴于返回数据是 `json`，用支持 `json` 的 PG 是最方便的
2. 环境

   Ubuntu 12.04.3 LTS
3. 安装 PG 9.3

   Google `postgresql 9.3 ppa` 可以看到官方 wiki 有详细说明

   {% highlight sh linenos %}
   -- 添加 apt 源
   sudo add-apt-repository ppa:pitti/postgresql
   sudo apt-get update
   sudo apt-get upgrade
   sudo apt-get install postgresql-9.3 pgadmin3
   sudo apt-get install postgresql-server-dev-9.3

   -- 其他信息
   -- deb http://apt.postgresql.org/pub/repos/apt/ precise-pgdg main
   -- 单独添加签名
   wget --quiet -O - https://www.postgresql.org/media/keys/ACCC4CF8.asc | sudo apt-key add -
   {% endhighlight %}
4. SUPER 用户 管理

   通过 APT 安装后，默认用户是 `postgres`

   {% highlight sh linenos %}
   -- 切换到用户 postgres
   sudo su - postgres

   -- 添加数据库
   createdb sample

   --  用 SUPER 角色登录
   psql
   {% endhighlight %}
5. 数据库与用户权限

   {% highlight sql linenos %}
   -- 创建新用户 授予数据库权限
   create user yoyo with password 'ppass';
   grant all privileges on database sample to yoyo;
   {% endhighlight %}
6. 表、RULE

   初期只需要维护一个只读，主键唯一的表

   {% highlight sql linenos %}
   -- 建表
   create table weibo_user (
       id bigserial PRIMARY KEY,
       screen_name varchar(30),
       name varchar(30),
       user_json json
   );
   {% endhighlight %}
   在有主键约束的情况下，如果在不检查是否有重复的情况下需要忽略主键冲突的行，在 `MySQL` 可以通过 `insert ignore into` 但是 `pg` 不支持，但是可以通过为表新建一个 rule 来忽略冲突行

   {% highlight sql linenos %}
   drop rule if exists "rule_insertignore_weibo_user" on "weibo_user";
   create rule "rule_insertignore_weibo_user" as on INSERT TO "weibo_user"
       where exists(select 1 from weibo_user
           where uid=NEW.uid and id=NEW.id)
       do instead nothing;
   {% endhighlight %}
7. 好了，用 `pg` 保存一段时间的数据再对其处理
