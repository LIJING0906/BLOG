---
title: JS中的this指向问题
dropcap: true
date: 2019-07-27 15:28:43
tags: 'javaScript'
categories: 'javaScript'
---
在上篇JS继承中涉及到的好几个知识点都想写，比如call()、apply()从而牵出和bind()的区别，由Object.create()想到与new、{}的区别，原型链以及作用域，然而在参考其他博客时发现，应该先把JS中的this指向弄明白。
《你不知道的JavaScript》(上卷)第二部分讲到了this，算是比较权威的关于this的讲解，但我觉得有些地方讲得还是晦涩难懂，需要结合一些博客来理解会容易理解一些。
# **为什么要用this**
this提供一种优雅的方式来隐式“传递”一个对象引用，在函数中显示传入一个上下文对象，避免在代码越来越复杂的情况下造成上下文对象混乱。
# **this到底是什么**
this就像它的词性一样，是个代词，表指代什么，在JS中表示指代某个对象。this是在函数运行时绑定到某个对象上，并不是在函数定义时被绑定的，因此this的绑定（即this的指向）与函数的声明位置没关系，只取决于函数的调用方式。
# **绑定规则**
说五种绑定规则之前，先说说不同作用域中this的指向，包括全局作用域（Global Scope）和局部作用域(Local Scope)。
**全局作用域（Global Scope）**
所有运行环境中JS运行时都只有唯一的全局对象，在浏览器中，全局对象是window;在node.js中全局对象是global。
在全局作用域中（任何函数体外的代码），this指向的是全局对象，不管是不是在严格模式下。
**局部作用域(Local Scope)**
局部作用域可以理解为{}包裹的区域，this的指向就根据调用方法不同而不同。
## 一、默认绑定
1. ***独立函数调用***，没有其他规则绑定时的默认规则，也是最常用的绑定规则，this指向全局对象window。
```javascript
function foo() { 
    console.log(this);
    console.log(this.a);
}       
var a = 2; 
foo(); // Window对象  2
```
2. ***严格模式下***，无法执行默认绑定把this绑定到全局对象上，因此，没有指定值时，this会绑定到undefined上。
虽然this的绑定规则完全取决于调用位置，但是只有foo()***运行***在非严格模式下时，默认绑定才能把this绑定到全局对象；严格模式下***调用***函数则不影响默认绑定。
```javascript
function foo() { // 运行在严格模式下，this会绑定到undefined
    "use strict";
    console.log(this.a);
}
var a = 2;
// 这里虽然foo()是在全局作用域中执行，但是foo里的代码运行在严格模式下，所以this被绑定到了undefined上。
foo(); // TypeError: Cannot read property 'a' of undefined

// --------------------------------------

function foo() {
    console.log(this.a);
}
var a = 2;
(function() { // 严格模式下调用函数则不影响默认绑定
    "use strict";
    foo(); // 2
})();
```
## 二、隐式绑定
当函数作为对象属性被调用时，函数中的this指向（被绑定到）调用这个函数的对象，这就是隐式绑定。注意：最后一层在调用中起作用，即最接近函数调用的那层起作用。
```javascript
function foo() { 
    console.log(this.a);
}
var a = 2;
var obj = { 
    a: 3,
    foo: foo 
};
obj.foo(); // 3
```
为什么是3？那就需要了解一下内存中基础类型数据和引用数据类型是怎么存储的，可以看看阮一峰的[JavaScript的this原理](http://www.ruanyifeng.com/blog/2018/06/javascript-this.html)。
上面代码的执行过程：获取obj.foo属性——>根据引用关系（引用地址）找到foo函数，执行函数调用。obj离foo（）最近，this被绑定到obj上。
1. 多层调用
```javascript
function foo() { 
    console.log(this.a);
}
var a = 2;
var obj1 = { 
    a: 4,
    foo: foo 
};
var obj2 = { 
    a: 3,
    obj1: obj1
};
obj2.obj1.foo(); // 4
```
同样看一下调用过程：获取obj2.obj1属性——>根据引用关系（引用地址）获取obj1对象——>再重复第二步找到foo函数——>执行函数调用。obj1离foo（）最近，this被绑定到obj1上。
2. 隐式丢失（函数别名）
```javascript
function foo() { 
    console.log(this.a);
}
var a = 2;
var obj = { 
    a: 3,
    foo: foo 
};
var bar = obj.foo; // 把foo（）的引用地址赋值给bar
bar(); // 2
```
为什么没有隐式绑定到obj上？因为obj.foo是引用类型的属性，它其实是一个指向foo()函数的引用地址[打印一下obj.foo就能发现，打印出来的是foo函数]，因此bar得到的是foo()函数的引用地址，调用bar()时this被绑定到全局对象window上了。
3. 隐式丢失（回调函数）
```javascript
function foo() {
    console.log(this.a);
}
function doFoo(fn) {
    // fn其实引用的是foo
    fn(); // <-- 调用位置！
}
var obj = {
    a: 2,
    foo: foo
};
var a = "oops, global"; // a是全局对象的属性
doFoo(obj.foo); // "oops, global"

// ----------------------------------------

// JS环境中内置的setTimeout()函数实现和下面的伪代码类似：
function setTimeout(fn, delay) {
    // 等待delay毫秒
    fn(); // <-- 调用位置！
}
```
道理同函数别名的this绑定丢失一样。
## 三、显示绑定
通过call()或apply()方法。第一个参数是一个对象，在调用函数时将这个对象绑定到this上。因为直接指定this的绑定对象，称之为显示绑定。
```javascript
function foo() { 
    console.log(this.a);
}
var a = 2;
var obj1 = { 
    a: 3,
};
var obj2 = { 
    a: 4,
};
foo.call(obj1); // 3 强制把this绑定到obj1上
foo.call(obj2); // 4 强制把this绑定到obj2上
```
显示绑定无法解决丢失绑定问题。
解决方案：
1. 硬绑定
```javascript
function foo() {
    console.log(this.a);
}
var obj = {
    a: 2
};
var bar = function() {
    foo.call(obj);
};
bar(); // 2 函数别名
setTimeout(bar, 100); // 2 回调函数
// 硬绑定的bar不可能再修改它的this
bar.call(window); // 2
```
典型应用场景是创建一个包裹函数，负责接收参数并返回值。
```javascript
function foo(something) {
    console.log(this.a, something);
    return this.a + something;
}

var obj = {
    a: 2
};

var bar = function() {
    return foo.apply(obj, arguments);
};

var b = bar(3); // 2 3
console.log(b); // 5
```
创建一个可以重复使用的辅助函数。
```javascript
function foo(something) {
    console.log( this.a, something );
    return this.a + something;
}
// 简单的辅助绑定函数
function bind(fn, obj) {
    return function() {
        return fn.apply( obj, arguments );
    }
}
var obj = {
    a: 2
};
var bar = bind(foo, obj);
var b = bar(3); // 2 3
console.log(b); // 5
```
ES5内置了Function.prototype.bind，bind会返回一个硬绑定的新函数，用法如下。
```javascript
function foo(something) {
    console.log(this.a, something);
    return this.a + something;
}
var obj = {
    a: 2
};
var bar = foo.bind(obj);
var b = bar(3); // 2 3
console.log(b); // 5
```
2. API调用的“上下文”
JS许多内置函数提供了一个可选参数，被称之为“上下文”（context），其作用和bind()一样，确保回调函数使用指定的this。这些函数实际上通过call()和apply()实现了显式绑定。
```javascript
function foo(el) {
    console.log(el, this.id);
}
var obj = {
    id: "awesome"
}
var myArray = [1, 2, 3]
// 调用foo()时把this绑定到obj
myArray.forEach(foo, obj);
// 1 awesome 2 awesome 3 awesome
```
## 四、new绑定
```javascript
function Func() {};
var func = new Func();
```
看看new操作符具体做了什么就明白new绑定了。
1. 创建一个空对象
```javascript
var obj = new Object();
```
2. 设置原型链
```javascript
obj.__proto__ = Func.prototype;
```
3. 让Func中的this指向obj，并执行Func的函数体。
```javascript
var result = Func.call(obj);
```
4. 判断Func的返回值类型：
如果是值类型，返回obj。如果是引用类型，就返回这个引用类型的对象。
```javascript
if (typeof(result) == "object"){
  func = result;
} else {
    func = obj;;
}
```
## 五、箭头函数绑定
箭头函数无法使用上述四条规则，而是根据外层（函数或者全局）作用域（词法作用域）来决定this。
1. 正常调用
```javascript
// 普通函数
function foo1(){     
    console.log(this.a);
}
// 箭头函数
var foo2 = () => {   
    console.log(this.a);
}
var a = 2;
var obj1 = { 
    a: 3,
    foo: foo1
};
var obj2 = { 
    a: 3,
    foo: foo2
};
// 调用普通函数
obj1.foo(); // 3
// 调用箭头函数
obj2.foo(); // 2 根据外层作用域，obj2.foo指向的foo2函数的外层是全局对象，因此输出2
foo2.call(obj2); // 2 ，箭头函数中显示绑定不会生效
```
2. 函数回调
```javascript
// 回调函数是普通函数
function foo1(){ 
    return function(){
        console.log(this.a);
    }   
}
// 回调函数是箭头函数
function foo2(){ 
    // 此处返回的箭头函数的外层是foo2被调用的作用域，foo2将在obj2.foo()时被调用，此时的外层作用域时obj2
    return () => {
        console.log(this.a);
    }   
}
var a = 2;
var obj1 = { 
    a: 3,
    foo: foo1 
};
var obj2 = { 
    a: 3,
    foo: foo2
};
// 执行普通回调
var bar1 = obj1.foo();
bar1(); // 2
// 执行箭头回调
var bar2 = obj2.foo();
bar2(); // 3
```
# **优先级**
优先级按照下面的顺序来进行判断: 
1. 函数是否在new中调用(new绑定)？如果是的话this绑定的是新创建的对象。 
2. 函数是否通过call、apply(显式绑定)或者硬绑定调用？如果是的话，this绑定的是指定的对象。 
3. 函数是否在某个上下文对象中调用(隐式绑定)？如果是的话，this绑定的是那个上下文对象。 
4. 如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到undefined，否则绑定到全局对象。

# **绑定例外**
在显示绑定中，把null或者undefined作为this的绑定对象传入call、apply或者bind，这些值在调用时会被忽略，实际应用的是默认规则。
1. 被忽略的this
```javascript
function foo() { 
    console.log(this.a);
}
var a = 2;
foo.call(null); // 2
foo.call(undefined); // 2
```
其实这不符合预期要求，如果某个函数确实使用了this，那会使用默认绑定把this绑定到全局对象上去。
安全的做法：传入一个特殊的对象（空对象），把this绑定到这个对象上，这样才不会对你的程序产生任何副作用。
JS中创建一个空对象最简单的方法是Object.create(null)，这个和{}很像，但是并不会创建Object.prototype这个委托，所以比{}更空。
```javascript
function foo(a, b) {
    console.log( "a:" + a + "，b:" + b );
}
// 创建一个空对象
var ø = Object.create(null);
// 把数组”展开“成参数
foo.apply(ø, [2, 3]); // a:2，b:3

// 使用bind()进行柯里化
var bar = foo.bind(ø, 2);
bar(3); // a:2，b:3 
```
2. 间接引用
间接引用时，调用这个函数会应用默认绑定规则。间接引用最容易在赋值时发生。
```javascript
// p.foo = o.foo的返回值是目标函数的引用，所以调用位置是foo()而不是p.foo()或者o.foo()
function foo() {
    console.log(this.a);
}
var a = 2;
var o = { a: 3, foo: foo };
var p = { a: 4 };
o.foo(); // 3
(p.foo = o.foo)(); // 2
```
3. 软绑定
硬绑定可以把this强制绑定到指定的对象（new除外），防止函数调用时使用默认绑定规则。但是会降低函数的灵活性，使用硬绑定之后就无法使用隐式绑定或者显式绑定来修改this。
如果给默认绑定指定一个全局对象和undefined以外的值，那就可以实现和硬绑定相同的效果，同时保留隐式绑定或者显示绑定修改this的能力。
```javascript
// 默认绑定规则，优先级排最后
// 如果this绑定到全局对象或者undefined，那就把指定的默认对象obj绑定到this,否则不会修改this
if(!Function.prototype.softBind) {
    Function.prototype.softBind = function(obj) {
        var fn = this;
        // 捕获所有curried参数
        var curried = [].slice.call(arguments, 1);
        var bound = function() {
            return fn.apply(
                (!this || this === (window || global)) ? obj : this,
                curried.concat.apply(curried, arguments)
            );
        };
        bound.prototype = Object.create(fn.prototype);
        return bound;
    };
}
```
使用：软绑定版本的foo()可以手动将this绑定到obj2或者obj3上，但如果应用默认绑定，则会将this绑定到obj。
```javascript
function foo() {
    console.log("name:" + this.name);
}
var obj = { name: "obj" },
    obj2 = { name: "obj2" },
    obj3 = { name: "obj3" };
// 默认绑定，应用软绑定，软绑定把this绑定到默认对象obj
var fooOBJ = foo.softBind(obj);
fooOBJ(); // name: obj 
// 隐式绑定规则
obj2.foo = foo.softBind(obj);
obj2.foo(); // name: obj2 <---- 看！！！
// 显式绑定规则
fooOBJ.call(obj3); // name: obj3 <---- 看！！！
// 绑定丢失，应用软绑定
setTimeout(obj2.foo, 10); // name: obj
```
# **总结**
我们在使用js的过程中，对于this的理解往往觉得比较困难，再调试过程中有时也会出现一些不符合预期的现象。很多时候，我们都是通过一些变通的方式（如：使用具体对象替换this）来规避的问题。