---
title: vue的过渡小记（多元素、多组件）
dropcap: true
date: 2018-11-06 11:30:37
tags: vue2.x
categories: vue
---
网页中，一般都存在切换的场景，vue提供了一个简单的切换过渡效果，用< transition>标签包裹要切换的部分即可。

# **多元素过渡**
### 绑定key
[vue官网](https://cn.vuejs.org/v2/guide/transitions.html#多个元素的过渡)上有写到：
> 当有**相同标签名**的元素切换时，需要通过***key***特性设置唯一的值来标记以让vue区分它们，否则vue为了效率只会替换相同标签内部的内容。即使在技术上没有必要，**在给< transition>组件中的多个元素设置key是宇哥更好的实践**

示例代码：
```html
    <transition>
        <button v-if="isEditing" key="save">Save</button>
        <button v-else key="edit">Edit</button>
    </transition>
```
上面栗子可以简写为：
```html
    <transition>
        <button :key="isEditing">{{isEditing ? 'Save' : 'Edit'}}</button>
    </transition>
```
### 过渡模式
vue提供两种过渡模式
+ ***in-out*** 新元素先进行过渡，完成之后当前元素过渡离开
+ ***out-in*** 当前元素先进行过渡，完成之后新元素过渡进入
```html
    <transition mode="in-out" name="fade">
        <!-- name属性的值是便于写样式 -->
    </transition>
```
# **多组件过渡**
多个组件过渡，就不需要***key***特性，使用[动态组件](https://lijing0906.github.io/post/vueis)就OK