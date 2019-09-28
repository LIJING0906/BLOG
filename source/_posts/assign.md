---
title: Object.assign的原理及其实现方式
dropcap: true
date: 2019-09-22 14:20:00
tags: 'javaScript'
categories: 'javaScript'
---
上周在总结赋值和深浅拷贝的时候提到了`Object.assign`这种浅拷贝方式。这周谈谈它的原理以及实现方式。
# 浅拷贝Object.assign
上篇文章有讲到它的定义和用法，主要是将所有**可枚举属性**的值从一个或多个源对象中复制到目标对象，同时返回目标对象。
语法如下：
```javascript
Object.assign(target,  ...source)
```
其中`target`是目标对象，`...source`是源对象，可以是一个或多个，返回修改后的目标对象。
如果目标对象和源对象具有相同属性，则目标对象的该属性将会被源对象的相同属性覆盖，后来的源对象的属性将会类似地覆盖早先的属性。
## 示例1
我们知道浅拷贝就是拷贝对象的第一层的基本类型值，以及第一层的引用类型地址。
```javascript
// 第一步
let a = {
    name: "Kitty",
    age: 18
}
let b = {
    name: "Jane",
    book: {
        title: "You Don't Know JS",
        price: "45"
    }
}
let c = Object.assign(a, b);
console.log(c);
// {
//     name: "Jane",
//     age: 18,
//     book: {title: "You Don't Know JS", price: "45"}
// } 
console.log(a === c); // true

// 第二步
b.name = "change";
b.book.price = "55";
console.log(b);
// {
//     name: "change",
//     book: {title: "You Don't Know JS", price: "55"}
// } 

// 第三步
console.log(a);
// {
//     name: "Jane",
//     age: 18,
//     book: {title: "You Don't Know JS", price: "55"}
// } 
```
 1、第一步中，使用`Object.assign`把源对象b中的属性复制到目标对象a中，把改变后的对象定义为c，可以看出b会替换掉a中相同的属性的值。上面的代码需要注意的是，返回对象c就是目标对象a。
 2、第二步中，修改源对象b的基本类型值（name）和引用类型值（book）。
 3、第三步中，浅拷贝之后目标对象a的基本类型值没有改变，但引用类型值被改变了，因为`Object.assign`拷贝的是属性值，当属性值是一个指向对象的引用时，它拷贝的那个引用地址。
 ## 示例2
 `String`类型和`Symbol`类型的属性都会被拷贝，而且不会跳过那些值为`null`或`undefined`的属性。
 ```javascript
 let a = {
    name: "Jane",
    age: 20
}
let b = {
    b1: Symbol("Jane"),
    b2: null,
    b3: undefined
}
let c = Object.assign(a, b);
console.log(c);
// {
//     name: "Jane",
//     age: 20,
//     b1: Symbol(Jane),
//     b2: null,
//     b3: undefined
// } 
console.log(a === c); // true
 ```
 # Object.assign模拟实现
 实现一个`Object.assign`大致思路如下:

 1、判断原生`Object`是否支持该函数，如果不存在的话创建一个函数`assign`,并使用`Object.defineProperty`将该函数绑定到`Object`上。
 2、判断参数是否正确（目标对象不能为空，我们可以直接设置{}传递进去，但必须设置值）。
 3、使用`Object()`转成对象，并保存为`to`,最后返回这个对象`to`。
 4、使用`for...in`循环遍历出所有可枚举的自有属性。并复制给新的目标对象（`hasOwnProperty`返回非原型链上的属性）。
 为了方便验证方便，使用`assign2`代替`assign`，注意以下模拟实现不支持`symbol`属性，因为`ES5`中根本没有`symbol`，实现代码如下：
 ```javascript
if (typeof Object.assign2 != 'function') {
    Object.defineProperty(Object, 'assign2', { // 注意点1
        value: function(target) {
            'use strict';
            if (target == null) { // 注意点2
                throw new Error('Cannot convert undefined or null to object');
            }
            var to = Object(target); // 注意点3
            for (var index = 1; index < arguments.length; index++) {
                var nextSource = arguments[index];
                if (nextSource != null) { // 注意点2
                    // 注意点4
                    for (var nextKey in nextSource) {
                        if (Object.prototype.hasOwnProperty.call(nextSource, nextKey)) {
                            to[nextKey] = nextSource[nextKey];
                        }
                    }
                }
            }
            return to;
        },
        writable: true,
        configurable: true
    })
}
 ```
 测试一下：
 ```javascript
let a = {
    name: "advanced",
    age: 18
}
let b = {
    name: "Jane",
    book: {
        title: "You Don't Know JS",
        price: "45"
    }
}
let c = Object.assign2(a, b);
console.log(c);
// {
//     name: "Jane",
//     age: 18,
//     book: {title: "You Don't Know JS", price: "45"}
// } 
console.log(a === c); // true
 ```
 ## 注意点1 可枚举型
 原生情况下挂载在`Object`上的属性是不可枚举的，但是直接在`Object`上挂载属性`a`之后是可枚举的，所以必须使用`Object.defineProperty`，并设置`enumerable: false`和`writable: true, configurable: true`。
 以下代码说明`Object`上的属性不可枚举：
 ```javascript
for (var i in Object) {
    console.log(Object[i]);
}
// 无输出
Object.keys(Object); // []
 ```
 我们可以使用2种方法查看`Object.assign`是否可枚举，使用`Object.getOwnPropertyDescriptor`或者`Object.propertyIsEnumerable`都可以，其中`propertyIsEnumerable(...)`会检查给定的属性名是否直接存在于对象中（而不是在原型链上）并且满足`enumerable: true`。具体用法如下：
 ```javascript
Object.getOwnPropertyDescriptor(Object, 'assign');
// {
//     value: ƒ, 
//     writable: true,     // 可写
//     enumerable: false,  // 不可枚举，注意这里是 false
//     configurable: true  // 可配置
// }
Object.propertyIsEnumerable('assign'); // false
 ```
