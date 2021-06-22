---
title: 封装 Vue 组件库
date: 2021-05-31 10:38:06
tags: [vue]
categories: [vue]
---

## CDD（Component-Driven Development)

* 自上而下
* 从组件级别开始，到页面级别结束

优点：

* 组件在最大程度被重用
* 并行开发
* 可视化测试

## 处理组件的边界情况

* $root         ----------响应式的
* $parent/$children        ----------响应式的
* $refs            ----------响应式的
* 依赖注入 provide / inject   --------非响应式的
* $attrs  : 把父组件中非 prop 属性绑定到内部组件
  1. 从父组件传给自定义子组件的属性，如果没有 prop 接收会自动设置到子组件内部的最外层标签上，如果是 class 和 style 的话，会合并最外层标签的 class 和 style
  2. 如果子组件中不想继承父组件传入的非 prop 属性，可以使用 inheritAttrs 禁用继承，然后通过 v-bind="$attrs" 把外部传入的非 prop 属性设置给希望的标签上，但是这不会改变 class 和 style

* $listeners ：把父组件中的 DOM 对象的原生事件绑定到内部组件

## 快速原型开发

* VueCLI 中提供了一个插件可以进行原型快速开发

* 需要额外安装一个全局的扩展

  ```bash
  npm install -g @vue/cli-service-global
  ```

* 使用 vue serve 快速查看组件的运行效果

  * vue serve 如果不指定参数默认会在当前目录找以下的入口文件

    ```bash
    main.js   index.js    App.vue   app.vue
    ```

  * 可以指定要加载的组件

    ```bash
    vue serve ./src/login.vue
    ```

### 基于 elementUI 开发

* 初始化 package.json ---------```npm init -y```
* 安装 ElementUI  ----------- ```vue add element```
* 加载 ElementUI，使用 Vue.use() 安装插件

### 组件分类

* 第三方组件
* 基础组件
* 业务组件

### 组件开发

#### 步骤条组件

```vue
<template>
  <div class="lg-steps">
    <div class="lg-steps-line"></div>
    <div
        class="lg-step"
        v-for="index in count"
        :key="index"
        :style="{color:index <= active ? activeColor : defaultColor }"
    >
      {{ index }}
    </div>
  </div>
</template>

<script>
  import './steps.css'
  export default {
    name: "LgSteps",
    props:{
      count:{
        type:Number,
        default:3
      },
      active:{
        type:Number,
        default: 0
      },
      activeColor:{
        type:String,
        default:'red'
      },
      defaultColor:{
        type:String,
        default:'green'
      }
    }
  }
</script>

<style scoped>

</style>
```



#### 表单组件

* Form 组件 依赖注入 整个form ``` provide (){ return { form:this }}```
* FormItem 接收 form,从中获取 models 和 rules ```inject:['form']```，
  * 表单验证 ``` npm i async-validator```

* Input组件中触发自定义事件 validate
* FormItem 渲染完毕注册自定义事件 validate

##### Form 组件

```vue
<template>
  <div class="lg-steps">
    <div class="lg-steps-line"></div>
    <div
        class="lg-step"
        v-for="index in count"
        :key="index"
        :style="{color:index <= active ? activeColor : defaultColor }"
    >
      {{ index }}
    </div>
  </div>
</template>

<script>
  import './steps.css'
  export default {
    name: "LgSteps",
    props:{
      count:{
        type:Number,
        default:3
      },
      active:{
        type:Number,
        default: 0
      },
      activeColor:{
        type:String,
        default:'red'
      },
      defaultColor:{
        type:String,
        default:'green'
      }
    }
  }
</script>

<style scoped>

</style>
```



##### FormItem 组件

```vue
<template>
  <div>
    <label>{{ label }}</label>
    <div>
      <slot></slot>
      <p v-if="errMessage">{{ errMessage }}</p>
    </div>
  </div>
</template>

<script>
  import AsyncValidator from 'async-validator'
  export default {
    name: "LgFormItem",
    inject:['form'],
    props:{
      label:{
        type:String
      },
      prop:{
        type:String
      }
    },
    data(){
      return {
        errMessage:''
      }
    },
    mounted(){
      this.$on('validate',() => {
        this.validate()
      })
    },
    methods:{
      validate(){
        if(!this.prop) return
        const value = this.form.model[this.prop]
        const rules = this.form.rules[this.prop]
        
        const descriptor = {
          [this.prop]:rules
        }
        
        const validator = new AsyncValidator(descriptor)
        return validator.validate({ [this.prop]:value },errs => {
          if(errs) {
            this.errMessage = errs[0].message
          }else{
            this.errMessage = ''
          }
        })
      }
    }
  }
</script>

<style scoped>

</style>
```



