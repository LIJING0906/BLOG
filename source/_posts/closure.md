---
title: JS闭包和作用域
dropcap: true
date: 2019-08-24 19:27:49
tags: 'javaScript'
categories: 'javaScript'
---
今天来啃闭包和作用域这块难啃的骨头。
# **什么是闭包**
> 闭包是函数和声明该函数的词法环境的组合。 ----MDN

> 闭包就是指有权访问另一个函数作用域中的变量的函数。 ----红宝书

# **什么是作用域**
作用域是一个变量和函数的作用范围，JS中函数内声明的所有变量在函数体内始终是可见的，在ES6前有全局作用域和局部作用域，但是没有块级作用域（catch只在其内部生效），局部变量的优先级高于全局变量。
# **作用域链**
Javascript中有一个执行上下文(execution context)的概念，它定义了变量或函数有权访问的其它数据，决定了他们各自的行为。每个执行环境都有一个与之关联的变量对象，环境中定义的所有变量和函数都保存在这个对象中。

当访问一个变量时，解释器会首先在当前作用域查找标示符，如果没有找到，就去父作用域找，直到找到该变量的标示符或者不在父作用域中，这就是作用域链。

作用域链和原型继承查找时的区别：如果去查找一个普通对象的属性，但是在当前对象和其原型中都找不到时，会返回undefined；但查找的属性在作用域链中不存在的话就会抛出ReferenceError。

作用域链的顶端是全局对象，在全局环境中定义的变量就会绑定到全局对象中。

闭包的作用域链包含着它自己的作用域，以及包含它的函数的作用域和全局作用域。
# **闭包的特性**
1. 函数嵌套函数
2. 内部函数可以访问外部作用域(或外部函数)的变量和参数
3. 参数和变量不会被回收机制回收，一直存在于内存中，除非手动清除
# **为什么要用闭包**
1. 希望变量长期存在内存中
2. 避免全局变量污染
# **闭包举例及应用**
## 最简单的闭包
```javascript
function outer() {
    var a = 1;
    return function() {
        return a; // 内部函数访问外部作用域的变量a
    }
}
var b = outer(); // 这一句执行完，变量a并没有被回收，因为要内部函数还需要引用
console.log(b()); // 1 执行内部函数，引用外部变量a
```
这是最简单的一种闭包，已经包含了闭包的三个特性。
## 释放对闭包的引用
```javascript
function makeAdder() {
    return function(x) {
        return x + y;
    }
}
var add5 = makeAdder(5);
var add10 = makeAdder(10);
console.log(add5(2)); // 7
console.log(add10(2)); // 12
// 释放对闭包的引用
add5 = null;
add10 = null;
```
上面的代码中，add5和add10都是闭包，它们共享相同的函数定义，但是保存了不同的环境，add5的环境中x是5，add10的环境中x是10。最后都通过null释放了对内部函数（闭包）的引用。

在javascript中，如果一个对象不再被引用，那么这个对象就会被垃圾回收机制回收；如果两个对象互相引用，而不再被第3者所引用，那么这两个互相引用的对象也会被回收。
##  "JavaScript中的函数运行在它们被定义的作用域里，而不是它们被执行的作用域里。" ——《JavaScript权威指南》
```javascript
var name = 'Jane';
function getName () {
  console.log(name);
}
function myName () {
  var name = 'hahaha';
  getName();
}
myName(); // Jane
```
## 闭包与for循环
```javascript
function arrFunc() {
    var arr = [];
    for (var i=0; i<10; i++) {
        arr[i] = function() {
            return i;
        }
    }
    return arr; // 函数声明的时候会把for循环执行一遍，最终i的值时10，后续函数被调用的时候传入的i都是10
}
console.log(arrFunc()[2]()); // 10，arrFunc()执行的结果是arr中有10个匿名函数，每个匿名函数执行的结果都是10
```
解决办法：
1. 用闭包
```javascript
function arrFunc() {
    var arr = [];
    for (var i=0; i<10; i++) {
        arr[i] = function(num) {
            return function() {
                return num;
            }
        }(i);
    }
    return arr;
}
console.log(arrFunc()[2]()); // 2
```
```javascript
function arrFunc() {
    var arr = [];
    for (var i=0; i<10; i++) {
        (function(i) {
            arr[i] = function() {
                return i;
            }
        })(i);
    }
    return arr;
}
console.log(arrFunc()[2]()); // 2
```
因为用了闭包，因此每一次循环都是独立的一个环境，对应不同的i的值，因此才能实现arr数组中是10个返回不同值的匿名函数。

