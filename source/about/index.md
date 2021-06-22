---
title: about
date: 2021-05-08 17:26:20
---

## hexo发生error：spawn failed错误的解决方法

### 问题描述：
* 先是出现错误：
	```error：spawn failed...```
* 然后经过一些博客的操作会出现以下问题：
	```fatal: cannot lock ref 'HEAD': unable to resolve reference HEAD: Invalid argument error: src refspec```
* 或者：
	```error: src refspec HEAD does not match any.```等等
	总结一下：
	**问题大多是因为git进行push或者hexo d的时候改变了一些.deploy_git文件下的内容。**

### 解决办法：
* 删除**.deploy_git**文件夹;

* 输入```git config --global core.autocrlf false```

* 然后，依次执行：
	
	```
	hexo clean
	hexo g
	hexo d
	```
	
	

