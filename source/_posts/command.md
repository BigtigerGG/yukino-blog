---
title: Vue 指令
date: 2020/1/20 23:30:25
tags:
- Command
catagories:
- Vue
- Vue Base
---
# Vue指令  

**每学习一个指令，将其用法和自己的理解记录下来**

## **v-bind**  

### **概述**

`v-bind` 指令是一个用于将 vue 实例中的数据属性或者prop动态地和 html 标签中的属性绑定 
起来。当 vue 实例中的数据属性发生改变时，html 标签中的属性也会响应式的变化。  

### **用法**

- 缩写： `:`
- 参数： `attr | prop`
- 修饰符：  
  - `.prop` 待续
  - `.camel` 待续
  - `.sync` 待续
- 说明：  
    动态的绑定一个或多个属性，或用一个prop到表达式。  

    在绑定 `style` 或 `class` 特性时，支持其它类型的值，如数组或者对象。 

    在绑定prop时，prop必须在子组件中声明。可以用修饰符指定不同的绑定类型。  

    没有参数时，可以绑定一个包含键值对的对象。**注意，此时 `class` 和 `style` 绑定不支持数组或者对象。  

    *下面是我自己的理解：*  
  
  - 动态绑定一个属性的时候，将null值作为参数，将会取消这个绑定事件。
- 示例  

        <!-- 绑定一个属性 -->
        <img v-bind:src="imageSrc">
    
        <!-- 动态特性名 (2.6.0+) -->
        <button v-bind:[key]="value"></button>
    
        <!-- 缩写 -->
        <img :src="imageSrc">
    
        <!-- 动态特性名缩写 (2.6.0+) -->
        <button :[key]="value"></button>
    
        <!-- 内联字符串拼接 -->
        <img :src="'/path/to/images/' + fileName">
    
        <!-- class 绑定 -->
        <div :class="{ red: isRed }"></div>
        <div :class="[classA, classB]"></div>
        <div :class="[classA, { classB: isB, classC: isC }]">
    
        <!-- style 绑定 -->
        <div :style="{ fontSize: size + 'px' }"></div>
        <div :style="[styleObjectA, styleObjectB]"></div>
    
        <!-- 绑定一个有属性的对象 -->
        <div v-bind="{ id: someProp, 'other-attr': otherProp }"></div>
    
        <!-- 通过 prop 修饰符绑定 DOM 属性 -->
        <div v-bind:text-content.prop="text"></div>
    
        <!-- prop 绑定。“prop”必须在 my-component 中声明。-->
        <my-component :prop="someThing"></my-component>
    
        <!-- 通过 $props 将父组件的 props 一起传给子组件 -->
        <child-component v-bind="$props"></child-component>
    
        <!-- XLink -->
        <svg><a :xlink:special="foo"></a></svg>
