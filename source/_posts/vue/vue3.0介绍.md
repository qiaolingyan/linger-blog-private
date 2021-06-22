---
layout: vue
title: vue3.0介绍
date: 2021-05-31 09:35:26
tags:[vue]
categories: [vue]
---

# vue 3 介绍

## 源码组织方式的变化

1. 源码采用 TypeScript 重写
2. 使用 Monorepo 管理项目结构

**packages 目录结构**

* compiler-core ：跟平台无关的编译器
* compiler-dom ：浏览器平台的编译器，依赖于 compiler-core
* compiler-sfc ：单文件组件编译器 ，依赖于 compiler-core 和 compiler-dom
* compiler-ssr ：服务端渲染编译器，依赖于 compiler-dom
* reactivity ：数据响应式系统，可独立使用
* runtime-core ：运行时，与平台无关的运行时
* runtime-dom：浏览器相关运行时，处理原生 dom 事件等
* runtime-test ： 测试的轻量级运行时，渲染出来的 dom 树是一个 js 对象
* server-renderer ： 服务端渲染
* shared ：公共 api
* size-check ：在 tree-shaking 之后检查 包的大小
* template-explorer ：在浏览器运行的实时编译组件，导出 render 函数
* vue ：完整版的 vue
* vue-compat

**构建版本**

* cjs ：commonjs
  * vue.cjs.js : 开发版，没有压缩
  * vue.cjs.prod.js 
* global
  * vue.global.js ：完整版的 vue
  * vue.global.prod.js
  * vue.runtime.global.js ：只包含运行时的构建版本
  * vue.runtime.global.prod.js
* browser
  * vue.esm-browser.js ：esmodule 完整版
  * vue.esm-browser.prod.js
  * vue.runtime.esm-browser.js
  * vue.runtime.esm-browser.prod.js
* bundler
  * vue.esm-bundler.js
  * vue.runtime.esm-bundler.js ：vue 最小版本，是 vue 体积更小

## Composition API

**RFC(Request For Comments)**

