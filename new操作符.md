### 手写new

demo

```ts
class People {
  name: string;
  constructor(name:string){
    this.name = name;
  }
}

const neo = new People('neo'); // People {name: 'neo'} 一个由People构造出来的对象
```



#### new原理

- 创建一个对象,对象的隐性原型链 ``` __proto__``` 需要指向该构造函数的```prototype```
  - ```const newObj.__proto__ = fn.prototype```
  - 可以用 ```const newObj = Object.create(fn.prototype)``` 实现
- 创建一个返回值, 她需要把构造函数的this指向上一步生成的新对象, 并将剩余参数导入
  - ```const result = fn.apply(newObj, arg)```
- 判断返回值是否为对象,如果是,返回result,不是的话返回newObject

#### MDN实现

```js
function newFn(fn, ...arg) {
  var obj = {}; // 创建一个空的简单JavaScript对象（即{}）
  obj.__proto__ = fn.prototype; // 为步骤1新创建的对象添加属性__proto__，将该属性链接至构造函数的原型对象；
  var result = fn.apply(obj, arg); // 将步骤1新创建的对象作为this的上下文 ；
  // 如果该函数没有返回对象，则返回this。
  let isObject = typeof result === 'object' && result !== null; // 是否不为null的对象
  let isFunction = typeof result === 'function'; // 是否是函数
  return isObject || isFunction ? result : obj; // 当是不为null的对象或函数时，返回；否则返回this
}
```



#### 简易实现

```ts
function newFN(fn, ...arg) {
  if (Object.prototype.toString.call(fn) !== '[object Function]') {
    throw TypeError(`${fn} is not a constructor`);
  }
  const _this = Object.create(fn.prototype); // 1. 创建一个__proto__ = 构造函数原型 的对象
  const result = fn.apply(_this, arg); // 2. 使用该对象为this，执行构造函数
  
  let isObject = typeof result === 'object' && result !== null; // 3.是否为不为null的对象
  let isFunction = typeof result === 'function'; // 3.是否是函数
  return （isObject || isFunction) ? result : _this; // 4. 构造函数fn返回的值不为null的对象 或者 函数, 则返回result，否则返回this。 因为new操作符里仅对构造函数里返回对象引用类型和函数类型的值作为返回，其他都返回新对象_this
}
```

