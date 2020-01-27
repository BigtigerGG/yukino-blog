---
title: Vuex 基础
date: 2020/1/27 16:51:29
tags:
- Vuex
categories:
- Vue
- Vuex
---
# **Vuex 基础**

这篇文章记录 Vue生态的状态管理模式—— Vuex，同时也记录了自我见解。  

## **什么是状态管理模式**  

通常，**一个 Vue 组件的数据被称为这个组件的状态**，它是驱动应用的数据源。Vue 的视图，也就是模板以声明式的方式将这些状态映射到 DOM 中。同时，Vue 的响应式机制能够根据这些状态的不同将变换的数据在视图中也能得到变化。  

通常，在简单的 Vue 组件中，数据的流动方向是“单向的“。如下图所示：  

![flow](/yukino-blog/images/flow.png)  

单项数据流让 Vue 组件的可预测性，可维护性得到了保证。  

但是当组件之间需要共享一些状态的时候，单项数据流很容易被破坏：  

- 多个视图依赖同一个状态（即多个组件需要共享同一份数据）
- 来自不同视图的行为需要变更同一个状态（即多个组件需要只改变一份数据对象）  

使用 props 传参可能在一定程度上能解决第一个问题，但是当组件的数量增多，这些嵌套的组件会让传参变得十分繁琐，降低了代码的复用性。并且传参对于兄弟组件之间的数据传递无能为力。  

对于问题二，如果使用父子组件之间直接引用，通常会导致不可维护的代码，因为它直接破坏了 Vue 的”单项数据流“理念。  

为了解决这个问题，Vue 将这些组件之间需要共享的状态抽离出来，并在此基础上继承了一些额外的功能和框架，形成了 Vue 的状态管理模式—— Vuex。  

## **合理地使用 Vuex**  

Vuex 始终是为了简化大型SPA（单页应用）的开发，如果在一些简单的应用中使用Vuex，它可能是多余且繁琐的。  

在简单的应用中，可以使用一个简单的 `store` 模式：  

    const store = {
        debug: true, // or false
        state:{
            sharedMessage: 'hello world'
        },
        clearSharedMessageAction(){
            // doSomeDebugThing
            if(debug) window.console.log("...")
            // doOtherThings
            // ...
            this.state.sharedMessage = ''
        },
        setSharedMessageAction(newValue){
            // doSomeDebugThing
            if(debug) window.console.log("...")
            this.state.sharedMessage = newValue
        }
    }

    new Vue({
        data(){
            return {
                sharedState: store.state,
                privateState: {
                    privateMessage: "private message"
                }
            }
        },
        methods:{
            clearSharedMessage: store.clearSharedMessageAction,
            setSharedMessage: value => store.setSharedMessageAction(value)
        },
        template: `
            <div>
                SharedMessage:
                {{sharedState.sharedMessage}}
                <br/>
                <input v-bind:value="sharedState.sharedMessage"
                       v-on:input="setSharedMessage($event.target.value)"
                />
            </div>
        `
    })

## **核心概念**  

### **State**

状态分布在 Vue 系统的各个组件中，这些状态通过 Vuex 解析成一个单一的状态树。一个 state 对象就包含了应用的全部层级状态。 

在入口文件中（main.js)，将 store 对象作为根实例 `app` 的一个选项：  

    const store = new Vuex.Store(Store) // 这里假设 Store 是模块系统导入的  

    new Vue({
        store, // store: store 的简写
        render:h => h(App)
    }).$mount('#app')  

这样，通过根实例的 `store` 选项，所有的子组件将其作为实例属性，通过 `this.$store` 来访问全局的单例 store 。通常，store 中的属性会作为组件中的计算属性，改变这些属性的 `Action` 会由组件中的 `methods` 中的方法来提交。

例如：  

    const Counter = {
        template: `
            <div>{{count}}
        `,
        computed:{
            count(){
                return this.$store.state.count
            }
        },
        methods:{
            increment(){
                this.$store.commit('increment')
            }
        }
    }

**Tips:**  

1. 当一个组件需要多个状态的时候，为了避免重复的为每一个状态声明计算属性，可以使用一个 `mapState` 的辅助函数来解决这个问题：  

        import { mapState } from 'vuex' // 对象解构

        export default{
            name: 'Counter',
            computed: mapState({
                // 在这个辅助函数中，
                // 可以访问将state作为函数的参数以访问 this.$store.state
                count: state=>state.count,
                // 可以将 state 中的状态作为字符串参数，等同于 this.$store.state
                // 这里将其作为组件计算属性的别名
                countAlias: 'count',
                // 通过使用普通函数，访问组件实例 this
                countPlusLocalCount(state) {
                    return state.count + this.localCount
                }
            })
        }  

        // 还可以在 mapState 函数中传入一个字符串数组，
        // 将 state 中的状态作为组件中的计算属性

        computed: mapState(['count'])

2. 既想要组件私有的计算属性，由想要共有的状态时，可以使用对象展开运算符 `...` ,将状态混入到计算属性对象中：  

        computed: {
            localComputed(){
                /*...*/
            },
            ...mapState(){
                /*...*/
            }
        }
    