##### Input 组件

```vue
<template>
  <div>
    <input v-bind="$attrs" :type="type" :value="value" @input="handleInput">
  </div>
</template>

<script>
  export default {
    name: "LgInput",
    inheritAttrs:false,
    props:{
      value:{
        type:String
      },
      type:{
        type:String,
        default:'text'
      }
    },
    methods:{
      handleInput(event){
        this.$emit('input',event.target.value)
        const findParent = parent => {
          while(parent){
            if(parent.$options.name === 'LgFormItem'){
              break
            }else{
              parent = parent.$parent
            }
          }
          return parent
        }
        const parent = findParent(this.$parent)
        
        if(parent){
          parent.$emit('validate')
        }
      }
    }
  }
</script>

<style scoped>

</style>
```



##### Button 组件

```vue
<template>
  <div>
    <button @click="handleClick">
      <slot></slot>
    </button>
  </div>
</template>

<script>
  export default {
    name: "LgButton",
    methods:{
      handleClick(event){
        this.$emit('click',event)
        event.preventDefault()
      }
    }
  }
</script>

<style scoped>

</style>
```



### 组件管理

1. Multirepo（Multiple Repository）

   每一个包对应一个项目

2. Monorepo（Monolithic Repository）

   一个项目仓库中管理多个模块 / 包

#### Monorepo 

* 组件库开发

* packages 文件夹下，每个组件对应一个文件夹，目录结构如下：

  ```
  -packages
  	- button
  		- __test__
  		- dist
  		- src
  			- button.vue
  		- index.js
  		- LICENCE
  		- package.json
  		- README.md
  	- form
  	- formitem
  	- input
  	- steps
  ```

  

### Storybook

* 可视化的组件展示平台
* 在隔离的开发环境中，以交互式的方式展示组件
* 独立开发组件
* 支持的框架
  * React 、React Native 、Vue 、Angular 
  * Ember 、HTML 、Svelte 、Mithril 、Riot

#### 安装

**自动安装**

```bash
npx -p @storybook/cli sb init --type vue
yarn add vue
yarn add vue-loader vue-template-compiler --dev
```

**手动安装**

略

### yarn 工作区

npm  不支持 workspaces

#### 开启 yarn 工作区

项目根目录的 package.json

```json
"private":true,
"workspaces":[
	"packages/*"
]
```

#### yarn workspaces 使用

* 给工作区根目录安装开发依赖

  ```bash
  yarn add jest -D -W
  ```

* 给指定工作区安装依赖

  ```bash
  yarn workspace q-button add lodash@4
  ```

* 给所有的工作区安装依赖

  ```bash
  yarn install
  ```



### Lerna

* Lerna 是一个优化使用 git 和 npm 管理多包仓库的工作流工具
* 用于管理具有多个包的 JavaScript 项目
* 它可以一键把代码提交到 git 和 npm 仓库

#### 使用

* 全局安装

  ```bash
  yarn global add lerna
  ```

* 初始化

  ```bash
  lerna init
  ```

* 发布

  ```bash
  lerna publish
  ```

#### npm 指令

* 登录 npm

  ```bash
  npm login
  ```

* 查看 npm 登录人

  ```bash
  npm whoami
  ```

* 查看镜像源

  ```bash
  npm config get registry
  ```

  

### Vue 组件单元测试

对函数的输入输出进行测试，使用断言，根据输入判断实际输出和预计输出是否一致

**组件单元测试**是指使用单元测试工具对组件的行为和状态进行测试，确保组件发布之后在项目使用过程中不会导致错误

**组件单元测试优点**：

* 提供描述组件行为的文档
* 节省手动测试的时间
* 减少研发新特性时产生的 bug
* 改进设计
* 促进重构

#### 配置环境

* 安装依赖

  * Vue Test Utils
  * Jest
  * vue-jest
  * babel-jest

  ```bash
  yarn add jest @vue/test-utils vue-jest babel-jest -D -W
  ```

* 配置测试脚本

  * package.json

  ```json
  "scripts":{
  	"test":"jest"
  }
  ```

