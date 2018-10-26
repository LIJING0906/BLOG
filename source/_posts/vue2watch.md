---
title: vue2的监听watch小爆料
dropcap: true
date: 2018-08-22 12:16:00
tags: vue
categories: vue
thumbnail:
---
跟vue相爱相杀这么久了，今天第一次来爆点小料(主要还是为自己做笔记)------vue2的watch监听

## 路由监听,也适用于普通变量（基本数据类型）的监听
#### 写法一,普通监听
```javascript
    watch: {
        $route(nVal, oVal){
            console.log(nVal) //打印的是路由对象
        }
    }
```
#### 写法二，深度监听
```javascript
    watch: {
        handler: function(nVal, oVal){
            console.log(nVal) //打印的是路由对象
        },
        deep: true
    }
```
#### 普通监听和深度监听的区别
普通监听只能监听普通（基本数据类型）的变量，如果想监听对象或者数组的变化，就需要深度监听。
但这里路由是特殊的，既能用普通监听，也能用深度监听

#### 写法三，调用监听方法
```javascript
    watch: {
        '$route': 'getRoute' //$route:'getRoute'也OK 
    },
    methods: {
        getRoute(){
            console.log(this.$route.path) //打印的是路由对象的path属性
        }
    }
```
#### 写法四，立即执行或阻止复用
有这样一种场景，page/a跳转到page/b，这两个页面是同一个组件，结果就是路径变了，但是组件的内容没变，造成这个结果的原因是：vue-router检测到这是同一个组件，决定复用这个组件，
所以写在钩子里的方法不会执行。有两个解决方法：
1. 
```javascript
    '$route': {
        handler: 'getRoute', //处理方法，写在methods里即可
        immediate: true //立即执行 
    }
```
2. 
```html
    <router-view :key='$route.fullpath'></router-view>
```
通过绑定唯一的值来阻止复用，可以说是一劳永逸，但是会牺牲一点点性能，鱼和熊掌不可兼得

## 监听对象，也适用于数组的监听
前面的路由监听中已经提到深度监听能监听对象和数组的变化
#### 对象整体的监听
```javascript
    data(){
        return {
            aObj:{
                aProp: 'a',
                bProp: 'b'
            }
        }
    },
    watch: {
        aObj: {
            handler: function(nVal, oVal){
                console.log(nVal) //打印的是aObj这个对象
            },
            deep: true
        }
    }
```
#### 对象某个属性的监听,这个也适用于对vuex的监听
```javascript
    data(){
        return {
            aObj:{
                aProp: 'a',
                bProp: 'b'
            }
        }
    },
    computed: {
        getA(){
            return this.aObj.aProp
        }
    },
    watch: {
        getA: function(nVal, oVal){
            console.log(nVal)
        }
    }
```
#### 最后的最后
以上是我目前工作中用到过的vue2的监听，主要目的还是为自己记笔记
工作太忙，没有心思继续搞动态的hexo博客了，先暂时用静态的吧