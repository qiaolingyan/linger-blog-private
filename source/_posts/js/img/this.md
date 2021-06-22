1.
```javascript
function foo() {
  console.log(this)
}

foo()  // window
window.foo()  // window
foo.call(1) // Number
```
2.
```javascript
const obj1 = {
    foo:function(){
        console.log(this)
    }
}
obj1.foo()  // obj1
const fn = obj1.foo
fn() // window
```
3.
```javascript
const obj2 = {
    foo:function(){
        function bar(){
            console.log(this)
        }
        bar()
    }
}
obj2.foo() // window
```
4.
```javascript
let length = 10
function fn1(){
    console.log(this.length)
}
const obj3 = {
    length:5,
    method(fn1){
        fn1()
        arguments[0]()
    }
}
obj3.method(fn,1,2)  // 10  3
```
### 总结
1.沿着作用域向上找最近的一个function（不是箭头函数），看这个function最终是怎样执行的
2.this的指向取决于所属function的调用方式，而不是定义
3.function调用一般分为以下几种情况：
  1.作为函数调用，即：foo()
    * 指向全局对象（globalThis),注意严格模式下是undefined
  2.作为方法调用，即：foo.bar() / foo.bar.baz() / foo['bar]() / foo[0]()
    * 指向最终调用这个方法的对象
  3.作为构造函数调用，即：new Foo()
    * 指向一个新的对象 Foo{}
  4.特殊调用，即：foo.call() / foo.apply() / foo.bind()
    * 参数指定成员
4.找不到所属的function，就是全局对象
5.箭头函数中的this指向
  * 继承自离他最近的作用域的this



### call、apply、bind
联系：都能改变this指向
* 区别：
    * call / apply 会立即调用当前函数，并修改函数的this指向，
而 bind 不会调用函数，也没有改变原函数的this指向，但是返回一个新函数，新函数的this指向改变了。
    * call / bind 方法传参一样，第一个是要改变函数this指向的对象，第二个以后的参数，将作为函数的实参传入。
而 apply，第一个是要改变函数this指向的对象，第二个是一个数组或者类数组，数组里面的每一个值，将作为函数的实参传入
```javascript
// call
Function.prototype.myCall = function (ctx,...args) {
  ctx = ctx || window
  ctx.fn = this
  const result = ctx.fn(...args)
  delete ctx.fn
  return result
}

// apply
Function.prototype.myApply = function (ctx,args=[]) {
  ctx = ctx || window
  ctx.fn = this
  const result = ctx.fn(...args)
  delete ctx.fn
  return result
}

// bind
Function.prototype.myBind = function (ctx,...args) {
  ctx = ctx || window
  return function(...args2){
    ctx.fn = this
    const result = ctx.fn(...args)
    delete ctx.fn
    return result
  }
}
```

