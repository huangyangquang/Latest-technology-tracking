# 并行执行多个独立的异步操作 - Promise.allSettled()

## 介绍
Promise.allSettled(promises) 是一个辅助函数，它并行运行 Promise, 并将已改变的状态（已完成或已拒绝）存放到一个数组中，返回给开发者使用。

当您有多个彼此不依赖（或者是说相互独立的）的异步任务，您想知道每一个promise任务的执行结果的时候（无论是已完成或已拒绝），通常使用它。

让我们看看 Promise.allSettled() 是如何使用的。


## 如何使用Promise.allSettled()

### 1.Promise.allSettled()
Promise.allSettled() 可用于**并行**执行**独立**的**异步**操作，并收集这些异步操作完成的结果。

该函数接受一个 Promise 数组（或通常是一个可迭代的）作为参数：
```
const statusesPromise = Promise.allSettled(promises);
```

当我们传入的参数`promises数组`中，每一项发生了状态的改变（无论是已完成或已拒绝）之后，`Promise.allSettled`都会将`promises数组`中每一项的执行结果，给我们存放在一个数组中，返回给开发者使用。

返回的数组`statusesPromise`中每一项的的格式如下：
1. `{ status: 'fulfilled', value: value }` — 如果对应的 promise 已完成  
   或者 
2. `{ status: 'rejected', reason: reason }` — 如果相应的 promise 已拒绝
![all-settled](./static/img/all-settled.svg)

当所有的promise都被resolved 或是 被rejected后，你可以使用.then对他们进行后续的操作：
```
statusesPromise.then(statuses => {
 statuses; // [{ status: '...', value: '...' }, ...]
});
```
或者使用 async/await 对他们进行后续操作:
```
const statuses = await statusesPromise;

statuses; // [{ status: '...', value: '...' }, ...]
```

### 2. example: 简单的使用Promise.allSettled()

在了解 Promise.allSettle() 之前，让我们定义 2 个简单的辅助函数:

首先，resolveTimeout(value, delay) —— 返回一个promise, 这个promise会在一段时间之后，触发resolve函数：
```
function resolveTimeout(value, delay) {
  return new Promise(
    resolve => setTimeout(() => resolve(value), delay)
  );
}
```

其次，rejectTimeout(reason, delay)——返回一个promise,这个promise会在一段时间之后，触发reject函数：
```
function rejectTimeout(reason, delay) {
  return new Promise(
    (r, reject) => setTimeout(() => reject(reason), delay)
  );
}
```
我们就使用这两个辅助函数来试验下 Promise.allSettled() 的功能吧。

#### 2.1. 所有promise已完成
下面，我会去同时执行一些异步的操作：
```
const statusesPromise = Promise.allSettled([
  resolveTimeout(['potatoes', 'tomatoes'], 1000),
  resolveTimeout(['oranges', 'apples'], 1000)
]);

// wait...
const statuses = await statusesPromise;

// after 1 second
console.log(statuses); 
// [
//   { status: 'fulfilled', value: ['potatoes', 'tomatoes'] },
//   { status: 'fulfilled', value: ['oranges', 'apples'] }
// ]
```
Promise.allSettled([...]) 返回一个数组 statusesPromise，两个异步操作都是并行执行的。  
`statusesPromise` 是 `Promise.allSettled([...])` 返回的一个数组，数组中的每一项都包含了对应的异步操作执行完成之后的状态：
1. 数组`statusesPromise`的第一项包含了完成状态：{ status: 'fulfilled', value: ['potatoes', 'tomatoes'] }  
同样，
3. 数组`statusesPromise`的第二项包含了完成状态：{ status: 'fulfilled', value: ['oranges', 'apples'] }。

#### 2.2 某个promise已拒绝
如果在多个并行独立的异步操作中，有一个异步操作是已拒绝的，那么`Promise.allSettled([...])`会怎么处理呢？
```
const statusesPromise = Promise.allSettled([
  resolveTimeout(['potatoes', 'tomatoes'], 1000),
  rejectTimeout(new Error('Out of fruits!'), 1000)
]);

// wait...
const statuses = await statusesPromise;

// after 1 second
console.log(statuses); 
// [
//   { status: 'fulfilled', value: ['potatoes', 'tomatoes'] },
//   { status: 'rejected', reason: Error('Out of fruits!') }
// ]
```
我看下结果，我们会很快明白：
1. 第一项由于触发了resolve函数，promise是已完成，数组的第一项是 { status: 'fulfilled', value: ['potatoes', 'tomatoes'] }

2. 第二项由于是触发reject函数，promise是已拒绝，数组的第二项状态被修改为rejected状态：{ status: 'rejected', reason: Error('Out of Fruits') }。

即使是有某一项是转变为rejected状态, 但是最后`Promise.allSettled`函数是会包含这一项，就它存放在返回值中。

#### 2.3 所有promise已拒绝
我们看下如下代码：
```
const statusesPromise = Promise.allSettled([
  rejectTimeout(new Error('Out of vegetables!'), 1000),
  rejectTimeout(new Error('Out of fruits!'), 1000)
]);

// wait...
const statuses = await statusesPromise;

// after 1 second
console.log(statuses); 
// [
//   { status: 'rejected', reason: Error('Out of vegetables!')  },
//   { status: 'rejected', reason: Error('Out of fruits!') }
// ]
```
在这种情况下， `Promise.allSettled`仍然会返回一个数组。 只是，该数组中每一项的状态都是rejected状态。

## 小结:
`Promise.allSettled(promises)` 允许您**并行**执行**互相独立**的promise, 并每一个promise执行之后的状态（已完成或拒绝）存放在数组中。

当您需要执行并行和独立的异步操作并获取到异步操作的所有结果时，Promise.allSettled(...) 非常有用，即使某些异步操作可能会失败。

## 兼容性：
参考： [兼容性](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled#%E6%B5%8F%E8%A7%88%E5%99%A8%E5%85%BC%E5%AE%B9) 很好


## 参考
- [TC39：Promise.allSettled](https://tc39.es/proposal-promise-allSettled/)
- [MDN: Promise.allSettled()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise/allSettled)


# 社交信息 / Social Links:
 ## (Welcome to pay attention, 欢迎关注)
Github：
[@huangyangquang](https://github.com/huangyangquang) | [最新技术追踪](https://github.com/huangyangquang/Latest-technology-tracking) | [javascript版算法](https://github.com/huangyangquang/Algorithm) | [早期前端知识总结 + 案例](https://github.com/huangyangquang/DEMO) | 欢迎Star✨✨✨


Social：
[新浪微博](https://weibo.com/u/6385661354) | [知乎](https://www.zhihu.com/people/cclv3) | [掘金](https://juejin.cn/user/2735240661699181) | [思否](https://segmentfault.com/u/c_z7wgq/articles) 

E-mail： fengquan.h@qq.com  

Old Blog：[CSDN](https://blog.csdn.net/huangyangquan3?type=blog)

微信公众号：前端学长Joshua  
<img src="http://qvf3q8r5e.hn-bkt.clouddn.com/wechatQrCode.jpg" width="50%">
 