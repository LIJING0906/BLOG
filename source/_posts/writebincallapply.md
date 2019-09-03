---
title: 模拟实现call()、apply()
dropcap: true
date: 2019-09-01 17:39:13
tags: 'javaScript'
categories: 'javaScript'
---
之前总结过`call()、apply()`的区别和应用场景，这次想总结如何模拟实现这两者，其实就是搞懂它们的原理。
# **模拟实现call()**
```javascript
var foo ={
  value:1
}
function bar(){
  console.log(this.value)
}
bar.call(foo); // 1
```
## call()的两个特点
1. 改变`this`的指向
2. 调用了`bar`函数

## 模拟实现第一步
在调用`call`时，我们希望`bar`里的`this`指向`foo`，我们可以把`bar`写进`foo`：
```javascript
var foo = {
    value: 1,
    bar: function() {
        console.log(this.value);
    }
}
foo.bar();
```
这样就改变了`this`指向，但`foo`却被添加了一个额外的属性，应该删除这个属性，大概思路：
```javascript
foo.fn = bar;
foo.fn();
delete foo.fn;
```
因此改进一下：
```javascript
Function.Prototype.call2 = function(context) {
    // 首先要获取调用call的函数，用this可以获取
    context.fn = this; // foo.fn = bar
    context.fn(); // foo.fn()
    delete context.fn; // delete foo.fn
}

// 测试
var foo = {
    value: 1
}
function bar() {
    console.log(this.value);
}
bar.call2(foo); // 1
```
## 模拟实现第二步
上面改进了依然存在一个问题：`bar`不能接收参数。所以我们可以从`arguments`中获取参数，从第二个到最后一个参数放到数组中，之所以不要第一个参数，是因为第一个参数是`this`。

我们知道`arguments`对象是类数组对象(类数组对象不能使用push/pop/shift/unshift等数组方法)，这里用ES3的实现把类数组对象转成数组：
```javascript
var args = [];
for(var i = 1, len = arguments.length; i < len; i++){
    args.push('arguments[' + i + ']');
}
```
执行后args为['arguments[1]', 'arguments[2]', 'arguments[3]']，如果直接用作参数，像这样`context.fn(args)`，那这样取到的每一个参数只是一个字符串，解决办法：
```javascript
eval('context.fn(' + args + ')');
```
因此，再改进的实现如下：
```javascript
Function.prototype.call2 = function(context) {
    context.fn = this;
    var args = [];
    for(var i = 1, len = arguments.length; i < len; i++) {
        args.push('arguments[' + i + ']');
    }
    eval('context.fn(' + args + ')');
    delete context.fn;
}

// 测试
var foo = {
    value: 1
}
function bar(name, age) {
    console.log(name);
    console.log(age);
    console.log(this.value);
}
bar.call2(foo, 'Cherry', 19); // Cherry 18 1
```
## 模拟实现第三步
`call`有两小点需要注意：
1. 第一个参数可以传`null`，当传`null`时，视为指向`window`
2. 函数是可以有返回值的

再改进一点如下：
```javascript
Function.prototype.call2 = function(context) {
    var context = context || window;
    context.fn = this;

    var args = [];
    for(var i = 1, len = arguments.length; i < len; i++) {
        args.push('arguments[' + i + ']');
    }
    var result = eval('context.fn(' + args + ')');

    delete context.fn;
    return result;
}
// 测试
var value = 3;
var obj = {
    value: 1
}
function bar(name, age) {
    console.log(this.value);
    return {
        value: this.value,
        name: name,
        age: age
    }
}
bar.call(null); // 3
console.log(bar.call2(obj, 'Cherry', 18));
// 1
// Object {
//    value: 1,
//    name: 'Cherry',
//    age: 18
// }
```
上面是用ES3的语法实现的，下面用ES6实现：
```javascript
Function.prototype.call2 = function(context) {
    context = context || window;
    context.fn = this;

    let args = [...arguments].slice(1);
    let result = content.fn(...args);

    delete context.fn;
    return result;
}
```
# **模拟实现apply()**
我们知道`apply()`和`call()`的原理一样，只是传参方式不同，因此，模拟实现也差不多：

ES3实现：
```javascript
Function.prototype.apply2 = function(context, arr) {
    var context = context || window;
    context.fn = this;

    var result;
    if (!arr) {
        result = context.fn()
    } else {
        var args = [];
        for(var i = 1, len = arr.length; i < len; i++) {
            args.push('arr[' + i + ']');
        }
        result = eval('context.fn(' + args + ')');
    }

    delete context.fn;
    return result;
}
```
ES6实现：
```javascript
Function.prototype.apply2 = function(context, arr) {
    context = context || window;
    context.fn = this;

    let result;
    if (!arr) {
        result = context.fn();
    } else {
        result = content.fn(...arr);
    }

    delete context.fn;
    return result;
}
```