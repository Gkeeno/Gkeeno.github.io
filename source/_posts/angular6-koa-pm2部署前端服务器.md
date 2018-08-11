---
title: angular6+koa+pm2部署前端服务器
date: 2018-08-11 17:36:07
tags:
---
> 本文不可避免地会出现以下概念，希望在阅读前能理解：
- angular6 的打包和初始化
- 基础的nodejs编程

# 前言
## 为何要部署前端服务器？
### 分离前后端
要知晓这个概念，首先就要区别出前后端的概念。  
在传统的web应用开发中，大多数会将浏览器作为区别前后端的标准：为用户展示界面的部分称为前端，为界面提供数据或提供业务逻辑服务的称为后端。

而在 传统单体应用 开发模式中，所有的页面都和后端代码放在一个项目中，部署时也是整个项目部署，前端开发人员甚至还要掌握一些后端开发的知识才能输出代码。  
为了让前后端分工更加明确，更好的各司其职，从长远来看，前后端分离也是一个必然的趋势和结果。

#### 解耦代码
![code-repo](http://wx1.sinaimg.cn/mw690/0060lm7Tly1fu5llb3mwhj30m80a2gmg.jpg)  
在传统的 单体应用 开发模式中，所有的代码都放在同一个项目中，不能单独运行，这个时候的项目是 前后端未分离的。
- **半分离**  
前后端共在一个项目，共用一个代码库，但是分别建立前端和后端的工程；  
前端不用关心后端代码是怎么写的，后端也不用担心前端代码是什么样的，但整个项目不能单独运行，需要后端和前端进行交互。

- **完全分离**  
前后端分别建立项目，代码库相互独立；  
前端能独立开发，依靠 mock程序或测试构造一个伪后端 运行   
后端也能独立开发，编写有详细的测试用例，保证API的可用性，并做好详细的接口文档，使其跟前端能够顺利集成。

#### 解耦部署
在部署单体应用中，整个项目打包上服务器即可运行，这个时候的项目是前后端未分离的。

部署半分离的前后端项目也是一样，发布整个项目即可。

完全分离前后端后，前后端可单独部署，本章结即是讲解部署前端项目部分。

> ⚠️ 前后端分离总体提升了部署和开发的复杂程度，对于小项目来说是过度设计，导致开发效率更低。

#### 分离提升性能瓶颈
有时候前端的流量和后端不在一个量级上，分离前后端可以使前后端采用不同的架构设计，使系统更加强大。

一些大厂的解决方案比如阿里云的控制台管理系统，部署时就采用反向代理服务器，把获取页面的请求路由到前端服务器，接口请求路由到后端服务器。  
> 你甚至可以在http response 的 header 里看到所使用的反向代理服务器的名称。

## 为何要选择koa作为前端服务器？
Koa 是一个基于 nodejs 平台的下一代 web 开发框架，相对最初火热的 Express 框架来说充分利用async 等一套优雅的方法，摒弃了"难看"的多层callback嵌套，能够快速而愉快地编写服务端应用程序。

而nodejs平台本身就具有着独特的异步、非阻塞I/O的特点，这也就意味着他特别适合I/O密集型操作，在处理并发量比较大的请求上能力比较强。  
此外，现代web开发框架（angular,vue,react）都基本上利用nodejs平台搭建前端项目，使用nodejs能无缝集成前端项目；  
前端项目打包生成静态文件后，直接运行nodejs程序，利用其高I/O向客户端提供静态文件，nodejs确实是一个不错的选择。

# 部署前端服务器
## 运行环境介绍
服务器系统：centerOS 7 （64位）  
本地环境： windows 10 （64位）  
连接工具：[MobaXterm](http://mobaxterm.mobatek.net/)

我的服务器是在阿里云买的ECS，购买的时候直接选了centerOS 7，服务器上必须安装nodejs运行环境，如何安装在这里不做赘述。  
安装完 nodejs 后用 npm 全局安装 pm2（一个nodejs程序托管系统），输入命令行`npm install -g pm2`即可。

## 前端项目介绍
项目采用angular+koa搭建的一个前端项目：  
![directory-tree](http://chuantu.biz/t6/356/1533969881x-1566660829.png)   
- `dist/spa-server`是我们前端项目打包后的静态文件，展示网站主要就是输出这些文件。
- `src` 是前端项目的源代码
- koa-app.js 是运行前端服务器的脚本
1. 先使用的angular cli生成前端项目脚手架，再使用`npm i koa koa-static koa-router`安装运行koa的依赖库。
2. 在项目根目录添加运行服务器的脚本 koa-app.js
添加以下基本代码:
```js
const koa = require("koa");
const router = require("koa-router")();
const static = require('koa-static'); 
const app = new koa();

const path = require('path');
const fs = require('fs');

var staticFileFolder = path.join(__dirname,"dist","spa-server");
var indexPath = path.join(staticFileFolder,"index.html");
/******************************
 * Set Static File
 * 设置静态文件后访问该文件路径 会直接返回不会走下一个中间件
 *****************************/
app.use(static(
  staticFileFolder
));

/******************************
 * logger
 *
 *****************************/
app.use(async (ctx, next) => {
  await next();
  console.log(`${ctx.method} : ${ctx.url} - ${hResponse}`);
});

/******************************
 * Routes
 * 
 *****************************/
 // 读取 index.html 的文件内容
const indexPage = fs.readFileSync(indexPath).toString();

router.get('*', async (ctx, next) => {
  ctx.response.body = indexPage;
}); 
app.use(router.middleware())



// 监听端口，启动服务器
app.listen(80);
```
上面的代码在路由部分做了特殊的处理，因为使用angular6搭建的网页为单页面应用程序(SPA)，路由部分是由前端来处理的。  
这种特性使得服务器要对请求的路径做出特殊响应：所有访问页面的路径都默认返回index.html的内容，然后交由前端的js来解析路径生成页面。[可参考angular官网说明](https://angular.cn/guide/deployment)

首先要在本地能够运行服务器，在项目根目录输入`node koa-app.js`运行服务器脚本。访问浏览器，页面如下:
![page](http://chuantu.biz/t6/356/1533972158x-1566661044.png)
本地运行可以正常访问的话，接下来就可以正式部署到服务器上了。

## 部署到服务器
### 精简上传文件
传输到服务器上运行的文件不必像本地开发一样，只需保留必须的文件即可，最终精简如下：
![page](http://chuantu.biz/t6/356/1533973186x-1404781120.png)
- `www` 是我们前端项目打包后的静态文件
- 由于静态文件的文件夹的名称和路径都变了, koa-app.js 中需修改相应代码
```js
var staticFileFolder = path.join(__dirname,"dist","spa-server"); 
// 改为如下:
var staticFileFolder = path.join(__dirname,"www"); 
```
- deploys.json 这个新增的文件用来引导 pm2的运行，基本内容为:
```json
{
  "apps": [
    {
      "name": "deploy-test1",
      "script": "./koa-app.js",
      "env": {
        "COMMON_VARIABLE": "true"
      },
      "env_production": {
        "NODE_ENV": "production"
      }
    }
  ]
}
```
`name` 是 托管到pm2上的程序名称  
`script` 是 程序的路径
- package.json 是nodejs程序的工程信息，随着项目初始化新建。包括我们程序的依赖项都在该处配置：
```json
{
  "name": "publish",
  "version": "1.0.0",
  "description": "",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "koa": "^2.5.2",
    "koa-router": "^7.4.0",
    "koa-static": "^5.0.0"
  }
}
```
`dependencies` 即是程序的依赖项,必须保留 `koa`,`koa-router`,`koa-static`

### 上传到服务器
这里我使用 MobaXterm 作为远程工具:  
保持目录结构,上传到服务器上，定位到项目的根目录位置下,输入`npm install`安装所有的依赖包，如下
![linux-server](http://chuantu.biz/t6/356/1533976856x-1404758437.png)

### 运行程序
安装完所有的依赖包程序就可以成功运行了,
还是定位到项目的根目录下，输入`pm2 start deploy.json`

![linux-server](http://chuantu.biz/t6/356/1533977903x-1404795589.png)

如果看到下面类似仪表盘的东西，恭喜你已经成功部署了前端服务器。
> pm2会帮你维持程序的运行防止意外挂掉，可以使用一些命令行查看程序的运行状态：
> `pm2 list`查看所有正在运行程序的状态快照
> `pm2 monit`打开监控仪表盘，监控所有运行的程序

这个时候就可以访问我们部署在服务器上的网站了:
![linux-server](http://chuantu.biz/t6/356/1533978560x-1404781120.png)
> - 如果不能访问网站可以搜索防火墙设置与端口开放的方法开放端口  
> - 阿里云等一些云服务器需配置安全策略，把一些端口设置白名单才可访问网站

# 后话
前端单独部署，如果跟后端服务器不在一个域名或端口的话还会有跨域的问题。  
为了解决这个问题,可以部署一个反向代理服务器，把前端的请求路由到前端服务器上，后端的请求路由到后端服务器上，对外都是只访问这个反向代理服务器，跨域问题也就迎刃而解。