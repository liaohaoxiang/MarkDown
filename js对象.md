### 对象属性描述（Descriptor）

对象的**每个属性**都会有一个描述对象，用来控制属性的行为，可以通过`Object.getOwnPropertyDescriptor`方法获取该属性的描述对象

```js
const dog = {name: 'WALL·E'}
Object.getOwnPropertyDescriptor(dog, 'name')
// {
//    "value": "WALL·E",
//    "writable": true,
//    "enumerable": true,
//    "configurable": true
//}
```

目前有四个操作会忽略`enumberable`为false的属性，即不可枚举

- `for...in`循环：只遍历对象自身的和 **继承的** **可枚举**属性，不包含Symbol属性 
- `Object.keys`：遍历对象自身的 **所有可枚举**的属性的key，不包含Symbol属性，返回一个数组
- `JSON.stringify`：只序列化对象自身的 **可枚举**属性
- `Object.assign`：忽略不可枚举属性，只拷贝对象自身的可枚举属性

### 对象操作

- 遍历对象
  - `for…in`
    - 循环遍历对象自身的和 **继承的** 可枚举属性，不包含Symbol属性 （最大问题就是会遍历到 `__proto__` 上的属性）
  - `Object.keys(obj)`
    - 返回一个数组，包含对象自身的 **可枚举属性**，不包含Symbol属性
    - PS：如果参数不是一个对象而是原始值，`ES5中会抛出TypeError，ES6中**强制转换**非对象参数为一个对象`
  - `Object.getOwnPropertyNames(obj)`
    - 返回一个数组，包含对象自身的 **可枚举和不可枚举属性**，但不包含Symbol属性
  - `Object.getOwnPropertySymbol(obj)`
    - 返回一个数组，包含对象自身的 **所有Symbol属性**的key
  - `Reflect.ownKeys(obj)`
    - 返回一个数组，包含对象自身的key，不管key是Symbol还是字符串，也不管是否可枚举
- 以上的遍历对象方法，遵循同样的次序规则
  - 首先遍历所有数值key，按照**数值**升序排列
  - 其次遍历所有字符串key，按照**加入时间**升序排列
  - 最后遍历Symbol的key，按照**加入时间**升序排列

### 扩展运算符的解构赋值

- 正常的解构赋值

  - ```js
    const person = {
      name: 'neo'
    };
    const { name } = person; // name === 'neo'
    ```

  - 这里在 **花括号的值** 能够正确对应上**对象里的key值**

- 扩展运算符的解构赋值

  - PS：扩展运算符使用解构赋值得到的对象，**不能复制继承**自原型对象的属性

  - ```js
    const person = Object.create({ name: 'holk', age: 25 }); // person原型是输入参数的对象
    person.height = 182;
    const { name, ...index } = person; // name是正常的解构，正常获取；index是扩展运算符解构
    const { age, height } = index; // 这里的age是undefined，height是182
    // 原因是因为index对象是通过【扩展运算符解构赋值】得来的，它不包含原型对象上的属性
    ```

  

