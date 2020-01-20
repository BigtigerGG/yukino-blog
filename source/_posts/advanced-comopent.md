---
title: 深入了解 Vue 组件
date: 2020/1/20 22:57:25
tags:
- Vue Component
categories:
- Vue
- Vue Base
permalink: /vue/vuebase/深入了解vue组件
---

# **深入了解组件**

这篇文章记录组件的高级知识，和自己对组件的总结。

## **组件的注册**

&emsp;&emsp;组件的注册包括全局注册和局部注册两种。
- 全局注册  

    即通过 `Vue.component({/\*\*...\*\*/})` 来注册组件。  
    全局注册的组件可用于任何新创建的 Vue 实例，也包括组件之间相互的使用。

- 局部注册  

    全局注册的组件，即便不想使用这个组件，它任然会包含在构建结果中。这造成了  
    用户下载的 JavaScript 的无谓的增加。  
    通过使用局部组件，可以避免这个问题。局部注册的方法如下：  

        // 通过使用变量，将组件的引用赋值给一个变量
        var ComponentA = { /\* ... \*/ }
        var ComponentB = { /\* ... \*/ }
        // 然后在一个 Vue 实例中的 components 定义你想使用的组件
        new Vue({
            el: "#app",
            components: {
                'component-a' : ComponentA,
                'component-b' : ComponentB
            }
        })
    
    需要**注意**的是：  
    - 对于 `components` 对象中的每一个属性来说，其属性名就是自定  
    义元素的名字，其属性值就是某一个组件的变量名。
    - 局部注册的组件在其子组件中不可用。如果想要在子组件中使用，  
    需要在子组件中注册这个组件。 

## **Prop**

### **prop 的类型**

通常prop的传递通过字符串数组的形式，比如这样：  
    
    props:['prop1', 'prop2', 'prop3']

但是，通常我们希望每一个 `prop` 都有自己指定值的类型。这时，我们可以用对象的形式  
列出 `prop`，这些属性的名称和值分别是prop各自的名称和类型。这样也可以对 prop。  
进行类型检查。例如：  

    props: {
        title: String,
        likes: Number,
        isPublished: Boolean
    }

### **prop 的传递应该是一个单向数据流**

所有的prop都使得其父子prop之间形成了一个**单向下行绑定**：父级prop的更新会向下流动  
到子组件中，但是反过来却不行。这样能够防止从子组件意外改变父级组件的状态，从而  
导致应用的数据流向难以理解。

每次父级组件发生更新的时候，子组件中的所有prop都会刷新为最新的值。这意味着不应该  
在一个子组件内部改变 prop。 

### **非 prop 的 Attribute**

一个非 prop 的 Attribute 是指传向一个组件，但是该组件并没有相应 prop 定义的Attribute。  
但是一个组件可以接受任意的 attribute ，**而这个attribute会被添加到这个组件的根元素上**。 

### **替换/合并已有的 attribute**

对于绝大数的 attribute，外部提供给组件的值会替换掉组件内部根元素设定好了的值。但是   
style 和 class 属性会合并两个值。

### **禁用 attribute 继承**

当不希望一个组件内部的根元素(组件)继承外部的属性，那么可以将这个组件的 `inheritAttrs`  
属性设置为 `false`。例如：

    Vue.component('demo-component', {
        inheritAttrs: false,
        template: /* ... */
    })

这样书写给 `demo-component` 这个组件的属性就不会继承到它的子组件中。

值得注意的是，这样做的好处在于：  
当一个组件的模板嵌套着很多子组件时，我们不希望配置给这个组件的属性直接继承在这个组件  
的根组件上，而是继承给某一个我们希望的子组件上面。 

具体的做法是：  

