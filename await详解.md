---
title: 'async/await中 await怎么解释？'
date: '2020-07-18'
kind: 'Tech'
---

> async 不用多说，代表这个函数有异步操作,关键是 await 如何理解

可以根据一个栗子引出思考

```js
async function async1() {
  console.log("a");
  await async2();
  console.log("b");
}
async function async2() {
  console.log('c');
}
async1();
new Promise(function (resolve) {
  console.log("d");
  resolve();
}).then(function () {
  console.log("e");
});
```

如果你是在 chrome 70 上做测试，那么返回的顺序将会是：

```shell
a
c
d
e
b
```

如果你在 chrome canary 73 或以上测试，那么返回的将会是：

```shell
a
c
d
b
e
```

两个测试结果的`async1 end` 会不一样，是因为 v8 引擎做了优化。

## 从 Promise 引出 Async 的修改原因

阅读了这遍文章后，对于 await 有进一步的看法[promise, async, await, execution order](https://github.com/xianshenglu/blog/issues/60)

开始前需要有一个认识，我们平时使用构造函数生成 Promise 实例，我们称为 RESOLVE(something)

```js
RESOLVE(p) === new Promise((resolve, reject) => {
    resolve(p);
  });
```

而另外一种创建 Promise 实例方式，更简洁，就是 Promise.resolve()

```js
Promise.resolve('foo') === new Promise(resolve => resolve('foo'));
```

直接上结论：

- Promise.resolve(`nonThenable`)与 RESOLVE(`nonThenable`) 全等。

```js
// 对于Promise参数为原始值，都等于这种情况。 ---摘要自《ES6标准入门》P289 by阮一峰
new Promise((resolve, reject) => {
  resolve(nonThenable);
});
```

- Promise.resolve(`thenable`)和 RESOLVE(`thenable`)是不同的。它们有不同的效果。


- RESOLVE(thenable)和 RESOLVE(promise)都可以转化为

```js
new Promise(resolve => resolve(thenable)) === new Promise(resolve=> {
  Promise.resolve().then(() => {
    thenable.then(resolve);
  });
});
```

- Promise.resolve(promise) === promise,即不作任何修改，原封不动返回这个 promise 实例。 而 Promise.resolve(nonPromiseThenable)可以转化为

```js
new Promise(resolve => {
  Promise.resolve().then(() => {
    nonPromiseThenable.then(resolve);
  });
});
```

疑点: 在 RESOLVE(thenable)中是 === Promise.resolve(nonPromiseThenable)

下面两个例子可以看出 两者在处理 Promise 为参数时所带来的的不同

```js
//Promise.resolve(promise)
let p1 = Promise.resolve(1);
Promise.resolve(p1).then(res => {
  console.log(res);
});
p1.then(res => {
  console.log(2);
});
//1
//2

//RESOLVE(promise)
let p1 = Promise.resolve(1);
new Promise((resolve, reject) => {
  resolve(p1);
}).then(res => {
  console.log(res);
});
p1.then(res => {
  console.log(2);
});
//2
//1
```

## 回到await 

[Normative: Reduce the number of ticks in async/await]('https://github.com/tc39/ecma262/pull/1250')

根据这个对于await的修改，揭露了await优先级提高的原因: 如果await后面的代码是promise，那么将使用Promise.resolve而不是使用RESOLVE()重新生成promise实例。

上面提到，Promise.resolve(promise)会原封不动把promise返回，因此时序上比RESOLVE提升了

所以await可以完全转换为promise.resolve()去理解

```js
const async1 = async function() {
  const res1 = await p1
  console.log(res1)
}
```
转换成

```js
const async1 = async function() {
  Promise.resolve(p1).then(res => {
    const res1 = res
    console.log(res1)
  })
}
```

<!-- ## Promise的语法糖
其实这道题问的是

> await P 怎么理解?

又因为 ```async```函数总是返回一个promise,所以其实就是在问

> await Promise 怎么理解?

```js
async function async1(){
  await P
  console.log('async1 end')
}
```

等价于

```js
async function async1() {
  return new Promise(resolve => {
    resolve(P)
  })
  return new Promise.resolve(p).then(() => {
    console.log('async1 end')
  })
}
```

```js
await === new Promise(resolve => {
    resolve('跟在你await后面的函数')
    }).then(()=>{'await下面的代码'})
``` -->

<!-- > 『RESOLVE(p)』接近于『Promise.resolve(p)』，不过有微妙而重要的区别：p 如果本身已经是 Promise 实例，Promise.resolve 会直接返回 p 而不是产生一个新 promise

## await [promise]

如果 P 是一个 promise 函数(如上述的 async2,因为 async 永远会返回一个 promise),那么在 await 规范更改后,await 就等价于

```js
(await async2()) ===
  Promise.resolve(async2()).then(res => {
    console.log('async1 end');
  });
```

那么对于 `resolve(async2())`，我们可以根据规范转换成：

```js
Promise.resolve().then((resolve) => {
  async2().then(resolve);
});
```

所以 async1 就变成了这样：

```js
async function async1() {
  return new Promise(resolve => {
    Promise.resolve().then(() => {
      async2().then(resolve);
    });
  }).then(() => {
    console.log('async1 end');
  });
}
``` -->