* Jest 配置文件

  * jest.config.js

  ```js
  module.exports = {
  	"testMatch":["**/__tests__/**/*.[jt]s?(x)"], // js jsx ts tsx
  	"moduleFileExtensions":[ 
  		"js",
  		"json",
  		"vue" // 告诉 Jest 处理 `*.vue` 文件
  	],
  	"transform":{
  		".*\\.(vue)$":"vue-jest", // 用 `vue-jest` 处理 `*.vue` 文件
  		".*\\.(js)$":"babel-jest" // 用 `babel-jest` 处理 `*.js` 文件
  	}
  }
  ```

* Babel 配置文件

  * babel.config.js

  ```js
  module.exports = {
  	presets:[
  		'@babel/preset-env'
  	]
  }
  ```

  * Babel 桥接

  ```bash
  yarn add babel-core@bridge -D -W
  ```

  

#### Jest 常用 API

1. 全局函数
   * describe(name, fn)      把相关测试组合在一起
   * test(name, fn)               测试方法
   * expect(value)                断言
2. 匹配器
   * toBe(value)                   判断值是否相等
   * toEqual(obj)                  判断对象是否相等
   * toContain(value)          判断数组或字符串中是否包含
3. 快照
   * toMatchSnapshot()

#### Vue Test Utils 常用 API

1. mount()
   * 创建一个包含被挂载和渲染 Vue 组件的 Wrapper
2. Wrapper
   * vm                wrapper 包裹的组件实例
   * props()         返回 Vue 实例选项中的 props 对象
   * html()           组件生成的 HTML 标签
   * find()            通过选择器返回匹配到的组件中的 DOM 元素
   * trigger()       触发 DOM 原生事件，自定义事件 wrapper.vm.$emit()

```js
// __test__ / input.test.js
import input from '../src/input.vue'
import { mount } from '@vue/test-utils'

describe('q-input',() => {
  test('input-text',() => {
    const wrapper = mount(input)
    expect(wrapper.html()).toContain('input type="text"')
  })
  
  test('input-password',() => {
    const wrapper = mount(input,{
      propsData:{
        type:'password'
      }
    })
    expect(wrapper.html()).toContain('input type="password"')
  })
  
  test('input-password',() => {
    const wrapper = mount(input,{
      propsData:{
        type:'password',
        value:'admin'
      }
    })
    expect(wrapper.props('value')).toBe('admin')
  })
  
  test('input-snapshot',() => {
    const wrapper = mount(input,{
      propsData:{
        type:'text',
        value:'admin'
      }j
    })
    expect(wrapper.vm.$el).toMatchSnapshot()
  })
})

```



### Rollup 打包

1. Rollup 是一个模块打包器
2. Rollup 支持 Tree-shaking 
3. 打包的结果比 Webpack 要小
4. 开发框架/组件库的时候使用 Rollup 更合适

#### 安装依赖

* Rollup
* rollup-plugin-terser
* rollup-plugin-vue@5.1.9  // 必须制定版本，这个版本是转换 vue 2,新版本转换 vue 3
* vue-template-compiler

#### 配置文件

只打包 button 文件

* 在 button 目录中创建 rollup.config.js

```js
import { terser } from 'rollup-plugin-terser'
import vue from 'rollup-plugin-vue'
module.exports = [
	{
		input:'index.js',
		output:[
			{
				file:'dist/index.js',
				format:'es'
			}
		],
		plugins:[
			vue({
				css:true,
				compileTemplate:true
			}),
			terser()
		]
	}
]
```

* button 目录下 package.json

```json
"scripts": {
    "build": "rollup -c"
  },
```

* 运行命令

```bash
yarn workspace q-button run build
```



打包全部文件

* 安装依赖

  ```bash
  yarn add @rollup/plugin-json rollup-plugin-postcss @rollup/plugin-node-resolve -D -W
  ```

