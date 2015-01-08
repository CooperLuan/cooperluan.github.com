---
layout: post
title: "shaping up with angular js 笔记"
description: ""
category: AngularJS
tags: [AngularJS, javascript, 笔记]
---
{% include JB/setup %}

此笔记来自 [codeschool 教材 ](http://campus.codeschool.com/courses/shaping-up-with-angular-js/contents)

教材中有些代码无效, 故这里只做笔记加深记忆

## action 绑定

定义 `ng-controller`/`ng-app`

```html
<div ng-controller="StoreController">
    ...
</div>
```

然后在 `javascript` 中定义 scope

```javascript
function StoreController() {
    alert("How are you");
}
```

## 绑定 `ng-app` 后定义 module 和 controller

```javascript
(function() {
    var app = angular.module('store', []);
    app.controller('StoreController', function() {
        this.product = {
            name: 'Dodecahedron',
            price: 2.95,
            description: '. . .',
        };
    })
})()
```

然后在 html 中绑定数据

```html
<div ng-controller="StoreController as store">
    <h1>{{ store.product.name }}</h1>
    <h2>{{ store.product.price }}</h2>
    <p>{{ store.product.description }}</p>
</div>
```

## angular 中的交互部分

用 `ng-show`/`ng-hide` 决定是否显示此 tag

```html
<div ng-controller="StoreController as store">
    <div ng-hide="store.product.soldOut">
        <h1>{{ store.product.name }}</h1>
        <h2>{{ store.product.price }}</h2>
        <p>{{ store.product.description }}</p>
        <!-- 根据 canPurchase 决定是否显示 Buy 按钮 -->
        <button ng-show="store.product.canPurchase">Buy</button>
    </div>
</div>
<script type="text/javascript">
(function() {
    var app = angular.module('store', []);
    app.controller('StoreController', function() {
        this.product = {
            name: 'Dodecahedron',
            price: 2.95,
            description: '. . .',
            canPurchase: true,
            soldOut: false,
        };
    })
})()
</script>
```

## angular 中的多组数据显示

> `ng-repeat`

```html
<div ng-controller="StoreController as store">
    <div ng-repeat="product in store.products">
        <h1>{{ product.name }}</h1>
        <h2>{{ product.price }}</h2>
        <p>{{ product.description }}</p>
        <!-- 根据 canPurchase 决定是否显示 Buy 按钮 -->
        <button ng-show="product.canPurchase">Buy</button>
    </div>
</div>
<script type="text/javascript">
(function() {
    var app = angular.module('store', []);
    app.controller('StoreController', function() {
        this.products = [{
            name: 'Dodecahedron',
            price: 2.95,
            description: '. . .',
            canPurchase: true,
            soldOut: false,
        }, {
            name: 'Channel',
            price: 37,
            description: '. . .',
            canPurchase: true,
            soldOut: true,
        }];
    })
})()
</script>
```

## angular 中的 filter

```html
<h2>{{ product.price | currency }}</h2>
<span>{{ '1388123412323' | date:'MM/dd/yyyy' }}</span>
<span>{{'octagon gem' | uppercase }}</span>
<span>{{ product.description | limitTo:10 }}</span>
<div ng-repeat="{{ product in store.products | limitTo:3 }}"></div>
<div ng-repeat="{{ product in store.products | orderBy:'-price' }}"></div>
```

## angular 中显示图片

```html
<img ng-src="{{ product.images[0].full }}">
<script type="text/javascript">
(function() {
    var app = angular.module('store', []);
    app.controller('StoreController', function() {
        this.product = {
            name: 'Dodecahedron',
            price: 2.95,
            description: '. . .',
            canPurchase: true,
            soldOut: false,
            images: [{
                full: '*****.jpg',
                thumb: '*****-thumb.jpg',
            }]
        };
    })
})()
</script>
```


## angular 控制 tab 显示动作

- 通过 `ng-init` 设置初始值 `<section ng-init="tab = 1">`
- 在 `li` 中通过 `ng-click` 赋值给 tab `<a ng-click="tab = 1">`
- 在 panel 中通过 `ng-show` 决定是否显示 `<div ng-show="tab === 1">`
- 动态改变 panel 属性 `<div ng-class="{ active:tab === 1 }">`

```html
<section ng-init="tab = 1">
 <ul class="nav nav-pills">
 <li> <a href ng-click="tab = 1" ng-class="{ active:tab === 1 }">Description</a> </li>
 <li> <a href ng-click="tab = 2">Specifications</a> </li>
 <li> <a href ng-click="tab = 3">Reviews</a> </li>
 </ul>
</section>

<div class="panel" ng-show="tab === 1">
    Hi there
</div>
<div class="panel" ng-show="tab === 2">
    Lord of king
</div>
<div class="panel" ng-show="tab === 3">
    Glass
</div>
```

## 用 js 简化 html 中的属性赋值

```html
<section ng-controller="PanelController as panel">
    <ul class="nav nav-pills">
         <li> <a href ng-click="panel.selectTab(1)" ng-class="{ active:tab === 1 }">Description</a> </li>
         <li> <a href ng-click="panel.selectTab(2)">Specifications</a> </li>
         <li> <a href ng-click="panel.selectTab(3)">Reviews</a> </li>
    </ul>
    <div class="panel" ng-show="panel.isSelected(1)">
        Hi there
    </div>
    <div class="panel" ng-show="panel.isSelected(2)">
        Lord of king
    </div>
    <div class="panel" ng-show="panel.isSelected(3)">
        Glass
    </div>
</section>

<script type="text/javascript">
(function() {
    var app = angular.module('panel', []);
    app.controller('PanelController', function() {
        this.tab = 1;
        this.selectTab = function(i) {
            this.tab = i;
        }
        this.isSelected = function(i) {
            return this.tab === i;
        }
    })
})()
</script>
```

## 可交互的 form

```html
<form name="commitReview"
    ng-controller="ReviewController as reviewCtrl"
    ng-submit="commitReview.$valid && reviewCtrl.addReview"
    novalidate>
Name: <input name="name" ng-model="reviewCtrl.review.name" required /><br>
Email: <input name="email" ng-model="reviewCtrl.review.email" required /><br>
<input type="submit" value="Submit" />
</form>
<script type="text/javascript">
(function() {
    var app = angular.module('review', []);
    app.controller('ReviewController', function() {
        this.review = {};
        this.reviews = [];
        this.addReview = function() {
            this.reviews.push(this.review);
        }
    })
})()
</script>
```
