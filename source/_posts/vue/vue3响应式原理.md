---
title: vue3响应式原理
date: 2021-06-01 11:36:02
tags: [vue]
categories: [vue]
---

项目地址：[https://gitee.com/endeavor1/vue3-reactivity](https://gitee.com/endeavor1/vue3-reactivity)

## vue3 响应式系统

1. Proxy 对象实现属性监听
2. 多层属性嵌套，在访问属性过程中处理下一级属性
3. 默认监听动态添加的属性
4. 默认监听属性的删除操作
5. 默认监听数组索引和 length 属性
6. 可以作为单独的模块使用

## 核心方法

1. reactive / ref / toRefs / computed
2. effect (watch 函数内部)
3. track
4. trigger

## 基础-Proxy 对象

1. Proxy

   ```js
   const target = {
       foo: 'xxx',
       bar: 'yyy'
   }
   const proxy = new Proxy(target, {
       get (target, key, receiver) {
           return Reflect.get(target, key, receiver)
       },
       // set 和 deleteProperty 中需要返回布尔类型的值
       // 在严格模式下，如果返回 false 的话会出现 Type Error 的异常
       set (target, key, value, receiver) {
           // 不加 return,严格模式下会报错
           return Reflect.set(target, key, value, receiver)
       },
       deleteProperty (target, key) {
           return Reflect.deleteProperty(target, key)
       }
   })
   ```

2. Proxy 和 Reflect 中使用的 receiver

   * Proxy 中 receiver：Proxy 或者继承 Proxy 的对象
   * Reflect 中 receiver：如果 target 对象中设置了 getter，getter 中的 this 指向 receiver

   ```js
   const obj = {
       get foo() {
           console.log(this)
           return this.bar
       }
   }
   
   const proxy = new Proxy(obj, {
       get (target, key, receiver) {
           if (key === 'bar') {
               return 'value - bar'
           }
           // 不加 receiver，上面 get 里this 是 obj 对象
           // return Reflect.get(target, key) 
           // 加 receiver，上面 get 里this 是 Proxy 对象
           return Reflect.get(target, key, receiver)
       }
   })
   console.log(proxy.foo)
   ```




## 响应式-工具函数

```js
// 判断是否是对象
const isObject = val => val !== null && typeof val === 'object'

// 递归处理，如果是对象，继续调用reactive将其处理为响应式
const convert = target => isObject(target) ? reactive(target) : target

// 判断某个对象本身是否具有某个属性
const hasOwnProperty = Object.prototype.hasOwnProperty
const hasOwn = (target,key) => hasOwnProperty.call(target,key)
```



## 响应式-reactive

1. 接收一个参数，判断这参数是否是对象
2. 创建拦截器对象 handler，设置 get / set / deleteProperty
3. 返回 Proxy 对象

```js
export function reactive(target) {
  // 判断这参数是否是对象
  if(!isObject(target)) return target
  
  // 创建拦截器对象 handler，设置 get / set / deleteProperty
  const handler = {
    get(target,key,receiver){
      // 收集依赖
      track(target,key)
      const result =  Reflect.get(target,key,receiver)
      // 判断得到的值是否是对象，如果是对象，会递归收集下一级的依赖
      return convert(result)
    },
    set(target,key,value,receiver){
      const oldValue = Reflect.get(target,key,receiver)
      let result = true
      // 判断新值是否与之前的值相同，如果相同就直接返回
      if(oldValue !== value){
        result = Reflect.set(target,key,value,receiver)
        // 触发更新
        trigger(target, key)
      }
      return result
    },
    deleteProperty(target,key){
      // 判断 target 中是否有 key 属性
      const hadKey = hasOwn(target,key)
      const result = Reflect.deleteProperty(target,key)
      if(hadKey && result){
        // 触发更新
        trigger(target, key)
      }
      return result
    }
  }
  // 返回 Proxy 对象
  return new Proxy(target,handler)
}
```



## 响应式-effect

调用一次 effect 回调函数

收集依赖-track ----- reactive 中的 get 方法里

触发更新-trigger ----- reactive 中的 set 和 deleteProperty 方法里

1. targetMap  : 记录目标对象和一个字典(depsMap)
   * new WeakMap() 类型：key 就是目标对象，value 是 depsMap
   * 弱引用 Map，目标对象失去引用后可以销毁
2. depsMap ：
   * new Map() 类型：key 就是目标对象中的属性名称，value 是 dep 
3. dep ：
   * new Set() 类型：存储的 是 effect 函数，一个属性可能对应多个函数

![img](C:\Users\qiaolingyan\AppData\Roaming\Typora\typora-user-images\image-20210601142920655.png)

```js
let activeEffect = null
export function effect(callbak) {
  activeEffect = callbak
  // 调用 callback 时会访问响应式对象属性，去收集依赖，收集依赖过程中需要把 callback 存储起来
  callbak()
  
  // 依赖收集完之后，把 activeEffect 置为 null，方便下次依赖收集
  activeEffect = null
}

// 收集依赖
let targetMap = new WeakMap()
export function track(target, key) {
  if(!activeEffect) return
  let depsMap = targetMap.get(target)
  if(!depsMap){
    // 创建一个 new Map() 存储到 depsMap 中，并且 把这个值存储到 targetMap 中
    targetMap.set(target,(depsMap = new Map()))
  }
  let dep = depsMap.get(key)
  if(!dep){
    depsMap.set(key,(dep = new Set()))
  }
  dep.add(activeEffect)
}

// 触发更新
export function trigger(target, key) {
  const depsMap = targetMap.get(target)
  if(!depsMap) return
  const dep = depsMap.get(key)
  if(!dep) return
  dep.forEach(effect => effect())
}
```



## 响应式-ref

```js
export function ref(raw) {
  // 判断 raw  是否是 ref 创建的对象，如果是的话直接返回
  if(isObject(raw) && raw.__v_isRef) return
  
  // 判断 raw 是否是对象，如果是对象的话调用 reactive 创建响应式对象，否则的话返回 raw
  let value = convert(raw)
  
  // 创建 ref 对象 ,并且返回
  const r = {
    __v_isRef:true, 
    get value(){
      track(r,'value')
      return value
    },
    set value(newValue){
      if(newValue !== value){
        raw = newValue
        value = convert(raw)
        trigger(r,'value')
      }
    }
  }
  return r
}
```



## ref 与 reactive 的区别

1. ref 可以把基本数据类型数据，转成响应式对象
2. ref 返回的对象，重新赋值成对象也是响应式的（给value 重新赋值）
3. reactive 返回的对象，重新赋值丢失响应式
4. reactive 返回的对象不可以解构



## 响应式-toRefs

1. 接收一个 reactive 返回的对象
2. 将reactive 返回的对象 内部的每个属性都转换为响应式
3. 把转换后的属性挂载到一个新的对象上返回

```js
export function toRefs(proxy) {
  // 判断函数的参数是否是 reactive 创建的对象
  
  // 遍历属性转换为 响应式，挂载到新的对象上返回
  const ret = proxy instanceof Array ? new Array(proxy.length) : {}
  for(const key in proxy){
    ret[key] = toProxyRef(proxy, key)
  }
  return ret
}

function toProxyRef(proxy, key) {
  const r = {
    __v_isRef:true,
    get value(){
      return proxy[key]
    },
    set value(newValue){
      proxy[key] = newValue
    }
  }
  return r
}
```



## 响应式-computed

1. 接收一个参数 getter，获取结果的函数
2. 返回一个 ref 创建的 具有 value 属性的对象
3. 调用 effect，在 effect 里调用 getter，把 getter的结果存到 ref 的 value 中

```js
export function computed(getter) {
  const result = ref()
  effect(() => (result.value = getter()))
  return result
}
```

