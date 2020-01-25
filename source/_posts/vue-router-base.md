---
title: Vue Router 基础
date: 2020/1/25 20:44:00
tags:
- Vue Router
categories:
- Vue 
- Vue Router
permalink: /vue/vuerouter/VueRouter基础
---

# **基础**

Vue Router 作为 Vue.js 的官方路由器，主要应用在大型单页应用的路由管理。
Vue Router 拥有以下功能：

- 嵌套的路由 / 视图表
- 模块化的、基于组件的路由配置
- 路由参数、查询、通配符
- 基于 Vue.js 的过度系统的视图过渡效果
- 细粒度的导航控制
- 带有自动激活的 Css Class 的链接
- Html5 历史模式或者 Hash 模式
- 自定义的滚动条行为

## **基础使用**

在模块化的构建系统中，例如 webpack ，要使用 Vue Router 需要先安装：

    npm install vue-router

这样 vue-router 作为 Vue 的一个插件来使用，在 main.js （入口文件）中需要先使用这个插件  

    import VueRouter from "vue-router"

    Vue.use(VueRouter)

这样在整个 app 环境中，都可以使用这个插件。

要使用 Vue Router，应该首先定义一个路由器的列表，像下面这样：  

    const routes = [
        {
            path: '/foo', // url path ，即路由路径
            component: AComponent // 一个组件对象，可以从其他文件中导入
        },
        {
            path: '/bar',
            component: AnotherComponent // 另一个组件对象
        }
    ]

然后可以在入口文件 main.js 中使用这个列表对象（导入），新建一个 VueRouter 对象：  

    const router = new VueRouter({
        routes // 等效于：routes:routes
    })

最后将这个 `router` 注册为 app 的选项：  

    new Vue({
        router,
        render: h=>h(App)
    }).$mount("#app")

在模板中，  

- 使用 `<router-link to="url">` 标签为一个路由设置路径，其中，`to` 属性是路由的路径  
这个标签会被默认渲染成一个 `<a>` 标签。
- 使用 `<router-view>` 标签定义路由的出口，路由匹配到的组件将会渲染在这里面。  

通过注入路由器，可以使用：

- `this.$router` 访问路由器
- `this.$route`  访问当前路由

## **动态路由匹配**

可以在路由参数的 `path` 属性中为一个路由设置动态路径，形如这样：  

    {
        path: '/user/:id',
        component: User
    }

这样，所有的形如：/user/8 、/user/7 这样的 url 都将映射到相同的路由。  

当匹配到一个路由的时候，参数值会被设置到 `this.$route.params`, 这样就能够在每一个组件中  
访问这个参数，上面例子中的 `User` 组件就可以利用这个参数，更新这个组件的模板。   

也可以为一个动态路由设置多个“路径参数”， 对应的值会设置到 `$route.params` 中, 例如：  

**模式** | **匹配路径** | **route.params**  
-|-|-|
/user/:username | /user/llp | `{username: 'llp'}`  
/user/:username/post/:post_id | /user/llp/post/113 | `{username: 'llp', post_id: '123'}`
 

## **响应路由参数的变化**

使用路由参数在渲染不同的组件切换时候，**原来的组件实例会被复用**，这意味着**组件的生命周期  
钩子不会再被调用**。

这个时候想要让组件对路由参数的变化做出响应的时候，可以利用 `watch` 选项检测 `$route` 的变化。 

## **嵌套路由**

嵌套路由用来解决组件嵌套中的动态路径，例如：  

路径：/user/foo/[profile] profile是动态路径的变量  
组件：  

    <user-component>
        <foo>
            <profile1/>
        <foo>
    </user-component>

一个渲染后的组件可以拥有自己的嵌套 `<router-view>`，要在嵌套的路由出口中渲染组件，需要在 `VueRouter` 的参数中使用 `children` 配置，例如：  

    const router = new VueRouter({
        routes: [
            {
                path: 'user/:id',
                component: User,
                children: [
                    {
                        path:'profile',
                        component: UserProfile
                    }
                ]
            }
        ]
    })

