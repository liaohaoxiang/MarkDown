### 学习技巧

- 断点调试（vscode 里的 debugger for chrome）
- 不要深度遍历源码（即不要遇到一个方法就进去看，一层一层看），要找到想看的点去分析，最好是带着问题分析。（比如nextTick原理）

### new Vue发生了什么

`new` 操作符会对class进行初始化，源码中Vue构造函数是

```js
function Vue (options) {
  if (process.env.NODE_ENV !== 'production' &&
    !(this instanceof Vue)
  ) {
    warn('Vue is a constructor and should be called with the `new` keyword');
  }
  this._init(options); //执行_init函数，传入options
}
```

<mark>这个`_init`是在Vue实例原型下的一个方法，是通过`initMixin`方法时加入</mark>

```js
// vue/src/core/instance/init.js
export function initMixin (Vue: Class<Component>) {
  Vue.prototype._init = function (options?: Object) {
    const vm: Component = this
    // ....还有其他代码
    // merge options
    if (options && options._isComponent) {
      // optimize internal component instantiation
      // since dynamic options merging is pretty slow, and none of the
      // internal component options needs special treatment.
      initInternalComponent(vm, options)
    } else {
      vm.$options = mergeOptions(
        resolveConstructorOptions(vm.constructor),
        options || {},
        vm
      )
    }
   // ....还有其他代码
    // expose real self
    vm._self = vm
    initLifecycle(vm)
    initEvents(vm)
    initRender(vm)
    callHook(vm, 'beforeCreate')
    initInjections(vm) // resolve injections before data/props
    initState(vm)
    initProvide(vm) // resolve provide after data/props
    callHook(vm, 'created')
    // ...
    if (vm.$options.el) {
      vm.$mount(vm.$options.el)
    }
  }
} 
```

`_init` 方法做了初始化的工作，比如uid，merge（合并） options然后放在$options下，生命周期,render,等等。。,还有一个`vm.$mount` 挂载页面到`options.el`上

### 为何可以用this.mydata访问data上的mydata属性

> 总结
>
> - new Vue后，执行initState，如果options.data存在，触发initData
> - initData里，检查data是否函数，true就执行data.call(vm,vm),false就把原值返回，如果原值为falsy，返回空对象
> - initData里，检查props,method,data里的key是否重复，重复触发告警
> - initData里最后一步，通过proxy增加一层转发
> - proxy里，通过Object.defineProperty改写getter和setter，使this.mydata的访问通过getter转发到访问this._data_mydata

这里的`initState` 可以解答出为什么可以用`this.mydata` 访问data里的数据

```js
export function initState (vm: Component) {
  vm._watchers = []
  const opts = vm.$options
  if (opts.props) initProps(vm, opts.props)
  if (opts.methods) initMethods(vm, opts.methods)
  if (opts.data) {
    initData(vm)
  } else {
    observe(vm._data = {}, true /* asRootData */)
  }
  if (opts.computed) initComputed(vm, opts.computed)
  if (opts.watch && opts.watch !== nativeWatch) {
    initWatch(vm, opts.watch)
  }
}
```

这里如果`$options` 定义了属性，就会按照这里的顺序初始化。主要看`initData`方法

```js
function initData (vm: Component) {
  let data = vm.$options.data
  data = vm._data = typeof data === 'function'
    ? getData(data, vm)
    : data || {}
  if (!isPlainObject(data)) {
    data = {}
    process.env.NODE_ENV !== 'production' && warn(
      'data functions should return an object:\n' +
      'https://vuejs.org/v2/guide/components.html#data-Must-Be-a-Function',
      vm
    )
  }
  // proxy data on instance
  const keys = Object.keys(data)
  const props = vm.$options.props
  const methods = vm.$options.methods
  let i = keys.length
  while (i--) {
    const key = keys[i]
    if (process.env.NODE_ENV !== 'production') {
      if (methods && hasOwn(methods, key)) {
        warn(
          `Method "${key}" has already been defined as a data property.`,
          vm
        )
      }
    }
    if (props && hasOwn(props, key)) {
      process.env.NODE_ENV !== 'production' && warn(
        `The data property "${key}" is already declared as a prop. ` +
        `Use prop default value instead.`,
        vm
      )
    } else if (!isReserved(key)) {
      proxy(vm, `_data`, key)
    }
  }
  // observe data
  observe(data, true /* asRootData */)
}
```

- 判断data是否是一个`function` ，true就调用`getData`，`getData`传入`data和vm实例`，代码主要是返回`data.call(vm,vm)`,绑定this到vm上；如果不是则直接用这个data； 这里得出的data会赋值给`vm._data`
- 判断data是否是对象（`isPlainObject`这个方法），如果不是就告警
- 循环判断`methods，props，data`上的key，不可以出现重复名称，因为最终都要挂载到vm实例，所以不可以冲突
- 实现 `this.mydata` 获取，靠的是`proxy`这个函数

```js
const sharedPropertyDefinition = {
  enumerable: true,
  configurable: true,
  get: noop,
  set: noop
}

export function proxy (target: Object, sourceKey: string, key: string) {
  sharedPropertyDefinition.get = function proxyGetter () {
    return this[sourceKey][key]
  }
  sharedPropertyDefinition.set = function proxySetter (val) {
    this[sourceKey][key] = val
  }
  Object.defineProperty(target, key, sharedPropertyDefinition)
}
```

- 通过`sharedPropertyDefinition`对象，定义`getter`和`setter` 两个函数，通过`Object.defineProperty`,对传入的vm实例和key，包装一层getter和setter，我们访问 `vm.key`，实质上这里的getter会访问到 `vm._data.key` ，就可以用 `this.mydata`,访问调用到`this._data.mydata`. 这里是代理访问的原理