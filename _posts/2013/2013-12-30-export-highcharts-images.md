---
layout: post
title: "Export highcharts images"
description: "pythhon 生成 highcharts 图表 png"
category: python
tags: [python, highcharts, export]
---
{% include JB/setup %}

需求: 后台生成 highcharts 图表图片

## 步骤

1. 到 [highcharts](http://highcharts.com) 下载源码包，文件夹 `exporting_server` 提供了下载导出脚本
2. 有三种方法: java/phantomjs/php，
3. 先看 php, 最终执行的是 `java -jar batik-rasterizer.jar -m image/png -d chart.png -w 300 temp/rand.svg`

   根据 Google `batik-rasterizer` 的结果来看，这个 jar 只提供了 `svg->images`，但是 svg 生成比较麻烦，我们可以用 json 文件直接生成
4. phantomjs 中有 README.md，信息很多，经过安装 phantomjs + 几轮测试后发现可行，唯一需要补充的地方是 `highcharts-convert.js`，里面提到需要 `highstock.js` 但是下载包中并未提供
5. 就用 phantomjs 了，只要搞定它就可以
6. 看看 [phantomjs](http://phantomjs.org/) 是什么/能做什么

## 安装依赖

1. [官方文档](http://phantomjs.org/build.html) 安装 phantomjs

       sudo apt-get update
       sudo apt-get install build-essential chrpath git-core libssl-dev libfontconfig1-dev
       git clone git://github.com/ariya/phantomjs.git
       cd phantomjs
       git checkout 1.9
       ./build.sh

   安装完成后将 bin/phantomjs copy 到任何地方可用
2. 将 phantomjs copy 到 exportint-sever/phantomjs 目录，这里有大部分的依赖文件
3. 下载 `highstock.js` 文件到 phantomjs 目录

## 导出

满足依赖了，从 `exporting-server/phantomjs/README.md` 可以看到详细使用方法，最后的执行命令为

    ./phantomjs highcharts-convert.js -infile options.json -outfile chart.png -scale 2.5 -width 600 -constr Chart -callback callback.js

`option.json` 的模板是这样的

    {
    xAxis: {
        categories: ['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec']
    },
    series: [{
        data: [29.9, 71.5, 106.4, 129.2, 144.0, 176.0, 135.6, 148.5, 216.4, 194.1, 95.6, 54.4]
    }]
    };

## 文件结构

    callback.js
    chart.png
    data.js
    highcharts-convert.js
    highcharts.js
    highcharts-more.js
    highstock.js
    jquery.1.9.1.min.js
    options1.json
    options.json
    phantomjs*
    README.md
    tmp/
