---
title: Nuxt部署到静态网站托管
date: 2020-04-19 12:12:57
categories: 
- 云开发
tags:
- 云开发
- Nuxt
- web前端
---

最开始了解到**Nuxt**是在vue SSR下了解到，用过之后感觉真香。 可以省去路由划分的时间，Nuxt.js 会读取该目录下所有的 .vue 文件并自动生成对应的路由配置、进一步封装Vuex等等。下面来介绍 **如何将Nuxt部署到静态托管上？**

**云开发（Tencent CloudBase，TCB）**是云端一体化的后端云服务 ，采用 **serverless** 架构，免去了移动应用构建中繁琐的服务器搭建和运维。同时云开发提供的静态托管、命令行工具（CLI）、Flutter SDK 等能力极大的降低了应用开发的门槛。

# 环境要求

> node.js

# 工具准备

> Nuxt脚手架： create-nuxt-app
> 云开发命令工具： @cloudbase/cli

# 安装

> 安装Nuxt脚手架
`npm i create-nuxt-app`

> 安装云开发命令工具 CLI
`npm i -g @cloudbase/cli`

> 测试 cloudbase/cli 是否安装成功 可以使用cloudbase或tcb命令
`cloudbase -v`

或

> `tcb -v`

> 更多命令可以移步[全部命令](https://docs.cloudbase.net/cli/intro.html#suo-you-ming-ling)
> 或运行命令
> `tcb -h`

# 创建Nuxt项目

`npx create-nuxt-app demo`

> 创建过程详细请[参考文档](https://zh.nuxtjs.org/guide/installation)

# 紧接着进入到项目目录下（这里是demo）

1. 在命令行下执行`npm run generate` 生成静态html文件
2. 在项目目录中会生成一个**dist**文件夹。该文件夹下的文件就是生成的静态文件


![](https://imgkr.cn-bj.ufileos.com/30916801-a6d4-4fe6-b46e-219e16183b77.png)


# 到此Nuxt部分就已经搞定了，现在要做的就是怎样将这个静态网站托管到云开发？

> 首先我们打开 [云开发](https://cloud.tencent.com/product/tcb)


![](https://imgkr.cn-bj.ufileos.com/90c36604-3f9d-4886-9918-c70680bad9a8.png)


> 选择或创建自己的云开发环境


![](https://imgkr.cn-bj.ufileos.com/24a28538-103f-4fc8-b83a-f69fa556c109.png)


> 这里要注意选择是**按量计费**的模式(只有按量计费才能开通静态网站托管)。


![](https://imgkr.cn-bj.ufileos.com/8a747438-fbc4-4b87-9c77-086a43f189cb.png)


> 创建成功后会自动对环境进行初始化(此过程大概2~3分钟)。


![](https://imgkr.cn-bj.ufileos.com/50f639a2-8aeb-4c8c-8565-401a7428face.png)


> 初始化成功后我们进到对应的环境中找到静态网站托管并开始使用


![](https://imgkr.cn-bj.ufileos.com/b6948f84-33d1-4b74-b3ad-97349e2b9f29.png)

等待静态网站服务初始化后就可以使用啦~

接下来我们就可以将nuxt的静态网站上传到云开发静态网站托管了。

首先执行登录命令

```
tcb login
```


![](https://imgkr.cn-bj.ufileos.com/db309378-b359-4681-8b71-fa7f92d0b0c6.png)


在弹出的页面进行授权


![](https://imgkr.cn-bj.ufileos.com/be7e01e5-8162-4000-8a7b-b3d2ef8df988.png)


接着，将静态网站进行部署到云开发静态网站托管

这里我们将dist文件夹下的所有文件都部署到静态网站托管中，执行命令

```
tcb hosting:deploy 文件夹  -e 云环境ID
```

这里的**文件夹**是将此文件夹下所有的文件都部署到云开发的根目录中，云环境ID可在环境ID下查看


![](https://imgkr.cn-bj.ufileos.com/28028fe9-b520-48b2-af88-fbd071b3747c.png)


因为我们希望将dist下的所有文件部署上去，所以上面的命令我们可以写成

```
tcb hosting:deploy ./dist -e demo-1cdbae
```


![](https://imgkr.cn-bj.ufileos.com/f3de2c77-5502-4afd-9a2c-e2702aa8ce97.png)


上传成功后我们会发现，静态网站托管中多了许多文件


![](https://imgkr.cn-bj.ufileos.com/9bb64c3d-f83f-46f6-b079-ba4b0629eb52.png)


那我们如何浏览呢？

云开发默认提供了一个与环境对应的默认域名，可以通过这个默认域名进行访问。


![](https://imgkr.cn-bj.ufileos.com/e384c6d2-2c88-4d03-af79-d0ce8bc93df0.png)

![](https://imgkr.cn-bj.ufileos.com/3a27ee13-b389-4cc2-979b-c6af8c30eadd.png)


这样至此我们的Nuxt就部署成功啦~

但**默认域名**存在限制下行速度10KB/S，如果正式使用的话需要添加一个已经备案的域名


![](https://imgkr.cn-bj.ufileos.com/f075e63d-9721-4c55-b1c8-e1b3c8beb01c.png)


并为其添加dns解析


![](https://imgkr.cn-bj.ufileos.com/b34123ff-33ea-4b85-bb9d-03d5f0f38489.png)

![](https://imgkr.cn-bj.ufileos.com/d7974b71-0aa7-473d-97ad-eb3c6c54837f.png)


如果可以ping通这个CNAME就可以进行使用自己的域名进行访问啦~~