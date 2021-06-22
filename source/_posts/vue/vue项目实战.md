---
title: vue项目实战
date: 2021-06-02 09:44:46
tags: [vue]
categories: [vue, 项目实战]
---

项目技术：vue2 + ts + elementui

## 搭建项目架构

### 创建项目

#### 使用 Vue CLI 创建项目

安装 Vue ClI

```bash
npm i -g @vue/cli
vue create edu-boss-fed

 Vue CLI v4.5.6
 ? Please pick a preset: Manually select features // 手动选择公共特性
 ? Check the features needed for your project: Babel, TS, Router,
Vuex, CSS Pre-processors, Linter
 ? Use class-style component syntax? Yes // TS 相关
 ? Use Babel alongside TypeScript (required for modern mode, autodetected polyfills, transpiling JSX)? Yes
 ? Use history mode for router? (Requires proper server setup for
index fallback in production) No
 ? Pick a CSS pre-processor (PostCSS, Autoprefixer and CSS Modules
are supported by default): Sass/SCSS (with dart-sass)
 ? Pick a linter / formatter config: Standard
 ? Pick additional lint features: Lint on save, Lint and fix on co
mmit
 ? Where do you prefer placing config for Babel, ESLint, etc.? In
dedicated config files // 保存到各自独立的配置文件中
 ? Save this as a preset for future projects? No

 ⚓ Running completion hooks...

 � Generating README.md...

 � Successfully created project topline-m-89.
 � Get started with the following commands:

 $ cd edu-boss-fed
 $ npm run serve
```

安装结束，启动开发服务

```bash
cd edu-boss-fed
npm run serve
```



### 使用 TypeScript 开发 Vue 项目

1. 全新项目

   脚手架安装时 选择 TypeScript

2. 已有项目

   ```bash
   vue add @vue/typescript
   ```

### TypeScript 相关配置介绍

#### 安装了 TypeScript 相关的依赖项

1. dependencies 依赖

| 依赖项                 | 说明                                      |
| ---------------------- | ----------------------------------------- |
| vue-class-component    | 提供使用 Class 语法写 Vue 组件            |
| vue-property-decorator | 在 Class 语法基础之上提供了一些辅助装饰器 |

2. devDependencies 依赖

| 依赖项                           | 说明                                                         |
| -------------------------------- | ------------------------------------------------------------ |
| @typescript-eslint/eslint-plugin | 使用 ESLint 校验 TypeScript 代码                             |
| @typescript-eslint/parser        | 将 TypeScript 转为 AST 供 ESLint 校验使用                    |
| @vue/cli-plugin-typescript       | 使用 TypeScript + ts-loader + fork-ts-checker-webpack-plugin 进行更快的类型检查 |
| @vue/eslint-config-typescript    | 兼容 ESLint 的 TypeScript 校验规则                           |
| typescript                       | TypeScript 编译器，提供类型校验和转换 JavaScript 功能        |

#### TypeScript 配置文件 ```tsconfig.json```

```json
{
  "compilerOptions": {
    "target": "esnext",
    "module": "esnext",
    "strict": true,
    "jsx": "preserve",
    "importHelpers": true,
    "moduleResolution": "node",
    "experimentalDecorators": true,
    "skipLibCheck": true,
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "sourceMap": true,
    "baseUrl": ".",
    "types": [
      "webpack-env"
    ],
    "paths": {
      "@/*": [
        "src/*"
      ]
    },
    "lib": [
      "esnext",
      "dom",
      "dom.iterable",
      "scripthost"
    ]
  },
  "include": [
    "src/**/*.ts",
    "src/**/*.tsx",
    "src/**/*.vue",
    "tests/**/*.ts",
    "tests/**/*.tsx"
  ],
  "exclude": [
    "node_modules"
  ]
}
```

#### shims-vue.d.ts 文件的作用

```typescript
// 主要用于 TypeScript 识别 .vue 文件模块
// TypeScript 默认不支持导入 .vue 模块。这个文件告诉 TypeScript 导入 .vue 文件模块都按 VueConstructor<Vue> 类型识别处理

declare module '*.vue' {
  import Vue from 'vue'
  export default Vue
}
```

#### shims-tsx.d.ts 文件的作用

```typescript
// 为 jsx 组件模板补充类型声明

import Vue, { VNode } from 'vue'

declare global {
  namespace JSX {
    // tslint:disable no-empty-interface
    interface Element extends VNode {}
    // tslint:disable no-empty-interface
    interface ElementClass extends Vue {}
    interface IntrinsicElements {
      [elem: string]: any
    }
  }
}
```

#### 定义组件的方式

##### 使用 Options APIs

* 组件仍然可以使用以前的方式定义（导出组件选项对象，或者使用 Vue.extend()）
* 但是当我们导出的是一个普通的对象，此时 TypeScript 无法推断出对应的类型
* 至于 VSCode 可以推断出类型成员的原因是因为我们使用了 Vue 插件
* 这个插件明确知道我们这里导出的是一个 Vue 对象
* 所以我们必须使用 ```Vue.extend()``` 方法确保 TypeScript 能够有正常的类型推断

```js
const component = {
	// 这里不会有类型推断
    // 因为 TypeScript 不能确认这是 Vue 组件的选项
}
```

```vue
<script lang="ts">
    import Vue from 'vue'
    export default Vue.extend({
        // 类型推断已启用
        name:'Button',
        data(){
            return {
                count: 1
            }
        }
    })
</script>
```

##### 使用 Class APIs 定义 Vue 组件

