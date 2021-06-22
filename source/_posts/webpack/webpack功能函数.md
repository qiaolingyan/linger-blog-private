### 简单打包分析

* 01 打包后的文件就是一个函数自调用，当前函数调用时传入一个对象

* 02 这个对象我们为了方便将之称为是模块定义，他就是一个键值对

* 03 这个键名就是当前被加载模块的文件名与某个目录的拼接

* 04 这个键值就是一个函数，和 node.js 里的模块加载有一些类似，会将被加载模块中的内容包裹于一个函数中

* 05 这个函数在将来某个时间点上会被调用，同时会接收到一定的参数，利用这些参数就可以实现模块的加载操作

* 06 针对于上述代码就相当于是将 {}（模块定义）传递给了 modules 

### 功能函数分析

当我们使用 webpack 进行打包的时候，最终都会产生一个或多个 js 文件，在这些文件中最终都会生成一个 自调用函数， 它接收一个对象作为参数（模块定义），它的键作为要查询的模块id，它的值作为要执行的函数，在执行函数过程中完成了当前模块id对应的模块内容加载。

针对于不同的模块类型，webpack 会使用不同的方法

1. **``` __webpack_require__.t```**

   *  01 接收二个参数，一个是 value 一般用于表示被加载的模块 id， 第二个值 mode 是一个二进制的数值

   * 02 t 方法内部做的第一件事就是调用自定义的 ```__webpack_require__``` 方法加载 value 对应的模块导出，重新赋值给 value 

   * 03 当获取到 value 值之后，余下的 8 4 ns 2 都是对当前的内容加工处理，然后返回使用

   * 04 当 mode & 8 成立时直接将 value 返回 ()
     * 4-1 当 mode & 1  mode & 8 同时成立，相当于加载的是一个 commonjs 规范，可以直接使用的导出内容，直接将value 返回

   * 05 当 mode & 4 成立时直接将 value 返回 ()
     * 5-2 当 mode & 1  mode & 4 同时成立，相当于加载的是一个 es module 规范，直接将value 返回

   * 06 当 mode & 8  mode & 4 都不成立，还要继续处理 value，定义一个 ns{}

     * 6-1 如果拿到的 value 是一个可以直接使用的内容，例如是一个字符串，将它挂载到 ns 的 default 属性上

     * 6-2 如果是一个对象，就会调用  ```__webpack_require__.d``` 给 ns 上添加每个属性，并给每个属性添加一个 getter

### commonjs模块打包分析

* require 加载 commonjs 规范，默认支持
* 只会调用```__webpack_require__```方法来获得值

### es module 模块打包分析

 * es module文件的导入
   *  调用```__webpack_require__.r```标记
   * 调用```__webpack_require__```方法来获得值
   *  将值传给```__webpack_require__.n```，里面调用 ```__webpack_require__.d```方法给exports身上添加一个 a 属性，值就是 exports

* commonjs 引入 es module 导出的文件
  * es module文件的导出 会调用 ```__webpack_require__.r```方法，给其添加一个标记，标记为 es module
  * es module文件的导出 会调用 ```__webpack_require__.d```方法，给 exports 身上添加属性，给这个属性添加一个 getter

### 懒加载实现

* 01 import（）可以实现指定模块的懒加载操作
* 02 当前懒加载的核心原理就是 jsonp
* 03 调用```__webpack_require__.e``` 生成一个 promise，.then 里调用```__webpack_require__.t```方法，第二个 .then 就是懒加载时我们写的 .then
* 03  ```__webpack_require__.t``` 方法可以针对于内容进行不同的处理（处理方式取决于传入的数值 8  6  7  3  2  1  ）

### 手写功能函数

