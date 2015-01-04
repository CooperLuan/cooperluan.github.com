---
layout: post
title: "lxml + XPath 处理 HTML 实践"
description: "XPath 基本语法 + lxml.etree.xpath 处理 HTML 实例"
category: python
tags: [python, XPath, lxml]
---
{% include JB/setup %}

用 python 处理 HTML 文本时，re/pyquery 的可维护性都不高，尝试用 XPath

文档来源 [XPath 教程](http://www.w3school.com.cn/xpath/index.asp)

### XPath 路径表达式

1. `nodename` 次节点的所有子节点
2. `/` 从根节点选取
3. `//` 从所有叶子节点选取
4. `.` 选取当前节点
5. `..` 选取当前节点父节点
6. `@` 选取属性
7. `*` 任何元素节点
8. `@*` 任何属性节点
9. `node()` 匹配任何类型的节点
10. `|` 选取多个路径
11. `::` 步 `轴名称::节点测试[谓语]`

### XPath 实例

1. `bookstore` bookstore 元素的所有子节点
2. `/bookstore` 选择根元素 bookstore
3. `//book` 选择所有 book 叶子节点
4. `bookstore//book` bookstore 节点下的所有 book 叶子节点
5. `//@lang` 名为 lang 的所有属性
6. `/bookstore/book[1]`   选取属于 bookstore 子元素的第一个 book 元素
7. `/bookstore/book[last()]` 选取属于 bookstore 子元素的最后一个 book 元素
8. `/bookstore/book[last()-1]`   选取属于 bookstore 子元素的倒数第二个 book 元素
9. `/bookstore/book[position()<3]`   选取最前面的两个属于 bookstore 元素的子元素的 book 元素
10. `//title[@lang]`  选取所有拥有名为 lang 的属性的 title 元素
11. `//title[@lang='eng']`    选取所有 title 元素，且这些元素拥有值为 eng 的 lang 属性
12. `/bookstore/book[price>35.00]`    选取 bookstore 元素的所有 book 元素，且其中的 price 元素的值须大于 35.00
13. `/bookstore/book[price>35.00]/title`  选取 bookstore 元素中的 book 元素的所有 title 元素，且其中的 price 元素的值须大于 35.00
14. `/bookstore/*` bookstore 的所有子元素
15. `//*` 所有元素
16. `//title[@*]` 带有 title 属性的所有元素
17. `//title | //price` 所有 title / price 节点
18. `child::book` 属于当前节点子元素的 book 节点
19. `attribute::lang` 选取当前节点的 lang 属性
20. `child::*` 选取当前节点所有子元素
21. `attribute::*` 当前节点的所有属性
22. `child::text()` 当前节点的所有文本子节点
23. `child::node()` 当前节点的所有子节点
24. `descendant::book` 当前节点的所有 book 后代
25. `ancestor::book` 当前节点的所有 book 父节点
26. `ancestor-or-self::book` 当前节点的所有 book 父节点 + 当前节点
27. `child::*/child::price` 当前节点的所有子节点的 price 子节点(第二代子节点)

### XPath 运算符

1. `|` 两个节点集
2. `+` `-` `*` `div` 加法 减法 乘法 除法
3. `=` `!=` `<` `<=` `>` `>=` `or` `and` `mod` mod 计算余数

### 函数

[XPath、XQuery 以及 XSLT 函数](http://www.w3school.com.cn/xpath/xpath_functions.asp)

XPath 含有超过 100 个内建的函数。这些函数用于字符串值、数值、日期和时间比较、节点和 QName 处理、序列处理、逻辑值等等

### 简单 lxml + XPath 处理 HTML 代码

{% highlight python linenos %}
# encoding: utf-8
from StringIO import StringIO

from lxml import etree

broken_html = """<html>
<head>
    <title>Tank</title>
</head>
<body>
<header>
    <ul>
        <li>Home</li>
        <li>About</li>
    </ul>
</header>
<div class="container">
    <div class="row">
        <div class="col-md-6">
            <table>
                <thead>
                    <th>ID</th>
                    <th>Name</th>
                </thead>
                <tbody>
                    <tr><td>1</td><td>Tank</td></tr>
                    <tr><td>2</td><td>Caption</td></tr>
                    <tr><td>3</td><td>Coco</td></tr>
                </tbody>
            </table>
        </div>
        <div class="col-md-6"></div>
    </div>
</div>
<footer>Powered by Cooper</footer>
</body>
</html>
"""

parser = etree.HTMLParser()
tree = etree.parse(StringIO(broken_html), parser)


def t_xpath():
    x_exp = "//div[@class='col-md-6']"
    print 'xpath', x_exp
    for foot in tree.xpath(x_exp):
        print foot.tag, foot.getparent().attrib

    x_exp = '//tr/td[2]'
    print 'xpath', x_exp
    for td in tree.xpath(x_exp):
        print td.tag, td.text

    x_exp = '//tr/td[1][text()>1]'
    print 'xpath', x_exp
    for td in tree.xpath(x_exp):
        print td.tag, td.text

    x_exp = '//div[@class]'
    print 'xpath', x_exp
    for td in tree.xpath(x_exp):
        print td.tag, td.attrib


def pretty_print():
    result = etree.tostring(
        tree.getroot(),
        pretty_print=True,
        method="html")
    print result

    html = etree.HTML(broken_html)
    print etree.tostring(html, pretty_print=True, method='html')


# pretty_print()
t_xpath()
{% endhighlight %}

输出

{% highlight json %}
xpath //div[@class='col-md-6']
div {'class': 'row'}
div {'class': 'row'}
xpath //tr/td[2]
td Tank
td Caption
td Coco
xpath //tr/td[1][text()>1]
td 2
td 3
xpath //div[@class]
div {'class': 'container'}
div {'class': 'row'}
div {'class': 'col-md-6'}
div {'class': 'col-md-6'}
{% endhighlight %}

### 未完待续
