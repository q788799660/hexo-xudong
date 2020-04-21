---
title: Next如何部署到静态网站托管
date: 2020-04-21 19:19:22
tags:
- 云开发
- Next
- web前端
categories: 云开发
---

​我们知道部署web应用程序的最佳方式是作为静态HTML应用程序，因为他对搜索引擎很友好，速度快等等，这对我们写个人博客这样的小型网站无异于非常nice。如果你的应用可以作为静态HTML，那么可以试试**Next.js**。
它可以把一个应用程序作为静态页面导出，那么导出的静态页面怎么部署到静态托管呢？我们以云开发静态托管服务为例。

**什么是云开发？**
可以理解为它为我们提前做好了很多的事(例如负载均衡，冷备热备，网络安全等等)，使我们只需关注业务逻辑即可。就像包饺子一样，提前有人给你准备好饺子馅和发好的面，我们只需要包饺子就可以了。详细了解可以到[云开发](https://cloud.tencent.com/product/tcb)查看

首先我们需要准备的环境以及工具如下：

**必要环境:**
> node.js
> 开通云环境

**开发工具：**
> create-next-app
> @cloudbase/cli

下面我们来详细操作~

首先我们进行安装create-next-app
    `npm i create-next-app`
以及云开发工具@cloudbase/cli
    `npm i @cloudbase/cli`
> npm命令是在安装node.js时自动安装

#### 构建Next项目
1. 利用脚手架创建一个项目
`npx create-next-app 项目名称`
此处项目名称为你的项目根目录名称

2. 创建完成后我们进入到项目中
`cd 项目名称`
3. 我们需要在跟目录中新建一个next.config.js文件

```js
module.exports = {
    exportTrailingSlash: true,
    exportPathMap: function () {
         return {
            '/': {page: '/'}
         };
     },
};
```
如果你希望生成的静态文件不只包含**首页和404页面**(Next自动生成),那么可以在next.config.js中加入`'/about': {page: '/about/about'}`,并在pages下新建一个about文件夹并创建about.js文件写入，如果只是测试生成首页和404就够了，那么直接跳到第4步即可
```js 
const about = () => (
  <div>about</div>
)
export default about
```
   附上next.config.js添加后的完整代码：

```js
module.exports = {
     exportTrailingSlash: true,
     exportPathMap: function () {
        return { 
            '/': {page: '/'},
            '/about': {page: '/about/about'}
        };
     },
};
```
4. 在package中加入一个script命令
`
 "export": "next export"
`
5. 我们运行下列代码来生成静态文件

```
    npm run build
    npm run export
```
我们发现根目录中生成了一个out文件夹，该文件夹下的所有文件就是我们生成的静态文件，所以接下来要做的事就是开通云环境并将其部署到静态网站托管。

![](https://imgkr.cn-bj.ufileos.com/575c3fbc-ee93-4475-a35c-20e6cc4db25e.png)


#### 开通云环境
1. 我们打开[云开发](https://console.cloud.tencent.com/tcb/env/index)并创建一个新的环境


![](https://imgkr.cn-bj.ufileos.com/4591475e-ceaa-446f-8e93-47fa4998f70b.png)


2. 这里要注意选择是**按量计费**的模式(只有按量计费才能开通静态网站托管)。


![](https://imgkr.cn-bj.ufileos.com/eaf7a7eb-66a1-4001-8a98-0ddfd5efa512.png)


3. 创建成功后会自动对环境进行初始化(此过程大概2~3分钟)。

![](https://imgkr.cn-bj.ufileos.com/000ce34f-f8f1-4a24-9926-956df09ce649.png)


4. 初始化成功后我们进到对应的环境中找到静态网站托管并开始使用


![](https://imgkr.cn-bj.ufileos.com/b114eba9-752a-4400-aee1-34cae97658b8.png)


等待静态网站服务初始化后就可以使用啦~
#### 部署上传

1. 首先在项目根目录下执行云开发登录命令
`tcb login`

![](https://imgkr.cn-bj.ufileos.com/a80cb627-7b5d-4f59-9348-2549b26b8298.png)


2. 在弹出的页面进行授权操作

![](https://imgkr.cn-bj.ufileos.com/b912149c-2796-4ef9-9c6c-c4c7921eb0ab.png)

3. 进行上传操作，这里我们希望out文件夹下所有的文件一并上传，所以，我们执行命令
`tcb hosting:deploy ./out -e 你的云开发环境ID`

![](https://imgkr.cn-bj.ufileos.com/4b8eadd6-8adc-4dfb-bf43-10bff8517303.png)

云环境ID可在环境ID下查看


![](https://imgkr.cn-bj.ufileos.com/81d38eb9-5525-4a41-8395-444cf96b6794.png)

![](https://imgkr.cn-bj.ufileos.com/db0e129f-c1f4-4fb9-8f1c-79ff9c4649d2.png)


4. 上传完成后我们在静态网站托管中可以看到我们out目录下的所有文件
   

![](https://imgkr.cn-bj.ufileos.com/61772c4a-08f8-48cb-9d6a-26f367a4a248.png)
  
5. 云开发默认提供了一个与环境对应的默认域名，可以通过这个默认域名进行访问。


![](https://imgkr.cn-bj.ufileos.com/8daf7ca4-c2e9-4047-9643-5c0611d856d0.png)


![](https://imgkr.cn-bj.ufileos.com/3d74b1ca-84e5-4bc9-9cb8-0499e23b608b.png)

#### 总结
我们总结一下步骤，大体上只有三步
1. 创建Next项目并生成静态文件
2. 开通云环境与静态网站托管服务
3. 使用云开发工具cloudbase/cli命令进行部署上传

