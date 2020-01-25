---
title: Vue 组件基础
date: 2020/1/25 20:39:00
tags:
- Vue Component
categories:
- Vue 
- Vue Base
---

# 组件

这篇文章概述了 Vue 的组件使用基础。更深入的组件了解 ==> [**深入了解 Vue 组件**](/yukino-blog/Vue/Vue-Base/advanced-component/)

## 基本介绍

### 定义
&emsp;&emsp;组件是可以复用的 Vue 实例，它可以被重复使用很多次，每当使用一个组件，  
就相当于 `new` 了一个新的 Vue 实例，所以它们名字相同的属性被各自维护。

### 示例
    // 定义一个 button-counter 的新组件
    Vue.component('button-counter', {
        data: function(){
            return {
                count:0
            }
        },
        template: "<button @click='count++'>click {{count}} {{count > 1 ? 'times' : 'time'}}</button>
    });
    new Vue({el:"component-demo"});

组件是可复用的 Vue 实例，且带有一个名字，可以在一个根 Vue 实例中以 html 标签的形，  
使用这个组件。
    
    // 在根实例中使用这个组件
    <div id='component-demo'>
        <button-counter></button-counter>
        <button-counter></button-counter> // 组件可以复用多次
    </div>

### **注意**
&emsp;&emsp;在注册组件的时候，data属性需要写成一个返回数据对象的函数，这样每个组件都有自己  
各自的数据，这份数据不会被共享。如果不写成函数，那么每一个组件都共享一份数据。然而  
组件应该是独立的。

*正确的：*  

    Vue.component('button-counter', {   
        data: function(){   
            return {    
                count: 0    
            }   
        }   
    })  

*错误的*  ‘

    Vue.component('button-counter', {
        data: {
            count: 0
        }
    })


&emsp;&emsp;在组件中书写字符串模板的时候，这个模板必须是一个单根元素（也就是有且仅有一个顶层  
节点）。

*正确的*

    template: `
        <div>
            <h2> this is a title </h2>
            <p>hello world</p>
        </div>
    `

*错误的*

```
template: `
    <h2> this is a title </h2>
    <p> hello world </p>
`
```


## 通过prop传递数据
### prop 的定义
&emsp;&emsp;prop是一个可以在组件上注册的一些自定义属性。当一个值传递给一个prop特性的时候，它   
就变成了那个组件实例的一个属性。例如，给一个博文组件传递一个标题，我们可以用一个`props`  
选项将其包含在该组件可接受的prop列表中

### 使用prop

&emsp;&emsp;在组件的定义中，props 字段作为一个prop列表，在模板中，可以使用这些prop。使用的方法  
是：将父组件传递的数据绑定在prop中，那么这个组件的prop就可以作为一个独立的属性来使用。  
例如：

```
Vue.component(post-item, {
    props:[post],
    template: `
        <div>
            <h2>{{post.id}}. {{post.text}}</h2>
            </br>
        </div>
        `
})
new Vue({
    el:'component-prop',
    data:{
        postList:[
            {id:0, text:"hello world"},
            {id:1, text:"awsome vue"}
        ]
    }
})
```

###

    <div id='component-prop'>
        <post-item v-for="postItem of postList"
                   v-bind:post="postItem"
                   v-bind:key="postItem.id"/>
    </div>












