---
title: 模拟实现bind()
dropcap: true
date: 2019-09-07 10:21:47
tags:
categories:
---
前面已经讨论过`bind()`的用法，这篇文章一步一步模拟实现`bind()`。
# **bind特点**
1. 可以指定this
2. 返回一个函数
3. 可以传入参数

## 模拟实现第一步
```javascript
Function.prototype.bind2 = function(context) {
    var self = this; // this是调用bind2的对象
    return function() { // 返回一个函数
        return self.apply(context); // 把this绑定给传入的上下文对象
    }
}

// 测试
var foo = {
    value: 2
}
var bar = function () {
    console.log(this.value);
}
var bf = bar.bind2(foo);
console.log(bf()); // 2
```
## 模拟实现第二步----传参
先看一个例子，在`bind()`的时候可以传参，在调用函数的又可以传参：
```javascript
var foo = {
    value: 2
}
function bar(name, age) {
    console.log(this.value);
    console.log(name);
    console.log(age);
}
var bf = bar.bind(foo, 'Jane');
bf(20); // 2 Jane 20
// bar.bind(foo, 'Jane')(20); // 2 Jane 20   把bind和调用写在一起
```
可以用`arguments`进行处理：
```javascript
Function.prototype.bind2 = function (context) {
    var self = this;
    // 获取bind时传入的参数，因为bind的第一个参数是指定的this，因此从第二个参数开始截取
    var args = Array.prototype.slice.call(arguments, 1);
    return function() {
        // 获取返回的函数传入的参数
        var bindArgs = Array.prototype.slice.call(arguments);
        return self.apply(context, args.concat(bindArgs)); // 合并两次获取到的参数
    }
}
// 测试
var foo = {
    value: 2
}
function bar(name, age) {
    return {
        value: this.value,
        name: name,
        age: age
    }
}
var bf = bar.bind2(foo, 'Jane');
bf(20); // { value: 2, name: "Jane", age: 20 }
```
## 模拟实现第三步
到上面的第二步，大部分的功能已经实现，但是还有一个难点，`bind()`有一个特性：
> 一个绑定函数也能使用new操作符创建对象：这种行为就像把原函数当成构造器，提供的this值被忽略，同时调用时的参数被提供给模拟函数。

也就是说，当`bind()`返回的函数作为构造函数的时候，`bind`时指定的`this`值会失效，但传入的参数依然生效。
```javascript
var value = 2;
var foo = {
    value: 1
}
function bar(name, age) {
    this.habit = 'shopping';
    console.log(this.value);
    console.log(name);
    console.log(age);
}
bar.prototype.friend = 'kevin';
var bindFoo = bar.bind(foo, 'Jane');
var obj = new bindFoo(20); // undefined "Jane" 20
console.log(obj.habit); // "shopping"
console.log(obj.friend); // "kevin"
```
上面的例子中，运行结果`this.value`输出是`undefined`，这个值既不是全局的`value`也不是`foo`对象的`value`，这说明`bind`的`this`失效了，而`new`操作符生成了一个新的对象，这个时候`this`指向的是`obj`。

OK，我们可以通过修改返回函数的原型来实现这种效果：
```javascript
Function.prototype.bind2 = function(context) {
    var self = this;
    var args = Array.prototype.slice.call(arguments, 1);

    var fBound = function() {
        var bindArgs = Array.prototype.slice.call(arguments);
        // 当作为构造函数时，this指向实例，此时`this instanceof fBound`结果为true，可以让实例获得来自绑定函数的值，即上例中实例会具有habit属性
        // 当作为普通函数时，this指向window，此时结果为false，将绑定函数的this指向context
        return self.apply(this instanceof fBound ? this : context, args.concat(bindArgs));
    }
    // 修改返回函数的prototype为绑定函数的prototype，实例就可以继承绑定函数的原型中的值，即上例中obj可以获取到bar原型上的friend
    fBound.prototype = this.prototype;
    return fBound;
}
// 测试直接用上面的例子把bind改成bind2即可测试，效果跟原生的bind一样
```
## 模拟实现第四步
上面实现中`fBound.prototype = this.prototype`有一个缺点，直接修改`fBound.prototype`的时候，也会直接修改`this.prototype`。
可以用一个空函数来中转：
```javascript
Function.prototype.bind2 = function(context) {
    var self = this;
    var args = Array.prototype.slice.call(arguments, 1);

    var fNOP = function() {};

    var fBound = function() {
        var bindArgs = Array.prototype.slice.call(arguments);
        // 当作为构造函数时，this指向实例，此时`this instanceof fBound`结果为true，可以让实例获得来自绑定函数的值，即上例中实例会具有habit属性
        // 当作为普通函数时，this指向window，此时结果为false，将绑定函数的this指向context
        return self.apply(this instanceof fBound ? this : context, args.concat(bindArgs));
    }
    // 修改返回函数的prototype为绑定函数的prototype，实例就可以继承绑定函数的原型中的值，即上例中obj可以获取到bar原型上的friend
    fNOP.prototype = this.prototype;
    fBound.prototype = new fNOP();
    return fBound;
}
```
## 模拟实现第五步
这一步要做一个判断：如果调用`bind()`的不是函数，需要抛出异常：
```javascript
if (typeof this !== 'function') {
    throw new Error("Function.prototype.bind - what is trying to be bound is not callable");
}
```
## 最终代码
```javascript
Function.prototype.bind2 = function(context) {
    if (typeof this !== 'function') {
        throw new Error('Function.prototype.bind - what is trying to be bound is not callable');
        var self = this;
        var args = Array.prototype.slice.call(arguments, 1);
        var fNOP = function() {};
        var fBound = function() {
            var bindArgs = Array.prototype.slice.call(arguments);
            return self.apply(this instanceof fNOP ? this : context, args.concat(bindArgs));
        }
        fNOP.prototype = this.prototype;
        fBound.prototype = new fNOP();
        return fBound;
    }
}
```