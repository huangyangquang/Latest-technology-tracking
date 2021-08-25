# 技术篇 - vue2.0与vue3.0之间创建数据的差异

这篇文章，将为你说明如何使用vue2的 options api 来创建数据，以及如何使用vue3的 composition api 来创建数据

同时，你将学习如何使用包装对象 ref 和 reactive 创建数据，何时使用它们以及为什么使用它们。

最后，你将了解到使用vue2的 options api 和 vue3的 composition api之间创建数据的差异

## options api
### 使用vue2的 options api 定义一个数据
在 Vue 2 中引入的 Options API 中，声明数据的方法之一，就是将它们添加到 `data() 函数返回的 JavaScript 对象`中。
```
// 创建一个名为name的数据，初始值为null
export default {
  data() {
    return {
      name: null
    }
  }
}
```
当我们使用options apic创建一个数据时，这个数据默认就是响应式的

> 数据响应式是什么？
> name 数据是响应式的，这意味着它可以绑定到模板中的 HTML 元素。
>
> 每当数据值发生变化时，视图就会更新，反之亦然，这也称为双向数据绑定。

### 在vue2中Javascript区域访问一个数据
假如我们要在此组件`的默认导出对象`内的任何位置，访问 name 数据，我们可以在 Options API 中使用`this`关键字。
```
// 在生命周期函数 mounted 中访问数据name
export default {
  ...
  mounted() {
    console.log(this.name);
  }
}
```

### 在vue2中Template区域访问一个数据
每当我们使用 Options API 创建数据时，数据不仅是响应式的，而且可以在 Vue 模板使用。

我们可以简单地使用双花括号来访问模板标签之间的 name 属性 --- {{name}}
```
<template>
  <div>
    {{name}}
  </div>
</template>
```
在 Vue 2 中，在模板template内，只允许有一个父元素：
```
<template>
  <div>
    {{name}}
  </div>
</template>
```
在vue2中，这是不允许的:
```
<template>
  <div>
    {{name}}
  </div>
  <div>
    {{name}}
  </div>
</template>

```
应该改为这样：
```
<template>
  <div> // 一个父元素
    <div>
        {{name}}
    </div>
    <div>
        {{name}}
    </div>
  </div>
</template>

```

## composition api（vue3）
Vue 3 的一大优点是我们可以直接使用 Options API 来创建响应式数据，就像上面的vue2例子一样。

此外，我们现在可以使用 Composition API, 来创建非常灵活的数据

在 Vue 3 中有两种创建响应式数据的方法，它们是：
- ref
- reactive

### ref()
在vue3中，我需要什么功能函数，就只需要导入对应地包既可
```
<script>
  import { ref } from "vue";
  export default {
     setup() {
       // 你的所有 数据 和 函数 都将在存在于steup函数中
     }
  } 
</script>
```
使用 Composition API，所有数据和函数都将包含在默认导出对象的 setup() 方法中：
```
<script>
  import { ref } from "vue";
  export default {
     setup() {
       let name = ref("");
     }
  } 
</script>
```
我们可以看见，name 变量的值是一个用 `ref 对象包裹的空字符串`

那么，ref()是什么呢？

ref 是一个包装对象，它接受一个内部值，并返回一个响应式和可变ref对象。

比如：变量 name 的内部值就是 空字符串`''`, 我们可以通过ref对象的 `.value` 属性来获取到这个内部值 


#### 在 JavaScript 中访问 Ref() 变量
要获取与 name 变量关联的值，我们只需要访问它的 `.value 属性`

ref() 对将有一个名为 `.value 的属性`，指向`内部值`
```
<script>
  import { ref } from "vue";
  export default {
     setup() {
       let name = ref("Raja");
       name.value = Raja Tamil; // Set
       console.log(name.value) // Get
     }
  } 
</script>
```
当然，如果你想要设置name变量的值(内部值)，也是需要通过`.value`来设置


在vue2中，使用 Options API，所有数据在创建后立即在模板中可用。  

但是，在vue3中，使用composition api，我们可以选择将数据和函数显式暴露给模板，而不是将所有的数据和函数都暴露给模板

