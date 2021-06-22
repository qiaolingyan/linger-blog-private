---
title: 虚拟Dom
date: 2021-05-31 10:38:06
tags: [vue]
categories: [vue]
---

### 1. 什么是虚拟Dom

virtual DOM (虚拟DOM)，是由普通的 js 对象来描述 DOM 对象

```
{
	sel:'div',
	text:'hello world'
}
```

### 2. 虚拟 DOM 的作用

* 维护视图和状态的关系
* 复杂视图情况下提升渲染性能
* 跨平台
  * 浏览器平台渲染DOM
  * 服务端渲染SSR（Nuxt.js/Next.js)
  * 原生应用（Weex/React Native)
  * 小程序（mpvue/uni-app）
* 虚拟 DOM 库
  * Snabbdom
    * 通过模块可扩展
    * 源码使用 ts 开发
  * virtual-dom

### 3. Snabbdom

* 安装 parcel

  ```
  md snabbdom-demo
  cd snabbdom-demo
  npm init -y
  npm install parcel-bundler -D
  ```

* 配置 scripts

  ```
  "scripts":{
  	"dev":"parcel index.html --open",
  	"build":"parcel build index.html"
  }
  ```

* 目录结构

#### 3.1 基本使用

* 模块

  作用：

  * Snabbdom 的核心库并不能处理 DOM 元素的属性/ 样式/ 事件等，可以通过注册 Snabbdom 默认提供的模块来实现
  * Snabbdom 中的模块可以用来扩展 Snabbdom 的功能
  * Snabbdom 中的模块的实现是通过注册全局的钩子函数来实现的

  官方提供的模块

  * attributes
  * props
  * dataset
  * class : 切换类样式
  * style
  * eventlisteners

  使用步骤

  * 导入需要的模块
  * init() 中注册模块
  * h()函数的第二个参数处处理模块

#### 3.2 源码

函数重载：函数名相同，参数个数不同或者参数类型不同（js中没有，ts中有）

patch 整体过程分析：

* patch(oldVnode, newVnode)
* 把新节点中变化的内容渲染到真实 DOM，最后返回新节点作为下一次处理的旧节点
* 对比新旧节点 Vnode 是否相同节点（节点的 key 和 sel 相同）
* 如果不是相同节点，删除之前的内容，重新渲染
* 如果是相同节点，再判断新的 VNode 是否有 text，如果有并且和 oldVnode 的 text 不同，直接更新文本内容
* 如果新的 VNode 有children，判断子节点是否有变化

