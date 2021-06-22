---
title: Vue-Router
date: 2021-06-01 20:38:06
tags: [Nodejs]
categories: [Nodejs]
---

## 什么是爬虫

使用程序获取并分析网页中的内容给特定业务应用使用

可以用多中语言进行爬虫

```js
const axios = require('axios')
const cheerio = require('cheerio')
const fs = require('fs')

const request = axios.create({
  baseURL:'https://cnodejs.org/' // 基础路径
})

async function main (page = 1){
  const { data }= await request.get(`/?tab=all&page=${page}`)
  const $ = cheerio.load(data);
  const articles = []
  const topics = $('#topic_list .cell')
  // 抓取当前页面数据
  for(let i = 0; i < topics.length; i++){
    const item = $(topics.get(i))
    const title =  item.find('.topic_title').text()
    const href = item.find('.topic_title').attr('href')
    const articleContent = await getArticleContent(href)
    articles.push({
      title:title,
      content:articleContent
    })
  }
  // fs.writeFileSync('./data.json',JSON.stringify(articles))
  console.log(`${page}页抓取`)
  // 当前页面抓取完毕，查看是否有下一页，递归调用 main
  fs.writeFileSync(`./data-${page}.json`,JSON.stringify(articles))
  if(!$('.pagination li').last().hasClass('disabled')){
    await main(page + 1)
  }
  
}

async function getArticleContent(link){
  const { data }= await request.get(link)
  const $ = cheerio.load(data);
  return $('.markdown-text').html()
}

main(1)
// 从第一页来时抓取数据
```