[https://class-component.vuejs.org](https://class-component.vuejs.org/)

在 TypeScript 下，Vue 组件可以使用一个继承自 Vue 类型的子类表示，这种类型需要使用 Component 装饰器去修饰

装饰器函数接收的参数就是以前的组件选项对象（data、props、methods之类）

```vue
// 基于类的 Vue 组件--使用 vue-class-component 装饰器
<script lang="ts">
    import Vue from 'vue'
    import Component from 'vue-class-component'

    // @Component 修饰符注明了此类为一个 Vue 组件
    @Component({
      // 所有的组件选项都可以放在这里
      template: '<button @click="onClick">Click!</button>',
      props:{
          size:String
      }
    })
    export default class MyComponent extends Vue {
      // 初始数据可以直接声明为实例的 property
      message: string = 'Hello!'

      // 组件方法也可以直接声明为实例的方法
      onClick (): void {
        window.alert(this.message)
      }
    }
</script>
```

##### 关于装饰器语法

[https://es6.ruanyifeng.com/#docs/decorator](https://es6.ruanyifeng.com/#docs/decorator)

装饰器 是 ES 草案中的一个特性，不建议在生产环境使用

类的装饰器

```typescript
function testable (target) {
    target.isTestable = true
}
@testable
class MyTestableClass {
    // ...
}
console.log(MyTestableClass.isTestable) // true
```

##### 使用 VuePropertyDecorator 创建 Vue 组件

[https://github.com/kaorun343/vue-property-decorator](https://github.com/kaorun343/vue-property-decorator)

该项目已经自动安装了

**个人建议 使用 Options APIs**:

* Class 语法仅仅是一种写法而已，最终还是要转换为普通的组件数据结构
* 装饰器语法还没有正式定稿发布，建议了解即可，正式发布之后再使用

#### 加入 Git 版本管理

1. 创建远程仓库
2. 将本地仓库推到线上

```bash
git init
git add .
git commit -m '提交日志'
git remote add origin 远程仓库地址
git push -u origin master
```

#### 项目目录结构说明

```dockerfile
 .
 ├── node_modules # 第三⽅包存储⽬录
 ├── public # 静态资源⽬录，任何放置在 public ⽂件夹的静态资源都会被简单的复
制，⽽不经过 webpack
 │ ├── favicon.ico
 │ └── index.html
 ├── src
 │ ├── assets # 公共资源⽬录，放图⽚等资源
 │ ├── components # 公共组件⽬录
 │ ├── router # 路由相关模块
 │ ├── store # 容器相关模块
 │ ├── views # 路由⻚⾯组件存储⽬录
 │ ├── App.vue # 根组件，最终被替换渲染到 index.html ⻚⾯中 #app ⼊⼝
节点
 │ ├── main.ts # 整个项⽬的启动⼊⼝模块
 │ ├── shims-tsx.d.ts # ⽀持以 .tsc 结尾的⽂件，在 Vue 项⽬中编写 jsx
代码
 │ └── shims-vue.d.ts # 让 TypeScript 识别 .vue 模块
 ├── .browserslistrc # 指定了项⽬的⽬标浏览器的范围。这个值会被 @babel/pre
set-env 和 Autoprefixer ⽤来确定需要转译的 JavaScript 特性和需要添加的 C
SS 浏览器前缀
 ├── .editorconfig # EditorConfig 帮助开发⼈员定义和维护跨编辑器（或IDE）
的统⼀的代码⻛格
 ├── .eslintrc.js # ESLint 的配置⽂件
 ├── .gitignore # Git 的忽略配置⽂件，告诉Git项⽬中要忽略的⽂件或⽂件夹
 ├── README.md # 说明⽂档
 ├── babel.config.js # Babel 配置⽂件
 ├── package-lock.json # 记录安装时的包的版本号，以保证⾃⼰或其他⼈在 npm i
nstall 时⼤家的依赖能保证⼀致
 ├── package.json # 包说明⽂件，记录了项⽬中使⽤到的第三⽅包依赖信息等内容
 └── tsconfig.json # TypeScript 配置⽂件
```

#### 调整初始目录结构

默认生成的目录结构不满足我们的开发需求，所以需要做一些自定义改动。

* 删除初始化的默认文件
* 新增调整我们需要的目录结构

修改 ```App.vue``` :

```vue
<template>
	<div id="app">
        <h1>App</h1>
        
        <!-- 根级路由出口 -->
        <router-view />
    </div>
</template>

<style lang="scss" scoped></style>
```

修改 ```router/index.ts``` :

```js
import Vue from 'vue'
import VueRouter, { RouteConfig } from 'vue-router'

Vue.use(VueRouter)

// 路由配置规则
const routes: Array<RouteConfig> = []

const router = new VueRouter({
  routes
})

export default router
```

删除默认示例文件：

* src/views/About.vue
* src/views/Home.vue
* src/components/HelloWorld.vue
* src/assets/logo.png

创建以下内容

* src/services 目录：接口模块
* src/utils 目录：存储一些工具模块
* src/styles 目录：存储一些样式资源

调整后的目录结构如下：

```dockerfile
 .
 ├── node_modules 
 ├── public 
 │ ├── favicon.ico
 │ └── index.html
 ├── src
 │ ├── assets 
 │ ├── components 
 │ ├── router 
 │ ├── services 
 │ ├── store 
 │ ├── styles
 │ ├── utils
 │ ├── views
 │ ├── App.vue 
 │ ├── main.ts 
 │ ├── shims-tsx.d.ts 
 │ └── shims-vue.d.ts 
 ├── .browserslistrc 
 ├── .editorconfig 
 ├── .eslintrc.js
 ├── .gitignore 
 ├── README.md
 ├── babel.config.js 
 ├── package-lock.json 
 ├── package.json 
 └── tsconfig.json
```

### 风格指南

本项⽬的⻛格指南主要是参照 vue 官⽅的[⻛格指南]([风格指南 — Vue.js (vuejs.org)](https://cn.vuejs.org/v2/style-guide/index.html))。在真正开始使⽤该项⽬之前建议先阅读⼀遍指南， 这能帮助让你写出更规范和统⼀的代码。当然每个团队都会有所区别。其中⼤部分规则也都配置在了 [eslint-plugin-vue](https://github.com/vuejs/eslint-plugin-vue)之中，当没有遵循规则的时候会报错，详细内容⻅[eslint]([ESLint | vue-element-admin (gitee.io)](https://panjiachen.gitee.io/vue-element-admin-site/zh/guide/advanced/eslint.html))章节。 

当然也有⼀些特殊的规范，是不能被 eslint 校验的。需要⼈为的⾃⼰注意，并且来遵循。最主要的就是⽂ 件的命名规则，这⾥拿 vue-element-admin 来举例。

#### Component

所有的```Component```文件都是以大写开头，除了```index.vue```

* ```@/components/BackToTop/index.vue ```
* ```@/components/Charts/Line.vue ```
* ```@/views/example/components/Button.vue```

#### JavaScript 文件

所有的```.js```文件都遵循横线连接（kebab-case)

* ```@/utils/open-window.js ```
* ```@/views/svg-icons/require-icons.js ```
* ```@/components/MarkdownEditor/default-options.js```

#### Views

在```views```文件下，代表路由的```.vue```文件都使用横线连接（kebab-case)，代表路由的文件夹也是使用同样的规则

* ```@/views/svg-icons/index.vue ```
* ```@/views/svg-icons/require-icons.js```

使⽤横线连接 (kebab-case)来命名``` views ```主要是出于以下⼏个考虑。

* 横线连接 (kebab-case) 也是官⽅推荐的命名规范之⼀ [⽂档 ]([风格指南 — Vue.js (vuejs.org)](https://cn.vuejs.org/v2/style-guide/index.html#单文件组件文件的大小写强烈推荐))
* views 下的 .vue ⽂件代表的是⼀个路由，所以它需要和 component 进⾏区分(component 都是⼤ 写开头) 
* ⻚⾯的 url 也都是横线连接的，⽐如 https://www.xxx.admin/export-excel ，所以路由对应 的 view 应该要保持统⼀ 
* 没有⼤⼩写敏感问题

### 代码规范和 ESLint

不管是多⼈合作还是个⼈项⽬，代码规范都是很重要的。这样做不仅可以很⼤程度地避免基本语法错误， 也保证了代码的可读性。 

这⾥主要说明以下⼏点： 

* 代码格式规范介绍 
* 我们项⽬中配置的具体代码规范是什么 
* 遇到代码格式规范错误怎么办 
* 如何⾃定义代码格式校验规范

#### 代码规范介绍

良好的代码格式规范更有利于： 

* 更好的多⼈协作 

* 更好的阅读 

* 更好的维护 ...

#### 标准是什么

没有绝对的标准，下⾯是⼀些⼤⼚商根据多数开发者的编码习惯制定的⼀些编码规范，仅供参考。

* 自己用：[JavaScript Standard Style]([JavaScript Standard Style (standardjs.com)](https://standardjs.com/))
* 工作大型团队用：[Airbnb JavaScript Style Guide]([GitHub - airbnb/javascript: JavaScript Style Guide](https://github.com/airbnb/javascript))
* 工作大型团队用：[Google JavaScript Style Guide]([Google JavaScript Style Guide](https://google.github.io/styleguide/jsguide.html))

#### 如何约束代码规范

只靠⼝头约定肯定是不⾏的，所以要利⽤⼯具来强制执⾏。 

* [JSLint ]([JSLint: The JavaScript Code Quality Tool](https://www.jslint.com/))
* [JSHint ]([JSHint, a JavaScript Code Quality Tool](https://jshint.com/))
* [ESLint]([JSHint, a JavaScript Code Quality Tool](https://jshint.com/)) ...

#### 项目中的代码规范是什么

ESLint 配置文件 ```.eslintrc.js```

```js
module.exports = {
  root: true,
  env: {
    node: true
  },
  // 插件：扩展了校验规则
  extends: [
    'plugin:vue/essential', // eslint-plugin-vue
    '@vue/standard', // @vue/eslint-config-standard
    '@vue/typescript/recommended' // @vue/eslint-config-typescript
  ],
  parserOptions: {
    ecmaVersion: 2020
  },
  // 自定义校验规则
  rules: {
    'no-console': process.env.NODE_ENV === 'production' ? 'warn' : 'off',
    'no-debugger': process.env.NODE_ENV === 'production' ? 'warn' : 'off',
      // 关闭 函数名与左括号之间的空格校验
    'space-before-function-paren': 0
  }
}
```

* **eslint-plugin-vue**
  * GitHub 仓库：[https://github.com/vuejs/eslint-plugin-vue ](https://github.com/vuejs/eslint-plugin-vue )
  * 官⽅⽂档：[https://eslint.vuejs.org/ ](https://eslint.vuejs.org/ )
  * 该插件使我们可以使⽤ ESLint 检查 ```.vue``` ⽂件的 ```<template>``` 和 ```<script>```
  * 查找语法错误
  * 查找对 Vue.js 指令的错误使用
  * 查找违反 Vue.js 样式指南的行为
* **@vue/eslint-config-standard**
  * [eslint-plugin-standard ](https://github.com/standard/eslint-plugin-standard)
  * [JavaScript Standard Style]([JavaScript Standard Style (standardjs.com)](https://standardjs.com/))
* **@vue/eslint-config-typescript**
  * 规则列表：https://github.com/typescript-eslint/typescript-eslint/tree/master/packages/eslint-plugin#supported-rules 

#### 如何自定义代码格式校验规范

[https://eslint.bootcss.com/docs/user-guide/getting-started](https://eslint.bootcss.com/docs/user-guide/getting-started)

```json
{
    "rules":{
        "semi":["errors","always"],
        "quotes":["errors","double"]
    }
}
```

ESLint 附带有⼤量的规则。你可以使⽤注释或配置⽂件修改你项⽬中要使⽤的规则。要改变⼀个规则设 置，你必须将规则 ID 设置为下列值之⼀：

* ```"off"`` 或 ```0``` - 关闭规则 
* ```"warn" ```或 ```1``` - 开启规则，使⽤警告级别的错误： ```warn``` (不会导致程序退出) 
* ```"error" ```或 ```2 ```- 开启规则，使⽤错误级别的错误： ```error``` (当被触发的时候，程序会退出)

为了在⽂件注释⾥配置规则，使⽤以下格式的注释：

```js
/* eslint eqeqeq: "off", curly: "error" */
```

在这个例⼦⾥，``` eqeqeq``` 规则被关闭，``` curly ```规则被打开，定义为错误级别。你也可以使⽤对应的数字 定义规则严重程度：

```js
/* eslint eqeqeq: 0, curly: 2 */
```

这个例⼦和上个例⼦是⼀样的，只不过它是⽤的数字⽽不是字符串。``` eqeqeq``` 规则是关闭的， ```curly``` 规 则被设置为错误级别。 如果⼀个规则有额外的选项，你可以使⽤数组字⾯量指定它们，⽐如：

```js
 /* eslint quotes: ["error", "double"], curly: 2 */
```

这条注释为规则 ```quotes``` 指定了 ```“double”```选项。数组的第⼀项总是规则的严重程度（数字或字符串）。

还可以使⽤ ```rules``` 连同错误级别和任何你想使⽤的选项，在配置⽂件中进⾏规则配置。例如：

```json
 {
     "rules": {
         "eqeqeq": "off",
         "curly": "error",
         "quotes": ["error", "double"]
     }
 }
```

配置定义在插件中的⼀个规则的时候，你必须使⽤ ```插件名/规则ID``` 的形式。⽐如：

```json
{
    "plugins":[
        "plugin1"
    ],
    "rules":{
        "eqeqed":"off",
        "curly":"error",
        "quotes":["error","double"],
        "plugin1/rule1":"error"
    }
}
```

在这些配置⽂件中，规则 ```plugin1/rule1 ```表示来⾃插件 ```plugin1``` 的 ```rule1``` 规则。你也可以使⽤ 这种格式的注释配置，⽐如：

```js
/* eslint "plugin1/rule1": "error" */
```

注意：当指定来⾃插件的规则时，确保删除 ```eslint-plugin-``` 前缀。ESLint 在内部只使⽤没有前缀的 名称去定位规则。

#### vscode 配置 ESLint

##### vscode  安装 eslint

安装并配置完成 ESLint 后，我们继续回到 VSCode 进⾏扩展设置，依次点击 ⽂件 > ⾸选项 > 设置 打 开 VSCode 配置⽂件,添加如下配置：

```json
{
    "files.autoSave": "off",
    "eslint.validate": [
        "javascript",
        "javascriptreact",
        "vue-html",
        {
            "language": "vue",
            "autoFix": true
        }
    ],
    "eslint.run": "onSave",
    "eslint.autoFixOnSave": true
}
```



#### 自动修复

```bash
npm run lint -- --fix
```



#### Git Hooks

### 布局

#### 导入 Element 组件库

Element，⼀套为开发者、设计师和产品经理准备的基于 Vue 2.0 的桌⾯端组件库。

* 官⽹：[https://element.eleme.cn/ ](https://element.eleme.cn/)
* GitHub 仓库：[https://github.com/ElemeFE/element](https://github.com/ElemeFE/element)

1. 安装 element

   ```bash
   npm i element-ui
   ```

2. 在```main.ts```中导入配置（完整引入）

   ```typescript
   import Vue from 'vue'
   import App from './App.vue'
   import router from './router'
   import store from './store'
   import ElementUI from 'element-ui'
   import 'element-ui/lib/theme-chalk/index.css'
   
   Vue.use(ElementUI)
   
   Vue.config.productionTip = false
   
   new Vue({
     router,
     store,
     render: h => h(App)
   }).$mount('#app')
   ```

3. 按需引入

   借助 [babel-plugin-component](https://github.com/QingWei-Li/babel-plugin-component)，我们可以只引入需要的组件，以达到减小项目体积的目的。

   首先，安装 babel-plugin-component：

   ```bash
   npm install babel-plugin-component -D
   ```

   然后，将 .babelrc 修改为:

   ```js
   {
     "presets": [["es2015", { "modules": false }]],
     "plugins": [
       [
         "component",
         {
           "libraryName": "element-ui",
           "styleLibraryName": "theme-chalk"
         }
       ]
     ]
   }
   ```

   接下来，如果你只希望引入部分组件，比如 Button 和 Select，那么需要在 main.js 中写入以下内容：

   ```typescript
   import Vue from 'vue';
   import router from './router'
   import store from './store'
   import { Button, Select } from 'element-ui';
   import App from './App.vue';
   
   Vue.component(Button.name, Button);
   Vue.component(Select.name, Select);
   /* 或写为
    * Vue.use(Button)
    * Vue.use(Select)
    */
   Vue.config.productionTip = false
   
   new Vue({
     router,
     store,
     render: h => h(App)
   }).$mount('#app')
   ```

4. 全局配置

   在引入 Element 时，可以传入一个全局配置对象。该对象目前支持 `size` 与 `zIndex` 字段。`size` 用于改变组件的默认尺寸，`zIndex` 设置弹框的初始 z-index（默认值：2000）。按照引入 Element 的方式，具体操作如下：

   * 完整引入 Element：

   ```js
   import Vue from 'vue';
   import Element from 'element-ui';
   Vue.use(Element, { size: 'small', zIndex: 3000 });
   ```

   * 按需引入 Element：

   ```js
   import Vue from 'vue';
   import { Button } from 'element-ui';
   
   Vue.prototype.$ELEMENT = { size: 'small', zIndex: 3000 };
   Vue.use(Button);
   ```



#### 样式处理

```dockerfile
 src/styles
 ├── index.scss # 全局样式（在⼊⼝模块被加载⽣效）
 ├── mixin.scss # 公共的 mixin 混⼊（可以把重复的样式封装为 mixin 混⼊到复⽤
的地⽅）
 ├── reset.scss # 重置基础样式
 └── variables.scss # 公共样式变量

```

* ```variables.scss```

  ```scss
  $primary-color: #40586F;
  $success-color: #51cf66;
  $warning-color: #fcc419;
  $danger-color: #ff6b6b;
  $info-color: #868e96; // #22b8cf;
  
  $body-bg: #E9EEF3; // #f5f5f9;
  
  $sidebar-bg: #F8F9FB;
  $navbar-bg: #F8F9FB;
  
  $font-family: system-ui, -apple-system, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
  ```

* ```index.scss```

  ```scss
  @import './variables.scss';
  
  // globals
  html {
    font-family: $font-family;
    -webkit-text-size-adjust: 100%;
    -webkit-tap-highlight-color: rgba(0, 0, 0, 0);
    // better Font Rendering
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
  }
  
  body {
    margin: 0;
    background-color: $body-bg;
  }
  
  // custom element theme
  $--color-primary: $primary-color;
  $--color-success: $success-color;
  $--color-warning: $warning-color;
  $--color-danger: $danger-color;
  $--color-info: $info-color;
  // change font path, required
  $--font-path: '~element-ui/lib/theme-chalk/fonts';
  // import element default theme
  @import '~element-ui/packages/theme-chalk/src/index';
  // node_modules/element-ui/packages/theme-chalk/src/common/var.scss
  
  // overrides
  
  // .el-menu-item, .el-submenu__title {
  //   height: 50px;
  //   line-height: 50px;
  // }
  
  .el-pagination {
    color: #868e96;
  }
  
  // components
  
  .status {
    display: inline-block;
    cursor: pointer;
    width: .875rem;
    height: .875rem;
    vertical-align: middle;
    border-radius: 50%;
  
    &-primary {
      background: $--color-primary;
    }
  
    &-success {
      background: $--color-success;
    }
  
    &-warning {
      background: $--color-warning;
    }
  
    &-danger {
      background: $--color-danger;
    }
  
    &-info {
      background: $--color-info;
    }
  }
  ```

  

* ```reset.scss```

  

* ```mixin.scss```

* ```main.ts``` 中引入

  ```typescript
  import Vue from 'vue'
  import App from './App.vue'
  import router from './router'
  import store from './store'
  import ElementUI from 'element-ui'
  
  // index.scss文件中已经引入了
  // import 'element-ui/lib/theme-chalk/index.css' 
  
  // 加载全局样式
  import './styles/index.scss'
  
  Vue.use(ElementUI)
  
  Vue.config.productionTip = false
  
  new Vue({
    router,
    store,
    render: h => h(App)
  }).$mount('#app')
  
  ```

  

##### ⽗组件改变⼦组件样式 深度选择器

建议你使⽤ ::v-deep 的写法，它不仅兼容了 css 的 >>> 写法，还兼容了 sass /deep/ 的写法。⽽且 它还是 vue 3.0 RFC 中指定的写法。 

⽽且原本 /deep/ 的写法也本身就被 Chrome 所废弃，你现在经常能在控制台中发现 Chrome 提示你 不要使⽤ /deep/ 的警告。

##### 共享全局样式变量

全局注入 ```variables.scss``` 的变量

有的时候你想要向 webpack 的预处理器 loader 传递选项。你可以使用 `vue.config.js` 中的 `css.loaderOptions` 选项。比如你可以这样向所有 Sass/Less 样式传入共享的全局变量：

参考： https://cli.vuejs.org/zh/guide/css.html#%E5%90%91%E9%A2%84%E5%A4%84%E7%90%86%E5%99%A8-loader-%E4%BC%A0%E9%80%92%E9%80%89%E9%A1%B9

```js
// vue.config.js
module.exports = {
  css: {
    loaderOptions: {
      // 默认情况下 `sass` 选项会同时对 `sass` 和 `scss` 语法同时生效
      // 因为 `scss` 语法在内部也是由 sass-loader 处理的
      // 但是在配置 `prependData` 选项的时候
      // `scss` 语法会要求语句结尾必须有分号，`sass` 则要求必须没有分号
      // 在这种情况下，我们可以使用 `scss` 选项，对 `scss` 语法进行单独配置
      scss: {
        additionalData: `@import "~@styles/variables.scss";`
      }
    }
  }
}
```

```vue
<template>
  <div id="app">
    <h1>App</h1>
    <p class="text">hello world</p>
    <!--根级路由出口-->
    <router-view />
  </div>
</template>
<script lang="ts">
import Vue from 'vue'

interface Foo {
  a: string
  b: number
}

export default Vue.extend({})
</script>
<style lang="scss" scoped>
// @import '~@/styles/variables.scss';
// 全局引入过，所以可以直接使用，不用再引入
.text {
  color: $danger-color;
}
</style>

```



### 服务端接口说明

后台为我们提供了数据接口，分别是：

* http://eduboss.lagou.com

  用户名：15510792995   密码：111111
  用户名： 18201288771  密码： 111111

* http://eduboss.lagou.com/boss/doc.html#/home

* http://edufront.lagou.com

  http://113.31.105.128/front/doc.html#/home

这两个接口都没有提供 CORS 跨域请求，所以需要在客户端配置服务端代理处理跨域请求

配置客户端层⾯的服务端代理跨域可以参考官⽅⽂档中的说明： 

* https://cli.vuejs.org/zh/config/#devserver-proxy 
* https://github.com/chimurai/http-proxy-middleware

下面是具体的操作流程：

```js
// vue.config.js
module.exports = {
  devServer: {
    proxy: {
      '/boss': {
        target: 'https://eduboss.lagou.com',
        // ws: true, // webSocket 协议
        changeOrigin: true // 把请求头中的 host 配置为 target
      },
      '/front': {
        target: 'http://edufront.lagou.com',
        changeOrigin: true
      }
    }
  }
}
```

#### 接口跨域问题

* cors ：生产环境可使用

  我最推荐的也是我⼯作中在使⽤的⽅式就是： cors 全称为 Cross Origin Resource Sharing（跨域资 源共享）。这种⽅案对于前端来说没有什么⼯作量，和正常发送请求写法上没有任何区别，⼯作量基本都 在后端这⾥。每⼀次请求，浏览器必须先以 OPTIONS 请求⽅式发送⼀个预请求（也不是所有请求都会发 送 options，展开介绍 ([点我CS | awesome-bookmarks (panjiachen.github.io)](https://panjiachen.github.io/awesome-bookmarks/blog/cs.html#cors)），通过预检请求从⽽获知服务器端对跨源请求⽀持的 HTTP ⽅法。在确认 服务器允许该跨源请求的情况下，再以实际的 HTTP 请求⽅法发送那个真正的请求。推荐的原因是：只要 第⼀次配好了，之后不管有多少接⼝和项⽬复⽤就可以了，⼀劳永逸的解决了跨域问题，⽽且不管是开发 环境还是正式环境都能⽅便的使⽤。详细 MDN[跨源资源共享（CORS） - HTTP | MDN (mozilla.org)](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/CORS) ⽂档

* webpack 的 proxy 代理：正向代理，生产环境不能使用

  [开发中 Server(devServer) | webpack 中文网 (webpackjs.com)](https://www.webpackjs.com/configuration/dev-server/#devserver-proxy)

* nginx 反向代理：生产环境可使用

#### 封装请求模块

安装 ```axios```

```bash
npm i axios
```

创建 ```src/utils/request.js```

```js
/**
 * 用户相关请求模块
*/

import request from '@/utils/request'
import qs from 'qs'

interface User {
  phone: string
  password: string
}

export const login = (data: User) => {
  return request({
    method: 'POST',
    url: '/front/user/login',
    // qs 转化参数后，axios 会自动将请求格式设置为 x-www-form-urlencoded，所以不需要在设置 headers 的 content-type，key=value&key=value
    // 不用 qs 转化，axios 默认发送的类型是 application/json {key:value,key:value}
    // 如果 data 是 formData 对象，则 content-type 是 multipart/form-data
    data: qs.stringify(data)
    // headers: {
    //   'content-type': 'application/x-www-form-urlencoded'
    // }
  })
}

export const getUserInfo = () => {
  return request({
    method: 'GET',
    url: '/front/user/getInfo'
  })
}

```

请求方法封装

``` services/user.ts```

```js
/**
 * 用户相关请求模块
*/

import request from '@/utils/request.js'
import qs from 'qs'

interface User {
  phone: string
  password: string
}

export const login = (data: User) => {
  return request({
    method: 'POST',
    url: '/front/user/login',
    // axios 默认发送的类型是 application/json 数据，这个接口的数据格式是 x-www-form-urlencoded,
    // 设置 headers,qs 序列化参数
    data: qs.stringify(data),
    headers: {
      'content-type': 'application/x-www-form-urlencoded'
    }
  })
}

```

#### 请求数据格式转换,请求格式为 x-www-form-urlencoded 的处理方式

axios 默认的请求数据格式为 ```‘application/json'```,如果接口支持的格式为其他需要进行转换

* 安装 ``` qs ```

  ```
  npm i qs
  ```

* 使用

  ```js
  /**
   * 用户相关请求模块
  */
  
  import request from '@/utils/request.js'
  import qs from 'qs'
  
  interface User {/*  */
    phone: string
    password: string
  }
  
  export const login = (data: User) => {
    return request({
      method: 'POST',
      url: '/front/user/login', // 这个接口的数据格式是 x-www-form-urlencoded,
      // qs 转化参数后，axios 会自动将请求格式设置为 x-www-form-urlencoded，所以不需要在设置 headers 的 content-type，key=value&key=value
      // 不用 qs 转化，axios 默认发送的类型是 application/json {key:value,key:value}
      // 如果 data 是 formData 对象，则 content-type 是 multipart/form-data
      data: qs.stringify(data)
      // headers: {
      //   'content-type': 'application/x-www-form-urlencoded'
      // }
    })
  }
  
  ```

  

#### 配置环境变量

#### 初始化路由页面组件

views 文件夹下创建以下目录

| 路径          | 说明       |
| ------------- | ---------- |
| /             | 首页       |
| /login        | 用户登录   |
| /role         | 角色管理   |
| /menu         | 菜单管理   |
| /resource     | 资源管理   |
| /course       | 课程管理   |
| /user         | 用户管理   |
| /advert       | 广告管理   |
| /advert-space | 广告位管理 |

#### 路由和侧边栏

```router/index.ts```

```typescript
import Vue from 'vue'
import VueRouter, { RouteConfig } from 'vue-router'

Vue.use(VueRouter)

// 路由配置规则
const routes: Array<RouteConfig> = [
  {
    path: '/',
    name: 'home',
    // 通过注释起个别名
    // 路由懒加载
    component: () => import(/* webpackChunkName:'home' */ '@/views/home/index.vue')
  },
  {
    path: '/login',
    name: 'login',
    component: () => import(/* webpackChunkName:'login' */ '@/views/login/index.vue')
  },
  {
    path: '/role',
    name: 'role',
    component: () => import(/* webpackChunkName:'role' */ '@/views/role/index.vue')
  },
  {
    path: '/menu',
    name: 'menu',
    component: () => import(/* webpackChunkName:'menu' */ '@/views/menu/index.vue')
  },
  {
    path: '/resource',
    name: 'resource',
    component: () => import(/* webpackChunkName:'resource' */ '@/views/resource/index.vue')
  },
  {
    path: '/course',
    name: 'course',
    component: () => import(/* webpackChunkName:'course' */ '@/views/course/index.vue')
  },
  {
    path: '/user',
    name: 'user',
    component: () => import(/* webpackChunkName:'user' */ '@/views/user/index.vue')
  },
  {
    path: '/advert',
    name: 'advert',
    component: () => import(/* webpackChunkName:'advert' */ '@/views/advert/index.vue')
  },
  {
    path: '/advert-space',
    name: 'advert-space',
    component: () => import(/* webpackChunkName:'advert-space' */ '@/views/advert-space/index.vue')
  },
  {
    path: '*', // 现在放哪个位置都行，vue-router 内部会放到最后，建议放到最后
    name: '404',
    component: () => import(/* webpackChunkName:'404' */ '@/views/error-page/404.vue')
  }
]

const router = new VueRouter({
  routes
})

export default router
```



#### Layout 布局

vue-router 嵌套路由 [https://router.vuejs.org/zh/guide/essentials/nested-routes.html](https://router.vuejs.org/zh/guide/essentials/nested-routes.html)

。。。借助 element  快速搭建

1. ```layout/index.vue```

```vue
<template>
  <el-container>
    <el-aside width="200px">
      <app-aside />
    </el-aside>
    <el-container>
      <el-header>
        <app-header />
      </el-header>
      <el-main>
        <!-- 子路由出口 -->
        <router-view />
      </el-main>
    </el-container>
  </el-container>
</template>

<script lang="ts">
import Vue from 'vue'
import AppAside from './components/app-aside.vue'
import AppHeader from './components/app-header.vue'

export default Vue.extend({
  name: 'LayoutIndex',
  components: {
    AppAside,
    AppHeader
  }
})
</script>

<style lang="scss" scoped>
.el-container {
  min-height: 100vh;
  min-width: 980px;
}

.el-header {
  background-color: #f8f9fb;
}

.el-aside {
  background-color: #d3dce6;
}

.el-main {
  background-color: #e9eef3;
}
</style>
```

2. 侧边栏 ```layout/components/app-aside.vue```

```vue
<template>
  <div class="aside">
    <el-menu default-active="2"
             class="el-menu-vertical-demo"
             @open="handleOpen"
             @close="handleClose"
             background-color="#545c64"
             text-color="#fff"
             active-text-color="#ffd04b"
             router>
      <el-submenu index="1">
        <template slot="title">
          <i class="el-icon-location"></i>
          <span>权限管理</span>
        </template>
        <el-menu-item-group>
          <el-menu-item index="/role">
            <i class="el-icon-location"></i>
            <span>角色列表</span>
          </el-menu-item>
          <el-menu-item index="/menu">
            <i class="el-icon-location"></i>
            <span>菜单管理</span>
          </el-menu-item>
          <el-menu-item index="/resource">
            <i class="el-icon-location"></i>
            <span>资源管理</span>
          </el-menu-item>
        </el-menu-item-group>
      </el-submenu>

      <el-menu-item index="/course">
        <i class="el-icon-menu"></i>
        <span slot="title">课程管理</span>
      </el-menu-item>
      <el-menu-item index="/user">
        <i class="el-icon-document"></i>
        <span slot="title">用户管理</span>
      </el-menu-item>
      <el-submenu index="4">
        <template slot="title">
          <i class="el-icon-location"></i>
          <span>广告管理</span>
        </template>
        <el-menu-item-group>
          <el-menu-item index="/advert">
            <i class="el-icon-location"></i>
            <span>广告列表</span>
          </el-menu-item>
          <el-menu-item index="/advert-space">
            <i class="el-icon-setting"></i>
            <span>广告位管理</span>
          </el-menu-item>
        </el-menu-item-group>
      </el-submenu>
    </el-menu>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
export default Vue.extend({
  name: 'AppAside',
  methods: {
    handleOpen(key: string, keyPath: string): void {
      console.log(key, keyPath)
    },
    handleClose(key: string, keyPath: string): void {
      console.log(key, keyPath)
    }
  }
})
</script>

<style lang="scss" scoped>
.aside {
  .el-menu {
    min-height: 100vh;
  }
}
</style>

```

3. 头部```layout/components/app-header.vue```

```vue
<template>
  <div class="header">
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>活动管理</el-breadcrumb-item>
      <el-breadcrumb-item>活动列表</el-breadcrumb-item>
      <el-breadcrumb-item>活动详情</el-breadcrumb-item>
    </el-breadcrumb>

    <el-dropdown>
      <span class="el-dropdown-link">
        <el-avatar shape="square"
                   :size="40"
                   src="https://cube.elemecdn.com/9/c2/f0ee8a3c7c9638a54940382568c9dpng.png"></el-avatar>
        <i class="el-icon-arrow-down el-icon--right"></i>
      </span>
      <el-dropdown-menu slot="dropdown">
        <el-dropdown-item>用户ID</el-dropdown-item>
        <el-dropdown-item divided>登出</el-dropdown-item>
      </el-dropdown-menu>
    </el-dropdown>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
export default Vue.extend({
  name: 'AppHeader'
})
</script>

<style lang="scss" scoped>
.header {
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: space-between;
  .el-dropdown-link {
    display: flex;
    align-items: center;
  }
}
</style>

```



[https://element.eleme.cn/#/zh-CN](https://element.eleme.cn/#/zh-CN)

```router/index.ts```

```typescript
import Vue from 'vue'
import VueRouter, { RouteConfig } from 'vue-router'
import Layout from '@/layout/index.vue'

Vue.use(VueRouter)

// 路由配置规则
const routes: Array<RouteConfig> = [
  {
    path: '/login',
    name: 'login',
    component: () => import(/* webpackChunkName:'login' */ '@/views/login/index.vue')
  },
  {
    path: '/',
    component: Layout,
    children: [
      {
        path: '',  // 默认子路由，匹配到 / 时会默认渲染这个组件
        name: 'home',
        // 通过注释起个别名
        component: () => import(/* webpackChunkName:'home' */ '@/views/home/index.vue')
      },
      {
        path: '/role',
        name: 'role',
        component: () => import(/* webpackChunkName:'role' */ '@/views/role/index.vue')
      },
      {
        path: '/menu',
        name: 'menu',
        component: () => import(/* webpackChunkName:'menu' */ '@/views/menu/index.vue')
      },
      {
        path: '/resource',
        name: 'resource',
        component: () => import(/* webpackChunkName:'resource' */ '@/views/resource/index.vue')
      },
      {
        path: '/course',
        name: 'course',
        component: () => import(/* webpackChunkName:'course' */ '@/views/course/index.vue')
      },
      {
        path: '/user',
        name: 'user',
        component: () => import(/* webpackChunkName:'user' */ '@/views/user/index.vue')
      },
      {
        path: '/advert',
        name: 'advert',
        component: () => import(/* webpackChunkName:'advert' */ '@/views/advert/index.vue')
      },
      {
        path: '/advert-space',
        name: 'advert-space',
        component: () => import(/* webpackChunkName:'advert-space' */ '@/views/advert-space/index.vue')
      }
    ]
  },
  {
    path: '*',
    name: '404',
    component: () => import(/* webpackChunkName:'404' */ '@/views/error-page/404.vue')
  }
]

const router = new VueRouter({
  routes
})

export default router

```



#### 登录页面布局

* 布局

* 登录接口请求

  * 请求格式处理，axios 默认发送的类型是 application/json 数据，这个接口的数据格式是 x-www-form-urlencoded，设置 headers,qs 序列化参数

  * 请求成功跳转到首页，提示登录成功

  * 请求失败提示错误原因

  * 表单校验 

    [https://github.com/yiminghe/async-validator](https://github.com/yiminghe/async-validator)

* 请求期间禁用按钮

```vue
<template>
  <div class="login">
    <h1>Edu boss管理系统</h1>
    <el-form ref="form"
             label-position="top"
             :model="form"
             :rules="rules"
             class="login-form"
             label-width="80px">
      <el-form-item label="手机号"
                    prop="phone">
        <el-input v-model="form.phone"></el-input>
      </el-form-item>
      <el-form-item label="密码"
                    prop="password">
        <el-input type="password"
                  v-model="form.password"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary"
                   class="login-btn"
                   :loading="isLoginLoading"
                   @click="onSubmit">登录</el-button>
      </el-form-item>
    </el-form>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import { login } from '@/services/user.ts'
import { Form } from 'element-ui'
export default Vue.extend({
  name: 'LoginIndex',
  data() {
    return {
      form: {
        phone: '18201288771',
        password: '111111'
      },
      rules: {
        phone: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          {
            pattern: /^1\d{10}$/,
            message: '请输入正确的手机号',
            trigger: 'blur'
          }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          {
            // pattern: /^1\d{10}$/,
            min: 6,
            max: 18,
            message: '长度在 6 到 18 个字符',
            trigger: 'blur'
          }
        ]
      },
      isLoginLoading: false
    }
  },
  methods: {
    async onSubmit() {
      try {
        // 1. 表单验证
        // this.$refs.form.validate()
        await (this.$refs.form as Form).validate()
        // 登录按钮显示 loading
        this.isLoginLoading = true
        // 2. 验证通过-提交表单
        const { data } = await login(this.form)
        // 3.处理请求结果
        //   失败：给出提示
        if (data.state !== 1) {
          this.$message.error(data.message)
        } else {
          //   成功：跳转到首页
          this.$router.push({
            name: 'home'
          })
          this.$message.success('登录成功')
        }
      } catch (err) {
        this.$message.error('登录失败' + err)
      }
      // 结束登录按钮 loading
      this.isLoginLoading = false
    }
  }
})
</script>

<style lang="scss" scoped>
.login {
  height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  .login-form {
    width: 300px;
    background: #ffffff;
    padding: 20px;
    border-radius: 5px;
    .login-btn {
      width: 100%;
    }
  }
}
</style>

```

#### 路由懒加载

当打包构建应⽤时，Javascript 包会变得⾮常⼤，影响⻚⾯加载速度。如果我们能把不同路由对应的组件 分割成不同的代码块，然后当路由被访问的时候才加载对应组件，这样就更加⾼效了。 

结合 Vue 的异步组件和 Webpack 的代码分割功能，轻松实现路由组件的懒加载。如：

```js
const Foo = () => import(/* webpackChunkName: 'foo' */ './Foo.vue')
```

当你的项⽬⻚⾯越来越多之后，在开发环境之中使⽤ lazy-loading 会变得不太合适，每次更改代码触 发热更新都会变得⾮常的慢。所以建议只在⽣产环境之中使⽤路由懒加载功能。

1. 安装依赖 ```npm install babel-plugin-dynamic-import-node -S -D ```
2. 在 ```babel.config.js``` 中添加插件

```js
module.exports = {
    presets: ['@vue/cli-plugin-babel/preset'],
    env:{
        development:{
            plugins:['dynamic-import-node']
        }
    }
}
```



## 用户登录和身份认证

#### 用户登录状态存储

1. 用户登录成功，记录登录状态，登录状态存放到 vuex 容器中，容器中的数据方便共享，组件间使用方便

   ```views/login/index.vue```

   ```js
   this.$store.commit('setUser', data.content)
   ```

   

2. 数据持久化，为了防止页面刷新数据丢失，存放到本地存储

   ```store/index.ts```

   ```typescript
   import Vue from 'vue'
   import Vuex from 'vuex'
   
   Vue.use(Vuex)
   
   export default new Vuex.Store({
     state: {
       // 类型警告报错解决 || 'null'
       user: JSON.parse(window.localStorage.getItem('user') || 'null') // 当前登录状态
     },
     mutations: {
       setUser(state, payload) {
         state.user = JSON.parse(payload)
   
         // 数据持久化，为了防止页面刷新数据丢失，存放到本地存储
         // 本地存储只能存放 字符串
         window.localStorage.setItem('user', payload)
       }
     },
     actions: {
     },
     modules: {
     }
   })
   
   ```

   

3. 访问需要登录的页面时，判断有没有登录状态 路由守卫

   使用路由元信息

   从哪跳转登录页面，登录成功之后跳转回去

   ```typescript
   const routes: Array<RouteConfig> = [
       ...
       {
           path: '/',
           component: Layout,
           meta: {
             requiresAuth: true
           },
           children:[...]
        }
       ...
   ]
   // 访问需要登录的页面时，判断有没有登录状态
   // 全局前置守卫
   router.beforeEach((to, from, next) => {
     // to.matched 是一个数组（匹配到的路由记录）
     // 判断匹配到的路由是否需要登录权限，只要一个匹配到就是，因父路由需要的话子路由也需要
     if (to.matched.some(record => record.meta.requiresAuth)) {
       // 判断登录状态
       if (!store.state.user) {
         next({
           name: 'login',
           // 通过 url 传递查询字符串参数
           query: {
             redirect: to.fullPath // 把登录成功需要返回的页面告诉登录页面
           }
         })
       }
     }
     next() // 允许通过
   })
   ```

   登录页面：

   ```js
   // 1. 登录成功，记录登录状态，状态需要能够全局访问，vuex
   this.$store.commit('setUser', data.content)
   // 2. 访问需要登录的页面时，判断有没有登录状态 路由守卫
   //   成功：跳转到首页
   this.$router.push((this.$route.query.redirect as string) || '/')
   this.$message.success('登录成功')
   ```

4. 头部信息获取

   ```typescript
   import store from '@/store/index.ts'
   
   export const getUserInfo = () => {
     return request({
       method: 'GET',
       url: '/front/user/getInfo',
       headers: {
         Authorization: store.state.user.access_token
       }
     })
   }
   ```

5. 设置统一 token

   请求拦截器里设置

   ```typescript
   // 请求拦截器
   request.interceptors.request.use(function (config) {
     // 改写 config 配置信息来实现业务功能的统一处理
     const { user } = store.state
     if (user && user.access_token) {
       config.headers = {
         Authorization: user.access_token
       }
     }
   
     // 一定要返回 config,否则请求发不出去了
     return config
   }, function (error) {
     return Promise.reject(error)
   })
   ```

6. 退出登录

   * 给组件注册原生事件 .native 修饰符

     ```vue
     <el-dropdown-item divided @click.native="handleLogout">登出</el-dropdown-item>
     
     handleLogout() {
           this.$confirm('确认退出吗?', '退出提示', {
             confirmButtonText: '确定',
             cancelButtonText: '取消',
             type: 'warning'
           })
             .then(() => {
               this.$store.commit('setUser', null)
               this.$router.push({
                 name: 'login'
               })
               this.$message({
                 type: 'success',
                 message: '退出成功!'
               })
             })
             .catch(() => {
               this.$message({
                 type: 'info',
                 message: '已取消退出'
               })
             })
         }
     ```

     

#### 身份认证

##### Token 过期

Token 过期是根据后台返回的信息判断的，后台会返回 401

* 可以让用户重新登录

* 刷新 Token

  * 在请求发起前拦截每个请求，判断 Token 的有效时间是否已过期，若已过期，则将请求挂起，先刷新 token 后在继续请求

    优点：在请求前拦截，能节省请求，省流量

    缺点：需要后端额外提供一个 token 过期时间的字段，使用了本地时间判断，若本地时间被篡改，特别是本地时间比服务器时间慢时，拦截会失败

  * 不在请求前拦截，而是拦截返回后的数据，先发起请求，接口返回过期后，先刷新 token ，再进行一次重试

    优点：不需额外的 token 过期字段，不需判断时间

    缺点：会消耗多一次请求，耗流量

```
登录接口响应数据：
	access_token：获取需要授权的接口数据
	expires_in：access_token过期时间
	refresh_token：刷新获取新的 access_token
access_token 有过期时间以及时间较短：
	为了安全
```

##### axios 错误处理 - 错误消息提示

**如果是自定义状态码，错误处理就写到第一个回调**

**下面几项为：第二个回调**

**请求收到响应了，但是状态码 超出 2xx**

* 400 - 请求参数错误

* 401 - token 无效(没有，过期，无效)

  * 如果有 refresh_token ,则使用refresh_token获取新的 access_token

  ​       成功了 -> 把本次失败的请求重新发出去

  ​       失败了 -> 跳转登录页面重新登录获取新的 token

     * 没有，则直接跳转登录页面

* 403 - 没有权限

* 404 - 请求资源不存在

* 500 - 服务端错误

**请求发出去了，但是没有收到响应**

**请求的时候设置请求时发生了一些事情，触发了错误**

```typescript
// 响应拦截器
request.interceptors.response.use(function (response) {
  // 状态码为 2xx
  // 如果是自定义状态码，错误处理就写到这里

  return response
}, async function (error) {
  // 超出 2xx 状态码
  // 如果是 HTTP 状态码，错误处理就写到这里

  if (error.response) { // 请求收到响应了，但是状态码 超出 2xx
    // const errMsgObj = {
    //   '400': '请求参数错误',
    //   '401': '',
    //   '403': '',
    //   '404': '',
    //   '500': ''
    // }
    // let { status } = error.response
    // status = status + ''
    // Message.error(errMsgObj[status])
    const { status } = error.response
    if (status === 400) {
      Message.error('请求参数错误')
    } else if (status === 401) {
      // token 无效(没有，过期，无效)
      // 如果有 refresh_token ,则使用refresh_token获取新的 access_token
      // 成功了 -> 把本次失败的请求重新发出去
      // 失败了 -> 跳转登录页面重新登录获取新的 token
      // 没有，则直接跳转登录页面
      if (!store.state.user) {
        redirectLogin()
        return Promise.reject(error)
      }

      // 刷新 token
      try {
        // 通过 axios.create() 发送请求，不用 request ,避免死循环
        const { data } = await axios.create()({
          method: 'POST',
          url: '/front/user/refresh_token',
          data: qs.stringify({
            refreshtoken: store.state.user.refresh_token
          })
        })
        // 成功了 -> 更新新的token,把本次失败的请求重新发出去
        store.commit('setUser', data.content)
        // error.config -> 失败请求的配置信息
        return request(error.config)
      } catch (err) {
        // 失败了 -> 清除当前用户登录状态，跳转登录页面
        store.commit('setUser', null)
        redirectLogin()
        return Promise.reject(error)
      }
    } if (status === 403) {
      Message.error('没有权限，请联系管理员')
    } if (status === 404) {
      Message.error('请求资源不存在')
    } if (status >= 500) {
      Message.error('服务端错误，请联系管理员')
    }
  } else if (error.request) { // 请求发出去了，但是没有收到响应
    Message.error('请求超时，请刷新重试')
  } else { // 请求的时候设置请求时发生了一些事情，触发了错误
    Message.error(`请求失败：${error.message}`)
  }
  // 请求失败的错误对象抛出，扔给上一个调用者
  return Promise.reject(error)
})
```



##### 多次请求

* 多个请求同时发，会触发多以请求刷新 Token

* refresh 只能请求一次，不然会出错
* 多个请求失败，只刷新一次 token ,重新请求 会把 其他的请求漏掉
* 思路：定义一个数组，定义一个刷新状态，1. 如果是刷新状态，则将请求挂起存放到数组中（返回一个promise，数组中 push 一个回调函数，回调函数里调用 promise的resolve，resolve里真正触发请求发送），2. 如果是非刷新状态，则触发刷新 token请求，请求成功则遍历 requests 数组调用其回调函数，重置 requests 数组，重新发送当前失败请求

```typescript
function refreshToken() {
  return axios.create()({
    method: 'POST',
    url: '/front/user/refresh_token',
    data: qs.stringify({
      refreshtoken: store.state.user.refresh_token
    })
  })
}
// 响应拦截器
let isRefreshing = false // 控制刷新 token 的状态
let requests: any[] = [] // 存储刷新 token 期间过来的请求
request.interceptors.response.use(function (response) {
  // 状态码为 2xx
  // 如果是自定义状态码，错误处理就写到这里

  return response
}, async function (error) {
  // 超出 2xx 状态码
  // 如果是 HTTP 状态码，错误处理就写到这里

  if (error.response) { // 请求收到响应了，但是状态码 超出 2xx
    // const errMsgObj = {
    //   '400': '请求参数错误',
    //   '401': '',
    //   '403': '',
    //   '404': '',
    //   '500': ''
    // }
    // let { status } = error.response
    // status = status + ''
    // Message.error(errMsgObj[status])
    const { status } = error.response
    if (status === 400) {
      Message.error('请求参数错误')
    } else if (status === 401) {
      // token 无效(没有，过期，无效)
      // 如果有 refresh_token ,则使用refresh_token获取新的 access_token
      // 成功了 -> 把本次失败的请求重新发出去
      // 失败了 -> 跳转登录页面重新登录获取新的 token
      // 没有，则直接跳转登录页面
      if (!store.state.user) {
        redirectLogin()
        return Promise.reject(error)
      }

      // 目前没有在刷新 token ,则尝试刷新 token
      if (!isRefreshing) {
        isRefreshing = true // 开启刷新状态
        // 刷新 token
        return refreshToken().then(res => {
          if (!res.data.success) {
            // 失败
            throw new Error('刷新 Token 失败')
          }
          // 成功了 -> 更新新的token,把本次失败的请求重新发出去
          store.commit('setUser', res.data.content)
          // error.config -> 失败请求的配置信息
          // 把 requests 数组队列中的请求重新发出去
          requests.forEach(cb => cb())
          // 重置 requests 数组
          requests = []
          return request(error.config)
        }).catch(error => {
          // 失败了 -> 清除当前用户登录状态，跳转登录页面
          store.commit('setUser', null)
          redirectLogin()
          return Promise.reject(error)
        }).finally(() => {
          isRefreshing = false // 重置刷新状态
        })
      }

      // 刷新状态下，把请求挂起，放到 requests 数组中
      return new Promise(resolve => {
        requests.push(() => {
          resolve(request(error.config))
        })
      })
    } if (status === 403) {
      Message.error('没有权限，请联系管理员')
    } if (status === 404) {
      Message.error('请求资源不存在')
    } if (status >= 500) {
      Message.error('服务端错误，请联系管理员')
    }
  } else if (error.request) { // 请求发出去了，但是没有收到响应
    Message.error('请求超时，请刷新重试')
  } else { // 请求的时候设置请求时发生了一些事情，触发了错误
    Message.error(`请求失败：${error.message}`)
  }
  // 请求失败的错误对象抛出，扔给上一个调用者
  return Promise.reject(error)
})
```



### 配置环境变量

知识点：

* 配置 Vue 项目中的环境变量
* [dotenv]([GitHub - motdotla/dotenv: Loads environment variables from .env for nodejs projects.](https://github.com/motdotla/dotenv))

```.env.development```

```
VUE_APP_API=http://eduboss.lagou.com
```



```.env.production```

```
VUE_APP_API=http://eduboss.lagou.com
```



## 用户权限

* 菜单列表

* 资源列表

* 角色列表

* 用户管理

  

## 课程管理

## 发布部署

## 项目中遇到的问题

### vscode 关闭 单引号转换为 双引号的问题

项目根目录 创建 ```.prettierrc.json```文件

```json
{
  "singleQuote": true,
  "semi": false
}
```

### core-js 报错

```
npm install core-js@3 --save
```

### TS 校验报错

```js
this.$refs.form.validate()
```

Property 'validate' does not exist on type 'Vue | Element | (Vue | Element)[]'. 

Property 'validate' does not exist on type 'Vue'.

解决方法：

1. 强制将 this.$refs.form 转换为 any 类型，不推荐

   ```js
   (this.$refs.form as any).validate()
   ```

2. 标注正确的类型

   ```js
   import { Form } from 'element-ui'
   (this.$refs.form as Form).validate()
   ```

   

### 上传文件 及 上传进度条

```js
export const upload = (data: any, onUploadProgress: (progressEvent: ProgressEvent) => void) => {
  // 该接口要求的请求数据类型是 multipart/form-data
  // 所以需要提交 FormData 数据对象
  return request({
    method: 'POST',
    url: '/boss/course/upload',
    data,
    // HTML5新增的上传响应事件 : progress
    onUploadProgress
  })
}
```



```vue
<!--
1. 组件需要根据绑定的数据进行图片预览
2. 组件需要把上传成功的图片地址同步到绑定的数据中
v-model 的本质还是父子组件通信
 给子组件传递一个 value 的数据
 默认监听 input 事件，修改绑定的数据
-->

<template>
  <div class="course-upload">
    <el-progress v-if="isUploading"
                 type="circle"
                 :percentage="percentage"
                 :width="178"
                 :status="percentage === 100 ? 'success' : undefined">
    </el-progress>
    <el-upload v-else
               class="avatar-uploader"
               action="/boss/course/upload"
               :http-request="handleUpload"
               :show-file-list="false"
               :before-upload="beforeAvatarUpload">
      <img v-if="imageUrl"
           :src="imageUrl"
           class="avatar">
      <i v-else
         class="el-icon-plus avatar-uploader-icon"></i>
    </el-upload>
  </div>
</template>

<script lang="ts">
import Vue from 'vue'
import { upload } from '@/services/course'
export default Vue.extend({
  name: 'CourseUpload',
  model: {
    prop: 'imageUrl',
    event: 'update'
  },
  props: {
    imageUrl: {
      type: String
    },
    limit: {
      type: Number,
      default: 2
    }
  },
  data() {
    return {
      isUploading: false,
      percentage: 0
    }
  },
  methods: {
    beforeAvatarUpload(file: any) {
      const isJPG = file.type === 'image/jpeg'
      const isLt2M = file.size / 1024 / 1024 < this.limit

      if (!isJPG) {
        this.$message.error('上传头像图片只能是 JPG 格式!')
      }
      if (!isLt2M) {
        this.$message.error(`上传头像图片大小不能超过 ${this.limit}MB!`)
      }
      return isJPG && isLt2M
    },
    async handleUpload(options: any) {
      try {
        this.isUploading = true
        const fd = new FormData()
        fd.append('file', options.file)
        const { data } = await upload(fd, e => {
          this.percentage = Math.floor((e.loaded / e.total) * 100)
        })
        if (data.code === '000000') {
          this.$emit('update', data.data.name)
        }
        this.isUploading = false
        this.percentage = 0
      } catch (e) {
        this.isUploading = false
      }
    }
  }
})
</script>

<style lang="scss" scoped>
.course-upload {
  ::v-deep .avatar-uploader .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }
  ::v-deep .avatar-uploader .el-upload:hover {
    border-color: #409eff;
  }
  .avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 178px;
    height: 178px;
    line-height: 178px;
    text-align: center;
  }
  .avatar {
    width: 178px;
    height: 178px;
    display: block;
  }
}
</style>

```



### 富文本编辑器

1. **wangeditor** [https://www.wangeditor.com/](https://www.wangeditor.com/)

   基于javascript和css开发的 Web富文本编辑器， 轻量、简洁、界面美观、易用、开源免费。

2. **TinyMCE** [https://www.tiny.cloud/docs/demo/full-featured/](https://www.tiny.cloud/docs/demo/full-featured/)

   TinyMCE是一个轻量级的基于浏览器的所见即所得编辑器，由JavaScript写成。它对IE6+和Firefox1.5+都有着非常良好的支持。功能齐全，界面美观，就是文档是英文的，对开发人员英文水平有一定要求。

3. **百度ueditor** [https://github.com/fex-team/ueditor](https://github.com/fex-team/ueditor)

   UEditor是由百度web前端研发部开发所见即所得富文本web编辑器，具有轻量，功能齐全，可定制，注重用户体验等特点，开源基于MIT协议，允许自由使用和修改代码，缺点是已经没有更新了

4. **kindeditor** [http://kindeditor.net/demo.php](http://kindeditor.net/demo.php)

   界面经典。

5. **Textbox** [https://www.textbox.io/](https://www.textbox.io/)

   Textbox是一款极简但功能强大的在线文本编辑器，支持桌面设备和移动设备。主要功能包含内置的图像处理和存储、文件拖放、拼写检查和自动更正。此外，该工具还实现了屏幕阅读器等辅助技术，并符合WAI-ARIA可访问性标准。

6. **CKEditor** [https://ckeditor.com/ckeditor-5/demo/](https://ckeditor.com/ckeditor-5/demo/)

   国外的，界面美观。

7. **quill** [https://quilljs.com/](https://quilljs.com/)

   功能强大，还可以编辑公式等

8. **medium-editor** [https://yabwe.github.io/medium-editor/](https://yabwe.github.io/medium-editor/)

   [https://github.com/yabwe/medium-editor](https://github.com/yabwe/medium-editor)

9. **simditor** [https://simditor.tower.im/](https://simditor.tower.im/)

   界面美观，功能较全。（个人博客考虑使用）

10. **summernote **[https://summernote.org/](https://summernote.org/)

    UI好看，精美

11. **jodit** [https://xdsoft.net/jodit/](https://xdsoft.net/jodit/)

    功能齐全

12. **Editor.md** [https://pandao.github.io/editor.md/](https://pandao.github.io/editor.md/)

    功能非常丰富的编辑器，左端编辑，右端预览，非常方便，完全免费

13. **froala Editor**[https://froala.com/wysiwyg-editor/](https://froala.com/wysiwyg-editor/)

    界面非常好看，功能非常强大，非常好用（非免费，可破解）

14. **syncfusion** [https://ej2.syncfusion.com/react/demos/#/material/rich-text-editor/tools](https://ej2.syncfusion.com/react/demos/#/material/rich-text-editor/tools)

15. **dhtmlxEditor** [https://dhtmlx.com/docs/products/dhtmlxRichText/](https://dhtmlx.com/docs/products/dhtmlxRichText/)

15. **bootstrap-wysiwyg** [https://www.zybuluo.com/mdeditor](https://www.zybuluo.com/mdeditor)

    bootstrap-wysiwyg是基于Bootstrap的轻型、免费开源的富文本编辑器，界面简洁大方。使用需要先引入bootstrap。

16. **eWebEditor** [http://www.ewebeditor.net/demo/](http://www.ewebeditor.net/demo/)

    eWebEditor外观和使用风格都和微软 Word很类似，功能很多。工具条可以定制，运行速度很快。导入文件接口很多，支持word、excel、pdf、ppt直接导入，目前版本不支持代码高亮，不适合纯技术平台使用，适合内容编辑人员使用

比较推荐：不在维护的项目不介意使用

* CKEditor
* quill
* medium-editor
* wangeditor
* ueditor
* tinymce

### 拖拽排序

element tree 树形组件



### 图片压缩

compressorjs [https://fengyuanchen.github.io/compressorjs/](https://fengyuanchen.github.io/compressorjs/)



### 阿里云视频点播

[https://help.aliyun.com/](https://help.aliyun.com/)

测试的阿里云账号ID：1618139964448548

### ts 扩展 window 数据类型

```shims-vue.d.ts```

```typescript
declare module '*.vue' {
  import Vue from 'vue'
  export default Vue
}

interface Window {
  AliyunUpload: any
}
```

### 将诸如路径字符串`/user/:name`转换为正则表达式。

```bash
npm install path-to-regexp --save
```

```js
const { pathToRegexp, match, parse, compile } = require("path-to-regexp");
 
// pathToRegexp(path, keys?, options?)
// match(path)
// parse(path)
// compile(path)
```

### 按需加载插件

1. 安装依赖 `npm install babel-plugin-dynamic-import-node -S -D`
2. 在 `babel.config.js` 中添加插件

```json
module.exports = {
  presets: ['@vue/cli-plugin-babel/preset'],
  env: {
    development: {
      plugins: ['dynamic-import-node']
    }
  }
}
```



### vue cli 打包部署本地预览

`dist` 目录需要启动一个 HTTP 服务器来访问

1. 使用一个 Node.js 静态文件服务器，例如 [serve](https://github.com/zeit/serve)

   ```bash
   npm install -g serve
   # -s 参数的意思是将其架设在 Single-Page Application 模式下
   # 这个模式会处理即将提到的路由问题
   serve -s dist
   ```

   问题：接口跨域

2. 自己搭建服务器

   ```js
   const express = require('express')
   const app = express()
   
   const path = require('path')
   
   // 托管 dist 目录，当访问 / 的时候，默认返回托管目录中的 index.html 文件
   app.use(express.static(path.join(__dirname, '../dist')))
   
   app.listen(3000, () => {
     console.log('running~~')
   })
   ```

   问题：接口跨域

   解决：[http-proxy-middleware](https://github.com/chimurai/http-proxy-middleware)中间件代理

   ```bash
   npm install --save-dev http-proxy-middleware
   ```

   ```js
   const express = require('express')
   const app = express()
   const path = require('path')
   const {
     createProxyMiddleware
   } = require('http-proxy-middleware');
   
   // 托管 dist 目录，当访问 / 的时候，默认返回托管目录中的 index.html 文件
   app.use(express.static(path.join(__dirname, '../dist')))
   app.use('/boss', createProxyMiddleware({
     target: 'http://eduboss.lagou.com',
     changeOrigin: true
   }));
   app.use('/front', createProxyMiddleware({
     target: 'http://edufront.lagou.com',
     changeOrigin: true
   }));
   app.listen(3000, () => {
     console.log('running~~')
   })
   ```

   ```json
   // package.json
    "preview": "node test-serve/app.js"
   ```

   



