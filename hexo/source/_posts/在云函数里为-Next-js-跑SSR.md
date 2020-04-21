---
title: 在云函数里为 Next.js 跑SSR
date: 2020-04-21 19:25:59
tags:
- 云开发
- Next
- web前端
- SSR
categories: 云开发
---


很多时候我们都希望首屏速度快，SEO友好，那么相比于客户端渲染，SSR渲染将是这方面的优势。
而Next.js、Nuxt.js都是SSR框架。本篇文章只用Next.js。
通常我们在部署SSR的时候，会担心运维等问题，但如果我们把它部署在云开发上就可以不必担心~
我们部署看看喽~

#### 环境准备
1. 安装[node.js](http://nodejs.cn/download/)
2. 安装云开发工具@cloudbase/cli
> npm i @cloudbase/cli

#### 搭建云环境
1. 首先在打开[云开发](https://console.cloud.tencent.com/tcb/env/index)并新建环境
![新建环境](https://imgkr.cn-bj.ufileos.com/345da882-e8ec-403b-993d-4bcc5f191481.png)

2. 创建完成后会自动进入环境初始化阶段，这个阶段大概持续2-3分钟。。
![初始化](https://imgkr.cn-bj.ufileos.com/8f364628-ab76-49b7-8818-ada8824d2fe9.png)

#### 初始化项目
当环境初始化完成后我们就可以进行初始化项目啦~
1. 使用 CLI 工具初始化一个云开发项目
> $ tcb init
```
tcb init
? 选择关联环境 xxx - [xxx-xxx]
? 请输入项目名称 nextSSR
? 选择模板语言 Node
? 选择云开发模板 Hello World
✔ 创建项目 cloudbase-next 成功！
```
初始化结束后的项目目录如下
```
nextSSR
└─. 
  │  .editorconfig
  │  .gitignorev1
  │  a.txt
  │  cloudbaserc.js
  │  README.md
  │  
  └─functions
      └─app
              index.js
```
2. 我们进入到项目中
> $ cd nextSSR
3. 然后在 functions文件夹下创建next.js应用：
> $ npm init next-app functions/next
等待初始化next.js项目
4. 初始化完成后在functions文件夹下会多出一个next的文件夹，这个便是我们的next应用

#### 配置next
1. 首先我们进入到next项目的根目录
> $ cd functions/next
2. 然后安装severless-http
> $ npm install --save serverless-http
3. 在next应用的根目录下`项目根目录/functions/next应用根目录`新建**index.js**,并将下列代码添加进去
```js
// index.js
const next = require('next')
const serverless = require('serverless-http')

const app = next({ dev: false })
const handle = app.getRequestHandler()

exports.main = async function(...args) {
    await app.prepare()
    return serverless((req, res) => {
        handle(req, res)
    })(...args)
}
```
4. 在next应用的根目录中新建**next.config.js**并将下列代码拷入
```js
// next.config.js
module.exports = {
    assetPrefix: '/next'
}
```
这样我们的项目就配置差不多了。

#### 项目的构建与发布
首先我们进入到functions/next目录中
执行`$ npm run build`
1. 然后回到项目根目录中，运行cli命令将代码上传到云函数

`$ tcb functions:deploy next`

2. 然后我们创建一个http服务

使用命令`$ cloudbase service:create -f next -p /next`
> -f表示HTTP Service路径绑定的云函数名称\
  -p表示Service Path，必须以"/"开头

```
$ cloudbase service:create -f next -p /next
✔ 云函数 HTTP service 创建成功！
```

![](https://imgkr.cn-bj.ufileos.com/6493f06d-9238-4b2c-b512-f077f53511db.png)

随后我们便可以通过**点击这个地址**进行访问啦。

3. 那我们上传到了哪里了呢？
我们进入到[云开发管理](https://console.cloud.tencent.com/tcb/scf/index)页面

![](https://imgkr.cn-bj.ufileos.com/6da1f8d3-d961-4a84-89ab-5304eac292f5.png)

我们看到在云函数的函数代码中可以找到我们刚才上传的文件

4. 我们点击预览即可浏览页面啦~
在函数配置可以通过触发云函数来进行浏览我们的页面

![](https://imgkr.cn-bj.ufileos.com/17846e0f-898b-44a1-a03b-d5db1eea7c89.png)

![](https://imgkr.cn-bj.ufileos.com/a5aeff1c-6a7c-4502-99f5-284991b3e428.png)

#### 对比
我们通过对比查看
  + 通过SSR渲染的页面加载速度
  
![](https://imgkr.cn-bj.ufileos.com/12b6f0e3-3e11-4ff2-814e-bb13e4a35872.png)

  
  + 非SSR的加载速度
  
![](https://imgkr.cn-bj.ufileos.com/4ebdeaeb-518b-4765-8532-8a50c2fd99bf.png)
可以看到有明显的速度提升啦~

最后想了解更多云开发的知识，可以猛戳[云开发文档](https://www.cloudbase.net/)啦~
