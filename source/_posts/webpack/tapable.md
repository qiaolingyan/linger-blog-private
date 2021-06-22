### tapable

本身是一个独立的库



#### 工作流程

1. 实例化 hook 注册事件监听
2. 通过 hook 触发事件监听
3. 执行懒编译生成的可执行代码

#### hook

本质是 tapable 实例对象

hook 执行机制可分为 同步 和 异步，异步 存在 并行 和 串行 两种模式

#### hook 执行特点

1. Hook：普通钩子，监听器之间互相独立不干扰
2. BailHook：熔断钩子，监听某个钩子返回非 undefined 时后续不执行
3. WaterfallHook：瀑布钩子，上一个监听的返回值可传递至下一个
4. LoopHook：循环钩子，如果当前未返回 false或者undefined 则一直执行

#### tapable 同步钩子

1. SyncHook
2. SyncBailHook
3. SyncWaterfallHook
4. SyncLoopHook

#### tapable 异步串行钩子

1. AsyncSeriesHook
2. AsyncSeriesBailHook
3. AsyncSeriesWaterfallHook

#### tapable 异步并行钩子

1. AsyncParalleHook
2. AsyncParalleBailHook



## 同步钩子使用

1. SyncHook

   ```
   // 导入钩子类
   const { SyncHook } = require('tapable')
   // 实例化类实例
   let hook = new SyncHook(['name', 'age'])
   // 添加事件监听
   hook.tap('fn1', function (name, age) {
     console.log('fn1--->', name, age)
   })
   hook.tap('fn2', function (name, age) {
     console.log('fn2--->', name, age)
   })
   hook.tap('fn3', function (name, age) {
     console.log('fn3--->', name, age)
   })
   // 触发事件监听
   hook.call('bom', 18)
   ```

2. SyncBailHook

   监听某个钩子返回非 undefined 时后续不执行

   ```
   // 导入钩子类
   const { SyncBailHook } = require('tapable')
   // 实例化类实例
   let hook = new SyncBailHook(['name','age'])
   // 添加事件监听
   hook.tap('fn1',function (name, age) {
     console.log('fn1--->',name,age)
   })
   hook.tap('fn2',function (name, age) {
     console.log('fn2--->',name,age)
     return undefined
   })
   hook.tap('fn3',function (name, age) {
     console.log('fn3--->',name,age)
   })
   // 触发事件监听
   hook.call('lg',18)
   ```

3. SyncWaterfallHook

   上一个监听的返回值可传递至下一个

   ```
   // 实例化类实例
   let hook = new SyncWaterfallHook(['name','age'])
   // 添加事件监听
   hook.tap('fn1',function (name, age) {
     console.log('fn1--->',name,age)
     return 'ret1'
   })
   hook.tap('fn2',function (name, age) {
     console.log('fn2--->',name,age)
     return 'ret2'
   })
   hook.tap('fn3',function (name, age) {
     console.log('fn3--->',name,age)
     return 'ret3'
   })
   // 触发事件监听
   hook.call('lg',33)
   ```

4. SyncLoopHook

   如果当前未返回 false或者undefined 则一直执行

   ```
   // 导入钩子类
   const { SyncLoopHook } = require('tapable')
   // 实例化类实例
   let hook = new SyncLoopHook(['name','age'])
   
   let count1 = 0
   let count2 = 0
   let count3 = 0
   
   // 添加事件监听
   hook.tap('fn1',function (name, age) {
     console.log('fn1--->',name,age)
     if(++count1 === 1){
       count1 = 0
       return undefined
     }
     return true
   })
   hook.tap('fn2',function (name, age) {
     console.log('fn2--->',name,age)
     if(++count2 === 2){
       count2 = 0
       return undefined
     }
     return true
   })
   hook.tap('fn3',function (name, age) {
     console.log('fn3--->',name,age)
   })
   // 触发事件监听
   hook.call('lg',12)
   ```

   

