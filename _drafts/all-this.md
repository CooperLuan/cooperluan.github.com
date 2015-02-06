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
事实不是这样的, JavaScript 中的 `this` 是一个潘多拉魔盒(译者注: 作者引用了哈利波特中黑魔法防御课老师用来关恶灵的衣柜..)

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

## object this

## DOM Event this

## HTML this
