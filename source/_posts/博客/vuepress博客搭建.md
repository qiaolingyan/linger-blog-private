---
title: vuepress博客搭建
date: 2021-06-22 10:38:06
tags: [博客]
categories: [博客]
---

### 前言

采用 VuePress 搭建博客

VuePress 由两部分组成：第一部分是一个[极简静态网站生成器 (opens new window)](https://github.com/vuejs/vuepress/tree/master/packages/%40vuepress/core)，它包含由 Vue 驱动的[主题系统](https://vuepress.vuejs.org/zh/theme/)和[插件 API](https://vuepress.vuejs.org/zh/plugin/)，另一个部分是为书写技术文档而优化的[默认主题](https://vuepress.vuejs.org/zh/theme/default-theme-config.html)，它的诞生初衷是为了支持 Vue 及其子项目的文档需求。

官网：[https://vuepress.vuejs.org/zh/guide/](https://vuepress.vuejs.org/zh/guide/)

github地址：[https://github.com/vuejs/vuepresshttps://github.com/BlogGuide/vuepress.blog.github.io)

### 准备工作

前提条件：VuePress 需要 [Node.js (opens new window)](https://nodejs.org/en/)>= 8.6

1. 创建并进入一个新目录

   ```bash
   mkdir vuepress-starter && cd vuepress-starter
   ```

2. 使用你喜欢的包管理器进行初始化

   ```bash
   yarn init # npm init
   ```

3. 将 VuePress 安装为本地依赖

   我们已经不再推荐全局安装 VuePress

   ```bash
   yarn add -D vuepress # npm install -D vuepress
   ```

   注意

   如果你的现有项目依赖了 webpack 3.x，我们推荐使用 [Yarn (opens new window)](https://classic.yarnpkg.com/zh-Hans/)而不是 npm 来安装 VuePress。因为在这种情形下，npm 会生成错误的依赖树。

4. 创建你的第一篇文档

   ```bash
   mkdir docs && echo '# Hello VuePress' > docs/README.md
   ```

5. 在 `package.json` 中添加一些 [scripts(opens new window)](https://classic.yarnpkg.com/zh-Hans/docs/package-json#toc-scripts)

   这一步骤是可选的，但我们推荐你完成它。在下文中，我们会默认这些 scripts 已经被添加。

   ```json
   {
     "scripts": {
       "docs:dev": "vuepress dev docs",
       "docs:build": "vuepress build docs"
     }
   }
   ```

6. 在本地启动服务器

   ```bash
   yarn docs:dev # npm run docs:dev
   ```

   VuePress 会在 [http://localhost:8080 (opens new window)](http://localhost:8080/)启动一个热重载的开发服务器。

