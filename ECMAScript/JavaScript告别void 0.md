# 现代JavaScript告别void 0


在JavaScript代码中，尤其是在一些比较老旧的项目代码中，你有时候可能会遇见这样的一个代码片 `void 0`

`void 0`操作，本质上是一个计算表达式，而且他最后会返回一个`undefined`这个原始值。`void 0`就是计算0这个值，实际上它没有进行任何操作，执行完之后直接返回`undefined`出来。所以，它实际上就是`undefined`的另外一种表达方式。

## 为什么`void 0`是`undefined`的另外一种表达方式呢？
我们先要知道，JavaScript被设计出来的时候，是比较匆忙的，所以有一些不太合理的地方。其中的一个不太合理的地方就是：`undefined`不是保留的关键字。相反，`undefined`只是全局对象window上的一个属性

**在ES5之前，全局对象window上的undefined属性是可以被修改的，因为可以被修改，就很容易造成一些不可以预知的问题**

我们可以看下以下的一个例子（tip：这个例子在现代JS引擎是不会生效的）：
```
// 在ES5之前的一个例子 - 这个例子在现代JS引擎是不会生效的:

// 在全局对象上，修改undefined属性:
undefined = "something else"; 

// 那么，可能会有一个潜在的问题就是，在其他的JS文件中，会做如下的判断:
var aVariable = 'something else'
if (aVariable === undefined) {
  // aVariable 这个变量的值为 "something else", 并不是原始值 'undefined'
  // 所以，代码 aVariable === undefined 会被判断为真
  doSomething();
}
```

也有另外的一些情况是，有些第三方包会直接修改全局对象上undefined这个属性。同时，我们在使用这个第三方包时，是通过script标签引入的，所以，这样也会造成我们的代码出现不可控制🙅的问题。

基于以上全局对象上undefined可以被任意修改的原因，同时又由于`void 0`操作，执行结束之后会返回一个`undefined`这个原始值。所以，在ES5语法之前的项目中，使用`void 0`操作是一个比较常见的代码行为。

## 在ES5之后，全局对象属性undefined的情况
可以任意修改全局对象window上的undefined属性，这个问题实在太大了，导致最后需要修改JavaScrit标准了。在ES5中，明确表示全局对象window上的undefined属性是一个只读属性，它不能被修改。
```
window.undefined = "something else";
console.log(undefined); // 在现代JS执行环境中，打印出
```

但是，这种修改似乎是无济于事的。为什么呢？
虽然，`undefined`作为一个全局属性不能被修改了，但是在JS标准中，它仍然不是一个关键字。所以，当`undefined`作为一个本地变量时，任然是可以被修改的。
```
const undefined = "something else";
let check = aVariable === void 0; // void 0 is needed here
```
将undefined作为一个本地变量是不规范的，所以，这个是你需要避免的写法。

在[eslint](https://eslint.org/docs/rules/no-undefined)中，是不允许将undefined作为一个变量名的。

## 在现代JS中，避免使用void 0
总而言之，在现代浏览器和JS引擎中，是没有必要继续使用void 0的：
1. 在ES5语法之后，全局对象上undefined不可以被任意修改的
2. 在在[eslint](https://eslint.org/docs/rules/no-undefined)中，是不允许将undefined作为一个变量名的

总之，void 0使得JS代码比较难以理解，因为你需要知道`void 0`实际上就是`undefined`的另外一种表达方式。


# 社交信息 / Social Links:
 ## (Welcome to pay attention, 欢迎关注)
Github：
[@huangyangquang](https://github.com/huangyangquang) | [最新技术追踪](https://github.com/huangyangquang/Latest-technology-tracking) | [javascript版算法](https://github.com/huangyangquang/Algorithm) | [早期前端知识总结 + 案例](https://github.com/huangyangquang/DEMO) | 欢迎Star✨✨✨


Social：
[新浪微博](https://weibo.com/u/6385661354) | [知乎](https://www.zhihu.com/people/cclv3) | [掘金](https://juejin.cn/user/2735240661699181) | [思否](https://segmentfault.com/u/c_z7wgq/articles) 

E-mail： fengquan.h@qq.com  

Old Blog：[CSDN](https://blog.csdn.net/huangyangquan3?type=blog)

微信公众号：前端学长Joshua  

<img src="  ../static/img/wechatQrCode.jpg" width="50%">
 