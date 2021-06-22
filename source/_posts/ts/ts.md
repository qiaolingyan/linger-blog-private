# 反馈问题

```js
// 有一个 promise 中的请求失败，取消其他请求
const controller = new AbortController();
let signal = controller.signal;
// axios
const requests = [ 
    fetch('index.html', { signal }),
    fetch('index.html', { signal }),
    fetch('ftp://example.com', { signal })
]

Promise
    .all(requests)
    .then(arr => {
        console.log(arr)
    })
    .catch(err => {
        console.log(err)
        controller.abort()
    })
```

react中map循环生成的DOM，绑定事件时，是推荐使用事件委托的方式，还是直接在map循环出来的每一个DOM上直接写onClick

## 性能优化

1. 对性能优化还是一知半解，不知在工作中如何去运用
2. 标记清除会造成空间碎片化，那么引用计数会吗？
3. v8引擎中的分代回收希望能给在细讲一下
   - https://juejin.cn/post/6927100870765051917

## 变量和函数

1. let到底存不存在变量提升，我看有的地方明确说了let也是会存在变量提升的，只是因为存在暂时性死区(从当前块的顶部到let声明的地方被称为暂时性死区)的原因，导致不可在这个区域内去使用该变量
2. 函数定义时有多个参数  但是调用的时候直传了部分参数 是不是只要按顺序传递有效参数，部分不传也是可以的呢 例如 function add(m,n){console.log(m+1)｝  add(2);   或者 funnction  add(m,n){console.log(n+1)｝  add(,2);
3. 老师您好，我想问下  lodash库中的debounce函数和memoize函数似乎都能实现缓存的效果，它们两者有什么区别呢？谢谢。
   - https://www.lodashjs.com/docs/lodash.debounce

```js
// 记忆函数
const _ = require('lodash')

function getArea (r) {
  console.log(r)
  return Math.PI * r * r
}

// let getAreaWithMemory = _.memoize(getArea)
// console.log(getAreaWithMemory(4))
// console.log(getAreaWithMemory(4))
// console.log(getAreaWithMemory(4))

// 模拟 memoize 方法的实现
function memoize (f) {
  let cache = {}
  return function () {
    let key = JSON.stringify(arguments)
    cache[key] = cache[key] || f.apply(f, arguments)
    return cache[key]
  }
}

```



## TS

1. 讲一下 Reflect.apply()
   
   1. fn.apply()
   
   - https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect/apply
   
2. TypeScript相关： 配置文件target设置了  es5，没有设置lib，symbol和Promise都不报错，把lib设为["ES2015"],console.log也没报错是怎么回事     

3. 和 Symbol.toStringTag 类似的属性还有什么？  对象的每一个方法都可以 Symbol.xxxTag 这样用吗？还是只有固定的几个

   - https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Symbol

## 其他

1. 事件委托的讲解以及事件委托的实际应用场景介绍

```html
<ul id="mv">
    <li class="one">1</li>
    <li class="two">2</li>
    <li>3</li>
</ul>

li.onclick = function () {}
```

```js
document.querySelector('#mv').addEventListener('click', function (e) {
  // 判断是否匹配目标元素
  if (e.target.nodeName.toLocaleLowerCase() === 'li') {
    console.log('the content is: ', target.innerHTML)
  }
})
```

2. 我想了解JavaScript到底有没有编译环节？part1-模块2-TypeScript语言-JavaScript类型系统特征中，老师说JavaScript没有编译环节。但是在《你不知道的JavaScript（上卷）》1.1中说“尽管通常将JavaScript归类为“动态”或“解释执行”语言，但事实上它是一门编译语言。”JavaScript引擎进行编译的步骤和传统的编译语言非常相似，在某些环节可能比预想的要复杂。”
   - 变量提升只是解释器扫描代码，以及开辟内存空间的一个过程，没有对代码编译
   - V8中的 JIT 会对执行一次以上的代码进行编译和优化处理

3. 在循环中将异步变成同步，除了原始for循环结合async await的方式实现，还有更好的实现方式么？例如：一个数组中全是Promise 但是数组的长度不定，数组中的每一个Promise的执行都要等前一个Promise的执行结果出来之后才能执行  

- 异步迭代器：https://es6.ruanyifeng.com/#docs/async-iterator

```js
const arr = [ fetch('./index.html'), fetch('./app.js') ]
async function fn () {
    for await (const res of arr) {
        console.log(res)
        // console.log(await res.text())
    }
}
fn()
```

4. JS编程的设计模式讲解（例如：工厂模式、订阅模式等等）   
5. https://gist.github.com/pissang/4d1cced7b7d32de41f9a815c27e4490e  以上链接中的“实现字符串 Query 的类型推导”部分代码看不大懂  希望能在课上帮助解释一下...

# 相关内容

## 关于 this 的回顾

```javascript
function foo () {
  console.log(this)
}
foo() //
window.foo() //
foo.call(1) //
```

```javascript
const obj1 = {
  foo: function () {
    console.log(this)
  }
}

obj1.foo() //
const fn = obj1.foo
fn() //
```


```javascript
const obj2 = {
  foo: function () {
    function bar () {
      console.log(this)
    }
    bar()
  }
}

obj2.foo()
```

关于 this 的总结：

1. 沿着作用域向上找最近的一个 function（不是箭头函数），看这个 function 最终是怎样执行的；
2. **this 的指向取决于所属 function 的调用方式，而不是定义；**
3. function 调用一般分为以下几种情况：
   1. 作为函数调用，即：`foo()`
      1. 指向全局对象（globalThis），注意严格模式问题，严格模式下是 undefined
   2. 作为方法调用，即：`foo.bar()` / `foo.bar.baz()` / `foo['bar']()` / `foo[0]()`
      1. 指向最终调用这个方法的对象
   3. 作为构造函数调用，即：`new Foo()`
      1. 指向一个新对象 `Foo {}`
   4. 特殊调用，即：`foo.call()` / `foo.apply()` / `foo.bind()`
      1. 参数指定成员
4. 找不到所属的 function，就是全局对象
5. 箭头函数中的 this 指向

```js
function fn () {
    let arrFn = () => {
        console.log(this)
    }
    arrFn()
}

const obj = {
    name: 'zs',
    fn: fn
}

obj.fn()   
fn()	  
```

then:

```javascript
var length = 10
function fn () {
  console.log(this.length)
}

const obj = {
  length: 5,
  method (fn) {
    fn()   //
    arguments[0]()  // 相当于 arguments.fn()
  }
}

obj.method(fn, 1, 2)
```

严格模式下原本应该指向全局的 `this` 都会指向 `undefined`

## ES 2020/2021 新特性

https://github.com/tc39/proposals/blob/master/finished-proposals.md

```js
// 空值合并运算符
function foo (option) {
  // 只有 size = null 或者 undefined
  option.size = option.size ?? 100
  
  const mode = option.mode || 'hash' 
  console.log(option)
}

foo({ size: 0 })

// 可选链运算符
const list = [
  {
    title: 'foo',
    author: {
      name: 'zs',
      email: 'zs@qq.com'
    }
  },
  {
    title: 'bar'
  }
]
list.forEach(item => {
  console.log(item.author?.name)
})
  
```

## 使用 TypeScript 的 Vue.js 项目差异

1）基本操作

1. 安装 @vue/cli 最新版本

2. 使用 @vue/cli 创建一个项目（不选 TypeScript)

3. 使用 @vue/cli 安装 TypeScript 插件

   ```bash
   vue add typescript
   ```

2）通过 Git Diff 对比介绍使用 TypeScript 的 Vue.js 项目差异

1. 安装了 @vue/cli-plugin-typescript 等插件
2. shims-tsx.d.ts 文件的作用
   1. 允许你以 .tsx 结尾的文件，在Vue项目中编写jsx代码
3. shims-vue.d.ts 文件的作用
   1. 用于 TypeScript 识别 .vue 文件
   2. TS 默认不支持 .vue 文件，这里 TS 导入.vue 文件都按 VueConstructor 处理
4. d.ts 的问题
   1. 该文件中定义的类型需要全局可用
   2. 这个文件中不能在最外层书写 import 或者 export，如果书写这个文件会有自己的作用域
   3. 如果这个文件中不写 import 或者 export，那么这个文件中定义的类型全局可用

## JavaScript 项目中的类型增强

JavaScript 项目中如何有更好的类型提示：JSDoc + [import-types](https://www.typescriptlang.org/docs/handbook/jsdoc-supported-types.html#import-types)

https://www.typescriptlang.org/docs/handbook/type-checking-javascript-files.html

https://www.typescriptlang.org/play/index.html?useJavaScript=truee=4#example/jsdoc-support

### 类型检查

`@ts-check`

### 类型注解

- `@type`
- `@typedef`

```js
// 检查类型的错误
// @ts-check

/**
 * @summary 操作 DOM
 * @author zs
 * @param { HTMLDivElement } a1 要操作的div
 * @param { string } a2 字符串
 * @returns { HTMLDivElement } 这是一个dom对象
 */
function fn (a1, a2) {
  return a1
}
fn()

/** @type { string } */
let name
name = 18

/** @typedef { 'open' | 'closed' } Status  */
/** @type { Status } */
const s = 'open'
```

1. 配置文件类型
   - vue.config.js
   - webpack.config.js

```js
/** @type {import('@vue/cli-service').ProjectOptions} */
module.exports = {
}

/** @type {import('webpack').Configuration} */
module.exports = {
}
```

1. router 类型

```js
// Vue.js 3.0 中
/** @type { import('vue-router').RouteRecordRaw[] } */
const routes = []

// Vue.js 2.x 中
/** @type { import('vue-router').RouteConfig[] } */
const routes = []
```



1. store 类型

```js
// Vue.js 3.0 和 Vue.js 2.x 一样
/** @type { import('vuex').MutationTree<typeof import('./state').default> } */
const mutations = {
  add (state) {
  }
}

export default mutations
```

### 类型补充声明

- types.d.ts

插件的类型扩展，使用类型补充声明

```js
import { AxiosInstance } from 'axios'

declare module 'vue/types/vue' {
  interface Vue {
    readonly axios: AxiosInstance
  }
}
```

```js
import axios from 'axios'

export default {
  install (Vue) {
    const instance = axios.create({
      baseURL: 'http://127.0.0.1/api/v1',
      timeout: 10000
    })
    Vue.prototype.axios = instance
  }
}


vm.axios

```



## 定义组件的几种不同方式

### 写法 1：使用 Options APIs

- 组件仍然可以使用以前的方式定义（导出组件选项对象，或者使用 Vue.extend()）
- 但是当我们导出的是一个普通的对象，此时 TypeScript 无法推断出对应的类型，
- 至于 VSCode 可以推断出类型成员的原因是因为我们使用了 Vue 插件，
- 这个插件明确知道我们这里导出的是一个 Vue 对象。
- 所以我们必须使用 `Vue.extend()` 方法确保 TypeScript 能够有正常的类型推断

```typescript
import Vue from 'vue'

export default Vue.extend({
  name: 'Button',
  data () {
    return {
      count: 1
    }
  },
  methods: {
    increment () {
      this.count++
    }
  }
})
```

### 写法 2：使用 Class APIs

在 TypeScript 下，Vue 的组件可以使用一个继承自 Vue 类型的子类表示，这种类型需要使用 Component 装饰器去修饰

装饰器函数接收的参数就是以前的组件选项对象（data、props、methods 之类）

```typescript
import Vue from 'vue'
import Component from 'vue-class-component' // 官方库

@Component({
  props: {
    size: String
  }
})
export default class Button extends Vue {
  private count: number = 1
  private text: string = 'Click me'

  get content () {
    return `${this.text} ${this.count}`
  }

  increment () { // 事件处理函数
    this.count++
  }

  mounted () { // 生命周期函数
    console.log('button is mounted')
  }
}
```

- Data: 使用类的实例属性声明
- Method: 使用类的实例方法声明
- Computed: 使用 Getter 属性声明
- 生命周期: 使用类的实例方法声明

**其它特性：例如 components, props, filters, directives 之类的，则需要使用修饰器参数传入！！！！！**

使用这种 class 风格的组件声明方式并没有什么特别的好处，只是为了提供给开发者多种编码风格的选择性

#### 装饰器

> [装饰器](https://github.com/tc39/proposal-decorators)是 ES 草案中的一个新特性，提供一种更好的面向切面编程（AOP）的体验，不过这个草案最近有可能发生重大调整，所以并不推荐。

TypeScript 中对装饰器的实现：https://www.staging-typescript.org/docs/handbook/decorators.html

- 类装饰器

  ```typescript
  function classDecorator (constructor: Function) {
    console.log('源类型：', constructor)
  }
  ```

- 方法装饰器

  ```typescript
  function methodDecorator (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log('目标对象：', target)
    console.log('属性名称：', propertyKey)
    console.log('属性描述符：', descriptor)
    // descriptor 指的就是 Object.definedProperty 传入的第三个参数
  }
  ```

- 访问器（getter/setter）装饰器

  ```typescript
  function accessorDecorator (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log('目标对象：', target)
    console.log('属性名称：', propertyKey)
    console.log('属性描述符：', descriptor)
  }
  ```

- 属性装饰器

- 参数装饰器


### 写法 3：使用 Class APIs + [vue-property-decorator](https://github.com/kaorun343/vue-property-decorator)

```typescript
import { Vue, Component, Prop } from 'vue-property-decorator'

@Component
export default class Button extends Vue {
  private count: number = 1
  private text: string = 'Click me'
  @Prop() readonly size?: string

  get content () {
    return `${this.text} ${this.count}`
  }

  increment () {
    this.count++
  }

  mounted () {
    console.log('button is mounted')
  }
}
```

这种方式继续放大了 class 这种组件定义方法。

### 推荐

No Class APIs，只用 Options APIs。

使用 Options APIs 最好是使用 `export default Vue.extend({ ... })` 而不是 `export default { ... }`。

其实 Vue.js 3.0 早期是想要放弃 Class APIs 的，不过无奈想要兼容，所以才保留下来了。



