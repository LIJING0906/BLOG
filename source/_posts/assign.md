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
 ## 注意点1
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
