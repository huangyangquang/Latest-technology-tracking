
<!-- TOC -->

- [面向未来的前端构建工具（一个开发服务器） - vite2.0 初识篇](#面向未来的前端构建工具一个开发服务器---vite20-初识篇)
  - [1. 什么是vite?](#1-什么是vite)
    - [如果是直接写commonJS 、UMD等前端模块化写，会怎么样呢？](#如果是直接写commonjs-umd等前端模块化写会怎么样呢)
  - [2. 为什么会出现vite?](#2-为什么会出现vite)
  - [3. 打包 和 构建之间有什么区别呢？（留坑）](#3-打包-和-构建之间有什么区别呢留坑)
  - [4. 为什么vite进行热更新的速度不会随着模块增多而变慢呢？](#4-为什么vite进行热更新的速度不会随着模块增多而变慢呢)
  - [5. vite的原理是什么？](#5-vite的原理是什么)
  - [6. 为什么vite是基于浏览器原生 ES imports实现的呢？  （留坑）](#6-为什么vite是基于浏览器原生-es-imports实现的呢--留坑)
  - [7. 通过实际，发现浏览器原生 ES imports需要开启本地服务器才可以访问。那么，为什么浏览器上运行ES imports需要开启本地服务器呢？  （留坑）](#7-通过实际发现浏览器原生-es-imports需要开启本地服务器才可以访问那么为什么浏览器上运行es-imports需要开启本地服务器呢--留坑)
- [参考](#参考)
- [其他提问](#其他提问)
- [备注](#备注)
- [社交信息 / Social Links:](#社交信息--social-links)
  - [(Welcome to pay attention, 欢迎关注)](#welcome-to-pay-attention-欢迎关注)

<!-- /TOC -->
# 面向未来的前端构建工具（一个开发服务器） - vite2.0 初识篇

## 1. 什么是vite?
借用作者的原话：
> Vite，**一个基于浏览器原生 ES imports 的开发服务器**。利用浏览器去解析 imports，在服务器端**按需编译返回**，**完全跳过了打包这个概念**，服务器随起随用。同时**不仅有 Vue 文件支持，还搞定了热更新，而且热更新的速度不会随着模块增多而变慢**。针对**生产环境则可以把同一份代码用 rollup 打包**。虽然现在还比较粗糙，但这个方向我觉得是有潜力的，做得好可以彻底解决改一行代码等半天热更新的问题。  

根据作者介绍，需注意到两个点：  
- 一个是 Vite 主要对应的场景是开发模式，原理之一就是拦截浏览器发出的 ES imports 请求并做相应处理。（生产模式是用 rollup 打包）
- 一个是 Vite 在开发模式下不需要打包，只需要编译浏览器发出的 HTTP 请求对应的文件即可，所以热更新速度很快。  

### 如果是直接写commonJS 、UMD等前端模块化写，会怎么样呢？
- 如果是导入的是直接本地开发的commonJs模块，是会报错的。（怎么解决呢？vite.config.js，留个坑，后边补上）
- 如果是导入的是通过npm/yarn下载的commonJs模块，会在预构建过程中将 CommonJS / UMD 转换为 ESM 格式。（关于预构建问题，留个坑，后边补上）


## 2. 为什么会出现vite?   
- vite的出现，是为了解决之前的构建工具上产生的什么问题嘛？    
- 相对于之前的构建工具，vite的优势在什么地方呢？  
请参考官网的解说： [为什么选 Vite](https://cn.vitejs.dev/guide/why.html) 

## 3. 打包 和 构建之间有什么区别呢？（留坑）

## 4. 为什么vite进行热更新的速度不会随着模块增多而变慢呢？
- 热更新的时候，Vite 只需要立即编译当前所修改的文件即可，所以响应速度非常快。（留坑，具体是怎么一个过程呢？）
- Vite 同时利用 HTTP 头来加速整个页面的重新加载（再次让浏览器为我们做更多事情）：源码模块的请求会根据 304 Not Modified 进行协商缓存，而依赖模块请求则会通过 Cache-Control: max-age=31536000,immutable 进行强缓存，因此一旦被缓存它们将不需要再次请求。

## 5. vite的原理是什么？  
- vite的实现需要从哪些方面进行分析？    
- vite的实现使用到了哪些技术？  
- vite在实现上解决了哪些难题呢？  
- 问题：
1. 引入路径问题。内部通过修改引用方式，留个坑，后边写个文章探究
2. 大量零散模块。如果不使用esbuild进行预编译的话，会因为文件间的依赖，产生多个请求
- 解法：
  1. 依赖预构建prebuild --- esbuild
  2. 利用HTTP 2的特性
  3. 真正的按需加载 (留坑，怎么实现的呢？)

## 6. 为什么vite是基于浏览器原生 ES imports实现的呢？  （留坑）
- 浏览器原生 ES imports有哪些优势的地方，是值得vite去使用的呢？

## 7. 通过实际，发现浏览器原生 ES imports需要开启本地服务器才可以访问。那么，为什么浏览器上运行ES imports需要开启本地服务器呢？  （留坑）

# 参考
- [为什么选 Vite](https://cn.vitejs.dev/guide/why.html)  
- [Vite 原理浅析](https://juejin.cn/post/6844904146915573773#heading-1)
- [Vite 2.0 初探](https://juejin.cn/post/6943821414747078687)
- [前端新玩具：Vite](https://github.com/zce/vite-essentials)
- [官方交流渠道](https://discord.com/)
- [官方文档](https://cn.vitejs.dev/guide/features.html)

# 其他提问
- webpack的热更新是怎么实现的呢？
- 有哪些其他的热更新的实现方式呢？

# 备注
这个初始篇，我会留坑比较多（实际上是对vite背后的深入挖掘）。这样子做，主要是想要把涉及到的技术点，在之后以一篇篇文章的形式整理出来。  
我是”留坑学长“


# 社交信息 / Social Links:
 ## (Welcome to pay attention, 欢迎关注)
Github：
[@huangyangquang](https://github.com/huangyangquang) | [最新技术追踪](https://github.com/huangyangquang/Latest-technology-tracking) | [javascript版算法](https://github.com/huangyangquang/Algorithm) | [早期前端知识总结 + 案例](https://github.com/huangyangquang/DEMO) | 欢迎Star✨✨✨


Social：
[新浪微博](https://weibo.com/u/6385661354) | [知乎](https://www.zhihu.com/people/cclv3) | [掘金](https://juejin.cn/user/2735240661699181) | [思否](https://segmentfault.com/u/c_z7wgq/articles) 

E-mail： fengquan.h@qq.com  

Old Blog：[CSDN](https://blog.csdn.net/huangyangquan3?type=blog)

微信公众号：前端学长Joshua  

<img src="../../static/img/wechatQrCode.jpg" width="50%">
 