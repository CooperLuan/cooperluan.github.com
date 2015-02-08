---
layout: post
title: "[译] all this / JAVAScript/NodeJS 中 this 那些事儿"
description: "粗译 all this 一文"
category: [javascript, Translation]
tags: [javascript, Translation]
---
{% include JB/setup %}

原文 [all this](http://bjorn.tipling.com/all-this)

> 译者注: 我在这之前只写过管理后台类型的 JS 代码, 所以翻译这篇含金量很高的文章会有一些问题, 一是之前没想过会有 this 这种困扰, 二是作者用了很多长句子, 有的问题只好生硬直译或者按照我理解的意思换上我的表达, 建议粗看后去看原文

你可能认为 JavaScript 中的 `this` 和面向对象语言如 `JAVA` 中的 `this` 一样, 是对实例属性的引用.
事实不是这样的, JavaScript 中的 `this` 是一个潘多拉魔盒(译者注: 作者引用了哈利波特中黑魔法防御课卢平教授用来关恶灵的柜子..)

下面是我想让我的工作伙伴都了解的 JavaScript 中 `this` 的用法, 内容很丰富, 有些知识点我花了几年的时间才能学到

## global this

在浏览器的全局作用域中, `this === window`

```html
<script type="text/javascript">
console.log(this === window); // true
</script>
```

在浏览器中, 用全局作用域中用 var 和给 `this`/`window` 赋值是一样的

```html
<script type="text/javascript">
var bar = "foo";
console.log(this.bar);      // foo
console.log(window.bar);    // foo
</script>
```

如果你定义变量时没有用 `var` 就相当于给 `this` 增加/改变 属性

```html
<script type="text/javascript">
foo  = "foo";
function testThis () {
    foo = "bar";
}
console.log(this.foo);  // foo
testThis();
console.log(this.foo);  // bar
</script>
```

在 node 的交互模式下, `this` 是顶级命名空间, `this === global`  
用 `var` 定义变量时, 会赋值给 `this`/`global`

```js
> this
{ ArrayBuffer: [Function: ArrayBuffer],
  Int8Array: { [Function: Int8Array] BYTES_PER_ELEMENT: 1 },
  Uint8Array: { [Function: Uint8Array] BYTES_PER_ELEMENT: 1 },
  Uint8ClampedArray: { [Function: Uint8ClampedArray] BYTES_PER_ELEMENT: 1 },
  Int16Array: { [Function: Int16Array] BYTES_PER_ELEMENT: 2 },
  Uint16Array: { [Function: Uint16Array] BYTES_PER_ELEMENT: 2 },
  Int32Array: { [Function: Int32Array] BYTES_PER_ELEMENT: 4 },
  Uint32Array: { [Function: Uint32Array] BYTES_PER_ELEMENT: 4 },
  Float32Array: { [Function: Float32Array] BYTES_PER_ELEMENT: 4 },
  Float64Array: { [Function: Float64Array] BYTES_PER_ELEMENT: 8 },
  DataView: [Function: DataView],
  global: [Circular],
  process: 
   { title: 'node',
     version: 'v0.10.29',
> this === global
true
> var foo = "bar";
> this.foo
bar
> global.foo
bar
```

在用 node 执行的脚本中, `this` 是一个空对象, 和 `global` 不一样  
用 `var` 赋值时, 不会像在浏览器中一样, 赋值给 `this`

```js
// test.js
console.log(this);
console.log(this === global);
var foo = "bar";
console.log(this.foo);
console.log(this.foo);
bar = "foo";
console.log(this.bar);
console.log(global.bar);
```

输出

```sh
$ node test.js
{}
false
undefined
undefined
undefined
foo
```

## function this

除了在 DOM 事件处理函数参数中提供的 `this` 参数外, 在 node 和浏览器的 `function` 中,
如果不是用 `new` 引用这个函数, `this === global`

```html
<script type="text/javascript">
foo = "bar";
function testThis() {
    this.foo = "foo";
}
console.log(this.foo);      // bar
testThis();
console.log(this.foo);      // foo

function testThis2() {
    this.foo = "f2f";
}
new testThis2();
console.log(this.foo);      // foo
console.log(new testThis2().foo)    // f2f     instance

function testThis3() {
    "use strict";
    this.foo = "foo3";
}
testThis3();        // Uncaught TypeError: Cannot set property 'foo' of undefined
console.log(new testThis3().foo);   // foo3
</script>
```

在 node 中的输出也一样

## prototype this

你创建的 function 是 `function object`, 创建后自动添加一个 `prototype` 属性, 可以用来赋值, 当你用 `new` 创建 `function object`
时, 你可以 通过 `this` 访问 `prototype` 属性中的值

```js
function Car() {
    console.log(this.foo);
}
Car.prototype.foo = "bar";
var car = new Car();        // bar
console.log(car.foo);       // bar
```

如果有多个 `function` 实例, 这些实例间共享 `prototype`, 可以重写实例属性

```js
function Car() {
}
Car.prototype.color = "B";
Car.prototype.logColor = function () {
    console.log(this.color);
}
Car.prototype.setColor = function (newColor) {
    this.color = newColor;
}

var car1 = new Car();
var car2 = new Car();
car1.logColor();        // B
car2.logColor();        // B

// share prototype in instances
Car.prototype.color = 'R';
car1.logColor();        // R
car2.logColor();        // R

// override individual instance property
car1.setColor("G");
car1.logColor();        // G
car2.logColor();        // R
```

实例中的 `this` 是一种特殊对象, 它实际上是一个关键词. 可以把它认为是访问 `prototype` 中属性的方式. 
如果直接给实例中的 `this` 赋值, 就访问不到 `prototype` 中的属性了, 删除赋值后可以恢复访问

```js
function Car() {
}
Car.prototype.color = 'B';
Car.prototype.colors = [];

var car1 = new Car();
var car2 = new Car();
car1.color = 'R';
console.log(car1.color);        // R
console.log(car2.color);        // B

car1.colors.push("O");
console.log(car1.colors);       // "[O]"
console.log(car2.colors);       // "[O]"

car2.colors.push("L");
console.log(car1.colors);       // ["O", "L"]
console.log(car2.colors);       // ["O", "L"]

car1.colors.pop("L");
console.log(car1.colors);       // ["O"]
console.log(car2.colors);       // ["O"]
```

不能在 `prototype` 上赋 array 类型的值, 如果你想让实例独享 array 类型的属性, 可以在 `function` 中创建这些属性

```js
function Car() {
    this.colors = [];
}
var car1 = new Car();
var car2 = new Car();

car1.colors.push("B");
console.log(car1.colors);       // ["B"]
console.log(car2.colors);       // []

car2.colors.push("G");
console.log(car1.colors);       // ["B"]
console.log(car2.colors);       // ["G"]
```

你可以把很多函数的 `prototype` 连起来组成 `prototype chain`, 这样用 `this` 引用属性时会沿着 `prototype chain` 

```js
function Car() {
  this.seats = 4;
}
Car.prototype.color = 'Blue';
function Driver() {
}
Driver.prototype = new Car();
var driver = new Driver();
console.log(driver.color, driver.seats);    // Blue 4
```

有人在 JavaScript 中用 `this` 模拟面向对象中的继承. 

> Any assignments to this in functions used to create the prototype chain will hide values defined further up the prototype chain.

不知怎么翻译, 但是从代码结果看, 对 `this` 的赋值符合 OO 的特性

```js
function Hard() {
  this.color = "Black";
}
function Car() {
  this.color = 'Orange';
}
Car.prototype = new Hard();
function Tank() {}
Tank.prototype = new Car();
function AirPlain() {}
AirPlain.prototype = new Hard();

console.log(new Tank().color);    // Orange
console.log(new AirPlain().color);  // Black
```

我把赋值到 `prototype` 的函数称作 `methods` 方法, 我在上面的例子中用了一些 `methods`(方法) 如 `logColor`.
这些 `methods`(方法) 和用来创建它的源函数一样, 有 `this` 这个魔法.

在 `prototype` 上定义的方法中引用的 `this` 指向当前实例

```js
function VW() {
  this.color = 'Black';
}
VW.prototype.logColor = function() {
  console.log(this.color);
}
function Volvo() {
  this.color = 'Slivery';
}
Volvo.prototype = new VW();

var car = new Volvo();
car.logColor();     // Slivery
```

在 JavaScript 中你可以嵌套函数, 也就是说可以在函数中定义函数, 当嵌套函数在闭包中引用在父函数中定义的变量时, 并不会继承 `this`

```js
var foo = "bar";
function Car() {}
Car.prototype.color = "Slivery";
Car.prototype.logColorInClosure = function() {
  var info = "Log color in closure";
  function logIt() {
    console.log(info, this.color);
    console.log(this.foor);
  }
  logIt();
}
var car  = new Car();
car.logColorInClosure();
// Log color in closure undefined
// bar
```

`logColorInClosure` 中的 `this` 是 `global` 中的对象(例子中的 `foo`), 在 `use strict` 情况下是 undefined. 这对于很多不熟悉 JavaScript 中 `this` 的人来说痛苦的根源.

有更糟糕的情况, 如果把实例方法当作一个函数参数, 方法中的 `this` 就恢复为对 `global` 的引用了, 在 `use strict` 的情况下是 `undefined`

```js
function Car() {
  this.color = 'Slivery';
  this.logColor = function() {
    console.log(this.color);
  }
}
function logColorInAnother(method) {
  method();
}
var car = new Car();
car.logColor();                 // Slivery
logColorInAnother(car.logColor);    // undefined
```

有的人倾向于用 `self` 捕捉 `this`, 避免同时用 `this`/`that`

```js
function Car() {}
Car.prototype.color = 'Slivery';
Car.prototype.logColor = function() {
  var self = this;
  function logColorInClosure() {
    console.log(self.color, this.color);
  }
  logColorInClosure();
}
var car = new Car();
car.logColor();         // Slivery undefined
```

但是如果你需要把方法当成值到处传递, 上面的方法就有问题了

```js
function Car() {}
Car.prototype.color = 'Slivery';
Car.prototype.logColor = function() {
  var self = this;
  function logColorInClosure() {
    console.log(self.color);
  }
  logColorInClosure();
}
var car = new Car();
car.logColor();           // Slivery

function doIt(method) {
  method();
}
doIt(car.logColor);       // undefined
```

你可以用函数的 `bind` 方法绑定实例后传递

```js
function Car() {}
Car.prototype.color = 'Slivery';
Car.prototype.logColor = function() {
  function logColorInClosure() {
    console.log(this.color);
  }
  logColorInClosure();
}
function doIt(method) {
  method();
}
var car = new Car();
doIt(car.logColor.bind(car));       // Slivery
```

也可以用 `apply`/`call` 立即调用函数或方法

```js
function Car() {}
Car.prototype.color = 'Slivery';
Car.prototype.logColor = function() {
  function logColorInClosure() {
    console.log(this.color);
  }
  logColorInClosure.apply(this);
  // logColorInClosure.call(this);
}
function doIt(method) {
  method();
}
var car = new Car();
doIt(car.logColor.bind(car));       // Slivery
```

对任何函数或方法, 你可以用 `bind` 代替 `this`, 即使它没有被赋值到 `prototype` 上

```js
function Car() {}
Car.prototype.color = 'Slivery';

function logColor() {
  console.log(this.color);
}
var car = new Car();
logColor.bind(car)();         // Slivery
logColor.apply(car);          // Slivery
logColor.call(car);           // Slivery
logColor();                   // undefined
```

如果你在 `function` 中返回了任何东西, 所有的魔法就都消失了

```js
function Car() {
  return {};
}
Car.prototype.color = "Slivery";
Car.prorotype.logColor = function() {
  console.log(this.color);
}
var car = new Car();
car.logColor();           // Uncaught TypeError: Cannot set property 'logColor' of undefined
```

奇怪的是, 如果你在 `function` 中返回一个字母/数字类型的值, 上面的现象就不会发生了, `return` 语句将被忽略. 永远不要在你要 `new` 的 `function` 中返回任何东西. 如果你想创建一个工厂模式, 用函数而不是 `new` 来创建实例, 当然这只是个建议.

你可以用 `Object.create` 代替 `new` 来创建对象

```js
function Car() {}
Car.prototype.color = 'Slivery';
Car.prototype.logColor = function() {
  console.log(this.color);
}
var car = Object.create(Car.prototype);
car.logColor();           // Slivery
```

但是这样的话就不会调用 `function` 中定义的过程了

```js
function Car() {
  this.color = "Black";
}
Car.prototype.color = 'Slivery';
Car.prototype.logColor = function() {
  console.log(this.color);
}
var car = Object.create(Car.prototype);
var car2 = new Car();
car.logColor();           // Slivery
car2.logColor();          // Black
```

因为 `Object.create` 不会调用 `function` 中定义的过程, 在创建继承模式的时候这个特性就有用了

```js
function Car() {
  this.color = "Black";
}
Car.prototype.color = 'Slivery';
Car.prototype.logColor = function() {
  console.log(this.color);
}

function Tank() {
  this.logColor(this.color);        // Slivery
  Car.apply(this);
  this.logColor(this.color);        // Black
}
Tank.prototype = Object.create(Car.prototype);
var tank = new Tank();
```

## object this

你可以在对象(object)上的任意函数里面使用 `this` 指向对象(object)上的其他属性, 这点和用 `new` 创建的实例不一样

```js
var obj = {
  foo: "bar",
  logFoo: function() {
    console.log(this.foo);
  }
}
obj.logFoo();         // bar
```

注意这里没有用 `new`/`Object.create` 也没有调用函数来创建 `obj`, 但是你也可以像函数实例一样 `bind` 到一个函数上

```js
var obj = {
  foo: "bar",
}
function logFoo() {
  console.log(this.foo);
}
logFoo.apply(obj);        // bar
logFoo.bind(obj)();       // bar
```

当你像下面这样用多级 `object` 时, `this` 无法回溯到上一层寻找它要的属性, 只有当它们的上一级 `object` 是同一个时 `this` 才有效

```js
var obj = {
  foo: "bar",
  logFoo: function() {
    function logIt() {
      console.log(this.foo);
    }
    logIt();
  },
  logFooDeeper: {
    logIt: function() {
      console.log(this.foo);
    }
  }
}
obj.logFoo();             // undefined
obj.logFooDeeper.logIt(); // undefined

但是你可以不通过 `this` 而直接引用你想要的值

```js
var obj = {
  foo: "bar",
  deeper: {
    logFoo: function() {
      console.log(obj.foo);
    }
  }
}
obj.deeper.logFoo();        // bar
```

## DOM Event this

在 `HTML DOM` 事件处理函数中, `this` 总是指向事件绑定的 `DOM` 元素

```js
function Listener() {
    document.getElementById("foo").addEventListener("click",
       this.handleClick);
}
Listener.prototype.handleClick = function (event) {
    console.log(this); //logs "<div id="foo"></div>"
}

var listener = new Listener();
document.getElementById("foo").click();
```

除非你手动绑定了一个 `this`

```js
function Listener() {
    document.getElementById("foo").addEventListener("click", 
        this.handleClick.bind(this));
}
Listener.prototype.handleClick = function (event) {
    console.log(this); //logs Listener {handleClick: function}
}

var listener = new Listener();
document.getElementById("foo").click();
```

## HTML this

在可以加 JavaScript 代码的所有属性中, `this` 指向当前元素

```html
<div id="foo" onclick="console.log(this);"></div>
<script type="text/javascript">
document.getElementById("foo").click(); //logs <div id="foo"...
</script>
```

## override this

不能重写 `this` 因为它是个关键词

```js
function test() {
  var this = {};
}
// Uncaught SyntaxError: Unexpected token this
```

无法通过语法检查

## eval this

你可以在 `eval` 中访问 `this`

```js
function Car() {
  this.color = "Slivery";
  this.logColor = function() {
    eval("console.log(this.color)");
  }
}
var car = new Car();
car.logColor();         // Slivery
```

这是个安全问题, 只能通过禁止使用 `eval` 来避免

通过 `Function` 创建的函数也可以访问 `this`

```js
function Car() {
  this.color = "Slivery";
  this.logColor = new Function("console.log(this.color)");
}
var car = new Car();
car.logColor();       // Slivery
```

## with this

你可以用 `with` 把 `this` 加到当前的作用域中来读写 `this` 中的属性

```js
function Car() {
  this.color = "Black",
  this.logColor = function() {
    with(this) {
      color = "Slivery";
      console.log(color);
    }
  }
}
var car = new Car();
car.logColor();         // Slivery
console.log(car.color); // Slivery
```

许多人认为这样不好因为 `with` 有歧义问题

## jQuery this

和 `HTML DOM` 事件处理中的 `this` 一样, `jQuery` 中的 `this` 在很多情况下也指向一个 DOM 元素, 如 `event handler` 和 `$.each` 这种方法

```html
<div class="foo bar1"></div>
<div class="foo bar2"></div>
<script type="text/javascript">
$(".foo").each(function () {
    console.log(this); //logs <div class="foo...
});
$(".foo").on("click", function () {
    console.log(this); //logs <div class="foo...
});
$(".foo").each(function () {
    this.click();
});
</script>
```

## thisArg this

如果你用 [`underscore.js`](http://underscorejs.org/)/[`lo-dash`](http://lodash.com/), 你大概会知道可以通过函数参数 `thisArg`(会被当作 `this`) 来传递一个实例, 比如 `_.each` 就是这样. `ECMAScript 5` 之后 JavaScript 中的原生方法也支持 `thisArg` 比如 `forEach`. 前面在演示 `bind/apply/call` 时, 就是用 `thisArg` 传递了实例

```js
function Thing(type) {
    this.type = type;
}
Thing.prototype.log = function (thing) {
    console.log(this.type, thing);
}
Thing.prototype.logThings = function (arr) {
   arr.forEach(this.log, this);
   // _.each(arr, this.log, this);
}

var thing = new Thing("fruit");
thing.logThings(["apples", "oranges", "strawberries", "bananas"]);
// fruit apples
// fruit oranges
// fruit strawberries
// fruit bananas
```

这样就不需要写很多嵌套绑定语句, 不需要再使用 `self`

## End

一些编程语言容易学, 比如 [Go](http://golang.org/) 的 SPEC 可以在很短时间内读完. 一旦读完 SPEC 就可以理解这门语言, 就不需要太多的担心技巧和陷阱, 更不用说实现细节了

JavaScript 并不是这样易懂的语言, SPEC 不易读懂, 有许多陷阱, 所以有许多人会讨论 [The God Parts](http://www.amazon.com/JavaScript-Good-Parts-Douglas-Crockford/dp/0596517742), 目前返现的最好的文档在 [Mozilla Developer Network](https://developer.mozilla.org/en-US/), 我推荐阅读那篇文档中关于 [this](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/this) 的部分, 并在用 Google 查找 JavaScript 相关的问题时加上 `mdn` 的前缀, 这样总能查找到最好的文档. 静态代码分析也非常有帮助, 我一般用 [jshint](http://www.jshint.com/)

如果有任何问题请在 Twitter 上联系 [我bjorntipling](https://twitter.com/bjorntipling)

更新: @hicksyfern 在他的博客上有一篇更简短的［some of this](http://tomhicks.github.io/code/2014/08/11/some-of-this.html)
