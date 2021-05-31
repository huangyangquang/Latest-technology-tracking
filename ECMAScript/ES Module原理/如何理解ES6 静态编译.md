# 如何理解ES6 静态编译?

<!-- TOC -->

- [如何理解ES6 静态编译?](#如何理解es6-静态编译)
- [实例](#实例)
- [参考](#参考)
- [留坑](#留坑)
- [社交信息 / Social Links:](#社交信息--social-links)
  - [(Welcome to pay attention, 欢迎关注)](#welcome-to-pay-attention-欢迎关注)

<!-- /TOC -->


请先阅读[深入ES Module, 浅析原理](https://github.com/huangyangquang/Latest-technology-tracking/blob/main/ECMAScript/ES%20Module%E5%8E%9F%E7%90%86/ES%20Module%E5%8E%9F%E7%90%86.md)


ES6 模块编译时执行会有以下两个特点：
1. import 命令会被 ~~JavaScript 引擎~~静态分析，优先于模块内的其他内容执行。
2. export 命令会有变量声明提前的效果。

那么现在，我们来探究：
1. 为什么import 命令会被 JavaScript 引擎静态分析，优先于模块内的其他内容执行呢？
2. 为什么export 命令会有变量声明提前的效果呢？


在[深入ES Module, 浅析原理](https://github.com/huangyangquang/Latest-technology-tracking/blob/main/ECMAScript/ES%20Module%E5%8E%9F%E7%90%86/ES%20Module%E5%8E%9F%E7%90%86.md)里，很清晰的指出。在浏览器 / Node 环境中，会有一个loader下载器去帮我们查找并且加载都有的文件。

具体如下：
loader先要判断出入口文件，在html文件里边，loader可以通过携带type="module"的script标签，来判断出是一个入口文件。  
<img src="./../static/img/5.png" width="50%">

当loader下载完入口文件之后，就会通过import语句的某一部分又被叫做“模块说明符”，"模块说明符"会告诉loader在哪儿可以找到下一个模块。
在浏览器中，浏览器会根据”模块说明符“（就是一个URL）去下载模块文件。  

<img src="./../static/img/6.png" width="50%">

**所以，在这个步骤，import命令会被静态分析。**  
但是，是被js引擎还是被loader下载器静态分析的，目前我还不确定。（知道的朋友可以联系我下）


第二个问题：为什么export 命令会有变量声明提前的效果？  
我们知道，在对文件进行解析之后，我们就从拥有一个入口文件到最后拥有一系列的module record。  
<img src="./../static/img/9.png" width="50%">  

下一步就是对模块进行实例化，并且将所有的模块实例链接起来。  

首先，JS引擎会创建了一个“模块环境记录（module environment record）”。  
这个模块环境记录管理着module record的变量，然后它在内存里面找到所有导出（export）的变量的“盒子(内存地址)。  
module environment record会一直监控着内存里面的哪个盒子和哪个export是相关联的。  
<img src="./../static/img/10.png" width="70%">

在这个时候，这些内存里面的盒子还没有获得它们的值，只有在求值这一步骤完成之后，真正的值才会被填充进去。

为了实例化模块图（module graph），JS引擎会做一个所谓深度优先后序遍历的操作。意思就是说，**JS引擎会先走到模块图的最底层 -- 找到不依赖任何其他模块的那些模块，并且设置好它们的导出（export）。**（所以，这一步就是 export 命令会有变量声明提前的效果的原因）

当JS引擎完成一个模块的所有导出的链接，它就会返回上一个层级去设置来自于这个模块的导入（import）。
需要注意的是，导出和导入都是指向同一片内存地址。  
先链接导出保证了所有的导入都能找到对应的导出。  
<img src="./../static/img/11.png" width="70%">

ES module使用所谓的“实时绑定”，导出的模块和导入的模块都指向同一段内存地址。

在这一步的最后，我们使得所有的模块实例导出/导入的变量的内存地址链接起来了。


# 实例
- 案例1：demo1 -  import 命令会被 ~~JavaScript 引擎~~静态分析，优先于模块内的其他内容执行
> // a.js  
> console.log('a.js')  
> import { foo } from './b';  
> 
> // b.js  
> export let foo = 1;  
> console.log('b.js 先执行');  
> 
> // 执行结果:  
> // b.js 先执行  
> // a.js  

从执行结果我们可以很直观地看出，虽然 a 模块中 import 引入晚于 console.log('a')，但是它被 ~~JS 引擎通过~~静态分析，提到模块执行的最前面，优于模块中的其他部分的执行。

由于 import 是静态执行，所以 import 具有提升效果,即 import 命令在模块中的位置并不影响程序的输出。

- 案例2：demo2 - export 命令会有变量声明提前的效果
> // a.js  
> import { foo } from './b';  
> console.log('a.js');  
> export const bar2 = () => {  
>   console.log('bar2');  
> }  
> export function bar3() {  
>   console.log('bar3');  
> }  
> 
> // b.js  
> export let foo = 1;  
> import * as a from './a';  
> console.log(a);  
> 
> // 执行结果:  
> // { bar: undefined, bar2: undefined, bar3: [Function: bar3] }  
> // a.js  

从上面的例子可以很直观地看出，a 模块引用了 b 模块，b 模块也引用了 a 模块，export 声明的变量也是优于模块其它内容的执行的，但是具体对变量赋值需要等到执行到相应代码的时候。

可能大家还会有疑问，为什么是先b.js先执行，而a.js是后执行的呢?  
这是因为es6 module在模块最后求值时，是按照深度优先倒序的规则来的。所以是先执行了b.js模块。




# 参考
- [深入理解 ES6 模块机制](https://zhuanlan.zhihu.com/p/33843378?group_id=947910338939686912)


# 留坑
- 如何理解CommonJS 运行时加载？


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
 