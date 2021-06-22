---
layout: vue
title: vue3-小练习-todolist
date: 2021-06-01 09:35:26
tags: [vue]
categories: [vue]
---

## 项目创建

仓库地址：[https://gitee.com/endeavor1/todolist-vue3](https://gitee.com/endeavor1/todolist-vue3)

* 升级 Vue ClI 脚手架 Vue CLI v4.5

  ```bash
  yarn global add @vue/cli
  # OR
  npm install -g @vue/cli
  ```

* 脚手架创建项目

  ```bash
  vue create todolist-app
  ```

* 使用 vite 快速构建 Vue 项目

  ```bash
  $ npm init @vitejs/app <project-name>
  $ cd <project-name>
  $ npm install
  $ npm run dev
  # OR
  $ yarn create @vitejs/app <project-name>
  $ cd <project-name>
  $ yarn
  $ yarn dev
  ```



## 案例 ToDoList 功能列表

* 添加待办事项

* 删除待办事项

* 编辑待办事项

* 切换待办事项

* 存储待办事项

  

1. 添加待办事项
2. 删除待办事项
3. 编辑待办事项
   * 双击待办事项，展示编辑文本框
   * 按回车或者编辑文本框失去焦点，修改数据
     * li 绑定 key 值为 todo.text 会导致 编辑文本框时 key 值发生变化，导致重新渲染，所以 key 值改为绑定 todo
   * 按 esc 取消编辑
   * 把编辑文本框清空按回车，删除这一项
   * 显示编辑文本框的时候获取焦点
     * 自定义指令：获取焦点
4. 切换待办事项
   * 点击 checkbox ，改变所有待办事项状态
     * 具有 get 和 set 的计算属性
   * All / Active / Completed
     * 锚点：监视地址中 hash 的变化，当组件挂载完成，注册 hashchange 事件，当组件销毁时 移除  hashchange 事件，在 hashchange 事件中获取当前 锚点的值，根据 hash 过滤 todos 列表的数据
     * 创建一个对象，封装 过滤 todos 列表的 3 个函数
   * 显示未完成待办项个数
   * 移除所有完成的项目
   * 如果没有待办项，隐藏 main 和 footer
5. 存储待办事项
   * watch 监听 todos，localStorage 存储



### 源码

**App.vue**

```vue
<template>
  <section id="app" class="todoapp">
    <header class="header">
      <h1>todos</h1>
      <input
          class="new-todo"
          placeholder="What needs to be done?"
          autocomplete="off"
          autofocus
          v-model="input"
          @keyup.enter="addTodo"
      >
    </header>
    <section class="main" v-show="count">
      <input id="toggle-all" class="toggle-all" type="checkbox" v-model="allDone">
      <label for="toggle-all">Mark all as complete</label>
      <ul class="todo-list">
        <li
            v-for="todo in filterTodos"
            :key="todo"
            :class="{ editing:todo === editingTodo, completed:todo.completed }"
        >
          <div class="view">
            <input class="toggle" type="checkbox" v-model="todo.completed">
            <label @dblclick="editTodo(todo)">{{ todo.text }}</label>
            <button class="destroy" @click="remove(todo)"></button>
          </div>
          <input
              class="edit"
              type="text"
              v-model="todo.text"
              v-editing-focus="todo === editingTodo"
              @keyup.enter="doneEdit(todo)"
              @blur="doneEdit(todo)"
              @keyup.esc="cancelEdit(todo)"
          >
        </li>
      </ul>
    </section>
    <footer class="footer" v-show="count">
      <span class="todo-count">
        <strong>{{ remainingCount }}</strong> {{ remainingCount > 1 ? 'items' : 'item' }} left
      </span>
      <ul class="filters">
        <li><a href="#/all">All</a></li>
        <li><a href="#/active">Active</a></li>
        <li><a href="#/completed">Completed</a></li>
      </ul>
      <button class="clear-completed" v-show="count > remainingCount" @click="removeCompleted">
        Clear completed
      </button>
    </footer>
  </section>
  <footer class="info">
    <p>Double-click to edit a todo</p>
    <!-- Remove the below line ↓ -->
    <p>Template by <a href="http://sindresorhus.com">Sindre Sorhus</a></p>
    <!-- Change this out with your name and url ↓ -->
    <p>Created by <a href="https://www.lagou.com">教瘦</a></p>
    <p>Part of <a href="http://todomvc.com">TodoMVC</a></p>
  </footer>
</template>

<script>
import './assets/index.css'
import useLocalStorage from './utils/useLocalStorage'
import { ref, computed, onMounted, onUnmounted, watchEffect } from 'vue'

const storage = useLocalStorage()

// 1. 添加待办事项
const useAdd = todos => {
  const input = ref('')
  const addTodo = () => {
    const text = input.value && input.value.trim()
    if(!text.length) return
    todos.value.unshift({
      text,
      completed:false
    })
    input.value = ''
  }
  return {
    input,
    addTodo
  }
}

// 2. 删除待办事项
const useRemove = todos => {
  const remove = todo => {
    const index = todos.value.indexOf(todo)
    todos.value.splice(index,1)
  }
  
  // 删除已完成待办事项
  const removeCompleted = () => {
    todos.value = todos.value.filter(todo => !todo.completed)
  }
  return {
    remove,
    removeCompleted
  }
}

// 3. 编辑待办事项
const useEdit = remove => {
  let beforeEditingText = ''
  const editingTodo = ref(null)
  // 双击待办事项，展示编辑文本框
  const editTodo = todo => {
    beforeEditingText = todo.text
    editingTodo.value = todo
  }
  // 按回车或者编辑文本框失去焦点，修改数据
  const doneEdit = todo => {
    if(!editingTodo.value) return
    todo.text = todo.text.trim()
    // 把编辑文本框清空按回车，删除这一项
    todo.text || remove(todo)
    editingTodo.value = null
  }
  // 按 esc 取消编辑
  const cancelEdit = todo => {
    editingTodo.value = null
    todo.text = beforeEditingText
  }
  return {
    editingTodo,
    editTodo,
    doneEdit,
    cancelEdit
  }
}

// 4. 切换待办项完成状态
const useFilter = todos => {
  // 具有 get 和 set 的计算属性
  const allDone = computed({
    get(){
      return !todos.value.filter(todo => !todo.completed).length
    },
    set(val){
      todos.value.forEach(todo => todo.completed = val)
    }
  })
  
  // 监视 hashchange 事件，监听 锚点 变化，过滤 todos
  const filter = {
    all:list => list,
    active:list => list.filter(todo => !todo.completed),
    completed:list => list.filter(todo => todo.completed)
  }
  const type = ref('all')
  const filterTodos = computed(() => filter[type.value](todos.value))
  // 未完成待办事项
  const remainingCount = computed(() => filter.active(todos.value).length)
  // 待办项总个数
  const count = computed(() => todos.value.length)
  const onHashChange = () => {
    const hash = window.location.hash.replace('#/','')
    if(filter[hash]){
      type.value = hash // 响应式数据，数据变化时要重新渲染模板
    }else{
      type.value = 'all'
      window.location.hash = ''
    }
  }
  onMounted(() => {
    window.addEventListener('hashchange',onHashChange)
  })
  onUnmounted(() => {
    window.removeEventListener('hashchange',onHashChange)
  })
  
  
  
  return {
    allDone,
    filterTodos,
    remainingCount,
    count
  }
}

// 5. 存储待办事项
const useStorage = () => {
  const KEY = 'TODOKEYS'
  const todos = ref(storage.getItem(KEY) || [])
  // watch(todos.value,(newVal,oldVal) => {
  //   console.log(newVal,oldVal)
  //   storage.setItem(KEY, newVal.value)
  // })
  watchEffect((val,old) => {
    console.log(123,val,old)
    let value = todos.value
    storage.setItem(KEY, value)
  })
  return todos
}

export default {
  name: 'App',
  setup(){
    const todos = useStorage()
    const { remove, removeCompleted } = useRemove(todos)
    return {
      todos,
      remove,
      removeCompleted,
      ...useAdd(todos),
      ...useEdit(remove),
      ...useFilter(todos)
    }
  },
  // 自定义指令，获取焦点
  directives:{
    editingFocus:(el,bingding) => {
      bingding.value && el.focus()
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
```



**useLocalStorage.js**

```js
function parse(str) {
  let value
  try{
    value = JSON.parse(str)
  }catch (e) {
    value = null
  }
  return value
}

function stringfy(obj) {
  let value
  try{
    value = JSON.stringify(obj)
  }catch (e) {
    value = null
  }
  return value
}

export default function useLocalStorage() {
  function setItem(key, value) {
    value = stringfy(value)
    window.localStorage.setItem(key, value)
  }
  function getItem(key) {
    let value = window.localStorage.getItem(key)
    if(value){
      value = parse(value)
    }
    return value
  }
  return {
    setItem,
    getItem
  }
}
```

