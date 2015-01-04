---
layout: post
title: "PGDoc Keyboard Guide"
description: "RT"
category: TamperMonkey
tags: [JS, monkey, TamperMonkey]
---
{% include JB/setup %}

用 TamperMonkey 的插件支持 PG 文档的 left/right 翻页

{% highlight javascript linenos %}
// ==UserScript==
// @name       PGDoc Keyword Guide
// @namespace  http://weibo.com/otcooper
// @version    0.1
// @description  enter something useful
// @match      http://www.postgresql.org/docs/*.html
// @noframes
// @run-at document-end
// @copyright  2014+, Cooper
// ==/UserScript==

// 加载 jQuery / keymaster 两个 JS 文件
var srcs = [
    'http://cdn.staticfile.org/jquery/2.1.0/jquery.min.js',
    'http://cdn.staticfile.org/keymaster/1.6.1/keymaster.min.js'
];
for (var i = srcs.length - 1; i >= 0; i--) {
    var tag = document.createElement("script");
    tag.type = "text/javascript";
    tag.src = srcs[i];
    document.body.appendChild(tag);
};

// 轮循直到 JS 对象加载完成
var intv_check = setInterval(check_load, 250);

function check_load() {
    console.log('check js loading');
    if(typeof jQuery !== 'undefined' && typeof key !== undefined) {
        console.log('loading finished');
        bind();
        clearInterval(intv_check);
    }
}

function bind() {
    key('left', function() {
        console.log('preview page');
        $("a[accesskey|='P']")[0].click();
    });
    key('right', function() {
        console.log('next page');
        $("a[accesskey|='N']")[0].click();
    });
}
{% endhighlight %}
