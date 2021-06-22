---
title: Vuex
date: 2021-05-31 10:38:06
tags: [vue]
categories: [vue]
---

### 状态管理

* state，驱动应用的数据源； 

* view，以声明方式将 state 映射到视图； 

* actions，响应在 view 上的用户输入导致的状态变化。



### 什么是 vuex

* vuex 是专门为 vue.js 设计的状态管理库
* vuex 采用集中式的方式存储需要共享的状态
* vuex 的作用是进行状态管理，解决复杂组件通信，数据共享
* vuex 集成到了 devtools 中，提供了 time-travel 时光旅行历史回滚功能

![img](C:\Users\qiaolingyan\AppData\Roaming\Typora\typora-user-images\image-20210430155510267.png)

1. store

   仓库，vuex的核心，唯一的，容器，包含应用的大部分状态

2. state

   Vuex 使用单一状态树，用一个对象就包含了全部的应用层级状态。

3. getter

   store 的计算属性，就像计算属性一样，getter 的返回值会根据它的依赖被缓存起来，且只有当它的依赖值发生了改变才会被重新计算。

4. mutation

   更改 Vuex 的 store 中的状态的唯一方法是提交 mutation。Vuex 中的 mutation 非常类似于事件：每个 mutation 都有一个字符串的 **事件类型 (type)** 和 一个 **回调函数 (handler)**。这个回调函数就是我们实际进行状态更改的地方

5. action

   Action 类似于 mutation，不同在于：

   - Action 提交的是 mutation，而不是直接变更状态。
   - Action 可以包含任意异步操作。

6. module

   由于使用单一状态树，应用的所有状态会集中到一个比较大的对象。当应用变得非常复杂时，store 对象就有可能变得相当臃肿。为了解决以上问题，Vuex 允许我们将 store 分割成**模块（module）**。每个模块拥有自己的 state、mutation、action、getter、甚至是嵌套子模块——从上至下进行同样方式的分割：

**使用**

mapState, mapGetters, mapMutations, mapActions，可传对象或数组

传对象可重命名，还可以通过state计算得出其他值

getters定义时可返回一个函数，页面调用 countAdd(5)

actions 异步，this.$store.dispatch("decreateAsync", num).then()

modules 命名空间设置 namespaced: true,，页面使用 ...mapState("product", ["products"])，可直接使用products

```
<script>
import { mapState, mapGetters, mapMutations, mapActions } from "vuex";
export default {
  name: "App",
  data() {
    return {
      count: 2,
      ass: 5
    };
  },
  computed: {
    ...mapState(["msg"]), // count: state => state.count
    ...mapState({ num: "count", message: "msg" }), // 给state重命名
    ...mapState({
      addNum(state) {
        return state.count + this.ass; // 通过计算得出新的值
      }
    }),
    ...mapGetters(["reverseMsg"]),
    ...mapGetters({ resMsg: "reverseMsg" }),
    ...mapGetters(["countAdd"]), // getter 在通过方法访问时，每次都会去进行调用，而不会缓存结果
    ...mapState("product", ["products"]) // 命名空间
  },
  methods: {
    ...mapMutations(["increate"]),
    ...mapMutations({
      add: "increate"
    }),
    ...mapActions(["increateAsync"]),
    ...mapActions({
      addAsync: "increateAsync"
    }),
    decreate(num) {
      this.$store.dispatch("decreateAsync", num).then(res => {
        console.log(res, this.count++);
      });
    },
    ...mapMutations(["setProducts"]),
    ...mapMutations("product", ["setProducts"]) // 命名空间
  }
};
</script>
```

#### vuex 插件

1. vuex 的插件就是一个函数

2. 这个函数接收一个 store 的参数

   ```
   const myPlugin = store => {
     // 当 store 初始化后调用
     store.subscribe((mutation, state) => {
       // 每次 mutation 之后调用
       // mutation 的格式为 { type, payload }
       // 如果是 cart 模块，则每次提交之后都更改 localStorage
       if (mutation.type.startsWith('cart/')) {
         window.localStorage.setItem('cart-products', JSON.stringify(state.cart.cartProducts))
     })
   }
   
   // 用
   const store = new Vuex.Store({
     // ...
     plugins: [myPlugin]
   })
   ```


### 手写 vuex

```
let _Vue = null
class Store {
  constructor(options) {
    const {
      state = {},
        getters = {},
        mutations = {},
        actions = {}
    } = options

    this.state = _Vue.observable(state)
    this.getters = Object.create(null)
    Object.keys(getters).forEach(key => {
      Object.defineProperty(this.getters, key, {
        get: () => getters[key](state)
      })
    })

    this._mutations = mutations
    this._actions = actions
  }

  commit(type, payload) {
    this._mutations[type](this.state, payload)
  }

  dispatch(type, payload) {
    this._actions[type](this, payload)
  }

}

function install(Vue) {
  _Vue = Vue
  _Vue.mixin({
    beforeCreate() {
      if (this.$options.store) {
        _Vue.prototype.$store = this.$options.store
      }
    }
  })
}

export default {
  Store,
  install
}
```

