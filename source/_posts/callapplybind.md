---
title: call()、apply()、bind()的区别
dropcap: true
date: 2019-08-03 11:49:24
tags: 'javaScript'
categories: 'javaScript'
---
这周讲讲call()、apply()和bind()的区别。[参考链接](https://www.cnblogs.com/moqiutao/p/7371988.html)
在javascript中，call()和apply()都是为了改变某个函数运行时的上下文（context）而存在的，换句话说，就是为了改变函数体内部this的指向。
JavaScript的一大特点是，函数存在「定义时上下文」和「运行时上下文」以及「上下文是可以改变的」这样的概念。
# **call()和apply()例子**
```javascript
function Fruits() {};
Fruits.prototype = {
    color: 'red',
    sayColor: function() {
        console.log(this.color);
    }
}
var apple = new Fruits();
apple.sayColor();    // red
```
如果有一个对象banana = {color : 'yellow'}想调用apple的sayColor()方法，那么我们可以通过call()或apply()改变sayColor()方法里的this指向来实现：
```javascript
banana = {
    color: 'yellow'
}
apple.sayColor.call(banana); // yellow
apple.sayColor.apply(banana); // yellow
```
所以，当一个object没有某个方法（本栗子中banana没有sayColor()方法），但是其他对象有（本栗子中apple有sayColor()方法），我们可以借助call()或apply()用其它对象的方法来操作。让我想到了JS的借用构造函数继承。
# **call()和apply()区别**
其实apply()和call()作用完全一样，只是接受参数的方式不太一样。
```javascript
var func = function(arg1, arg2) {};
func.call(this, arg1, arg2);
func.apply(this, [arg1, arg2]);
```
第一个参数——this是你想指定的上下文，它可以是任何一个JS对象(JS中一切皆对象)，当第一个参数为null、undefined的时候，默认指向window。call()需要把参数按顺序传递进去，而apply()则是把参数放在数组里。
因此，在确定参数个数的情况下用call()更好，参数一目了然；参数个数不确定时用apply()更好。
# **bind()**
和call()很相似，第一个参数是this的指向，从第二个参数开始是接收的参数列表。区别在于bind()方法返回值是函数以及bind()接收的参数列表的使用。
1. bind返回值是函数
```javascript
var obj = {
    name: 'Dot'
}
function printName() {
    console.log(this.name);
}
var dot = printName.bind(obj);
console.log(dot); // function () { … }
dot();  // Dot
printName(); // 这里打印为空，因为window对象里name属性的值默认为空，可以自行打印一下便知
```
bind()方法不会立即执行，而是返回一个改变了上下文this后的函数。而原函数printName()中的this并没有被改变，依旧指向全局对象window。
2. 参数的使用
```javascript
function fn(a, b, c) {
    console.log(a, b, c);
}
var fn1 = fn.bind(null, 'Dot');
fn('A', 'B', 'C'); // A B C
fn1('A', 'B', 'C'); // Dot A B
fn1('B', 'C'); // Dot B C
fn.call(null, 'Dot'); // Dot undefined undefined
```
call()是把第二个及以后的参数作为fn()方法的实参传进去，而fn1()方法的实参实则是在bind()中参数的基础上再往后排。
***有时候我们也用bind方法实现函数柯里化（是把接受多个参数的函数变换成接受一个单一参数（最初函数的第一个参数）的函数，并且返回接受余下的参数而且返回结果的新函数的技术）***
```javascript
var add = function(x) {
  return function(y) {
    return x + y;
  };
};
var increment = add(1);
var addTen = add(10);
increment(2); // 3
addTen(2); // 12
```
***在低版本浏览器没有bind()方法，我们也可以自己实现一个***
```javascript
if (!Function.prototype.bind) {
    Function.prototype.bind = function () {
        var self = this,                        // 保存原函数
            context = [].shift.call(arguments), // 保存需要绑定的this上下文
            args = [].slice.call(arguments);    // 剩余的参数转为数组
        return function () {                    // 返回一个新函数
            self.apply(context, [].concat.call(args, [].slice.call(arguments)));
        }
    }
}
```
# **应用场景**
1. 求数组中的最大和最小值
```javascript
var arr = [2, 1, 89, 3, 46];
var max = Math.max.apply(null, arr); // 89
var min = Math.min.apply(null, arr); // 1
```
2. 将类数组转化为数组
```javascript
// var trueArr = Array.prototype.slice.call(arrayLike);
function list() {
  return Array.prototype.slice.call(arguments);
}
var list1 = list(1, 2, 3);
console.log(list1);  // [1, 2, 3]
```
3. 数组追加
```javascript
var arr1 = [1,2,3];
var arr2 = [4,5,6];
var total = [].push.apply(arr1, arr2);
// var total = Array.prototype.push.apply(arr1, arr2);
console.log(total); // 6
console.log(arr1); // [1, 2, 3, 4, 5, 6]
console.log(arr2); // [4,5,6]
```
4. 判断变量类型
```javascript
function isArray(obj){
    return Object.prototype.toString.call(obj) == '[object Array]';
}
isArray([]) // true
isArray('dot') // false
```
5. 利用call和apply做继承(借用构造函数)
```javascript
function Person(name, age){
    // 这里的this都指向实例
    this.name = name;
    this.age = age;
    this.sayAge = function() {
        console.log(this.age);
    }
}
function Female(){
    Person.apply(this, arguments); // 将父元素所有方法在这里执行一遍就继承了
}
var dot = new Female('Dot', 2);
```
6. 使用log代理console.log
```javascript
function log() {
  console.log.apply(console, arguments);
}
```
7. 绑定函数
MDN对bind()的解释是：bind()方法会创建一个新函数，称为绑定函数，当调用这个绑定函数时，绑定函数会以创建它时传入 bind()方法的第一个参数作为 this，传入 bind() 方法的第二个以及以后的参数加上绑定函数运行时本身的参数按照顺序作为原函数的参数来调用原函数。
```javascript
this.num = 9; 
var mymodule = {
  num: 81,
  getNum: function() { 
    console.log(this.num);
  }
};
mymodule.getNum(); // 81
var getNum = mymodule.getNum;
getNum(); // 9, 因为在这个例子中，"this"指向全局对象
var boundGetNum = getNum.bind(mymodule);
boundGetNum(); // 81
```
# **总结**
call()、apply()和bind()函数存在的区别:
bind()返回对应函数，便于稍后调用；apply()和call()则是立即调用。
除此外，在ES6的箭头函数下，call()和apply()将失效，对于箭头函数来说：
* 箭头函数体内的this对象，就是定义时所在的对象，而不是使用时所在的对象，所以不需要类似于var _this = this这种丑陋的写法
* 箭头函数不可以当作构造函数，也就是说不可以使用new命令， 否则会抛出一个错误
* 箭头函数不可以使用arguments对象，该对象在函数体内不存在，如果要用， 可以用Rest参数代替
* 不可以使用yield命令，因此箭头函数不能用作Generator函数，什么是Generator函数可自行查阅资料，推荐阅读阮一峰[Generator函数的含义与用法](http://www.ruanyifeng.com/blog/2015/04/generator.html)、[Generator函数的异步应用](http://es6.ruanyifeng.com/#docs/generator-async)