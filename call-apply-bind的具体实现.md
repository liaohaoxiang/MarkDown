### 手写apply

规范描述 [ECMAScript5.1中文版](http://yanhaijing.com/es5/#322)

apply方法语法： ``` Function.prototype.applyFn = function apply(thisArg, argsArray) ```

实现原理：通过`隐式调用` 来改变函数执行时的`this` ，隐式调用实际上就是在对象上执行函数

需要考虑的情况

1. this 是否为 function， 抛出一个TypeError异常

   - >  原因： 有可能会不为function的对象执行这个applyFn方法，比如一个简单对象用`__proto__` 连接到Function.prototype

2. argArray 是 null 或 undefined ，argsArray=[ ]

   - > 原因： 规范中描述 如果 argArray 是 null 或 undefined, 则
     > 返回提供 thisArg 作为 this 值并以空参数列表调用 func 的 [[Call]] 内部方法的结果。

3. argArray类型是否为object，抛出一个 TypeError 异常 

   - > 原因： ECMAScript 5 开始可以使用类数组对象，不一定是数组

实现时注意：浏览器环境下，如果apply传入的thisArg是`undefined`或者`null`的话，会指定到全局对象`window`

代码:

```js
// 辅助函数，返回windows
function getGlobalObject(){
    return this;
}

Function.prototype.applyFn = function apply(thisArg, argsArray) {
  if (typeof this !== 'function') {   // 当前this指向执行该函数的对象，如果不是函数类型，那么就抛出错误
    throw TypeError(this + ' is not a function');
  };
  if (argsArray === void 0 || argsArray === null ) {  // 没有传入argsArray或者传入null，则为空数组
    argsArray = [];
  };
  if (argsArray !== new Object(argsArray)) {
    throw new TypeError('CreateListFromArrayLike called on non-object');    // 如果argsArray不是object类型，抛出异常
  };
  if (thisArg === void 0 || thisArg === null) { // 如果thisArg传入undefined或者null，指向全局对象
    thisArg = getGlobalObject();
  };
  
  // apply实现
  thisArg = new Object(thisArg); // ES3改版，thisArg传入基础类型的值会将其包装成引用类型
  var _fn = Symbol(); // 传入一个相对唯一的值 （可以用UUID）
  var originalVal = thisArg[_fn];
  var hasOriginalVal = thisArg.hasOwnProperty(_fn);  // 万一万一还是有相同的key值，需要把这个值储存起来
  
  thisArg[_fn] = this; // 赋值这个函数到thisArg这个对象，key为_fn,value为需要apply的函数
  var result = thisArg[_fn](...argsArray);  // 以argsArray为参数执行[[Call]]内部方法，运行时的this是thisArg
  delete thisArg[_fn]; // 删除这个key
 
  // 确定是否恢复被我们覆盖的_fn 这个key的值
  if (hasOriginalVal) {
    var originalVal = thisArg[_fn];
  }
  return result;
}
```