* 配置文件

  项目根目录创建 rollup.config.js

  ```js
  import fs from 'fs'
  import path from 'path'
  import json from '@rollup/plugin-json'
  import vue from 'rollup-plugin-vue'
  import postcss from 'rollup-plugin-postcss'
  import { terser } from 'rollup-plugin-terser'
  import { nodeResolve } from '@rollup/plugin-node-resolve'
  
  const isDev = process.env.NODE_ENV !== 'production'
  
  // 公共插件配置
  const plugins = [
  	vue({
  		// Dynamically inject css as a <style> tag
          css:true,
          // Explicitly convert template to render function
          compileTemplate:true 
      }),
      json(),
      nodeResolve(),
      postcss({
          // 把 css 插入到 style 中
          // inject :true,
          // 把 css 放到 js 同一目录
          extract:true
      })
  ]
  
  // 如果不是开发环境，开启压缩
  isDev || plugins.push(terser())
  
  // package 文件夹路径
  const root = path.resolve(__dirname,'packages')
  
  module.exports = fs.readdirSync(root)
  	// 过滤，只保留文件夹
  	.filter(item => fs.statSync(path.resolve(root, item)).isDirectory())
  	// 为每一个文件夹创建对应的配置
  	.map(item => {
      	const pkg = require(path.resolve(root,item,'package.json'))
          return {
              input:path.resolve(root, item, 'index.js'),
              output:[
                  {
                      exports:'auto',
                      file:path.resolve(root, item, pkg.main),
                      format:'cjs'
                  },
                  {
                      exports:'auto',
                      file:path.resolve(root, item, pkg.module),
                      format:'es'
                  },
              ],
              plugins:plugins
          }
  	})
  ```

* 在每一个包中设置 package.json 中的 main 和 module 字段

  ```json
  "main":"dist/cjs/index.js"
  "module":"dist/es/index.js"
  ```

* 根目录的 package.json 中配置 scripts

  ```json
  "build":"rollup -c"
  ```

  * 遇到的问题

    Error: PostCSS plugin postcss-noop-plugin requires PostCSS 8.

  * 解决

    yarn add postcss-loader autoprefixer@8.0.0 -D -W

    ```js
    postcss({
        plugins:[
          require('autoprefixer')({overrideBrowserlist:['> 0.15% in CN']})
        ]
      })
    ```

### 设置环境变量

* 安装依赖  cross-env ,可以跨平台设置环境变量

  ```bash
  yarn add cross-env -D -W
  ```

* 修改 项目根目录 package.json 中 scripts 中的 build

  ```json
  "scripts":{
      "build:prod":"cross-env NODE_ENV=production rollup -c",
      "build:dev":"cross-env NODE_ENV=development rollup -c"
  }
  ```

### 清理

* 清理所有包中的 node_modules

  项目根目录 package.json

  ```json
  "scripts":{
      "clean": "lerna clean"
  }
  ```

  

* 清理 所有的 dist

  * 安装依赖 rimraf

    ```bash
    yarn add rimraf -D -W
    ```

  * 每个包的  package.json

    ```json
    "scripts": {
        "del": "rimraf dist"
      },
    ```

  * 执行所有包中的 del 命令

    ```
    yarn workspaces run del
    ```



### 基于模板生成组件基本结构

plop

* 安装 plop

  ```bash
  yarn add plop -D -W
  ```

* 创建模板 plop-template 目录

* plopfile.js 配置文件

  ```js
  module.exports = plop => {
    plop.setGenerator('component', {
      description: 'create a custom component',
      prompts: [
        {
          type: 'input',
          name: 'name',
          message: 'component name',
          default: 'MyComponent'
        }
      ],
      actions: [
        {
          type: 'add',
          path: 'packages/{{name}}/src/{{name}}.vue',
          templateFile: 'plop-template/component/src/component.hbs'
        },
        {
          type: 'add',
          path: 'packages/{{name}}/__tests__/{{name}}.test.js',
          templateFile: 'plop-template/component/__tests__/component.test.hbs'
        },
        {
          type: 'add',
          path: 'packages/{{name}}/stories/{{name}}.stories.js',
          templateFile: 'plop-template/component/stories/component.stories.hbs'
        },
        {
          type: 'add',
          path: 'packages/{{name}}/index.js',
          templateFile: 'plop-template/component/index.hbs'
        },
        {
          type: 'add',
          path: 'packages/{{name}}/LICENSE',
          templateFile: 'plop-template/component/LICENSE'
        },
        {
          type: 'add',
          path: 'packages/{{name}}/package.json',
          templateFile: 'plop-template/component/package.hbs'
        },
        {
          type: 'add',
          path: 'packages/{{name}}/README.md',
          templateFile: 'plop-template/component/README.hbs'
        }
      ]
    })
  }
  ```

* 根目录 package.json 文件添加命令

  ```json
  "scripts": {
      "plop": "plop"
    },
  ```

* 运行命令

  ```
  yarn plop
  ```

  



