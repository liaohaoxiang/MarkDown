---
title: 'Vue响应式原理'

date: '2021-03-10'

kind: 'Tech'
---



### 从Object.defineProperty开始

> Object.defineProperty() 方法会直接在一个对象上定义一个新属性，或者修改一个对象的现有属性，并返回此对象。—— 
>
> [MDN]: https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty	"MDN"

根据 

[Vue官网]: https://cn.vuejs.org/v2/guide/reactivity.html

的解释, Vue是根据```Object.defineProperty``` 把 属性全部转化为 ```getter/setter``` ,这个API是ES5的```shim``` 特性,也因此不可用于IE8及更低的版本



一句话,这个API可以将一个 对象 的 ```key值``` 动态的加入,并设置它key值的属性

```js
const obj = {};

Object.defineProperty(obj, "mykey", {
  enumerable: false,
  configurable: false,
  writable: false,
  value: "static"
});

console.log(obj.mykey); // "static"
```

值得注意的是这四个属性

- enumerable

  可枚举性, **默认false** ,常用场景: ```for … in``` ,```Object.keys``` 都是需要可枚举的属性

- configurable

  可配置性,**默认false** ,当值为true时 才可**改变属性的描述符(即是否可改变除 value 和 writable 特性外的其他特性)** 和 **删除该属性**

- writable

  可写性,**默认false**, 当值为true时 才可以**赋值**

- value

  值, **默认undefined** ,可以是**任意**的JavaScript值

  

> 注意: 如果一个描述符同时拥有 ```value``` 或 ```writable``` 和 ```get``` 或 ```set``` 键，则会产生一个异常。



### 重点是getter/setter

上面是这个API的四个描述符介绍, Vue却是 通过改变 get/set 描述符去实现的响应式.

```js
const obj = {};

Object.defineProperty(obj, "mykey", {
  enumerable: false,
  configurable: false,
  // 使用了方法名称缩写（ES2015 特性）
  // 下面两个缩写等价于：
  // get : function() { ... },
  // set : function(newValue) { ... },
  get(){
		console.log('I get it')
	},
  set(value) {
		console.log(`I set in ${value}`)
	},
});
obj.mykey = 'string' // "I set in string"
console.log(obj.mykey); // "I get it"
```



Vue组件实例其实是一个 **watch** 实例,它会把```data```里的数据都改写```getter/setter``` ,通过 ```watch``` 的监听,每当对象改变属性的值,就会触发setter方法,通知到watcher上,触发re-render,从而更新view; 当对象被放在data时, getter方法也会**收集作为依赖** ,反馈到watcher中等待使用.



### 局限性和作出对应的策略

JavaScript对于 ```引用类型``` 是无法作检测的, 所以对与 **数组** 和 **对象** ,我们需要特殊的方法去回避



#### 对于对象

由于使用的是```Object.defineProperty``` 监听,所以每次我们需要向Vue 明确指出需要哪些属性是 响应式的, 提供给Vue 收集依赖.

```js
var vm = new Vue({
  data:{
    myObject: {
      name: 'haox'
    }
  }
})

// `vm.a` 是响应式的

vm.b = 2
// `vm.b` 是非响应式的
```

对于已经创建的Vue实例, 只能使用```Vue.set(object, propertyName, value)``` 的方法向 **嵌套对象** 添加响应式属性

```js
this.$set(this.myObject, 'age', '20')
```



#### 对于数组

```js
var vm = new Vue({
  data: {
    items: ['a', 'b', 'c']
  }
})
vm.items[1] = 'x' // 不是响应性的
vm.items.length = 2 // 不是响应性的
```

第一种使用下标去改变数组项,可以使用```Vue.set(vm.items, indexOfItem, newValue)``` 去解决

第二种需要用到```splice``` 方法实现,```vm.items.splice(newLength)``` 

> 都可以通过splice方法解决两种问题,因为splice改变原数组, this.items已经被收集,所以修改时会触发setter



以上.