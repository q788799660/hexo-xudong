---
title: 让NuxtSSR在云函数中飞起来
date: 2020-04-22 17:35:05
tags: 
- 云开发
- Nuxt
- web前端
- SSR
categories: 云开发
---
我们以往部署*Nuxt*到服务器需要pm2进行进程管理，还需要考虑到服务器的性能，负载均衡、网络安全等一系列运维问题。往往我们做的却不是最优的，那么为什么我们不将它交给专业运维的人去配置呢？

我们只去关心应用层面的业务逻辑，去关心用户的交互体验，这才是我们该做的事~ 

所以，*云开发*它来了！！它可以很完美的帮我们解决以上的问题，提升我们的开发效率，将所有精力放在业务逻辑以及用户的交互上。

*那如何在云开发中让我的Nuxt的SSR跑起来呢* 往下看👇

### 开发需求
#### node.js环境
我们需要用到npm以及云函数是基于[node.js](https://nodejs.org/en/) -v8.9 所以node.js必不可少~

#### create-nuxt-app
用到`create-nuxt-app`来创建一个nuxt项目

安装: `npm i create-nuxt-app -g`

#### @cloudbase/cli
用来进行快速、方便的部署项目，管理云开发资源。

安装： `npm i @cloudbase/cli -g` 

### 构建云开发项目
首先进行登录授权 `tcb login`，在弹出的窗口进行授权👇

![登录授权](https://imgkr.cn-bj.ufileos.com/f5d8bb9d-01c0-4697-a44e-00ee84110230.png)

新建一个[云环境](https://console.cloud.tencent.com/tcb/env/index)👇


![新建云环境](https://imgkr.cn-bj.ufileos.com/4f994c84-55b4-40da-b887-1cd49d7f6763.png)

在弹出窗口新建，我们选择*按量计费*并将*开启免费资源*选项勾上

**注：每个账户只能创建一个开启免费资源的云环境**

点击立即开通后，云环境会自动进行初始化。（过程大概持续2-3分钟）耐心等待即可~

待我们初始化完成后，使用命令`tcb init` 选择进行关联的云环境
```bash
$ tcb init
√ 选择关联环境 · nuxt - [nuxt-1a3208:空] 
√ 请输入项目名称 · nuxtSSR
√ 选择开发语言 · Node 
√ 选择云开发模板 · Hello World
√ 创建项目 nuxtSSR 成功！
```
创建完成后我们使用命令`tcb env:list`来查看云环境信息，并将云环境ID复制下来~

我们进入到云开发项目目录nuxt中 

此时的目录结构是这样的👇

```
.        
├── functions // 云函数目录
├── .editorconfig 
├── .gitignore 
├── cloudbaserc.js // 项目配置文件 
└── README.md
```
在cloudbaserc.js中将envID改成自己的云环境ID

![](https://imgkr.cn-bj.ufileos.com/d2d7b506-58c0-4901-be9a-ee9ee19726f7.png)

我们进入到functions中来新建一个云函数，在functions中一个文件夹为一个**云函数**。
`cd functions`
接下来我们就在**functions**下构建nuxt项目喽~

### 构建Nuxt项目

#### 创建一个项目
`npx create-nuxt-app nuxt`

安装node的时候会自动安装npm 而在npm5.2.0之后又会自动加入npx 所以这个命令不必单独安装
``` bash
$ npx create-nuxt-app nuxt
create-nuxt-app v2.15.0
✨  Generating Nuxt.js project in nuxt
? Project name nuxt # 项目的名称
? Project description My badass Nuxt.js project # 项目的描述
? Author name dxd   # 作者的名字
? Choose programming language JavaScript # 选择程序语言 我们选择JavaScript
? Choose the package manager Npm  # 选择包管理工具  我们选择npm
? Choose UI framework None # 选择UI框架 因为本次演示没有用到，所以选择none
? Choose custom server framework None (Recommended) # Server框架，同上
? Choose Nuxt.js modules (Press <space> to select, <a> to toggle all, <i> to invert selection) # 选择模块
? Choose linting tools (Press <space> to select, <a> to toggle all, <i> to invert selection)  # 选择JS检查工具
? Choose test framework None # 选择测试框架
? Choose rendering mode Universal (SSR) # 是否开启SSR服务端渲染，选择Universal开启
? Choose development tools (Press <space> to select, <a> to toggle all, <i> to invert selection)
```
创建成功后的样子子👇

![](https://imgkr.cn-bj.ufileos.com/24170ba0-ccec-48e1-8fe8-af512ba39cbb.png)

到此 我们就新建了一个名为nuxt的**云函数**~

那么函数的入口文件在哪呢？

我们需要为他新建一个index.js的入口文件。

并在其中写入
```js
//index.js
const {Nuxt} = require('nuxt')
const serverless = require('serverless-http')
let config = require('./nuxt.config.js')
const nuxt = new Nuxt(config)
exports.main = async (...args) => {
  console.log(config.dev)
  await nuxt.ready();
  return serverless((req,res) => {
        nuxt.render(req,res)
  })(...args)
}
```
上述代码中，我们用到了`serverless-http`,所以我们来安装它~

安装： `npm i serverless-http --save`

#### 配置项目
我们在nuxt.config.js中加入`dev:false`

![](https://imgkr.cn-bj.ufileos.com/668e9026-4481-4d20-af7f-7cc4f98e0e57.png)

否则部署云函数后会报下面错误👇~
```js
{
"errorCode":1,
"errorMessage":"user code exception caught",
"stackTrace":"EROFS: read-only file system, rmdir '/var/user/.nuxt'\n
Error: EROFS: read-only file system, rmdir '/var/user/.nuxt'"
}
```
#### 打包项目
我们在nuxt项目(functions/nuxt)中使用`npm run build`进行打包，会生成.nuxt文件夹

打包完成后回到云开发根目录 使用命令`tcb functions:deploy nuxt`
```bash
$ tcb functions:deploy nuxt
? 未找到函数发布配置，是否使用默认配置（仅适用于 Node.js 云函数） Yes
√ [nux] 云函数部署成功！
```

接着我们需要为这个云函数新建一个HTTP连接
`tcb service:create -f nuxt -p /nuxt`

```bash
$ tcb service:create -f nuxt -p /nuxt
√ 云函数 HTTP Service 创建成功！
点击访问> https://xxxx.service.tcloudbase.com/nuxt
```

我们点击 上面返回的连接即可看到我们部署的页面啦~👇

![](https://imgkr.cn-bj.ufileos.com/164a928c-3ebb-4102-9ca2-3da1ec94a169.png)

如果报错*超时*像这样👇~

`{"errorCode":-1,"errorMessage":"Task timed out after 3 seconds"}`

我们可以适当将超时时间延长一点~(默认超时时间为3s)

选择对应的云函数

在[函数配置](https://console.cloud.tencent.com/tcb/scf/index?envId=)中点击编辑
![](https://imgkr.cn-bj.ufileos.com/7abd024c-7997-495d-94b8-9e1ad9647883.png)

### 总结
#### NuxtSSR部署三步走

1. 构建云开发项目
2. 在云函数中构建nuxt项目并配置
3. 部署云函数，并为其新建HTTP连接