这种闭包可以用于循环添加DOM事件。

2. 用ES6的let
```javascript
function arrFunc() {
    var arr = [];
    for (let i=0; i<10; i++) {
        arr[i] = function() {
            return i;
        }
    }
    return arr;
}
console.log(arrFunc()[2]()); // 2
```
# **闭包中的this**
```javascript
var obj = {
    name: 'object',
    getName: function() {
        return function() {
            return this.name;
        }
    }
}
console.log(obj.getName()()); // ''
```
`obj.getName()()`分成两步来看，第一步执行`obj.getName()`得到匿名函数`function(){return this.name}`，第二步在全局作用域中执行`function(){return this.name}()`，`this`指向`window`对象，它的`name`属性的值默认是为空的，因此打印空字符串。

想让`this`指向`obj`,可以用一下方法：
```javascript
var obj = {
    name: 'object',
    getName: function() {
        var that = this; // 把this保存起来
        return function() {
            return that.name; // 返回that.name
        }
    }
}
console.log(obj.getName()()); // object
```
下面这种情况也要注意：
```javascript
var name = 'window';
var obj = {
    name: 'object',
    getName: function() {
        return this.name;
    }
}
console.log(obj.getName()); // object
(obj.name = obj.name)(); // window
```
`(obj.getName = obj.getName)`赋值语句返回的是等号右边的值，在全局作用域中返回，所以`(obj.getName = obj.getName)();`的`this`
## 设计私有的方法和变量
任何在函数中定义的变量，都可以认为是私有变量，因为不能在函数外部访问这些变量。私有变量包括函数的参数、局部变量和函数内定义的其他函数。

把有权访问私有变量的公有方法称为特权方法（privileged method）。
```javascript
function Animal() {
    // 定义私有变量series和run
    var series = '哺乳动物';
    function run() {
        console.log('I can run!');
    };
    // 特权方法，可以访问私有变量series
    this.getSeries = function() {
        return series;
    }
}
var dog = new Animal();
console.log(dog.getSeries()); // 哺乳动物
```
了解两个概念：
1. 单例：只有一个实例的对象。JS中通常用字面量来创建单例
2. 模块模式：我的理解就是用闭包的方式为单例创建私有方法和私有属性的模式

用普通模式创建单例：
```javascript
var objA = {
    name: 'Jane',
    speak: function() {
        console.log('I can speak!');
    },
    getName: function() {
        return this.name;
    }
}
```
用模块模式创建单例：
```javascript
var listObj = (function() {
    // 定义私有变量
    var list = [];
    // 定义特权方法
    var add = function(item) {
        list.push(item);
    };
    var getAll = function() {
        return list;
    };
    var getLength = function() {
        return list.length;
    };
    return {
        add: add,
        getAll: getAll,
        getLength: getLength
    };
})()
listObj.add({
    id: 0,
    name: 'hahaha'
});
console.log(listObj.getAll()); // [{ id: 0, name: 'hahaha' }]
console.log(listObj.getLength()); // 1
```
# **闭包的缺陷**
闭包的缺点就是常驻内存会增大内存使用量，并且使用不当很容易造成内存泄露。

如果不是因为某些特殊任务而需要闭包，在没有必要的情况下，在其它函数中创建函数是不明智的，因为闭包对脚本性能具有负面影响，包括处理速度和内存消耗。
# **一道面试题**
```javascript
function fun(n, o) {
    console.log(o);
    return {
        fun: function(m) {
            return fun(m, n);
        }
    };
}
var a = fun(0); // undefined，同时a={fun:funtion(m){return fun(m, 0)}}
a.fun(1); // 0，同时得到{fun:funtion(1){return fun(1, 1)}} 外层函数传入n=1,内层函数的n也被置为1
a.fun(2); // 0
a.fun(3); // 0

var b = fun(0).fun(1); // 0
b.fun(2); // 1
b.fun(3); // 1

var c = fun(0).fun(1).fun(2).fun(3); // undefined 0 1 2
```