可以看出，`children` 选项中的元素任然是一系列 *路由配置对象*。  
需要**注意**的是：  
`children` 配置对象中的 `path` 如果以 `/` 开头，这个嵌套路径会被当做整个域的根路径。 

当没有匹配到嵌套路由的时候，我们有时候可能想要渲染一个默认的组件，这个时候可以设置一个空 
路由：  

    children: [
        path: '',
        component: DefaultComonent
    ]  

## **编程式的路由导航**

因为可以在 Vue 组件实例中访问路由实例 `this.$router` ，所以可以在 Vue 实例中编写路由导航，实现 `<route-link>` 标签功能。  

<font size="4">`this.$router.push(location, onComplete?, onAbort?)`</font>

导航到不同的 URL，可以使用 `router.push` 方法。该方法向 histroy 栈中添加一个新的记录，`<route-link>` 标签的内部实现原理也是使用该方法。

**声明式** | **编程式**
-|-|
`<router-link to="...">` | `this.$router.push({...})`  

### **push函数中的参数**  

- history  
  1. 可以是一个字符串  
   
        字符串对象表示一个字符串路径  
  2. 可以是一个对象

        对象表示一个描述地址的对象，这个对象中的参数可以有：

        `name` : 一个字符串路径, 只能表示根路径下一级路径名，配合 `params` 使用  
        `path` : 一个字符串路径，可以完整表示一个路径。使用后，`param` 属性会被忽略  
        `params` : 动态路径参数，一个参数对象
        `query` : 查询参数，一个参数对象  

- onComplete : 一个导航成功后的回调函数
- onAbort : 导航到相同路径或在当前导航完成之前导航到另一个不同的路由进行的调用（请求分发）
  
*Tips*：  
    
如果该函数中未指定后面两个回调参数，该函数返回的是一个 `Promise`， 相应的回调可以在 `then`   
(onComplete), `catch` (onAbort) 中定义。


<font size="4">`this.$router.replace(location, onComplete?, onAbort?)`</font>

作用和 `router.push()` 一样，唯一的不同就是，它不会像 `histroy` 添加新纪录，而是替换掉当前的 `histroy` 记录。  

**声明式** | **编程式**  
-|-|  
`<router-link v-bind:to="..." replace>` | `router.replace(...)`  

<font size="4">`this.$router.go(n)`</font>  

这个方法的参数式一个参数，意思是在 `histroy` 记录中向前进或者向后退多少步，类似 `window.histroy.go(n)` 。  

## **命名路由**  

在配置路由器的时候，路由对象中的 `name` 属性，可以为这个路由配置一个名字，当链接一个路由  
或者在执行某些跳转的时候，就显得很方便了。例如：  

    const router = new VueRouter({
        routes: [
            {
                path: '/user/:userId',
                name: 'user',
                component: User
            }
        ]
    })

要链接一个路由，可以给 `<router-link>` 标签中的 `to` 属性传递一个对象：  

    <router-link v-bind:to="{name: 'user', params:{userId: 123}}">User</router-link>

类似的，`$router.push()` 方法作用是一样的：  

    $router.push({name:'user', params:{userId:123}})  

## **命名视图**

如果想在同一级展示多个视图，而不是嵌套视图，可以将路由器配置对象中的 `component` 换成  
`components`, 使之成为一个组件。这些组件可以对应多个视图，如果要让路由出口与组件对  
应，应该给 `<router-view>` 配置一个 `name` 属性，如果没有名字，其默认名字为 `default`。设置好名字后，可以在路由器配置中分别配置对应名字的组件。例如：  

    const router = new VueRouter({
        routes:[
            {
                path: '/',
                components:{
                    default:Foo,
                    nameA: Bar,
                    nameB: FooBar
                }
            }
        ]
    })

### **嵌套命名视图**

当我们将嵌套视图和命名视图的用法结合起来的时候，可以实现更好的功能。

