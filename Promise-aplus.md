```js
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";

class MyPromise {
  constructor(executor) {
    try {
      executor(this.resolve, this.reject);
    } catch (e) {
      this.reject(e);
    }
  }
  // static常量
  status = PENDING;
  value = undefined;
  reason = undefined;
  onResolvedCallbacks = [];
  onRejectedCallbacks = [];

  // 模仿内核带的resolve和reject方法
  resolve = (value) => {
    // 只能在pending时改变状态和赋resolve的值
    if (this.status === PENDING) {
      this.status = FULFILLED;
      this.value = value;
      // 发布-订阅模式 主要是兼容promise的异步操作,这里是作为发布的操作,把收集好的订阅一次性调用
      while (this.onResolvedCallbacks.length) {
        console.log(
          `resolve | 发布事件 | this.onResolvedCallbacks.length = ${this.onResolvedCallbacks.length}`
        );
        this.onResolvedCallbacks.shift()(value);
      }
    }
  };

  reject = (reason) => {
    if (this.status === PENDING) {
      this.status = REJECTED;
      this.reason = reason;
      while (this.onRejectedCallbacks.length) {
        this.onRejectedCallbacks.shift()(reason);
      }
    }
  };

  then(onFulfilled, onRejected) {
    console.log("--then开始--", `onFulfilled = ${onFulfilled}`);
    // 如果then回调 不传或者不是函数，就使用默认函数 ---- 值穿透
    onFulfilled =
      typeof onFulfilled === "function" ? onFulfilled : (value) => value;
    onRejected =
      typeof onRejected === "function"
        ? onRejected
        : (reason) => {
            throw reason;
          };
    // 为实现链式调用,then方法return一个新的promise,会立即执行
    const promise2 = new MyPromise((resolve, reject) => {
      const fulfilledMicrotask = () => {
        // 使用queueMicrotask创建微任务,等待promise2初始化成功后,测试循环调用
        queueMicrotask(() => {
          console.log(`${FULFILLED}的微任务, 通过执行then里的回调得出x`);
          try {
            const x = onFulfilled(this.value); //将then的回调函数的返回值传入
            resolvePromise(promise2, x, resolve, reject);
          } catch (error) {
            reject(error);
          }
          console.log(`in queueMicrotask, 微任务结束`);
        });
      };
      const rejectedMicrotask = () => {
        queueMicrotask(() => {
          try {
            const x = onRejected(this.reason);
            resolvePromise(promise2, x, resolve, reject);
          } catch (error) {
            reject(error);
          }
        });
      };
      if (this.status === FULFILLED) {
        fulfilledMicrotask();
      }

      if (this.status === REJECTED) {
        rejectedMicrotask();
      }

      if (this.status === PENDING) {
        // 如果promise的execute是异步的,由于状态还没变更,
        // 缓存then的回调函数,等到resolve或者reject执行后,
        // 通过发布-订阅的模式,由resolve/reject 调用then的回调函数
        this.onResolvedCallbacks.push(fulfilledMicrotask);
        this.onRejectedCallbacks.push(rejectedMicrotask);
      }
    });
    console.log("--then结束-返回新 promise2 =", promise2);
    return promise2;
  }
  // Promise.resolve方法
  static resolve(value) {
    // 如果value是一个promise对象，直接返回
    if (value instanceof MyPromise) {
      return value;
    }
    // 如果value是一个thenable对象，调用value.then方法
    if (value && typeof value.then === "function") {
      return new MyPromise((resolve, reject) => {
        value.then(resolve, reject);
      });
    }
    // 如果value是一个普通对象，返回一个新的promise对象
    return new MyPromise((resolve) => {
      resolve(value);
    });
  }
  // Promise.reject方法
  static reject(reason) {
    return new MyPromise((_, reject) => {
      reject(reason);
    });
  }
}

function resolvePromise(promise2, x, resolve, reject) {
  console.log("--进入resolvePromise-- x =", x);
  // 防止在then函数里返回自己
  // p2 = p1.then(_ => { return p2; });
  if (promise2 === x) {
    console.log("循环调用");
    return reject(
      new TypeError("Chaining cycle detected for promise #<Promise>")
    );
  }

  let isCalled = false;
  // 判断x如果是promise (为兼容第三方代码, 判断可以是不为null的对象或者函数)
  if ((typeof x === "object" && x !== null) || typeof x === "function") {
    // 如果是函数,把它的then拿出来
    try {
      let then = x.then;
      // 如果then是函数,就调用它,并传入resolve和reject
      if (typeof then === "function") {
        console.log(`resolvePromise | then是function | then = ${then}`);
        then.call(
          x,
          (y) => {
            if (isCalled) {
              return;
            }
            isCalled = true;
            resolvePromise(promise2, y, resolve, reject);
          },
          (r) => {
            if (isCalled) {
              return;
            }
            isCalled = true;
            reject(r);
          }
        );
      } else {
        // 如果then不是函数,以x为参数执行promise
        resolve(x);
      }
    } catch (error) {
      if (isCalled) {
        return;
      }
      called = true;
      reject(error);
    }
  } else {
    // 如果不为对象和函数,是其他类型(包括null),直接返回
    resolve(x);
  }
}
```