来看看直接在`Object`上挂载属性`a`之后可枚举的情况：
```javascript
Object.a = function() {
    console.log('log a');
}
Object.getOwnpropertyDescriptor(Object, 'a');
// {
//      value: ƒ, 
//      writable: true, 
//      enumerable: true,  // 注意这里是 true
//      configurable: true
// }
Object.propertyIsEnumerable('a'); // true
```
因为`Object.assign`是不可枚举的，所以不能用直接挂载的方式（可枚举）来模拟实现，必须用`Object.defineProperty`来设置`writable: true, enumerable: false, configurable: true`，当然默认情况下都是`false`。
```javascript
Object.defineProperty(Object, 'b', {
    value: function() {
        console.log('log b');
    }
})
Object.getOwnPropertyDescriptor(Object, 'b');
// {
//     value: ƒ, 
//     writable: false,     // 可写
//     enumerable: false,  // 不可枚举，注意这里是 false
//     configurable: false  // 可配置
// }
```
## 注意点2 判断参数是否正确
因为`undefined`和`null`是相等的，即`undefined == null`返回`true`，只需要按照如下方式判断就好了。
```javascript
if (target == null) { // TypeError if undefined or null
    throw new TypeError('Cannot convert undefined or null to object');
}
```
## 注意点3 原始类型被包装为对象
```javascript
var v1 = 'abc';
var v2 = true;
var v3 = 10;
var v4 = Symbol('foo');
var obj = Object.assgin({}, v1, null, v2, undefined, v3, v4);
// 原始类型会被包装，null和undefined会被忽略
// 注意，只有字符串的包装对象才可能有自身可枚举属性
console.log(obj); // {'0': 'a', '1': 'b', '2': 'c'}
```
上面的代码可以看出v1、v2、v3实际上被忽略了，原因在于他们自身没有**可枚举属性**。
```javascript
var v1 = 'abc';
var v2 = true;
var v3 = 10;
var v4 = Symbol('foo');

// Object.keys() 返回一个数组，包含所有可枚举属性
// 只会查找对象直接包含的属性，不查找[[Prototype]]链
Object.keys( v1 ); // [ '0', '1', '2' ]
Object.keys( v2 ); // []
Object.keys( v3 ); // []
Object.keys( v4 ); // []
Object.keys( v5 ); // TypeError: Cannot convert undefined or null to object

// Object.getOwnPropertyNames(..) 返回一个数组，包含所有属性，无论它们是否可枚举
// 只会查找对象直接包含的属性，不查找[[Prototype]]链
Object.getOwnPropertyNames( v1 ); // [ '0', '1', '2', 'length' ]
Object.getOwnPropertyNames( v2 ); // []
Object.getOwnPropertyNames( v3 ); // []
Object.getOwnPropertyNames( v4 ); // []
Object.getOwnPropertyNames( v5 ); // TypeError: Cannot convert undefined or null to object
```
但是下面的代码是可以执行的：
```javascript
var a = 'abc';
var b = {
    v1: 'def',
    v2: true,
    v3: 10,
    v4: Symbol('foo'),
    v5: null,
    v6: undefined
}
var obj = Objec.assign(a, b);
console.log(obj);
// {
//     [String: 'abc']
//     v1: 'def',
//     v2: true,
//     v3: 10,
//     v4: Symbol('foo'),
//     v5: null,
//     v6: undefined
// }
```
原因很简单，因为此时`undefined`、`true`等不适 作为对象，而是作为对象`b`的属性值，对象`b`是可枚举的。
```javascript
Object.keys(b); // [ 'v1', 'v2', 'v3', 'v4', 'v5', 'v6' ]
```
这里其实又可以看出一个问题来，那就是目标对象如果是原始类型，会被包装成对象，对应上面的代码就是目标对象`a`会被包装成`[String: 'abc']`，那模拟实现时应该如何处理呢？很简单，使用`Object()`就OK。
```javascript
var a = 'abc';
console.log(Object(a)); // [String: 'abc']
```
到这里已经介绍了很多知识了，让我们再来延伸一下。
```javascript
var a = 'abc';
var b = 'def';
Object.assign(a, b); // TypeError: Cannot assign to read only property '0' of object '[object String]'
```
报错的原因在于`Object.assgin()`时，其属性描述符为不可写，即`writable: false`。
```javascript
var myObject = Object('abc');
Object.getOwnPropertyNames(myObject);
// [ '0', '1', '2', 'length' ]

Object.getOwnPropertyDescriptor(myObject, '0');
// { 
//   value: 'a',
//   writable: false, // 注意这里
//   enumerable: true,
//   configurable: false 
// }
```
## 注意点4 存在性
如何在不访问属性值的情况下判断对象中是否存在某个属性呢：
```javascript
var anotherObject = {
    a: 1
};

// 创建一个关联到anotherObject的对象
var myObject = Object.create(anotherObject);
myObject.b = 2;

('a' in myObject); // true
('b' in myObject); // true

myObject.hasOwnProperty('a'); // false
myObject.hasOwnProperty('b'); // true
```
上边用`in`操作符和`hasOwnProperty`方法，区别如下：
1. `in`操作符会检查属性是否在对象及其`[[Prototype]]`原型链上
2. `hasOwnProperty()`只会检查属性是否存在于`myObject`对象中，不会检查`[[Prototype]]`原型链
`Object.assign`方法肯定不会拷贝原型链上的属性，所以模拟实现时需要用`hasOwnProperty()`判断处理下，但是直接使用`myObject.hasOwnProperty()`是有问题的，因为有的对象可能没有连接到`Object.prototype`上（比如通过 `Object.create(null)`来创建），这种情况下，使用`myObject.hasOwnProperty()`就会失败。

```javascript
var myObject = Object.create(null);
myObject.b = 2;

('b' in myObject); // true

myObject.hasOwnProperty( 'b' ); // TypeError: myObject.hasOwnProperty is not a function
```
解决方法也很简单，使用之前介绍的`call`就可以了，使用如下:
```javascript
var myObject = Object.create(null);
myObject.b = 2;

Object.prototype.hasOwnProperty.call(myObject, 'b'); // true
```