1. 禁用属性继承（即 `inheritAttrs: false`）
2. 将组件的实例属性 [`$attrs`](https://cn.vuejs.org/v2/api/#vm-attrs) , 绑定到指定的子组件或者元素上面  

需要**注意**的是：  

`inheritAttrs: false` 并**不会**影响到 style 和 class 的绑定。也就是说，这两个属性  
还是会绑定在模板的根元素（子组件）上面。

## **自定义事件**

### **事件名**

不同于组件和prop，事件名不会存在任何的自动化的大小写转换，而是触发的事件名必须  
完全匹配这个事件所用的名称。  

建议始终使用短横线分割的小写标识符。

### **自定义组件的 `v-model`**

`v-model` 用在组件上面的时候，会默认利用名为 `value` 的 prop 和名为 `input` 的事件，  
当使用一些像单选框，复选框这样的输入表单控件的时候，这个指令就达不到我们想要  
的效果。这个时候，我们可以使用 Vue 实例中的 `model` 选项用来避免这样的冲突。  
例如：  

    Vue.component('base-checkbox', {
        model: {
            prop: 'checked',
            event: 'change'
        },
        props: {
            checked: Boolean
        },
        template: `
            <input type="checkbox"
                   :checked="checked"
                   @change="$emit('change', $event.target.checked)"
            >
        `
    })

这样当在 `base-checkbox` 这个组件上使用 `v-model` 的时候，将会创建一个和checked  
双向绑定的响应式关系。

### **使用 $listeners 属性将原生事件绑定到组件中**

绑定原生 DOM 事件的方法是在 `v-on` 这个指令上面加上 `.native` 这个修饰符。    
但是，当在一个组件上绑定一个原生事件并且组件内部尝试监听一个特定的元  
素的时候，父级的 .native 监听器将会失败。Vue 不会报错，但是相应的事件处  
理函数将不会执行。  

这个时候，可以使用 Vue 提供的 $listeners 属性将组件上所有的事件监听器指  
向这个组件上的指定元素。具体的方法是：

    // 例如我们想要将一个组件上面的 focus 事件监听器指向这个组件内部的一个 input 元素
    Vue.component('base-input', {
        inheritAttrs: false,
        props: ['input']
        computed: {
            inputListeners: function(){
                let vm = this;
                // 使用 Object.assign 方法将多个对象合并
                return Object.assign({}, 
                    this.$listeners,
                    {
                        input: function(){
                            vm.$emit('input', $event.target.value);
                        }
                    }
                )
            }
        },
        template: `
            <label>
                <input type="text"
                       v-bind="$attrs"
                       :input="input"
                       v-on="inputListeners">
            </label>
        `
    })

这样使用 `base-input` 就相当于使用普通的 `input` 了，也就是说，这个组件  
已经是透明的了。

## **插槽**

### **插槽的基本使用** 

当我们想要在一个组件中插入一段模板或者HTML的时候，可以使用 Vue 提供  
的插槽机制。如果不使用插槽，并且在一个自定义组件中插入了一段模板或者  
HTML的话，那么 Vue 将不会渲染这一部分的内容。

插槽的具体使用方式如下：

    // 假如这是一个名为 <my-slot> 组件的模板内容
    <label>
        <span>你好</span>
        <slot></slot>
    </label>
###
可以像使用 html 元素一样使用这个组件

    <my-slot>
        Vue
    </my-slot>

### 
经过 Vue 渲染后的结果为：

    你好Vue

### **具名插槽**

当想要使用多个插槽的时候，可以在 `<slot>` 元素上面添加一个 `name` 特性，这个    
特性可以作为识别这个插槽的标识。然后在具体使用这个模板的时候，使用 `v-slot`  
指令，将 name 作为这个指令的参数，作用在一个模板组件 `<template>` 上面。那  
么这个模板的内容将被填充在这个插槽里面。未使用 name 特性的 `<slot>` 将会有  
一个默认的 name —— default。未使用 `<template>` 的内容将会视作默认插槽的内  
容。使用这种方式定义的插槽叫做**具名插槽**。

**注：**
和其他可缩写指令一样，`v-slot` 指令可以缩写为 `#`, 但必须指定参数，也就是  
说不能这样用：`#=`
### **作用域插槽**

当想在父级组件作用域中使用子组件中作用域中的数据的时候，可以使用 Vue 提供的  
作用域插槽的解决方案。
具体的使用方法如下：

    // 假如在父级组件或者 rawHtml 中需要使用一个叫做 isCompleted 的属性
    // 这是子组件
    var todoList = {
        props:{
            todoItems: Array
        },
        computed: {
            filteredTodoItems: function(){
                return this.todoItems.filter(item => item.title);
            }
        },
        // 绑定在<slot>标签中的特性被称为插槽 prop , 这个prop在父级作用域中
        // 可以通过 v-slot 指令提供的插槽 prop 名字来访问
        template: `
            <ul>
                <li v-for="item of filteredTodoItems
                    :key="item.id">
                    <slot v-bind:todo="item">
                        {{item.title}}
                    </slot>
                    <input type="checkbox" v-model="item.isCompleted">
                </li>
            </ul>
        `
    };

    new Vue({
        el:"#component-slot-prop",
        data: {
            todoItems: [
                {
                    id: 0,
                    title: "学习 Vue 的插槽用法",
                    isCompleted: false
                },
                {},
                {
                    id: 3,
                    title: "学习 JVM 的知识",
                    isCompleted: false
                }
            ]
        },
        components: {
            "todo-list": todoList
        }
    });

    // 在父级组件中访问这个插槽 prop，根据 isCompleted 的值来条件渲染
    <div id="component-slot-prop">
    <!-- 这里使用解构对象给 v-slot 指令赋值 -->
        <todo-list v-slot="{ todo }"
                   :todo-items="todoItems">
            <span v-if="todo.isCompleted"> ✔ </span>
            {{todo.title}}
        </todo-list>
    </div>

**注：**  
作用域插槽的更多用法 ==> [Vue Virtural Scroller](https://github.com/Akryum/vue-virtual-scroller "一个基于 Vue 的虚拟滑动器，旨在提高性能")

## **动态组件 & 异步组件**

当我们想要根据条件，动态渲染某一个组件的时候，可以使用 Vue 提供的  
动态组件功能。例如，当我们选择一个导航栏标题的时候，内容区域根据标  
题的不同使用不同的组件。

具体的做法是使用一个 Vue 提供的内置组件 `<component>`，在这个组件上使  
用 `v-bind:is` 指令来选择不同的组件。

例如一个组件的模板是这样的：

    <button v-for="tab of tabs"
            v-on:click="currentTab=tab"/>
    <component v-bind:is="currentTabComponent">

那么当我们点击不同的导航栏标题的时候，`<component>` 组件就会根据 `currentTab`  
的不同，选择不同的组件。

### **在动态组件上面使用 `keep-alive` 来使得组件得以缓存**

一般情况下，动态组件会随着组件的切换，重新渲染一个新的组件对象。也就是说，当  
我们切换组件的时候，之前那个组件的内容在重新切回这个组件的时候会消失不见。为  
了保持之前那个组件的状态，可以使用 `<keep-alive>` 元素将 `component` 包裹起来，这  
样失活的组件将会被缓存起来。

具体的做法是：  

    <keep-alive>
        <component v-bind:is="currentTabComponent">
    </keep-alive>

需要注意的是：这个需要保持活性的组件应该始终拥有自己的名字，无论是通过组件的  
`name` 选项，还是通过全局注册/局部注册。

### **异步组件**
当想要异步从服务器中获取某一个组件的时候，我们可以使用**异步组件**的功能。Vue提供  
的异步组件的使用方法是通过一个工厂函数来定义一个需要异步加载的组件，Vue 只有在  
这个组件需要被加载的时候才会运行这个工厂函数，且 Vue 会将这个已经加载好了的组件  
缓存起来供未来渲染。  
像这样定义一个异步组件：  

    Vue.component('async-component',function(resolve, reject){
        // 这里使用 setTimeout 来模拟异步获取
        setTimeout(()=>{
            // 假如 data 是获取到的数据或者模板
            if(data){
                resolve(data) // 这个是 promise 对象的使用，表示成功获取到数据
            }else{
                reject(reason) // 假如 reason 是失败的原因
            }
        })
    })

其中，当我们成功获取到数据的时候，Vue 会调用 `resolve` 回调函数，反之则调用 `reject`   
回调函数。

## **处理边界情况**

### **访问子组件或者子元素**
当我们想要在父级组件中访问子组件或者子元素对象的时候，可以在这个子组件或者子  
元素中添加 ref 这个属性。这样我们就可以在父组件中通过 $refs 这个组件实例属性来获  
取这个组件中所有带有 ref 这个属性的子组件对象。

需要**注意**的是：通过这种方式获取的子组件属性并不是响应式的，这仅仅是为了获取子  
组件临时状态的一种方法, 不应该在模板或者计算属性中访问 `$refs`

### **依赖注入**
当我们想要让一个组件的所有子组件都能够获取它的一个属性的话，我们可以使用**依赖注入**。  
具体的做法是在这个组件中添加一个 `provide` 选项，选项中提供一个共享属性，例如：

    provide: function(){
        return {
            getMap: this.map
        }
    }

相应的，在这个组件的子组件中提供一个 `inject` 选项来注入这个属性，像这样：  

    inject:['getMap']

那么，这个子组件就可以访问这个 `map` 属性，而不需要知道这个属性来自于哪个父级组件。  
相应的，提供这个属性的父级组件也不知道那个子组件会使用它。

需要**注意**的是：这种做法并不是响应式的。也就是说，子组件获取到的这个属性即使更改了  
父级组件的那个属性并不会随之更改。

### **程序化事件侦听器**
当想要在程序流程中自定义某一个组件的事件监听和取消监听，可以使用：  
- `$on(event, eventHandler)` 监听一个事件
- `$once(event, eventHandler)` 一次性监听一个事件（只监听一次）
- `$off(event, eventHandler)` 取消监听一个事件

### **循环引用**

1. 递归组件

    组件是可以在模板中调用自己的，这样就形成了一个递归组件。但是需要注意设置递归  
    的退出条件（例如最终会得到一个 `false` 的 `v-if` 组件），否则这个组件会无穷递归下  
    去，Vue 会报一个 `max stack size exceed` 的错误。

2. 组件之间的循环引用

    当我们想要在组件之间实现类似一种*组合模式*的时候，也就是像文件系统那样，一个文  
    件可以是一个文件夹，也可以是一个文件，很容易出现这样一种组件的使用方式：  

        // <folder>组件的模板内容
        template: `
            <p>
                <span>{{folder.name}}</span>
                <folder-contents v-bind:children="folder.children"/>  
            </p>
        `
    ##
        // <folder-contents> 组件的模板内容
        template： `
            <ul>
                <li v-for="child of children">
                    <!--这里表示这个 child 是一个文件目录-->
                    <folder v-if="child.children" v-bind:folder="child"/>
                    <!--这里表示这个 child 是一个文件-->
                    <span v-else>{{child.name}}</span>
                </li>
            </ul>
        `
    
    看似这两个组件这么使用好像没什么问题，实际上，这里会出现一个逻辑上的矛盾：当 Vue   
    想要渲染 `folder` 组件的时候，发现它的子组件是 `folder-contents` ，当想要渲染  
    `folder-contents` 组件的时候，却发现需要渲染它的父组件。当使用全局注册这两个组件  
    的时候， Vue 会解决这个矛盾，但是当使用一个像 *webpack*  这样的模块系统的时候，组件  
    之间的依赖是通过导入一个单文件形成的，这样就会出现循环依赖的错误。  

    解决方法是给模块系统设置一个依赖点，也就是当渲染一个组件的时候，人为的设置它所依赖  
    的组件先不用渲染，只给一个定义，这样就退出了这个循环依赖。在这个例子中，`<folder>`  
    组件就是这个点，可以在组件的生命周期钩子 `beforeCreate` 中先给一个定义：  

        beforeCreate: () => require("./folder-component.vue").default  
    
    或者使用 *webpack* 的异步导入方法 `import` ：  

        components: {
            "folder-component" : () => import("./folder-component.vue")
        }
