
### 手写promise
1. Promise就是一个类，在执行这个类的时候，需要传递一个执行器进去，执行器会立即执行
2. Promise 中有三种状态，分别为 fulfilled 成功、rejected 失败、pending 等待
  *  pending -> fulfilled
  * pending -> rejected
  * 一旦状态确定就不可更改
3. resolve和reject函数是用来更改状态的
  * resolve: fulfilled
  * reject: rejected
4. then 方法内部做的事情就判断状态，如果是成功，调用成功的回调函数，如果状态是失败，调用失败的回调函数，then方法是被定义在原型对象中的
5. then成功回调有一个参数，表示成功之后的值，then失败回调有一个参数表示失败后的原因
6. 同一个promise对象下面的then方法是可以被调用多次的
7. then方法是可以被链式调用的，后面then方法的回调函数拿到值的是上一个then方法的回调函数的返回值

```javascript
/*
  1. Promise 就是一个类 在执行这个类的时候 需要传递一个执行器进去 执行器会立即执行
  2. Promise 中有三种状态 分别为 成功 fulfilled 失败 rejected 等待 pending
    pending -> fulfilled
    pending -> rejected
    一旦状态确定就不可更改
  3. resolve和reject函数是用来更改状态的
    resolve: fulfilled
    reject: rejected
  4. then方法内部做的事情就判断状态 如果状态是成功 调用成功的回调函数 如果状态是失败 调用失败回调函数 then方法是被定义在原型对象中的
  5. then成功回调有一个参数 表示成功之后的值 then失败回调有一个参数 表示失败后的原因
  6. 同一个promise对象下面的then方法是可以被调用多次的
  7. then方法是可以被链式调用的, 后面then方法的回调函数拿到值的是上一个then方法的回调函数的返回值
*/

// 定义状态常量，使编辑器可以自动提醒
const PENDING  = 'pending'
const FULFILLED = 'fulfilled'
const REJECTED = 'rejected'

class MyPromise {
  PromiseState = PENDING  // 对象状态
  PromiseResult = null  // 对象结果
  
  tmpFulfilled = [] // 暂存成功回调
  tmpRejected = [] // 暂存失败回调
  
  // 构造方法，new 的时候自动执行
  constructor(executor){
    try{
      executor(this.resolve,this.reject)
    }catch(e){
      this.reject(e) // 异常时设为失败状态
    }
  }
  
  // 设置成功状态，结果，因为是直接调用，所以箭头函数使其this指向MyPromise实例
  resolve = value => {
    // 如果状态不为待定，则函数直接返回，确保状态不可逆
    if(this.PromiseState !== PENDING) return
    this.PromiseState = FULFILLED
    this.PromiseResult = value
    // 设置成功状态时，循环执行暂存的成功回调函数（promise可以多次then调用）
    while (this.tmpFulfilled.length) this.tmpFulfilled.shift()()
  }
  // 设置失败状态，结果
  resolve = reason => {
    if(this.PromiseState !== PENDING) return
    this.PromiseState = REJECTED
    this.PromiseResult = reason
    while (this.tmpRejected.length) this.tmpRejected.shift()()
  }
  
  then(onFulfilled,onRejected){
    // 参数不是函数时，给默认值
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value
    onRejected = typeof onRejected === 'function' ? onRejected : reason => {throw reason}
    const thenPromise = new MyPromise((resolve,reject) => {
      let resolvePromise = cb => {
        // 创建一个微任务
        queueMicrotask(() => {
          try{
            let x = cb(this.PromiseResult)
            if(x === thenPromise){
              return reject(new TypeError('Chaining cycle detected for promise #<Promise>'))
            }
            if(x instanceof MyPromise){
              x.then(val => {
                resolvePromise(() => val)
              },reason => {
                resolvePromise(() => reason)
              })
            }else{
              resolve(x)
            }
          }catch(e){
            reject(e)
          }
        })
      }
      
      if(this.PromiseState === FULFILLED){
        resolvePromise(onFulfilled)
      }else if(this.PromiseState === REJECTED){
        resolvePromise(onRejected)
      }else{
        this.tmpFulfilled.push(resolvePromise.bind(this,onFulfilled))
        this.tmpRejected.push(resolvePromise.bind(this,onRejected))
      }
    })
    return thenPromise
  }
  
  static all(arr){
    const result = []
    let count = 0
    return new MyPromise((resolve, reject) => {
      let addData = (i,val) => {
        result[i] = val
        count++
        if(count === arr.length) resolve(result)
      }
      arr.forEach((item,index) => {
        if(item instanceof MyPromise){
          item.then(val => addData(index,val),reject)
        }else{
          addData(index,item)
        }
      })
    })
  }
  
  static race(arr){
    return new MyPromise((resolve,reject) => {
      arr.forEach(item => {
        if(item instanceof MyPromise){
          item.then(resolve,reject)
        }else{
          queueMicrotask(() => {
            resolve(item)
          })
        }
      })
    })
  }
  
  static resolve(param){
    if(param instanceof MyPromise) return param
    return new MyPromise(resolve => resolve(param))
  }
  
  static reject(param){
    return new MyPromise((resolve,reject) => reject(param))
  }
  
  finally(callback){
    /*return this.then(v => {
      return MyPromise
      .resolve(callback())
      .then (() => v)
    }, r => {
      return MyPromise
      .resolve(callback())
      .then (() => { throw r })
    })*/
    
    let x = typeof callback === 'function' ? callback() : callback
    return MyPromise.resolve(x).then(() => this,reason => {throw reason})
  }
  
  catch(onRejected){
    return this.then(undefined,onRejected)
  }
}

MyPromise.deferred = function () {
  var result = {}
  result.promise = new MyPromise(function (resolve,reject) {
    result.resolve = resolve
    result.reject = reject
  })
  return result
}

module.exports = MyPromise

```