## 异步钩子使用

对于异步钩子的使用，在添加事件监听时会存在三种方式：tap tapAsync tapPromise

### 异步并行

1. AsyncParalleHook

   ```
   const { AsyncParallelHook } = require('tapable')
   
   let hook = new AsyncParallelHook(['name'])
   
   // 对于异步钩子的使用，在添加事件监听时会存在三种方式：tap tapAsync tapPromise
   ```

   ```
   // 01 tap
   console.time('time')
   hook.tap('fn1',function (name) {
     console.log('fn1--->',name)
   })
   hook.tap('fn2',function (name) {
     console.log('fn2--->',name)
   })
   hook.callAsync('lg',() => {
     console.log('lg-last')
     console.timeEnd('time')
   })
   ```

   ```
   // 02 tapAsync
   console.time('time')
   hook.tapAsync('fn1',function (name,callback) {
     setTimeout(() => {
       console.log('fn1--->',name)
       callback()
     },1000)
   })
   hook.tapAsync('fn2',function (name,callback) {
     setTimeout(() => {
       console.log('fn2--->',name)
       callback()
     },2000)
   })
   
   hook.callAsync('lg',() => {
     console.log('lg-last')
     console.timeEnd('time')
   })
   ```

   ```
   // 03 tapPromise
   console.time('time')
   hook.tapPromise('fn1',function (name) {
     return new Promise((resolve,reject) => {
       setTimeout(() => {
         console.log('fn1--->',name)
         resolve()
       },1000)
     })
   })
   hook.tapPromise('fn2',function (name) {
     return new Promise((resolve,reject) => {
       setTimeout(() => {
         console.log('fn2--->',name)
         resolve()
       },2000)
     })
   })
   
   hook.promise('foo').then(() => {
     console.log('end')
     console.timeEnd('time')
   })
   ```

   

2. AsyncParalleBailHook

   ```
   const { AsyncParallelBailHook } = require('tapable')
   
   let hook = new AsyncParallelBailHook(['name'])
   
   // 对于异步钩子的使用，在添加事件监听时会存在三种方式： tap tapAsync tapPromise
   
   // tapAsync
   console.time('time')
   hook.tapAsync('fn1',function (name,callback) {
     setTimeout(() => {
       console.log('fn1--->',name)
       callback()
     },1000)
   })
   hook.tapAsync('fn2',function (name,callback) {
     setTimeout(() => {
       console.log('fn2--->',name)
       callback('err')
     },2000)
   })
   hook.tapAsync('fn3',function (name,callback) {
     setTimeout(() => {
       console.log('fn3--->',name)
       callback()
     },3000)
   })
   hook.callAsync('zce',() => {
     console.log('lg-last')
     console.timeEnd('time')
   })
   ```

   

### 异步串行

1. AsyncSeriesHook

   ```
   const { AsyncSeriesHook } = require('tapable')
   
   let hook = new AsyncSeriesHook(['name'])
   
   // 对于异步钩子的使用，在添加事件监听时会存在三种方式： tap tapAsync tapPromise
   
   // tapPromise
   console.time('time')
   hook.tapPromise('fn1',function (name) {
     return new Promise((resolve,reject) => {
       setTimeout(() => {
         console.log('fn1--->',name)
         resolve()
       },1000)
     })
   })
   hook.tapPromise('fn2',function (name) {
     return new Promise((resolve,reject) => {
       setTimeout(() => {
         console.log('fn2--->',name)
         resolve()
       },2000)
     })
   })
   
   hook.promise('aqs').then(() => {
     console.log('aqs-end')
     console.timeEnd('time')
   })
   ```

   

## 手写 SyncHook