这也就意味着，我们可以创建一个只在setup函数中使用的变量

现在，我们只需要将`name变量`，作为setup函数的return值中的一个属性暴露出去即可
```
<script>
  import { ref } from "vue";
  export default {
     setup() {
       let name = ref("Raja");
       return {
          username: name
       }
     }
 } 
</script>
```


#### 在vue3中Template区域访问一个数据
和vue2一样，我们只是需要通过`{{name}}`，就可以在模板中访问到响应式数据name了
```
<template>
   {{name}}
</template>
```

> 在vue3中，不在需要一个父级的节点来包裹所有的元素了  
现在我们可以在模板标签中拥有兄弟姐妹的 div 元素：
```
<template>
    <div> {{name}} </div>
    <div> {{name}} </div>
    <div> {{name}} </div>
</template>
```


### Reactive()
在composition api中，创建一个响应式数据的另外一种方式：`reactive()`

可能你有疑问，为什么在vue3中，会有两种创建响应式数据的方式呢？  
- ref() 是用于原始值的，比如：字符串，数字等  
- Reactive() 是用于对象的，它接受一个对象并返回原始对象的`proxy代理`。

而实际上，ref也是可以用于对象的，源码如下：
```
// 简单看下：在源码中，会判断传递的给ref() 的参数是不是一个对象。如果是对象，就使用reactive()
const convert = (val) => shared.isObject(val) ? reactive(val) : val;
```

下面，我们看看怎么使用 reactive:  
与 ref 类似，使用reactive, 并将 Javascript 对象作为初始值传递给 reactive() 作为参数。
```
<script>
  import { reactive } from "vue";
  export default {
     setup() {
       let book = reactive({title: "The Best Vue 3 Book", price:19.99});
     }
 } 
</script>
```


#### 从 Reactive() 变量中获取值
我们可以`直接通过变量book,就可以访问到属性`，而不需要像ref,需要通过.value：
```
<script>
  import { reactive } from "vue";
  export default {
     setup() {
       let book = reactive({title: "The Best Vue 3 Book", price:19.99});
       console.log(book.title);
       console.log(book.price);
     }
 } 
</script>
```


#### 在vue3中Template区域访问一个数据
与前面的示例类似，我们需要做的就是将此变量,作为属性添加到 setup() 函数返回的 JavaScript 对象中
```
<script>
  import { reactive } from "vue";
  export default {
     setup() {
       let book = reactive({title: "The Best Vue 3 Book", price:19.99});
       return {
         book
       }
     }
 } 
</script>
```

在模板中访问这个属性时，和往常一样，也是使用`{{}}`:
```
<template>
  {{book.title}}
  {{book.price}}
</template>
```

我们可以来测试下响应式的效果，我们设置一个定时器，在2秒之后，改变对象book,上的属性：
```
<script>
  import { reactive } from "vue";
  export default {
     setup() {
       let book = reactive({title: "Vue 3 Book", price:29.99});
        
       setTimeout(() => {
          book.title = "Vue 3 is awesome";
          book.price = 19.99;
       }, 2000)
       
       return {
         book
       }
     }
 } 
</script>
```
在2秒之后，我们可以发现视图上的值发生了变化，这样就可以轻易查看响应式的效果了

## 没有响应式的数据
vue3的另外一个好处，当我们有需要是，可以创建一个私有的，且非响应式的数据：
```
<script>
  import { reactive } from "vue";
  export default {
     setup() {
       let book = {title: "Vue 3 Book", price:29.99};
       return {
         book
       }
     }
 } 
</script>
```

## 小结
- 响应式  
  在vue2 (options api)，所有作为 data()函数 的返回值的数据都是具备响应式的  
  在vue3 (composition api)，数据可以被声明为响应式，也可以不是响应式。对于响应式数据，可以使用reactive 或者 ref 来进行创建
- 模板 (template)  
  在vue2 (options api)，最外层必须要有一个父节点  
  在vue3 (composition api)，最外层可以不需要父节点进行包裹








