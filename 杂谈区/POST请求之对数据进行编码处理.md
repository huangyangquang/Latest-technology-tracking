# POST请求 之 对数据进行编码处理
<!-- TOC -->

- [POST请求 之 对数据进行编码处理](#post请求-之-对数据进行编码处理)
  - [URLSearchParams](#urlsearchparams)
    - [URLSearchParams 的读取和转换操作](#urlsearchparams-的读取和转换操作)
    - [url.searchParams](#urlsearchparams-1)
    - [让 URLSearchParams 作为Fetch的请求体(body)](#让-urlsearchparams-作为fetch的请求体body)
  - [FormData](#formdata)
    - [让 FormData 作为Fetch的请求体(body)](#让-formdata-作为fetch的请求体body)
    - [转换为 URLSearchParams](#转换为-urlsearchparams)
    - [将Fetch的body读取为 FormData](#将fetch的body读取为-formdata)
  - [其他可以作为Fetch的body的格式](#其他可以作为fetch的body的格式)
    - [Blobs](#blobs)
    - [Strings](#strings)
    - [Buffers](#buffers)
    - [Streams](#streams)
  - [最后的福利：将 FormData 转换为 JSON](#最后的福利将-formdata-转换为-json)
- [参考](#参考)
- [社交信息 / Social Links:](#社交信息--social-links)

<!-- /TOC -->
好,来。我们先来先来看个代码例子：

```
async function isPositive(text) {
  const response = await fetch(`http://text-processing.com/api/sentiment/`, {
    method: 'POST',
    body: `text=${text}`,
    headers: {
      'Content-Type': 'application/x-www-form-urlencoded',
    },
  });
  const json = await response.json();
  return json.label === 'pos';
}
```

这块代码写得比较糟糕，可能会导致安全问题。 为什么呢？因为：`text=${text}` 这块地方存在问题：

未转义的文本被添加到具有定义编码的格式中。就是说，这里的`text`变量，它是没有经过转义（或者说是编码）就直接被写到了请求体中，而在这个请求中，是有要求编码格式的`'Content-Type': 'application/x-www-form-urlencoded'`。

这种写法有点类似于 SQL/HTML 注入，因为某种旨在作为“值”的东西（指类似`text变量`的一些值）可以直接与格式进行交互。

所以，我将深入研究正确的方法，同时也会浏览一些相关的、鲜为人知的 API：

## URLSearchParams

URLSearchParams 可以用来 处理编码和解码 application/x-www-form-urlencoded 数据。 它非常方便，因为，嗯……

> The application/x-www-form-urlencoded format is in many ways an aberrant monstrosity, the result of many years of implementation accidents and compromises leading to a set of requirements necessary for interoperability, but in no way representing good design practices. In particular, readers are cautioned to pay close attention to the twisted details involving repeated (and in some cases nested) conversions between character encodings and byte sequences. Unfortunately the format is in widespread use due to the prevalence of HTML forms. — The URL standard

......所以,是的，非常不建议你自己 对 `application/x-www-form-urlencoded 的数据`进行编码/解码。 

下面是URLSearchParams的工作原理：
```
const searchParams = new URLSearchParams();
searchParams.set('foo', 'bar');
searchParams.set('hello', 'world');

// Logs 'foo=bar&hello=world'
console.log(searchParams.toString());
```

URLSearchParams这个构造函数还可以接受一个[key, value]对的数组，或一个产生[key, value]对的迭代器：
```
const searchParams = new URLSearchParams([
  ['foo', 'bar'],
  ['hello', 'world'],
]);

// Logs 'foo=bar&hello=world'
console.log(searchParams.toString());
```

或者是一个对象：
```
const searchParams = new URLSearchParams({
  foo: 'bar',
  hello: 'world',
});

// Logs 'foo=bar&hello=world'
console.log(searchParams.toString());
```

或者是一个字符串：
```
const searchParams = new URLSearchParams('foo=bar&hello=world');

// Logs 'foo=bar&hello=world'
console.log(searchParams.toString());
```

### URLSearchParams 的读取和转换操作

读取（指对数据进行枚举等读取操作）和转换（指将其转为数组或者对象等） URLSearchParams 的方法还是很多的，[MDN](https://developer.mozilla.org/en-US/docs/Web/API/URLSearchParams) 上都有详细说明。

如果在某些场景下，您想处理所有数据，那么它的迭代器就派上用场了：
```
const searchParams = new URLSearchParams('foo=bar&hello=world');

for (const [key, value] of searchParams) {
  console.log(key, value);
}
```

这同时意味着您可以轻松地将其转换为[key, value]对数组：
```
// To [['foo', 'bar'], ['hello', 'world']]
const keyValuePairs = [...searchParams];
```

或者将它与支持生成key-value对的迭代器的 API 一起使用，例如 Object.fromEntries，可以把它转换为一个对象：
```
// To { foo: 'bar', hello: 'world' }
const data = Object.fromEntries(searchParams);
```

但是，请注意，转换为对象有时是有损转换的哦：就是可能会造成某些值得丢失
```
const searchParams = new URLSearchParams([
  ['foo', 'bar'],
  ['foo', 'hello'],
]);

// Logs "foo=bar&foo=hello"
console.log(searchParams.toString());

// To { foo: 'hello' }
const data = Object.fromEntries(searchParams);
```

### url.searchParams
URL 对象上有一个 searchParams 属性，非常方便地获取到请求参数：
```
const url = new URL('https://jakearchibald.com/?foo=bar&hello=world');

// Logs 'world'
console.log(url.searchParams.get('hello'));
```

不幸的是，在`window.location`上没有`location.searchParams`这个属性。

 这是因为 window.location 由于它的某些属性如何跨源工作而变得复杂。 例如设置 otherWindow.location.href 可以跨源工作，但不允许获取它。 
 
 但是无论如何，我们都要解决它，让我们能比较容易地从地址栏中获取到请求参数：
```
// Boo, undefined
location.searchParams;

const url = new URL(location.href);
// Yay, defined!
url.searchParams;

// Or:
const searchParams = new URLSearchParams(location.search);
```

### 让 URLSearchParams 作为Fetch的请求体(body)

好的，现在我们进入正题。 文章开头示例中的代码存在一些问题，因为它没有进行转义输入：
```
const value = 'hello&world';
const badEncoding = `text=${value}`;

// 😬 Logs [['text', 'hello'], ['world', '']]
console.log([...new URLSearchParams(badEncoding)]);

const correctEncoding = new URLSearchParams({ text: value });

// Logs 'text=hello%26world'
console.log(correctEncoding.toString());
```

为方便使用，`URLSearchParams对象` 是可以直接被用作`请求Request` 或 `响应Response` 的 `主体body`，因此文章开头的“正确”代码版本是：
```
async function isPositive(text) {
  const response = await fetch(`http://text-processing.com/api/sentiment/`, {
    method: 'POST',
    body: new URLSearchParams({ text }),
  });
  const json = await response.json();
  return json.label === 'pos';
}
```

如果使用 URLSearchParams 作为`body`，则 `Content-Type` 字段会**自动设置**为 `application/x-www-form-urlencoded`

您不能将`请求Request` 或 `响应Response` 的 `主体body`读取为 URLSearchParams对象，但我们有一些方法可以解决这个问题……

## FormData

FormData 对象可以表示 HTML 表单的一组key-value的数据。 同时key值也可以是文件，就像 `<input type="file">` 一样。

您可以直接给 FormData 对象 添加数据：
```
const formData = new FormData();
formData.set('foo', 'bar');
formData.set('hello', 'world');
```

FormData对象也是一个迭代器，因此您可以将它转换为键值对数组或对象，就像使用 URLSearchParams 一样。但是，与 URLSearchParams 不同的是，您可以将 HTML 表单直接读取为 FormData：

```
const formElement = document.querySelector('form');
const formData = new FormData(formElement);
console.log(formData.get('username'));
```
这样，您就可以轻松地从表单中获取到数据了。 我经常使用这种方式，所以发现这比单独从每个元素中获取数据要容易得多。

### 让 FormData 作为Fetch的请求体(body)

与 URLSearchParams 类似，您可以直接使用 FormData 作为 fetch body：
```
const formData = new FormData();
formData.set('foo', 'bar');
formData.set('hello', 'world');

fetch(url, {
  method: 'POST',
  body: formData,
});
```

这会**自动**将 Content-Type 标头设置为 `multipart/form-data`，并以这种格式发送数据：

```
const formData = new FormData();
formData.set('foo', 'bar');
formData.set('hello', 'world');

const request = new Request('', { method: 'POST', body: formData });
console.log(await request.text());
```

...console.log打印出如下内容：

```
------WebKitFormBoundaryUekOXqmLphEavsu5
Content-Disposition: form-data; name="foo"

bar
------WebKitFormBoundaryUekOXqmLphEavsu5
Content-Disposition: form-data; name="hello"

world
------WebKitFormBoundaryUekOXqmLphEavsu5--
```

这就是使用 `multipart/form-data 格式`发送数据时body的样子。 它比 application/x-www-form-urlencoded 更复杂，但它可以包含`文件数据`。但是，某些服务器无法处理`multipart/form-data `，比如：Express。 如果你想在 Express 中支持 multipart/form-data，你需要使用一些库来帮忙了比如： [busboy](https://www.npmjs.com/package/busboy) 或 [formidable](https://www.npmjs.com/package/formidable) 


但是，如果您想将`表单`作为 `application/x-www-form-urlencoded` 发送怎么办？ 嗯…

### 转换为 URLSearchParams

因为 URLSearchParams 构造函数可以接受一个生成键值对的迭代器，而 FormData 的迭代器正是这样做的，它可以生成键值对，因此您可以将 FormData 转换为 URLSearchParams：

```
const formElement = document.querySelector('form');
const formData = new FormData(formElement);
const searchParams = new URLSearchParams(formData);

fetch(url, {
  method: 'POST',
  body: searchParams,
});
```

但是，如果表单数据包含文件数据，则此转换过程将抛出错误。 因为`application/x-www-form-urlencoded` 不能表示`文件数据`，所以 URLSearchParams 也不能。

### 将Fetch的body读取为 FormData

您还可以将 Request 或 Response 对象读取为 FormData：
```
const formData = await request.formData();
```

如果Request 或 Response的body是 `multipart/form-data` 或 `application/x-www-form-urlencoded`，这个方法是很有效。 它对于服务器中处理表单提交特别有用。

## 其他可以作为Fetch的body的格式

还有一些其他`格式format`可以作为Fetch的body：

### Blobs
Blob 对象（同时，File也可以作为Fetch的body， 因为它继承自 Blob）可以作为Fetch的body：

```
fetch(url, {
  method: 'POST',
  body: blob,
});
```

这会自动将 Content-Type 设置为 blob.type 的值。

### Strings

```
fetch(url, {
  method: 'POST',
  body: JSON.stringify({ hello: 'world' }),
  headers: { 'Content-Type': 'application/json' },
});
```

这会**自动**将 Content-Type 设置为 text/plain;charset=UTF-8，但它可以被覆盖，就像我上面所做的那样,将 Content-Type 设置为 `application/json`

### Buffers

ArrayBuffer 对象，以及由数组缓冲区支持的任何东西，例如 Uint8Array，都可以用作Fetch的body:

```
fetch(url, {
  method: 'POST',
  body: new Uint8Array([
    // …
  ]),
  headers: { 'Content-Type': 'image/png' },
});
```

这不会自动设置 Content-Type 字段，因此您需要自己进行设置。

### Streams
最后，获取主体可以是流（stream）！ 对于 Response 对象，这可以[让服务端获取不一样的开发体验](https://jakearchibald.com/2016/streams-ftw/)，而且[它们也可以与request一起使用](https://web.dev/fetch-upload-streaming/)。


所以，千万不要尝试自己处理 `multipart/form-data` 或 `application/x-www-form-urlencoded` 格式的数据，让 FormData 和 URLSearchParams 来帮我们完成这项艰苦的工作！

## 最后的福利：将 FormData 转换为 JSON

目前有个问题，就是：

如何将 FormData 序列化为 JSON 而不会丢失数据？

表单可以包含这样的字段：
```
<select multiple name="tvShows">
  <option>Motherland</option>
  <option>Taskmaster</option>
  …
</select>
```

当然，您可以选择多个值，或者您可以有多个具有相同名称的输入：
```
<fieldset>
  <legend>TV Shows</legend>
  <label>
    <input type="checkbox" name="tvShows" value="Motherland" />
    Motherland
  </label>
  <label>
    <input type="checkbox" name="tvShows" value="Taskmaster" />
    Taskmaster
  </label>
  …
</fieldset>
```

最后获取到数据的结果是一个`具有多个同名字段的 FormData 对象`，如下所示：
```
const formData = new FormData();
formData.append('foo', 'bar');
formData.append('tvShows', 'Motherland');
formData.append('tvShows', 'Taskmaster');
```

就像我们在 URLSearchParams 中看到的，一些对象的转换是有损的（部分属性是会被剔除丢的）：
```
// { foo: 'bar', tvShows: 'Taskmaster' }
const data = Object.fromEntries(formData);
```

有以下几种方法可以避免数据丢失，而且最终仍然可以将fromData数据序列化 JSON。 

首先，转为[key, value]对数组：
```
// [['foo', 'bar'], ['tvShows', 'Motherland'], ['tvShows', 'Taskmaster']]
const data = [...formData];
```

但是如果你想要转为一个对象而不是一个数组，你可以这样做：
```
const data = Object.fromEntries(
  // Get a de-duped set of keys
  [...new Set(formData.keys())]
    // Map to [key, arrayOfValues]
    .map((key) => [key, formData.getAll(key)]),
);
```

...上诉代码的data变量，最终是：

```
{
  "foo": ["bar"],
  "tvShows": ["Motherland", "Taskmaster"]
}
```

我比较倾向于数据中每个值都是一个数组，即使它只有一个项目。 因为这可以防止服务器上的大量代码分支，并可以简化验证。 虽然，您有可能更倾向于 PHP/Perl 约定，其中以 [] 结尾的字段名称表示“这应该生成一个数组“， 如下：

```
<select multiple name="tvShows[]">
  …
</select>
```

并我们来转换它：

```
const data = Object.fromEntries(
  // Get a de-duped set of keys
  [...new Set(formData.keys())].map((key) =>
    key.endsWith('[]')
      ? // Remove [] from the end and get an array of values
        [key.slice(0, -2), formData.getAll(key)]
      : // Use the key as-is and get a single value
        [key, formData.get(key)],
  ),
);
```

...上诉代码的data变量，最终是：

```
{
  "foo": "bar",
  "tvShows": ["Motherland", "Taskmaster"]
}
```

> 注意：如果form表单中包含文件数据，请不要尝试将表单转换为 JSON。 如果是这种`form表单中包含文件数据`的情况，那么使用 `multipart/form-data` 会好得多。


# 参考
- [Encoding data for POST requests](https://jakearchibald.com/2021/encoding-data-for-post-requests/?utm_source=ESnextNews.com&utm_medium=Weekly+Newsletter&utm_campaign=2021-07-06)

# 社交信息 / Social Links:

(Welcome to pay attention, 欢迎关注)

Github：
[@huangyangquang](https://github.com/huangyangquang) | [最新技术追踪](https://github.com/huangyangquang/Latest-technology-tracking) | [javascript版算法](https://github.com/huangyangquang/Algorithm) | [早期前端知识总结 + 案例](https://github.com/huangyangquang/DEMO) | 欢迎Star✨✨✨


Social：
[新浪微博](https://weibo.com/u/6385661354) | [知乎](https://www.zhihu.com/people/cclv3) | [掘金](https://juejin.cn/user/2735240661699181) | [思否](https://segmentfault.com/u/c_z7wgq/articles) 

E-mail： fengquan.h@qq.com  

Old Blog：[CSDN](https://blog.csdn.net/huangyangquan3?type=blog)

微信公众号：前端学长Joshua  

![微信公众号](../static/img/wechatQrCode.jpg)