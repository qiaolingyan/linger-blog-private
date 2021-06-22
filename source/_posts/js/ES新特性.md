### let 和 const
* var 
  * 声明提升，使用undefined初始化
  * 全局作用域
  * 可以仅声明不初始化
  * 可以重复定义
  * 可以多次赋值
  * 可以声明前访问
* let
  * 不存在变量提升，会存入暂时性死区
  * 块级作用域
  * 可以仅声明不初始化
  * 不可以重复定义
  * 可以多次赋值
  * 不可以声明前访问（Cannot access 'tmp' before initialization）
* const
  * 不存在变量提升，会存入暂时性死区
  * 块级作用域
  * 必须在声明时初始化
  * 不可以重复定义
  * 基本数据类型不可以，引用数据类型仅可以改变值
  * 不可以声明前访问（Cannot access 'tmp' before initialization）

### Arrow functions
1、箭头函数特点
  * 箭头函数没有自己的this，箭头函数的this不是调用的时候决定的，而是在定义的时候处在的对象就是它的this
  * 箭头函数没有自己的arguments，指向离它最近包裹它的函数的arguments
  * 箭头函数没有显示原型属性
  * 箭头函数不能new调用

### Classes（类）
1. 实例方法
2. 静态方法
3. 类的继承

```javascript
class Person1 {
  // 当前类型的构造函数,this为当前实例对象
  constructor(name){
    this.name = name
  }
  // 实例方法 调用：this.say()
  say(){
    console.log(`hi,my name is ${this.name}`)
  }
  // 静态方法 调用：Person1.create('tom')
  // 静态方法里的this不会指向实例对象，而是指向Person
  static create(name){
    return new Person1(name)
  }
}
class Students extends Person1{
  constructor(name,number){
    super(name)  // 继承父类的属性
    this.number = number
  }
  hello(){
    super.say() // 调用父类的方法
    console.log(`my school number is ${this.number}`)
  }
}

const s = new Students('jack','100')
s.hello()


```
### Promises
### Generators
### Modules
### Template literals（模板字⾯量）
1. 普通模板字符串
```javascript
const str = `hello es2015,this is a \`string\`` //支持转义
const str1 = `hello es2015,
  this is a \`string\``    // 可直接换行
const name1 = 'tom'
const msg = `hey,${name1} -- ${1+2} -- ${Math.random()}`  // 支持变量及数学运算
```

2. 带标签的模板字符串
```javascript
const name2 = 'tom'
const gender = true
function myTagFunc(strings,name,gender) {
  // 对模板字符串进行分割,根据${}
  console.log(strings,name,gender)  // strings:[ 'hey,', ' is a ', '' ] ---  name:'tom' ---gender:true
  const sex = gender ? 'boy' : 'girl'
  return strings[0] + name + strings[1] + sex + strings[2]
}
const result = myTagFunc`hey,${name2} is a ${gender}`
console.log(result)  // hey,tom is a boy
```

### Default parameters（默认参数）

### Enhanced object literals（对象字⾯量增强）
### Destructuring assignments（解构分配）
### Spread operator（展开操作符）
### Symbol
* 用途
  * 作为对象的属性，可以作为对象的私有属性
  * 阻止对象上属性名的冲突，每个Symbol都独一无二的，可以保证不与其他属性名产生冲突
  * 使用Symbol来代替常量
### for ... of 循环
### Map 和 Set
### Proxy
### 浅拷贝与深拷贝
* 浅拷贝：针对Object,Array这种复杂数据类型，浅拷贝复制一层对象的属性，属性中的值是基本数据类型直接复制值，如果是引用数据类型复制内存地址的指针，所以在修改复制后的变量里引用类型的里面的值时，会导致原始数据也被修改
  * Object.assign({},target)
* 深拷贝：针对Object,Array这种复杂数据类型，深拷贝递归复制了所有的层级，新数据和原始数据不存在联系，因此在修改复制后的变量里引用类型的里面的值时，不会导致原始数据也被修改
  * JSON.parse(JSON.stringfy())
