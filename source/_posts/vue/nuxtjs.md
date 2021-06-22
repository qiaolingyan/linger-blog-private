---
title: Nuxt.js
date: 2021-05-31 10:38:06
tags: [vue, 服务端渲染]
categories: [vue]
---

### 1. 使用方式

1. 初始项目

   * ```bash
     npm init nuxt-app <project-name>
     ```

   * ```bash
     mkdir <project-name>
     cd <project-name>
     touch package.json
     
     // package.json
     {
       "name": "my-app",
       "scripts": {
         "dev": "nuxt",
         "build": "nuxt build",
         "generate": "nuxt generate",
         "start": "nuxt start"
       }
     }
     
     yarn add nuxt
     mkdir pages
     touch pages/index.vue
     yarn dev
     ```

2. 已有的 Node.js 服务端项目

   * 直接把 Nuxt 当做一个中间件集成到 Node Web Server 中

3. 现有的 Vue.js 项目

   * 非常熟悉 Nuxt.js 

   * 至少哦10% 的代码改动

### 2. 异步数据 asyncData

1. asyncData 只有页面组件中才调用，子组件不会触发调用，，要使用数据的话，需要页面组件传给子组件

2. asyncData 中没有 this

3. 当你想要动态页面内容有利于 SEO 或者是提升首屏渲染速度的时候，就在 asyncData 中发请求拿数据

4. 上下文对象 

   * 获取路由参数 context.route.params 或者 context.params

   ```js
   async asyncData(context){
         const {data} = await axios({
           method:'GET',
           url:'http://localhost:3000/data.json'
         })
         const id = JSON.parse(context.params.id)
         return {
           name:data.posts.find(item => item.id === id).name
         }
       }
   ```

   

### 3. 生命周期

1. 服务端生效的钩子函数
   * beforeCreate
   * created
   * errorCaptured

### 4. nuxt 渲染流程

![img](C:\Users\qiaolingyan\AppData\Roaming\Typora\typora-user-images\image-20210508153033790.png)

### 5. nuxt 命令

| 命令          | 描述                                                         |
| ------------- | ------------------------------------------------------------ |
| nuxt          | 启动一个热加载的 web 服务器（开发模式）localhost:3000        |
| nuxt build    | 利用 webpack 编译应用，压缩 JS 和 CSS 资源（发布用）         |
| nuxt start    | 以生产模式启动一个 Web 服务器（需要先执行 nuxt build）       |
| nuxt generate | 编译应用，并依据路由配置生成对应的 HTML 文件（用于静态站点的部署） |

### 6. 最简单的部署方式

1. 配置 Host + Port

   ```js
   // nuxt.config.js
   // 添加
    server: {
       host: '0.0.0.0', // 
       port: 3000
     }
   ```

2. 压缩发布包，压缩以下包上传

   * .nuxt  打包生成的文件

   * static 静态资源

   * nuxt.config.js  配置
   * package.json
   * package-lock.json

3. 把发布包传到服务端

   1. 连接到服务端，命令  ```ssh root@39.105.28.5```
   2. 查看 ``` ls```
   3. ``` mkdir realworld-nuxtjs```
   4. ```cd realworld-nuxtjs```
   5. 打印路径 ```pwd```，复制路径
   6. 退出 exit 服务端或者重新打开一个命令行窗口
   7. 把本地的压缩包传到远程服务器，路径不能有中文名 ```scp ./realworld-nuxtjs.zip root@39.105.28.5:/root/realworld-nuxtjs```
   8. 连接服务器，查看ls
   9. 解压 ```unzip realworld-nuxtjs.zip```
   10. ls 查看,-a查看隐藏目录 ```ls -a```
   11. 安装依赖 ```npm i```
   12. 启动服务 ```npm run start```
   13. 访问 公网IP + 端口号

4. 解压

5. 安装依赖

6. 启动服务

连接服务器
  ssh -o StrictHostKeyChecking=no root@106.75.182.115
  ls
创建目录
  mkdir realword-nuxtjs
进入目录
  cd realword-nuxtjs
查看路径
  pwd
退出
  exit



### 7. 使用 PM2 启动 Node 服务，在后台运行应用

1. [https://github.com/Unitech/pm](https://github.com/Unitech/pm)
2. [官方文档 https://pm2.io](https://pm2.io)
3. 服务端 安装 ```npm install --global pm2```
4. 启动 pm2 start 脚本路径 ```pm2 start npm -- start```
5. 关闭 ```pm2 stop 6```，6 是 id

#### 7.1 PM2 常用命令

| 命令        | 说明                             |
| ----------- | -------------------------------- |
| pm2 list    | 查看应用列表                     |
| pm2 start   | 启动应用                         |
| pm2 stop    | 停止应用                         |
| pm2 reload  | 重载应用（原有实例慢慢消灭）     |
| pm2 restart | 重启应用（原有程序杀死，再开启） |
| pm2 delete  | 删除应用                         |

### 8. 自动化部署

![img](./img/部署.jpg)

#### 8.1 CI/CD 服务

* Jenkins
* Gitlab Ci
* GitHub Actions 本案例采用
* Travis CI
* Circle CI

#### 8.2 环境准备

* Linux 服务器
* 把代码提交到 GitHub 远程仓库

#### 8.3 配置GitHub Access Token

github 的 token

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

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

```
gridsome-blog token : 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
```

* 生成： [https://github.com/settings/tokens](https://github.com/settings/tokens)
* 配置到项目的 Secrets 中：[https://github.com/lipengzhou/realworld-nuxtjs/settings/secrets](https://github.com/lipengzhou/realworld-nuxtjs/settings/secrets)

#### 8.4 配置 GitHub Actions 执行脚本

* 在项目根目录创建 .github/workflows 目录

* 下载 main.yml 到 workflows 目录中

  [https://gist.github.com/lipengzhou/b92f80142afa37aea397da47366bd872](https://gist.github.com/lipengzhou/b92f80142afa37aea397da47366bd872)

* 修改配置

* 配置 PM2 配置文件

  ```json
  // pm2.config.json
  {
  	"apps":[
  		{
  			"name":"RealWorld",
  			"script":"npm",
  			"args":"start"
  		}
  	]
  }
  ```

  

* 提交更新

* 查看自动部署状态

* 访问网站

* 提交更新

  * git add .
  * git commit -m '部署更新测试'
  * git push

  

  * git add .

  * git tag v0.1.0  // 创建标签

  * git tag // 查看标签

  * git push origin v0.1.0 // 推送
  
    

