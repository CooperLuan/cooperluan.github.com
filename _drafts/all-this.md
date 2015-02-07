---
layout: post
title: "[译] all this / JAVAScript/NodeJS 中 this 那些事儿"
description: "粗译 all this 一文"
category: [javascript, Translation]
tags: [javascript, Translation]
---
{% include JB/setup %}

原文 [all this](http://bjorn.tipling.com/all-this)

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

## DOM Event this

## HTML this
