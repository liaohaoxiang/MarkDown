### v-show 和 v-if的区别

v-show 都会渲染，只是条件不符合时，会加入`style="display: none"` ，适合多次转换，dom一直存在

v-if 只会渲染符合条件的模板，如果频繁渲染会导致dom重复销毁和新增



### v-for为什么要key

> 当 Vue 正在更新使用 v-for 渲染的元素列表时，它默认使用“就地更新”的策略
>
> 为了给 Vue 一个提示，以便它能跟踪每个节点的身份，从而重用和重新排序现有元素，你需要为每项提供一个唯一 key attribute
>
> 建议尽可能在使用 v-for 时提供 key attribute，除非遍历输出的 DOM 内容非常简单，或者是刻意依赖默认行为以获取性能上的提升。
>
>  —— Vue文档

- v-for不可以和v-if一起使用，v-for优先级会高于v-if，即使不展示也会先渲染，耗费性能。可以把v-if提升到父组件使用。
- key的作用是 **组件更新时，判断两个节点是否相同** ；
  - 如果是 **简单的，没有状态的组件** ，那么`不加key的性能会高于加key`，因为唯一key在每次更新时都不能找到可复用的节点，需要不断销毁/创建vnode，添加和移除DOM对性能的影响更大。不带key时节点能够复用，省去了销毁/创建组件的开销。
  - 如果是**有状态组件**，那么需要有 **唯一key** 为每次渲染列表时确保能完全更新所有组件，使其拥有正确的状态。避免【原地复用】带来的副作用
- 总结
  - 不用key
    - **原地复用节点**，因为`a.key === a.key === undefined` ，会判断新旧节点为同一个节点，所以不会 **重新创建/删除接口节点** ，只会在节点上 **比较更新** ，在渲染性能方面会得到提升
    - **无法维持组件状态** ，原地复用的副作用
  - 用key
    - **维持组件状态** ，key唯一标识了组件，不会再比较新旧节点上直接判断为同一节点，**而是会继续在接下来的节点中寻找** key相同的节点，能找到相同的key就**复用节点**，不能找到就**增加/删除节点** 。
    - **查找性能提升** ，有key的时候会把key生成hash，这样在查找的时候就是hash查找，时间复杂度为O(1)
    - **节点复用带来的性能提升** ，因为key唯一标识了组件，所以**尽可能的对组件进行复用** ，那么**创建/删除**节点就变少，性能提升

### v-model原理

- 修饰符

  - `v-model.lazy` 由原来的`input`事件触发，改变为`change` 事件触发
  - `v-model.trim` 去掉字符串首尾空格
  - `v-model.number` 强制输入Number类型

- 自定义v-model

  - `v-model` 在内部为不同的输入元素使用不同的 property 并抛出不同的事件：

    - text 和 textarea 元素使用 `value` property 和 `input` 事件；
    - checkbox 和 radio 使用 `checked` property 和 `change` 事件；
    - select 字段将 `value` 作为 prop 并将 `change` 作为事件。

  - 在组件里

    - ```vue
      <template>
        <div>
          <input type="text" :value="value" v-on="inputEvent">
        </div>
      </template>
      <script>
      export default {
        props: {
          value: String,
        },
        data(){
          return {
            inputEvent: {
              input: (e) => this.$emit('input', e.target.value)
            }
          }
        }
      }
      </script>
      ```

  - 使用（消费）该组件

    - ```vue
      <Vmodel v-model="parentMsg" /> // typeof parentMsg == 'string'
      ```

  - 可以看出`v-model`是把 `parentMsg` 当作一个 `props` 传入到组件中，组件里通过 `value` 去接收该 `props` （input框默认是value，checkbox和radio默认的props是`checked`）

  - 组件接收到value后，绑定到input框，相当于`:value="parentMsg"` 。`v-on` 可以传入一个对象，该对象是以事件名称为`key` ，`handler` 为 `value` 。这里input框的输入事件是input事件，然后`v-model` 同时会传入一个 `@input` 事件到组件中，必须要用`$emit` 去触发该事件，并把target.value 赋值到第一个参数

  - 总结：v-model 传入一个名为value的props，和一个input事件。value用于绑定事件的值，input事件用于监听输入变化从而把变更值回调到value，达到双向绑定。