例如：  

    {
        path:'/settings',
        component: UserSettings,
        children:[
            {
                path:'emails', // /settings/emails
                component: UserEmailSetting
            },
            {
                path: 'profile',
                components:{
                    default: UserProfile,
                    helper: UserProfilePreview
                }
            }
        ]
    }

    <div>
        <h1>User Settings</h1>
        <NavBar/>
        <router-view/>
        <router-view name='helper'>
    </div>


## **重定向和别名**

### **重定向**

当我们访问一个 url 地址的时候，我们实际上想让浏览器跳转到另一个地址，可以使用重定向的功能  

具体的用法是在路由配置对象中添加一个 `redirect` 参数，例如：  

    {
        path: '/a',
        component: ComponentA,
        redirect: '/b'
    }

这样，当我们访问 `/a` 的时候，浏览器会跳转到 `/b` 

### **别名**

与重定向不同的是，别名的用法为：

    {
        path: '/a',
        component: ComponetA,
        ailas: '/b'
    }

这样当我们访问 `/b` 的时候，URL会映射为 `/b` ,但是路由将会映射到 '/a'。  

别名的作用： 可以自由的将 UI 结构映射到任意结构的 URL，而不是受限于配置的嵌套路由结构。  

## **路由组件传参**  

在组件中使用 `$route` 来获取当前路由的路由参数，会造成组件和路由的高度耦合，不利于组件  
的复用。因此，可以在路由配置对象中设置 `props` 选项，利用 prop 将路由参数传递给组件，这  
样组件就可以在任何地方使用。  

例如：  

    const User = {
        props: {
            id: Number
        },
        template: `
            <div>User {{id}} </div>
        `
    }

    const router = new VueRouter({
        routes:[
            {
                path: '/user/:id', 
                component: User,
                props: true // 布尔参数将会设置路由参数为 prop
            },
            
            // 对于命名视图，需要在 props 选项中为每一个命名视图设置
            {
                path: '/user/:id',
                component: {
                    default: User,
                    sideBar: SideBar
                },
                props: {
                    default: true,
                    sideBar: false
                }
            }
        ]
    })

### **props 选项的几种模式**  

上面的例子已经展示了 props 选项作为布尔值的作用，props 一共可以设置一下几种模式  

- 布尔模式  

  当 props 选项设置为布尔值的时候的时候，当值为 `true` 时 `$route.params` 
  将会设置为组件的prop

- 对象模式  
  
  当props 选项被设置为对象的时候，这个对象将会按照原样被设置为组件的 prop ，例如

        const router = new VueRouter({
            routes:[
                {
                    path:'/user/:id',
                    component: User,
                    props:{
                        show: false
                    }
                }
            ]
        })
    
    这样 `{show:false}` 这个对象将会作为组件的 prop 传递给组件，当需要传递的 prop 是静态的时候使用。
- 函数模式  
  
  当 props 选项是一个函数的时候，组件将会将函数的返回值作为 prop。当需要基于路由的值  
  转换为另一个对象的的时候可以利用这个模式。  

**需要注意的是**  

最好将传递的 prop 设置为静态的值，因为它只会在路由发生变化的时候才会改变。  


## **H5 Histroy 模式**

VueRouter 默认使用 hash 模式，这个模式使用当前的 URL 来模拟生成一个完整的 URL，当 URL 改变的时候，页面不会重新加载。  

这种模式的特点就是在当前 URL 的末尾生成一个 # 符号。例如：  

    https://router.vuejs.org/zh/guide/essentials/history-mode.html#%E5%90%8E%E7%AB%AF%E9%85%8D%E7%BD%AE%E4%BE%8B%E5%AD%90

为了去掉这个 # ,可以使用 **history** 模式。这种模式下，浏览器的 URL 就像正常的 URL 一样。  

需要注意的是，在这种模式下，后台需要正确的配置。因为如果页面是一个单页应用，一些不返回页  
面的 URL 会返回 404。  

一个解决方案是：配置一个所有情况下的候选资源，如果找不到这个资源，至少应该返回 `index.html` 页面。这样需要在前端处理 404 等错误页面。  

