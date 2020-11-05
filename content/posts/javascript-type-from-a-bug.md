---
title: "从一个Bug说起"
date: 2020-11-05T11:19:58+08:00
draft: false
toc: false
images:
tags:
  - JavaScript
---
## JavaScript的类型判断

### 类型

基本类型 Undefined Null Boolean Number String Symble

复杂类型 Object Array RegExp Date Function (Set Map)

其他 Document Error .etc

### 判断方法

#### typeof

```js
let bool = true
let num = 1
let str = 'abc'
let und = undefined
let nul = null
let arr = [1,2,3]
let obj = {}
let fun = function(){}
let reg = new RegExp()
let date = new Date()


console.log(typeof bool); //boolean
console.log(typeof num); //number
console.log(typeof str); //string
console.log(typeof und); //undefined
console.log(typeof nul); //object
console.log(typeof arr); //object
console.log(typeof obj); //object
console.log(typeof reg); //object
console.log(typeof fun); //function
console.log(typeof date); //object
```

#### instanceof

```js
console.log(reg instanceof Object); //true
console.log(reg instanceof RegExp); //true
console.log(und instanceof Object); // false
console.log(nul instanceof Object); // false

// 思考
console.log(bool instanceof Boolean);  //false
let bool2 = new Boolean();
console.log(bool2 instanceof Boolean); //true

console.log(fun instanceof Function) //true
let fun = function(){this.name="22";};
console.log(fun instanceof function ) //false

var obj = Object.create(null);
typeof obj              // "object"
obj instanceof Object   // false
```

instanceof它不仅检测构造找个对象的构造器，还检测原型链。所以它可以检测继承而来的属性。

搞点事情

```js
const s = new String('123');

s instanceof String; // true
s instanceof Object; // true


s.__proto__ = Object.prototype;

s instanceof String; // false
s instanceof Object; // true
```

```js
function Animal (name) {
    this.name = name
}

const lion = new Animal('lion');
lion instanceof Animal // true
```

#### constructor

```js
console.log(bool.constructor === Boolean);// true
console.log(num.constructor === Number);// true
console.log(str.constructor === String);// true
console.log(arr.constructor === Array);// true
console.log(obj.constructor === Object);// true
console.log(fun.constructor === Function);// true

function Foo() {}
var f = new Foo();
f.constructor.name // "Foo"
```

#### prototype

Object.prototype.toString.call

```js
console.log(Object.prototype.toString.call(bool));//[object Boolean]
console.log(Object.prototype.toString.call(num));//[object Number]
console.log(Object.prototype.toString.call(str));//[object String]
console.log(Object.prototype.toString.call(und));//[object Undefined]
console.log(Object.prototype.toString.call(nul));//[object Null]
console.log(Object.prototype.toString.call(arr));//[object Array]
console.log(Object.prototype.toString.call(obj));//[object Object]
console.log(Object.prototype.toString.call(fun));//[object Function]
```

也来皮一下

```js
const o = {
    get [Symbol.toStringTag] () {
        return 'Array'
    }
}

Object.prototype.toString.call(o) // [object Array]
```

#### 衍生

lodash

```js
_.isArrayLike(function(){return arguments}());
_.isArrayLike([1, 2, 3])
_.isArrayLike(document.body.children)
_.isArrayLike('abc')
_.isNil(null|undefined)
```

原生方法 Array.isArray() Number.isNaN()(isNaN()的问题是什么) etc.

其实这是某天组内分享的准备内容,原因还要从一个bug说起.(然而这个bug我已经忘了,只剩下这么一篇干巴巴的记录)
