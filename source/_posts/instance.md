---
title: Vue 实例
date: 2020/1/20 23:21:25
tags:
- Vue Instance
categories:
- Vue
- Vue Base
---
# Vue 实例  
  
## 创建一个 Vue 实例

每一个 Vue 应用都是通过 `Vue` 函数创建一个新的 Vue 实例开始的  

    var vm = new Vue({
        // 选项
    });  

当创建一个 Vue 实例时，可以传入一个选项对象。  
每一个 Vue 应用通过 `new Vue` 创建的**根 Vue 实例**，以及可选的，嵌套的、可复用的*组件* 树  
组成。
  
## 数据与方法

### 数据  

数据作为一个 Vue 实例的属性，当这个属性被改变时，视图将会产生响应(也就是同步的改变)，因为这个属性被加入到了 Vue 的响应式系统中了。

为一个 Vue 实例添加数据的方式如下:  

    var vm = new Vue({
        data: {
            msg: "hello vue!"
        }
    });

这样就为 vm 这个 Vue 实例添加了一个属性名为 msg 的数据，他的值为 “hello vue” 。

### 方法

方法作为一个 vue 实例的方法，通过在 vue 实例中定义它们，就可以在一些需要动态改变数据的地方用  
到这些方法

为一个 Vue 实例添加方法的方式如下：

    var vm = new Vue({
        data: {
            msg: "hello vue"
        },
        methods: {
            reverseMsg: function(){
                this.msg = this.msg.split('').reverse().join('');
            }
        }
    });

这里为 vm 这个 vue 实例添加了一个 `reverseMsg` 的方法,  这个方法将实例中的msg属性这个字符串反  
转了
> **<font color='red' size='4'>注意</font>**  
> 不要在选项属性或回调上使用箭头函数，比如 `created: () => console.log(this.a) `  
> 或 `vm.$watch('a', newValue => this.myMethod())`。因为箭头函数并没有 `this`， 
> `this` 会作为变量一直向上级词法作用域查找，直至找到为止，经常导致 `Uncaught   
> TypeError: Cannot read property of undefined `或 `Uncaught TypeError:  
> this.myMethod is not a function` 之类的错误。

## Vue  实例的生命周期
  
![Vue实例生命周期示意图][lifecycle]











[lifecycle]: /yukino-blog/images/vue-instance-lifecycle.png