```
// Hook.js

class Hook {
  constructor(args = []){
    this.args = args
    this.taps = [] // 将来用于存放组装好的 {}
    this._x = undefined // 将来在代码工厂函数中会给 _x = [f1,f2,f3,...]
  }
  
  tap(options,fn){
    if(typeof options === 'string'){
      options = {
        name:options
      }
    }
    options = Object.assign({fn},options) // { fn: ... name:fn1 }
    
    // 调用以下方法将组装好的 options  添加至 []
    this._insert(options)
  }
  
  tapAsync(options,fn){
    if(typeof options === 'string'){
      options = {
        name:options
      }
    }
    options = Object.assign({fn},options) // { fn: ... name:fn1 }
  
    // 调用以下方法将组装好的 options  添加至 []
    this._insert(options)
  }
  
  _insert(options){
    this.taps[this.taps.length] = options
  }
  
  call(...args){
    // 01 创建将来要具体执行的函数代码结构
    let callFn = this._createCall()
    // 02 调用上述的函数（传参）args传入进去
    return callFn.apply(this,args)
  }
  
  callAsync(...args){
    let callFn = this._createCall()
    return callFn.apply(this,args)
  }
  
  _createCall(){
    return this.compile({
      taps:this.taps,
      args:this.args
    })
  }
}

module.exports = Hook
```

```
// SyncHook.js

let Hook = require('./Hook')

class HookCodeFactory {
  args(){
    return this.options.args.join(',')  // ["name","age"]  ==》name,age
  }
  head(){
    return `var _x = this._x;`
  }
  content(){
    let code = ``
    for(var i = 0; i < this.options.taps.length; i++){
      // code += `var _fn0 = _x[0];_fn[0](name,age)`
      code += `var _fn${i} = _x[${i}];_fn${i}(${this.args()});`
    }
    return code
  }
  setup(instance,options){
    this.options = options // 这里的操作在源码中是通过 init 方法实现的
    instance._x = options.taps.map( o => o.fn)
  }
  create(options){ // 创建一段可执行的代码体然后返回
    let fn
    // fn = new Function('name,age','var _x = this._x;var_fn0 = _x[0]; _fn0(name,age);')
    fn = new Function(
      this.args(),
      this.head() + this.content()
    )
    return fn
  }
}

let factory = new HookCodeFactory()

class SyncHook extends Hook{
  constructor(args){
    super(args)
  }
  
  compile(options){ // {taps:[{},{}], args:[name,age]}
    factory.setup(this,options)
    return factory.create(options)
  }
}

module.exports = SyncHook
```





## 手写 AsyncParalleHook

```
// .AsyncParalleHook
let Hook = require('./Hook')

class HookCodeFactory {
  args({after,before} = {}){
    let allArgs = this.options.args
    if(before)allArgs = [before].concat(allArgs)
    if(after) allArgs = allArgs.concat(after)
    return allArgs.join(',')  // ["name","age"]  ==》name,age
  }
  head(){
    return `"use strict";var _context;var _x = this._x;`
  }
  content(){
    let code = `var _counter = ${this.options.taps.length}; var _done = (function () {
      _callback();
    });`
    for(var i = 0; i < this.options.taps.length; i++){
      code += `var _fn${i} = _x[${i}];_fn${i}(name, (function () {
        if (--_counter === 0) _done();
      }));`
    }
    return code
  }
  setup(instance,options){
    this.options = options // 这里的操作在源码中是通过 init 方法实现的
    instance._x = options.taps.map( o => o.fn)
  }
  create(options){ // 创建一段可执行的代码体然后返回
    let fn
    // fn = new Function('name,age','var _x = this._x;var_fn0 = _x[0]; _fn0(name,age);')
    fn = new Function(
      this.args({ after:'_callback' }),
      this.head() + this.content()
    )
    return fn
  }
}

let factory = new HookCodeFactory()

class AsyncParalleHook extends Hook{
  constructor(args){
    super(args)
  }
  
  compile(options){ // {taps:[{},{}], args:[name,age]}
    factory.setup(this,options)
    return factory.create(options)
  }
}

module.exports = AsyncParalleHook
```

