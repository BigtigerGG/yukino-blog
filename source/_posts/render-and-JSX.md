---
title: 渲染函数和 JSX
date: 2020/1/25 21:55:00
tags:
- Render Function
- JSX
categories:
- Vue 
- Vue Base
---
# **渲染函数和 JSX**
vascript 框架，这也是 Vue 简单易学的原因。然而有些时候，我们需要  
Javascript 完全编程的能力，可以使用 Vue 提供的 **渲染函数（render）** 来做到这一点。

## **节点、树以及虚拟 DOM**

Vue 使用的元素（组件）节点和原生的 DOM 节点是不一样的，这个节点被 Vue 加工过，包含了  
这个节点的和它的子节点的信息。 这样的节点叫做”虚拟节点“（virtual node），所谓的虚拟 DOM  
就是由这些虚拟节点构成的 DOM 树所构成。

### **`createElement`** 参数

在 `render` 函数中，通常使用 `createElement` 函数来创建虚拟节点，下面是这个函数所接受的参数  

    /**
    * @return {VNode}
    */
    createElement(
        /**
        * @param {String | Object | Function}
        * 一个 Html 标签名、组件对象或者解析了上述任意
        * 一种的 async 函数，必填项
        */
        ’div',
        /**
        * @param {Object}
        * 一个对应模板属性的数据对象，例如 props，data 等，可选
        */
        {
            // props: {}    
            // data: function(){}
        },
        /**
        * @param {String | Array}
        * 子级虚拟节点列表
        * 也可以由子级文本字符串构成
        */
        [
            "strings",
            // createElement()
        ]
    )

需要**注意**的是，一个组件的中的所有 VNode 必须是唯一的，也就是所有的 VNode 都应该是一个独立  
对象。当我们想要渲染多个重复的 VNode 的时候，我们可以使用工厂函数，例如：

    createElement(
        'div',
        Array.apply(null, {length:20}) 
            .map(()=>{
                return createElement(
                    'p',
                    'hi'
                )
            })
    )

## **使用插件**
`Vue.use()` 使用插件。需要在调用 `new Vue()` 启动应用之前完成。

## **使用过滤器**

Vue 可以自定义过滤器，用于一些常见的文本格式化。过滤器一般用在两个地方: 模板表达式中，和`v-bind`  
表达式中。过滤器应该被添加在表达式的尾部，由管道符号（`|`）表示。  

例如：

    {{ message | capitalize}} <!-- 首字母大写 -->

    <div v-bind:id = "rawId | formatId">

过滤器可以被定义在 Vue 实例的 `filters` 选项下作为一个组件过滤器：

    filters: {
        capitalize: function(value){
            if(!value) return '';
            value = value.toString();
            return value.charAt(0).toUppercase() + value.slice(1);
        }
    }

也可以被定义为全局过滤器：

    Vue.filter('capitalize', function(value){
        if(!value) return '';
            value = value.toString();
            return value.charAt(0).toUppercase() + value.slice(1);
    })

需要**注意**的是  

- 当全局过滤器和局部过滤器重名的时候，会优先使用局部过滤器
- 过滤器会始终将表达式的值作为第一个参数
- 过滤器可以接受参数，例如：`message | capitalize(arg0, arg1)`, 但是该函数的第一个参数是 `message`