* [https://github.com/vuejs/rfcs](https://github.com/vuejs/rfcs)

**Composition API RFC**

* [https://composition-api.vuejs.org](https://composition-api.vuejs.org)

**设计动机**

* Options API -- vue 2.0 中
  * 包含一个描述组件选项（data、methods、props等）的对象
  * Options API 开发复杂组件，同一个功能逻辑的代码被拆分到不同选项

* Composition API -- vue 3.0 中
  * Vue.js 3.0 新增的一组 API
  * 一组基于函数的 API
  * 可以更灵活的组织组件的逻辑

## 性能提升

**响应式系统升级**

* Vue.js 2.x 中响应式系统的核心 defineProperty
* Vue.js 3.x 中使用 Proxy 对象重写响应式系统
  * 可以监听动态新增的属性
  * 可以监听删除的属性
  * 可以监听数组的索引和 length 属性

**编译优化**

[[https://vue-next-template-explorer.netlify.app](https://vue-next-template-explorer.netlify.app/)]

* Vue.js 2.x 中通过标记静态根节点，优化 diff 的过程
* Vue.js 3.0 中标记和提升所有的静态节点，diff 的时候只需对比动态节点内容
  * Fragments（升级 vetur 插件）
  * 静态提升
  * Patch flag
  * 缓存事件处理函数

**源码体积的优化**

* Vue.js 3.0 中移除了一些不常用的 API
  * 例如：inline-template、filter 等
* Tree-shaking

## Vite 构建工具

**ES Module**

* 现代浏览器都支持 ES Module(IE 不支持)

* 通过下面的方式加载模块

  * <script type="module" src="..." ></script>

* 支持模块的 script 默认延迟加载

  * 类似于 script 标签设置 defer
  * 在文档解析完成后，触发 DOMContentLoaded 事件前执行

**Vite as Vue-CLI**

* Vite 在开发模式下不需要打包可以直接运行
* Vue-CLI 开发模式下必须对项目打包才可以运行
* Vite 在生产环境下使用 Rollup 打包
  * 基于 ES Module 的方式打包
* Vue-CLI 使用 Webpack 打包

**Vite 特点**

* 快速冷启动
* 按需编译
* 模块热更新

**Vite 创建项目**

* Vite 创建项目

  ```bash
  npm init vite-app <project-name>
  cd <project-name>
  npm install
  npm run dev
  ```

* 基于模板创建项目

  ```bash
  npm init vite-app --template react
  npm init vite-app --template preact
  ```




# Composition API

## **createApp**

​	创建 vue 对象

## **setup**

​	Composition API 入口

## **reactive**

​	把一个对象转换为响应式对象

## 生命周期钩子函数

因为 `setup` 是围绕 `beforeCreate` 和 `created` 生命周期钩子运行的，所以不需要显式地定义它们。换句话说，在这些钩子中编写的任何代码都应该直接在 `setup` 函数中编写。

| **选项式 API** | **Hook inside** `setup` |                                              |
| -------------- | :---------------------- | -------------------------------------------- |
| beforeCreate   | Not needed*             |                                              |
| created        | Not needed*             |                                              |
| beforeMount    | onBeforeMount           |                                              |
| mounted        | onMounted               |                                              |
| beforeUpdate   | onBeforeUpdate          |                                              |
| updated        | onUpdated               |                                              |
| beforeUnmount  | onBeforeUnmount         |                                              |
| unmounted      | onUnmounted             | render函数重新调用时触发（首次调用也会触发） |
| errorCaptured  | onErrorCaptured         | render函数重新调用时触发（首次调用不会触发） |

## 函数

1. ### **toRefs**

   * 作用：传入的对象必须是一个代理对象，它将一个响应式对象的所有属性 都转换为响应式对象
   * 原理：内部会创建一个新的对象，然后遍历这个传入的代理对象的所有属性，把所有属性的值都转换为响应式对象，挂载到新创建的对象上，将这个新的对象返回。内部会为代理对象的每一个属性创建一个 具有 value属性 的对象，value 具有 getter 和 setter

2. ### **ref**

   * 作用：将基本数据类型转换为响应式对象
   * 返回一个对象 ```{value:0}```,value 具有 getter 和 setter

3. ### **reactive**

   * 作用：把一个对象转换为响应式对象

## Computed

1. 第一种用法：接受一个 getter 函数，并为从 getter 返回的值返回一个不变的响应式 [ref](https://v3.cn.vuejs.org/api/refs-api.html#ref) 对象。

   ```js
   const count = ref(1)
   computed(() => count.value + 1)
   
   console.log(plusOne.value) // 2
   plusOne.value++ // 错误
   ```

2. 第二种用法：也可以使用具有 `get` 和 `set` 函数的对象来创建可写的 ref 对象。

   ```js
   const count = ref(1)
   const plusOne = computed({
       get:() => count.value + 1,
       set:val => {
           count.value = val - 1
       }
   })
   
   plusOne.value = 1
   console.log(count.value) // 0
   ```

## wacth

* watch 的三个参数
  * 第一个参数：要监听的数据
  * 第二个参数：监听到数据变化后执行的函数，这个函数有两个参数，分别是新值和旧值
  * 第三个参数选项对象，deep 和 immediate
* watch 的返回值
  * 取消监听的函数

1. 侦听一个单一源：侦听器数据源可以是具有返回值的 getter 函数，也可以是 [ref](https://v3.cn.vuejs.org/api/refs-api.html#ref)

   ```js
   // 侦听 一个 getter
   const state = reactive({count:0})
   watch(
   	() => state.count,
       (count,prevCount) => { ... }
   )
   
   // 侦听一个 ref
   const count = ref(0)
   watch(count,(count,prevCount) => {
       ...
   })
   ```

2. 侦听多个源：侦听器还可以使用数组同时侦听多个源

   ```js
   wacth([fooRef, barRef], ([foo, bar], [prevFoo, prevBar]) => {
       ...
   })
   ```

## WatchEffect

在响应式地跟踪其依赖项时立即运行一个函数，并在更改依赖项时重新运行它。

* 是 watch 函数的简化版本，也用来监视数据的变化

* 接收一个函数作为参数，监听函数内响应式数据的变化

  ```js
  const count = ref(0)
  const stop = watchEffect(() => console.log(count.value)) // -> logs 0
  // stop 函数：取消 watch 监听
  
  setTimeout(() => {
      count.value++  // -> logs 1
  })
  
  ```

* watch 的返回值

  * 取消监听的函数

## 自定义指令

**vue 2**

```js
Vue.directive('focus',{
    bind(el, binding, vnode, prevVnode){},
    inserted(){},
    updated(){}, // remove
    componentUpdated(){},
    unbind(){}
})
```

**vue 3**

```js
app.directive('focus',{
    beforeMount(el, binding, vnode, prevVnode){},
    mounted(el){  el.focus() },
    beforeUpdate(){}, // new
    updated(){},
    beforeUnmount(){}, // new
    unmounted(){}
})

// 在 mounted 和 updated 时触发相同行为，而不关心其他的钩子函数。那么你可以通过将这个回调函数传递给指令来实现：
app.directive('pin', (el, binding) => {
  el.style.position = 'fixed'
  const s = binding.arg || 'top' // binding.arg 是我们传递给指令的参数
  el.style[s] = binding.value + 'px'  // binding.value 是我们传递给指令的值
})
```

* created：在绑定元素的 attribute 或事件监听器被应用之前被调用。在指令需要附加须要在普通的 v-on 事件监听器前调用的事件监听器时，这很有用
* beforeMount：当指令第一次绑定到元素并且在挂载父组件之前调用
* mounted：在绑定元素的父组件被挂载后调用
* beforeUpdate：在更新包含组件的 VNode 之前调用
* updated：在包含组件的 VNode 及其子组件的 VNode 更新后调用
* beforeUnmount：在卸载绑定元素的父组件之前调用
* unmounted：当指令与元素解除绑定且父组件已卸载时，只调用一次

