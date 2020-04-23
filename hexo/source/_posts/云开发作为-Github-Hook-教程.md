---
title: 云开发作为 Github Hook 教程
date: 2020-04-23 18:40:28
tags:
- 云开发
- web前端
- Github
categories: "云开发"
---

我们通常会有需求：将新push到Github上的代码自动触发其他事件

Github为我们提供了webHooks，它类似于发布订阅模式，它订阅了GitHub.com上的某些事件。当这些事件之一被触发时，将向WebHook的配置 URL 发送 *HTTP POST payload*。

例如 我们向Github新push上了代码，webHooks就会监听到这个push事件，随后向配置的URL发送HTTP POST payload

[webHooks](https://developer.github.com/webhooks/) 文档戳这

而云开发中的云函数刚好匹配这一需求，当我们在云上部署一个云函数并为其创建一个 HTTP 触发路径，顾名思义通过这个路径可以触发对应的云函数。所以我们可以将webHooks与云函数进行结合~

> push到Github => webHooks监听到push事件 => webHooks通过配置的URL触发云函数 => 在云函数中触发事件

在对大概流程有一个了解后，来进行具体的实践操作~

### 开发前准备
我们需要用到一只[node.js](http://nodejs.cn/)，一只@cloudbase/cli

[@cloudbase/cli](https://docs.cloudbase.net/cli/intro.html#an-zhuang-cloudbase-cli) 是一个开源的命令行界面交互工具，用于帮助用户快速、方便的部署项目，管理云开发资源。

#### 安装node

直接去官网Download就可以啦~

#### 安装@cloudbase/cli

`npm i @cloudbase/cli -g`


### 开通云环境
既然我们要用到云函数，那么它当然需要有个云环境~

我们进入到[云开发](https://console.cloud.tencent.com/tcb/env/index),新建一个环境~如果还没有开通云环境可以勾选**开启免费资源**

**注： 每个账户只能开通一个享有免费资源的环境**

![](https://imgkr.cn-bj.ufileos.com/d8fc2802-dffc-4e3c-a7a7-9941dc35f85f.png)

创建后会进行自动初始化环境(大概2-3分钟)~

我们可以使用cli工具进行查看环境状态也可以在[控制台](https://console.cloud.tencent.com/tcb/env/index)进行查看

我们使用cli命令 `tcb env:list` 来进行查看环境状态

![](https://imgkr.cn-bj.ufileos.com/37b1eb6a-3366-4124-acaf-ebbfcd19f513.png)
当环境状态变为正常就代表初始化已经完成可以正常使用啦~

### 初始化云开发项目

使用命令 `tcb init` 创建一个云开发项目

```bash
$ tcb init
√ 选择关联环境 · xxxx - [xxx-xxxx:空] 
√ 请输入项目名称 · cloudFunction
√ 选择开发语言 · Node 
√ 选择云开发模板 · Hello World
√ 创建项目 cloudFunction 成功！
```
创建完成的云开发目录结构如下👇

```bash
.        
├── functions // 云函数目录
      └─app  
├── .editorconfig 
├── .gitignore 
├── cloudbaserc.js // 项目配置文件 
└── README.md
```
在**functions**中一个目录代表一个云函数，我们创建完成会自动为我们添加一个名为 app 的云函数

我们可以将app修改一下，当然也可以新建一个云函数~

将app修改为webHooks

将云函数的入口文件也就是 index.js 添加一个日志输出👇
```js
exports.main = async (event) => {
    console.log('触发了')
}
```
### 部署云函数
此时的云函数还在本地，我们需要将它部署到云端。通过命令`tcb functions:deploy webHooks`
```bash
$ tcb functions:deploy webHooks
? 未找到函数发布配置，是否使用默认配置（仅适用于 Node.js 云函数） Yes
√ [webHooks] 云函数部署成功！
```

部署后我们可以通过命令查看云函数详情 `tcb functions:detail webHooks -e 环境ID`

注： 环境ID可通过命令 `tcb env:list` 查看
```bash
$ tcb functions:detail webHooks -e 对应的环境ID
云函数 [webHooks] 详情：

┌─────────────────────┬─────────────────────┐
│        信息         │         值          │
├─────────────────────┼─────────────────────┤
│       环境 Id       │     对应的环境ID    │
├─────────────────────┼─────────────────────┤
│      函数名称       │      webHooks       │
├─────────────────────┼─────────────────────┤
│        状态         │      部署完成       │
├─────────────────────┼─────────────────────┤
│    代码大小（B）    │         220         │
├─────────────────────┼─────────────────────┤
│ 环境变量(key=value) │         无          │
├─────────────────────┼─────────────────────┤
│      执行方法       │     index.main      │
├─────────────────────┼─────────────────────┤
│    内存配置(MB)     │         256         │
├─────────────────────┼─────────────────────┤
│      运行环境       │      Nodejs8.9      │
├─────────────────────┼─────────────────────┤
│     超时时间(S)     │          3          │
├─────────────────────┼─────────────────────┤
│      网络配置       │         无          │
├─────────────────────┼─────────────────────┤
│       触发器        │         无          │
├─────────────────────┼─────────────────────┤
│      修改时间       │ 2020-04-23 16:07:51 │
├─────────────────────┼─────────────────────┤
│    自动安装依赖     │        TRUE         │
└─────────────────────┴─────────────────────┘

函数代码（Java 函数以及入口大于 1 M 的函数不会显示）

// 返回输入参数
exports.main = async (event) => {
    console.log('触发了')
}
```

### 创建 HTTP 触发路径
当云函数部署完成后我们怎样才能触发这个函数呢？

我们需要为它创建一个触发路径，每当我们进入到这个URL都会触发这个云函数

通过命令 `tcb service:create -f webHooks -p /webHooks` 为云函数创建一个HTTP触发路径
```bash
tcb service:create -f webHooks -p /webHooks     
√ 云函数 HTTP Service 创建成功！
```
随后在控制台会生成一个连接，这个连接就是触发这个云函数的路径👇

![](https://imgkr.cn-bj.ufileos.com/ed757a07-e030-4e49-83eb-bb49cedb59f3.png)

### 创建webHooks
我们云函数搞定之后剩下的就是webHooks的创建啦~
我们进入到对应的Github仓库中，点击Setting，并ADD一个webhook

![](https://imgkr.cn-bj.ufileos.com/8c3e1dc9-6ee7-45db-9e87-c27f1a577db4.png)


![](https://imgkr.cn-bj.ufileos.com/402b11b2-c85b-4681-ae81-4f94468d721b.png)
添加完成后我们可以进行测试喽~

### 测试

向你的Github上进行push操作

随后在[云开发](https://console.cloud.tencent.com/tcb/scf/index)控制台内查看对应云函数的日志

![](https://imgkr.cn-bj.ufileos.com/2ee969b8-2b11-4954-9b5d-3f9e8a39a782.png)

发现打印出来了 ‘触发了’ 三个字~，说明成功实现了对push操作的监听，并触发云函数~

### 总结
总结一句话👇

将webhooks的URL配置到**云函数**的 HTTP 触发路径即可实现监听~
