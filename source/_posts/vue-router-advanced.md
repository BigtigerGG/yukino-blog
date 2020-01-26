---
title: Vue Router 进阶
date: 2020/1/26 18:35:38
tags:
- Vue Router
categories:
- Vue
- Vue Router
---
# **Vue Router 进阶**

这篇文章包含 Vue Router 的进阶内容，以及自己的经验总结。  

## **导航守卫**

在应用中的页面 URL 改变的时候，Vue Router 提供了一系列的钩子函数来处理路由改变过程中需要  
处理的逻辑。这些钩子函数就是导航守卫，这里的导航是指：*当前的路由发生了改变（也就是说页  
面的 URL 发生了改变）*。  

需要注意的是，**路由中参数，或者查询的改变并不会影响进入/离开的导航守卫**。这时想要应对这些  
变化，需要观察 `$route` 对象或者使用 `beforeRouteUpdate` 导航守卫。  

导航守卫根据作用域的不同分为：  

- 全局守卫
- 路由独享守卫
- 组件内守卫

## **全局守卫**

全局守卫在所有的路由变换时都可以被触发。按照触发时间不同被分为：  

- 全局前置守卫
- 全局解析守卫
- 全局后置钩子  

### **全局前置守卫**  

使用 `router.beforeEach()` 可以注册一个全局前置守卫：  

    const router = new VueRouter({
        routes
    })
    router.beforeEach((to, from, next)=>{
        // ...
    })

可以注册一个或者多个这样的守卫，当一个导航被触发的时候，这些守卫按照创建的顺序依次被调用  
守卫们时异步解析执行，在所有守卫解析完成前路由的变化过程会一直处于 **等待中**  
每一个守卫方法接受三个参数：  

- `to:Route` : 即将要进入的 路由对象  
- `from:Route` : 当前导航正要离开的路由对象  
- `next : Function` : **这个方法必须被调用从而保证这个钩子被成功解析**  
  - `next()`:进行管道中的下一个钩子。如果所有的导航钩子执行完了，则导航的状态则是： **conformed**。
  - `next(false)` ：中断当前的导航。如果当前的 URL 改变了，URL 会被重置到 `from` 路由对应的位置。  
  - `next('/')` 或者 `next({path:'/'})` : 跳转到一个不同的地址，当前的导航被中断，然后进入一个新的导航。
  - `next(Error)` ： 传入一个 `Error` 实例， 导航被终止并将错误传递到 `router.onError()` 注册过的回调函数。  
  
### **全局解析守卫**  

使用 `router.beforeResolve` 来注册一个全局解析守卫。它的作用和全局前置守卫类似，唯一的区别是全局解析守卫的触发时间是在当前导航被确认之前，在所有的组件和组件内守卫解析完毕时才被调用。  

### **全局后置守卫**  

顾名思义，全局后置守卫在当前的路由离开的时候被触发。这种守卫不会接受 `next` 函数作为回调，也不会改变导航本身。  

## **路由独享的守卫**  

在路由对象中可以配置这个路由独享的导航守卫：  

    const router = new VueRouter({
        routes: [
            {
                name:'foo',
                path:'/foo',
                component:Foo,
                beforeEnter: (to, from, next)=>{
                    //...
                }
            }
        ]
    })

## **组件内的守卫**  

可以在组件内直接定义这些导航守卫：  

- `beforeRouterEnter`
- `beforeRouterUpdate`
- `beforeRouterLeave`

`beforeRouterEnter` 守卫不能直接访问组件实例 `this` ，因为在触发这个钩子函数的时候，这个组件还没有被创建。  

不过可以在这个钩子的 `next` 参数中接受一个 `vm` 组件对象作为当前组件实例。在导航被确认的时候执行这个回调函数：  

    beforeEnter: (to, from, next)=>{
        next(vm=>{
            // 这个vm作为当前的组件实例
        })
    }

其他两个组件内守卫因为被触发的时候组件已经被创建，所以可以访问 `this`。

## **完整的导航解析过程** 

1. 导航被触发。
2. 在失活的组件里调用离开守卫。
3. 调用全局的 beforeEach 守卫。
4. 在重用的组件里调用 beforeRouteUpdate 守卫 (2.2+)。
5. 在路由配置里调用 beforeEnter。
6. 解析异步路由组件。
7. 在被激活的组件里调用 beforeRouteEnter。
8. 调用全局的 beforeResolve 守卫 (2.5+)。
9. 导航被确认。
10. 用全局的 afterEach 钩子。
11. 触发 DOM 更新。
12. 用创建好的实例调用 beforeRouteEnter 守卫中传给 next 的回调函数