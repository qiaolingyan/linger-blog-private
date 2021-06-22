---

title: vite
date: 2021-06-01 15:22:02
tags: [vue, vite]
categories: [打包]

---

项目地址：[https://gitee.com/endeavor1/q-vite-cli](https://gitee.com/endeavor1/q-vite-cli)

## Vite 概念

* Vite 是一个面向现代浏览器的一个更轻、更快的 Web 应用开发工具
* 它基于 ECMAScript 标准原生模块系统（ES Module）实现
* 解决 webpack 在开发阶段 使用 dev-server 冷启动时间过长，另外 热更新 反应慢的问题



## Vite 项目依赖

* Vite
* @vue/compiler-sfc



## Vite 创建项目

* 使用 npm 

  ```bash
  $ npm init @vitejs/app <project-name>
  $ cd <project-name>
  $ npm install
  $ npm run dev
  ```

  

* 使用 yarn

  ```bash
  $ yarn create @vitejs/app <project-name>
  $ cd <project-name>
  $ yarn
  $ yarn dev
  ```

  

## 基础使用

### ```vite serve```

* 编译是在服务器端
* 模块的处理是在请求到服务器端处理的
* 采用即时编译，按需编译

![img](C:\Users\qiaolingyan\AppData\Roaming\Typora\typora-user-images\image-20210601154824361.png)

* vue-cli-service serve 对比
  * 先打包，打包结果存储到内存中才会启动 web serve ，不管模块是否使用到，都会打包到 bundle里，项目体积大，打包速度就会变慢

![img](./img/vue-cli.jpg)



* #### HMR

  * Vite HMR

    * 立即编译当前所修改的文件

  * Webpack HMR

    * 会自动以这个文件为入口重写 build 一次，所有涉及到的依赖也都会被加载一遍

      

### ```vite build```

* Rollup
* Dynamic import (代码切割)
  * Polyfill

**打包 or 不打包**：

使用 Webpack 打包的两个原因：

* 浏览器环境并不支持模块化

* 零散的模块文件会产生大量的 HTTP 请求

  

## Vite 特性

1. 快速冷启动
2. 模块热更新
3. 按需编译
4. 开箱即用

## Vite 核心功能

1. 静态 Web 服务器
2. 编译单文件组件
   * 拦截浏览器不识别的模块，并处理
3. HMR



## Vite 实现原理

1. vite 通过对请求路径劫持获取资源的内容返回给浏览器
2. vite 热更新实现在 client 端，webSocket 监听了一些更新的类型
3. vite 热更新实现在 server 端，通过 watcher 监听页面改动

**使用 koa 开启一个 服务器**

1. package.json 文件中添加 bin 字段，作为 cli 工具的入口

   ```json
   "name": "q-vite-cli",
   "bin": "index.js",
   ```

2. index.js 文件里开启 Koa 服务器

   ```js
   #!/usr/bin/env node
   
   const Koa = require('koa')
   const send = require('koa-send')
   const app = new Koa()
   // 1. 开启静态文件服务器
   app.use(async (ctx,next) => {
     await send(ctx, ctx.path, { root: process.cwd(), index: 'index.html'})
     await next()
   })
   app.listen(3000)
   console.log('Server running @ http://localhost:3000')
   ```

3. npm link 

4. vue 3 项目中 使用  ```q-vite-cli``` ,启动项目

   * 报错： ```Uncaught TypeError: Failed to resolve module specifier "vue". Relative references must start with either "/", "./", or "../".```
   * 原因：```main.js``` 请求中  ```import { createApp } from 'vue'```时**报错**

**修改第三方模块的路径**

需要创建两个中间件：

* 一个中间件用来修改 node_modules 模块的引入路径，修改为 **/@modules/模块名称**

  ```js
  // 把流转换为字符串
  const streamToString = stream => new Promise((resolve,reject) => {
    const chunks = []
    stream.on('data',chunk => chunks.push(chunk))
    stream.on('end',() => resolve(Buffer.concat(chunks).toString('utf-8')))
    stream.on('error',reject)
  })
  
  // 2. 修改第三方模块的路径
  app.use(async (ctx,next) => {
    if(ctx.type === 'application/javascript'){
      // ctx.body 是一个流
      const contents = await streamToString(ctx.body)
      // import Vue from 'vue'
      // import App from './App.vue' || '/App.vue'
      ctx.body = contents.replace(/(from\s+['"])(?![\.\/])/g,'$1/@modules/')
    }
  })
  ```

  更改配置后需关闭服务器，重新启动  ```q-vite-cli``` 

* 另一个中间件是当请求处理回来后判断请求中是否有 **/@modules/模块名称**

  * 需要放在 开启静态文件服务器 中间件之前

**加载第三方模块 **

* 需要放在 开启静态文件服务器 中间件之前

  ```js
  const path = require('path')
  
  // 3. 加载第三方模块
  app.use(async (ctx,next) => {
    // ctx.path --> /@modules/vue
    if(ctx.path.startsWith('/@modules')){
      const moduleName = ctx.path.substr(10)
      const pkgPath = path.join(process.cwd(),'node_modules',moduleName,'package.json')
      const pkg = require(pkgPath)
      ctx.path = path.join('/node_modules',moduleName,pkg.module)
    }
    console.log(ctx.path)
    await next()
  })
  ```



**编译单文件组件**

* 开启静态文件服务器中间件 和 修改第三方模块的路径中间件 之间

* 需要使用 @vue/compiler-sfc ```npm i @vue/compiler-sfc```

  ```js
  const { Readable } = require('stream')
  const compilerSFC = require('@vue/compiler-sfc')
  
  // 把字符串转换为流
  const stringTostream = text => {
    const stream = new Readable()
    stream.push(text)
    stream.push(null)
    return stream
  }
  
  // 4. 处理单文件组件
  app.use(async (ctx,next) => {
    if(ctx.path.endsWith('.vue')){
      const contents = await streamToString(ctx.body)
      const { descriptor } = compilerSFC.parse(contents)
      let code
      if(!ctx.query.type){
        // 单文件组件第一次请求
        code = descriptor.script.content
        code = code.replace(/export\s+default\s+/g,'const __script = ')
        code += `
          import { render as __render } from "${ctx.path}?type=template"
          __script.render = __render
          export default __script
        `
      }else if(ctx.query.type === 'template'){
        // 单文件组件第二次请求
        const templateRender = compilerSFC.compileTemplate({ source: descriptor.template.content })
        code = templateRender.code
      }
      ctx.type = 'application/javascript'
      ctx.body = stringTostream(code)
    }
    await next()
  })
  ```

  

* 报错：process is not defined，解决：

  ```js
  // 2. 修改第三方模块的路径
  app.use(async (ctx,next) => {
    if(ctx.type === 'application/javascript'){
      // ctx.body 是一个流
      const contents = await streamToString(ctx.body)
      // import Vue from 'vue'
      // import App from './App.vue'
      ctx.body = contents
        .replace(/(from\s+['"])(?![\.\/])/g,'$1/@modules/')
        .replace(/process\.env\.NODE_ENV/g,'"development"')
    }
  })
  ```

  

  







