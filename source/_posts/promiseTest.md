---
title: Promise面试题
dropcap: true
date: 2019-12-01 21:16:55
tags: 'javaScript'
categories: 'javaScript'
---
前面的文章梳理了一下有关浏览器进程、事件循环机制、微任务和红任务、Promise的相关知识，这篇文章想讲讲有关这些知识的一些面试题。
[参考文章](https://www.jianshu.com/p/65a71f3b8e35)
# **题目一**
```javascript
const promise = new Promise((resolve, reject) => {
    console.log(1)
    resolve()
    console.log(2)
})
promise.then(() => {
    console.log(3)
})
console.log(4)
// 输出结果如下：
// 1
// 2
// 4
// 3
```
**解读：** 
1. 代码从上往下被放入执行栈中，执行栈先执行所有的同步任务，当遇到异步任务时，把异步任务挂起，等待执行栈中没有同步任务了，就从任务队列中取出异步任务来执行，
2. Promise的创建是同步的，因此会先打印1，遇到异步任务resolve()就把resolve()推入事件队列，
3. 继续执行resolve()之后的同步任务，所以打印2，
4. promise.then()是异步任务，等待执行栈空闲再执行，
5. 执行同步任务打印 4，
6. 执行栈中没有同步任务，取出异步任务开始执行，因此打印 3。

# **题目二**
```javascript
const promise1 = new Promise((resolve, reject) => {
  setTimeout(() => {
    resolve('success')
  }, 1000)
})
const promise2 = promise1.then(() => {
  throw new Error('error!!!')
})

console.log('promise1', promise1)
console.log('promise2', promise2)

setTimeout(() => {
  console.log('promise1', promise1)
  console.log('promise2', promise2)
}, 2000)
// 执行结果如下：
// promise1 Promise { <pending> }
// promise2 Promise { <pending> }
// (node:50928) UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 1): Error: error!!!
// (node:50928) [DEP0018] DeprecationWarning: Unhandled promise rejections are deprecated. In the future, promise rejections that are not handled will terminate the Node.js process with a non-zero exit code.
// promise1 Promise { 'success' }
// promise2 Promise {
// <rejected> Error: error!!!
//    at promise.then (...)
//    at <anonymous> }
```
**解读：** promise 有 3 种状态：pending、fulfilled 和 rejected 。状态改变只能是 pending->fulfilled 或者 pending->rejected ，状态一旦改变则不能再变。上面 promise2 并不是 promise1 ，而是返回的一个新的 Promise 实例。
# **题目三**
```javascript
const promise = new Promise((resolve, reject) => {
  resolve('success1')
  reject('error')
  resolve('success2')
})

promise.then((res) => {
   console.log('then: ', res)
 }).catch((err) => {
   console.log('catch: ', err)
 })
 // 执行结果如下：
 // then: success1
```
**解读：**  Promise 的 resolve 或 reject 只有第一次执行有效，多次调用没有任何作用，呼应代码二结论： promise 状态一旦改变则不能再变。
# **题目四**
```javascript
Promise.resolve(1).then((res) => {
   console.log(res)
   return 2
 }).catch((err) => {
   return 3
 }).then((res) => {
   console.log(res)
 })
 // 执行结果如下：
 // 1
 // 2
```
**解读：** promise 可以链式调用。提起链式调用我们通常会想到通过 return this 实现，不过 Promise 并不是这样实现的。promise 每次调用 .then 或者 .catch 都会返回一个新的 promise，从而实现了链式调用。
[Promise](https://lijing0906.github.io/post/promise)中讲过Promise.resolve('success');等价于new Promise(resolve => { resolve('success'); });，因此先打印1，然后return 2，这个then被调用之后返回一个Promise，因此在第二个then时会打印2。
# **题目五**
```javascript
const promise = new Promise((resolve, reject) => {
  setTimeout(() => {
    console.log('once')
    resolve('success')
  }, 1000)
})

const start = Date.now()
promise.then((res) => {
  console.log(res, Date.now() - start)
})
promise.then((res) => {
  console.log(res, Date.now() - start)
})
// 执行结果如下：
// once
// success 1002
// success 1003
```
**解读：**  promise 的 .then 或者.catch可以被调用多次，但这里 Promise构造函数只执行一次。或者说 promise内部状态一经改变，并且有了一个值，那么后续每次调用 .then或者 .catch都会直接拿到该值。Date.now() - start的值看浏览器的执行效率，可能一样，可能相差一点点。
# **题目六**

```javascript
Promise.resolve().then(() => {
  return new Error('error!!!')
 }).then((res) => {
   console.log('then: ', res)
 }).catch((err) => {
   console.log('catch: ', err)
 })
 // 执行结果如下：
 // then: Error: error!!!
 //   at Promise.resolve.then (...)
 //   at ...
```
**解读：** .then或者 .catch中 return一个 error对象并不会抛出错误，所以不会被后续的 .catch捕获，需要改成其中一种：
```javascript
return Promise.reject(new Error('error!!!'))
throw new Error('error!!!')
```
因为返回任意一个非 promise的值都会被包裹成 promise对象，即 return new Error('error!!!')等价于 return Promise.resolve(new Error('error!!!'))。
# **题目七**
```javascript
const promise = Promise.resolve().then(() => {
   return promise
 })
promise.catch(console.error)
// 执行结果如下：
// TypeError: Chaining cycle detected for promise #<Promise>
//    at <anonymous>
//    at process._tickCallback (internal/process/next_tick.js:188:7)
//    at Function.Module.runMain (module.js:667:11)
//    at startup (bootstrap_node.js:187:16)
//    at bootstrap_node.js:607:3
```
**解读：** .then或 .catch返回的值不能是 promise本身，否则会造成死循环。类似于：
```javascript
process.nextTick(function tick () {
  console.log('tick')
  process.nextTick(tick)
})
```
# **题目八**
```javascript
Promise.resolve(1)
  .then(2)
  .then(Promise.resolve(3))
  .then(console.log)
// 执行结果如下：
// 1
```
**解读：** .then或者 .catch的参数期望是函数，传入非函数则会发生值穿透。
# **题目九**
```javascript
Promise.resolve().then(function success (res) {
  throw new Error('error')
 }, function fail1 (e) {
   console.error('fail1: ', e)
 }).catch(function fail2 (e) {
   console.error('fail2: ', e)
 })
 // 执行结果如下：
 // fail2: Error: error
 //   at success (...)
 //   at ...
```
**解读：** .then可以接收两个参数，第一个是处理成功的函数，第二个是处理错误的函数。.catch是 .then第二个参数的简便写法，但是它们用法上有一点需要注意：.then的第二个处理错误的函数捕获不了第一个处理成功的函数抛出的错误，而后续的 .catch可以捕获之前的错误。当然以下代码也能达到同样的效果：
```javascript
Promise.resolve().then(function success1 (res) {
  throw new Error('error')
 }, function fail1 (e) {
   console.error('fail1: ', e)
 }).then(function success2 (res) {
 }, function fail2 (e) {
   console.error('fail2: ', e)
 })
```
# **题目十**
```javascript
process.nextTick(() => {
  console.log('nextTick')
})
Promise.resolve().then(() => {
  console.log('then')
})
setImmediate(() => {
  console.log('setImmediate')
})
console.log('end')
// 执行结果如下：
// end
// nextTick
// then
// setImmediate
```
**解读：** process.nextTick和 promise.then都属于 microtask，而 setImmediate属于 macrotask，在事件循环的 check阶段执行。事件循环的每个阶段（macrotask）之间都会执行 microtask，事件循环的开始会先执行一次 microtask。
