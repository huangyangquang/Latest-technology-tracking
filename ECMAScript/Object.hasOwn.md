# Object.hasOwn 替换掉 Object.prototype.hasOwnProperty
2021 年 6 月 25 日发布 · 标记为ECMAScript

建议使用 Object.hasOwn() 方法，因为它使得 Object.prototype.hasOwnProperty() 更易于使用。

## 阶段
目前这个提案还在第三阶段

## Object.hasOwn提案为什么会出现呢？
目前，这样的代码是很常见：
```
const hasOwnProperty = Object.prototype.hasOwnProperty;

if (hasOwnProperty.call(object, 'foo')) {
  // `object` has property `foo`.
}
```

或者有些库使得使用 Object.prototype.hasOwnProperty 更简单：  
[npm: has](https://www.npmjs.com/package/has)  
[npm: lodash.has](https://www.npmjs.com/package/lodash.has)  
[See Related](https://github.com/tc39/proposal-accessible-object-hasownproperty#related)

使用 Object.hasOwn 这个新函数，我们可以将上边代码简写为：
```
if (Object.hasOwn(object, 'foo')) {
  // `object` has property `foo`.
}
```

在日常开发中，有某些常见的用法，使得 Object.prototype 上的方法有时可能不可用或是会被重新定义了。

比如：
### 1. Object.create(null)：
Object.create(null) 会创建一个不从 Object.prototype 继承的对象，这使得Object.prototype 上的方法无法访问。
```
Object.create(null).hasOwnProperty("foo")
// Uncaught TypeError: Object.create(...).hasOwnProperty is not a function
```

### 2. 重新定义 hasOwnProperty：
如果你对对象的内置属性进行了重新赋值改写，那么你在调用某个属性（比如：.hasOwnProperty）时，肯定调用的不是对象的内置属性
```
let object = {
  hasOwnProperty() {
    throw new Error("gotcha!")
  }
}

object.hasOwnProperty("foo")
// Uncaught Error: gotcha!
```

## 3. ESLint no-prototype-builtins
在ESLint的[规则 built-in rule](https://eslint.org/docs/rules/no-prototype-builtins) 中，是禁止直接使用 Object.prototypes 内置函数

ESLint 文档中关于 no-prototype-builtins 说明:

对这一规则的错误例子：
```
/*eslint no-prototype-builtins: "error"*/
var hasBarProperty = foo.hasOwnProperty("bar");
...
```

对这一规则的正确例子：
```
/*eslint no-prototype-builtins: "error"*/
var hasBarProperty = Object.prototype.hasOwnProperty.call(foo, "bar");
...
```

## 提案
该提案添加了一个 Object.hasOwn(object, property) 方法，其行为与调用 hasOwnProperty.call(object, property) 相同
```
let object = { foo: false }
Object.hasOwn(object, "foo") // true

let object2 = Object.create({ foo: true })
Object.hasOwn(object2, "foo") // false

let object3 = Object.create(null)
Object.hasOwn(object3, "foo") // false
```

## 为什么不是 使用 Object.hasOwnProperty(object, property)？
Object.hasOwnProperty(property) 今天已经存在，因为 Object 本身继承自 Object.prototype，所以定义一个具有不同名称的新方法将是一个比较明显的变化。

## 为什么命名为 hasOwn 呢？
See [Issue #3](https://github.com/tc39/proposal-accessible-object-hasownproperty)

> Object.hasOwn 已经在 V8 v9.3 中，执行时在后边加上 --harmony-object-has-own 的标志就可以使用，Chrome 中不久之后也会推出它。

# 参考
- [Accessible Object.prototype.hasOwnProperty()](https://github.com/tc39/proposal-accessible-object-hasownproperty)

# 社交信息 / Social Links:
 ## (Welcome to pay attention, 欢迎关注)
Github：
[@huangyangquang](https://github.com/huangyangquang) | [最新技术追踪](https://github.com/huangyangquang/Latest-technology-tracking) | [javascript版算法](https://github.com/huangyangquang/Algorithm) | [早期前端知识总结 + 案例](https://github.com/huangyangquang/DEMO) | 欢迎Star✨✨✨


Social：
[新浪微博](https://weibo.com/u/6385661354) | [知乎](https://www.zhihu.com/people/cclv3) | [掘金](https://juejin.cn/user/2735240661699181) | [思否](https://segmentfault.com/u/c_z7wgq/articles) 

E-mail： fengquan.h@qq.com  

Old Blog：[CSDN](https://blog.csdn.net/huangyangquan3?type=blog)

微信公众号：前端学长Joshua  

<img src="../static/img/wechatQrCode.jpg" width="50%">
 