```
(function (modules) {

  // 14 定义 webpackJsonpCallback 实现：合并模块定义，改变 promise 状态执行后续行为
  function webpackJsonpCallback(data) {
    // 01 获取需要被动态加载的模块 id
    let chunkIds = data[0]
    // 02 获取需要被动态加载的模块依赖关系对象
    let moreMoudles = data[1]
    // 03 循环判断 chunkIds 里对应的模块内容是否已经完成了加载
    let chunkId, resolves = []
    for (let i = 0; i < chunkIds.length; i++) {
      chunkId = chunkIds[i]
      if (Object.prototype.hasOwnProperty.call(installedChunks, chunkId) && installedChunks[chunkId]) {
        resolves.push(installedChunks[chunkId][0])
      }
      // 更新当前的 chunk 状态
      installedChunks[chunkId] = 0
    }

    for (moduleId in moreMoudles) {
      if (Object.prototype.hasOwnProperty.call(moreMoudles, moduleId)) {
        modules[moduleId] = moreMoudles[moduleId]
      }
    }

    while (resolves.length) {
      resolves.shift()()
    }
  }

  // 01 定义对象用于将来缓存被加载过的模块
  let installedModules = {}

  // 15 定义 installedChunks 用于标识某个 chunkId 对应的 chunk 是否完成了加载
  let installedChunks = {
    main: 0
  }

  // 02 定义一个 __webpack_require__ 方法来提换 import require 加载操作
  function __webpack_require__(moduleId) {
    // 2-1 判断当前缓存中是否存在要被加载的模块内容，如果存在则直接返回
    if (installedModules[moduleId]) return installedModules[moduleId].exports

    // 2-2 如果当前缓存中不存在，则需要我们自己定义 {}，执行被导入的模块内容加载
    let module = installedModules[moduleId] = {
      i: moduleId,
      l: false,
      exports: {}
    }

    // 2-3 调用当前 moduelId 对应的函数完成对应内容的加载
    modules[moduleId].call(module.exports, module, module.exports, __webpack_require__)

    // 2-4 当上述的方法调用完成之后，我们就可以修改 l 的值用于标识当前模块已经加载完成了
    module.l = true

    // 2-5 加载工作完成之后，要将拿回来的内容返回至调用的位置
    return module.exports
  }

  // 03 定义 m 属性用于保存 modules
  __webpack_require__.m = modules

  // 04 定义 c 属性用于保存 cache
  __webpack_require__.c = installedModules

  // 05 定义 o 方法用于判断对象的身上是否存在指定的属性
  __webpack_require__.o = function (object, property) {
    return Object.prototype.hasOwnProperty(object, property)
  }

  // 06 定义 d 方法用于在对象的身上添加指定的属性，同时给该属性提供一个 getter
  __webpack_require__.d = function (exports, name, getter) {
    if (!__webpack_require__.o(exports, name)) {
      Object.defineProperty(exports, name, {
        enumerable: true,
        get: getter
      })
    }
  }

  // 07 定义 r 方法用于标识当前模块是  es6
  __webpack_require__.r = function (exports) {
    if (typeof Symbol !== "undefined" && Symbol.toStringTag) {
      Object.defineProperty(exports, Symbol.toStringTag, {
        value: 'Module'
      })
    }
    Object.defineProperty(exports, '__esModule', {
      value: true
    })
  }

  // 08 定义 n 方法用于设置具体的 getter
  __webpack_require__.n = function (module) {
    let getter = module && module.__esModule ?
      function getDefault() {
        return module['default']
      } :
      function getModuleExports() {
        return module
      }
    __webpack_require__.d(getter, 'a', getter)
  }

  // 17 定义 jsonpScriptSrc 实现 src 的处理
  function jsonpScriptSrc(chunkId) {
    return __webpack_require__.p + "" + chunkId + '.built.js'
  }

  // 16 定义 e 方法用于实现： 实现json来加载内容，利用 promise 来实现异步加载操作
  __webpack_require__.e = function (chunkId) {
    // 01 定义一个数组用于存放 promise
    let promises = []

    // 02 获取 chunkId 对应的 chunk 是否已经完成了加载
    let installedChunkData = installedChunks[chunkId]

    // 03 依据当前是否已完成加载的状态来执行后续的逻辑
    if (installedChunkData !== 0) {
      if (installedChunkData) {
        promises.push()
      } else {
        let promise = new Promise((resolve, reject) => {
          installedChunkData = installedChunks[chunkId] = [resolve, reject]
        })
        promises.push(installedChunkData[2] = promise)

        // 创建标签
        let script = document.createElement('script')
        // 设置 src
        script.src = jsonpScriptSrc(chunkId)
        // 写入 script 标签
        document.head.appendChild(script)
      }
    }

    // 执行 promise
    return Promise.all(promises)
  }

  // 11 定义 t 方法，用于加载指定 value 的模块内容，之后对内容进行处理再返回
  __webpack_require__.t = function (value, mode) {
    // 01 加载 value 对应的模块内容（value 一般就是模块id）
    // 加载之后的内容又重新赋值给 value
    if (mode & 1) {
      value = __webpack_require__(value)
    }

    // 如果成立，加载了可以直接返回使用的内容(commonjs)
    if (mode & 8) {
      return value
    }

    // 如果成立，加载了可以直接返回使用的内容(es module)
    if ((mode & 4) && typeof value === 'object' && value && value.__esModule) {
      return value
    }

    // 如果 8  和  4  都没有成立，则需要自定义 ns 来通过 default 属性返回内容
    let ns = Object.create(null)

    __webpack_require__.r(ns)

    Object.defineProperty(ns, 'default', {
      enumerable: true,
      value: value
    })

    if (mode & 2 && typeof value !== 'string') {
      for (let key in value) {
        __webpack_require__.d(ns, key, function (key) {
          return value[key]
        }.bind(null, key))
      }
    }

    return ns
  }

  // 09 定义 p 属性用于保存资源访问路径
  __webpack_require__.p = ''

  // 11 定义变量存放数组
  let jsonpArray = window['webpackJsonp'] = window['webpackJsonp'] || []

  // 12 保存原生的 push 方法
  let oldJsonpFunction = jsonpArray.push.bind(jsonpArray)

  // 13 重写原生的 push 方法
  jsonpArray.push = webpackJsonpCallback

  // 10 调用 __webpack_require__ 方法，执行模块导入与加载操作
  return __webpack_require__(__webpack_require__.s = './src/index.js')
})
({

  "./src/index.js": (function (module, exports, __webpack_require__) {
    let oBtn = document.getElementById('btn')
    oBtn.addEventListener('click', function () {
      __webpack_require__.e( /*! import() | login */ "login").then(__webpack_require__.t.bind(null, /*! ./login.js */ "./src/login.js", 7)).then((login) => {
        console.log(login)
      })
    })
    console.log('index.js')
  })

});
```