### nextTick原理

在**下次 DOM 更新循环结束之后**执行延迟回调。nextTick主要使用了宏任务和微任务。根据执行环境分别尝试采用

- 微任务 `microTimerFunc`
  - Promise （vue 不自带prolifill）
  - MutationObserver (不支持IE)
- 宏任务`macroTimerFunc` （优先级从高到低）
  - setImmediate （高版本 IE 和低版本 Edge 才支持的特性）
  - setTimeout

定义了一个异步方法，多次调用nextTick会将方法存入队列中，通过这个异步方法清空当前队列。

```js
var callbacks = [];
var pending = false;

function flushCallbacks () {
  pending = false;
  var copies = callbacks.slice(0);
  callbacks.length = 0;
  for (var i = 0; i < copies.length; i++) {
    copies[i]();
  }
```



### Vue的生命周期

- `beforeCreate` new Vue()之后触发的第一个钩子，在当前阶段data、methods、computed以及watch上的**数据和方法都不能被访问**
- `created` 在实例创建完成后发生，当前阶段**已经完成了数据观测**，可以使用/更改数据，在这里更改数据**不会触发生命周期updated函数**。当前阶段无法与Dom进行交互，如果非要想，可以通过`vm.$nextTick来访问Dom`
- `beforeMount` 发生在**挂载之前** ，当前阶段**虚拟Dom已经创建完成**，即将开始渲染。
- `mounted` 在挂载完成后发生，真实的Dom挂载完毕，数据完成双向绑定，**可以访问到Dom节点，使用$refs属性对Dom进行操作**
- `beforeUpdate` 发生在更新之前，响应式数据发生更新，虚拟dom**重新渲染之前被触发**，你可以在当前阶段进行更改数据，不会造成重渲染
- `updated` 发生在更新完成之后，当前阶段组件Dom已完成更新。避免在此期间更改数据，因为这可能会**导致无限循环的更新**。



### computed 和 watch

- computed有缓存， data不变就不会重新计算
- watch是一个对象时，property有
  - handler
  - deep 是否深度
  - immeditate 是否立即执行
- watch监听引用类型，拿不到oldVal

当有一些数据需要**随着另外一些数据变化时**，建议使用computed。
当有一个要**执行一些业务逻辑或异步操作时**建议使用watcher



### keep-alive

- `keep-alive`可以实现组件缓存，当组件切换时不会对当前组件进行卸载。

- 常用的两个属性`include/exclude`，允许组件有条件的进行缓存。

- 两个生命周期`activated/deactivated`，用来得知当前组件是否处于活跃状态。

- ```vue
  <keep-alive include="a,b">
    <component :is="view"></component>
  </keep-alive>
  ```

  

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

这个`_init`是在Vue实例原型下的一个方法，是通过`initMixin`方法时加入

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

这里如果`$options` 定义了属性，就会按照这里的顺序初始化。主要看`initData`属性

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

- 判断data是否是一个`function` ，true就调用`getData`，这里传入`data和vm实例`，代码主要是返回`data.call(vm,vm)`,绑定this到vm上；如果不是则直接用这个data； 这里得出的data会赋值给`vm_data`
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





### 事件处理

> Vue事件是挂载到e.currentTarget上的

- 监听事件
  - 通过 ```v-on``` 指令监听 DOM 事件
  - 内联处理器中的方法可以通过``` v-on:click="say('hi')"``` 来调用方法
  - 如果内联语句需要用到 原始的 DOM 事件，可以用特殊变量 ```$event``` 传入到方法 ```v-on:click="fn('123', $event)"```
- 事件修饰符
  - `@click.stop` 事件的 `event.stopPropagation()` ，阻止事件的冒泡和捕获两种传播手段
  - `@click.prevent`  事件listener 的`event.preventDefault()`，阻止事件的默认行为
  - `@click.capture` 事件listener 使用事件捕获模式
  - `@click.self` 只当`event.target`是当前元素自身时才触发 handler
  - `@click.once` 事件只触发一次
  - `@click.passive` 阻止事件 listener 使用 preventDefault 方法，告诉浏览器你不想阻止默认行为
- 按键修饰符
  - 监听键盘事件，检查详细的按键
  - `@keyup.enter` 当 handler 里的`$event.key` 为`Enter` 时才触发
