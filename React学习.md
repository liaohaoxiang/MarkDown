### 什么是React

概念题：类似题目 - 谈一谈你对React的理想。容易线性思考，缺乏整体性，会失去问题主动权。
解题思路：

- 讲概念，一句话解释技术本质
- 说用途，简短说明**技术用途**
- 理思路，简要说明**核心技术思路**
- 优缺点列举，独特优势和个别缺点，不能踩一捧一

- React是一个网页UI库
  - 历史上，05年JQ诞生，浏览器兼容是最大的问题，本质上只是一个工具函数。如何解决代码组织和复用率，成为最大问题。
  - 09年Angular出世，mvc模式，提供了路由、双向绑定、指令、组件等框架特性。双向绑定是最大的特色。但是太过复杂的系统设计，导致痛点没有解决。
  - 2010年，Backbone.js 的普及。
  - React ： View = fn(props)，只有组件，没有页面、控制器和模型。**只关心数据和组件**。
  - 设计模式中描述：组合优于技巧。构建UI视图时，组合组件**始终是最优的解决方案**。
- React核心：声明式，组件化，通用性



回答： 

- React是一个**构建UI的js库**，**通过组件化解决View层开发复用问题**，本质是一个组件化框架

- 核心设计思路有三点：声明式，组件化，通用性

  - 声明式优势在于直观和组件，不用机械化的编写代码

  - 组件化优势在于View的拆分与模块**复用**，更容易做到高内聚低耦合

  - 通用性优势在于一次学习，应用到各个场景，比如React Native，React 360，主要靠虚拟DOM保证实现

缺点：没有提供一套完整的解决方案，开发大型项目需要向社区整合解决方案，学习成本加深

加分：对React优化的看法，对虚拟DOM的看法，谈谈对React的工程化结构和设计模式

### 为什么React要用JSX

其实问的是：为什么不用其他，非要用JSX （比较论证）

考点：深挖知识面广度，对流行框架模板方案是否了解，技术方案调研能力

回答思路：

- 一句话解释JSX： JSX是一个JS的语法扩展，类似于XML的ES语法扩展。主要用于声明React元素，需要通过Babel转换为React.createElement。
- 核心概念： React本身不强制使用JSX，不用JSX可以依赖React.createElement生成组件,**核心是语法糖**。React需要将组件转化为虚拟DOM，**XML在树结构的描述上天生具有可读性强的优势**
  - 设计初衷：关注点分离（组件是React的基本单位），即将代码分隔为不同部分的设计原则，是面向对象设计的核心概念。**关注点分离的价值** 在于**简化程序的开发和维护**，当关注点分开时，各部分可以重复使用，独立开发和更新。具有价值的是可以改进或者修改一段代码，**无须知道其他部分细节**
- 方案对比：
  - 模板：React不引入模板的概念，模板的语法、指令会引入更多依赖
  - 模板字符串：结构会多次内部嵌套，语法提示差
  - JXON：语法提示差

回答： 

- JSX是一个JS的语法扩展，类似于XML。**JSX主要用于声明React元素**，但React不强制使用JSX，即使使用也会用**babel插件编译为React.createElement**，本质只是**语法糖**。
- 从这里看出，React团队并不想引入JavaScript以外的开发体系，通过关注点分离，保持开发纯粹性
- 模板不应该是开发时的关注点，模板字符串的结构会造成多次内部嵌套，语法提示也会变差，JXON也是因为语法提示变差影响开发



### 思考：Babel插件如何实现JSX到JS的编译

实现原理：Babel读取代码生成AST，将AST传入插件层进行转换，把JSX结构转换成React.createElement函数

代码在 BABEL_JSX_TRANSFORM_DEMO 项目中

```js
/**
 * 对jsx语法编译成js的babel插件
 * @param {babel.type} t 就是@babel/types 函数返回一个具有visitor属性的对象
 */
module.exports = function({types: t}) {
  return {
    name: 'jsx_transform-plugin',
    visitor: {
      JSXElement(path) {
        const { openingElement } = path.node; // JSXOpeningElement

        const tagName = openingElement.name.name; // 获取这个JSX标签的名字 demo里的div
        const attributes = t.nullLiteral(); // 不考虑 JSX上的props，直接传递null
        const args = [t.stringLiteral(tagName), attributes]; // 调用React.createElement需要传递的参数,这里是 'div' 和 props = null;

        const reactIdentifier = t.identifier("React"); // React object
        const createElementIdentifier = t.identifier("createElement"); // createElement
        const callee = t.memberExpression(
          reactIdentifier,
          createElementIdentifier
        ); // React.createElement

        const callRCExpression = t.callExpression(callee, args); // 生成React.createElement('xxx', null, children)
        callRCExpression.arguments = callRCExpression.arguments.concat(path.node.children); //将children参数合并进去

        path.replaceWith(callRCExpression, path.node); // 用生成的createElement结构替换之前的jsx结构
      },
      //处理 JSXText 节点
      JSXText(path) {
        const nodeText = path.node.value;
        path.replaceWith(t.stringLiteral(nodeText), path.node); // 直接用 string 替换 原来的节点
      },
    }
  }
}
```



### 如何避免生命周期中的坑

破题：如何避免坑——你见过多少坑——为什么会有坑

- 在不恰当的周期里**调用了不适合的代码**
- 在需要调用时却**忘了调用**

题目思路： 

- 基于周期，确认使用方式
- 基于每个周期的职责，确认调用范围

流程梳理：

- 挂载阶段
  - constructor，用于初始化state和绑定事件。社区已经去除，原因是
    - 不推荐在这里出力初始化以外的逻辑
    - 不属于React生命周期，只是class初始化函数
    - 通过移除，简洁代码
  - `static getDerivedStateFromProps()`， 会在**调用 render 方法之前调用**，用于props变化时更新state，**它应返回一个对象来更新 state**，如果返回 null 则不更新任何内容。官方不推荐
    - 当props传入时触发
    - 当state发送变化时触发
    - 当forceUpdate被调用时
  - UNNSAFE_componentWillMount，用于挂载前操作，目前**标记为弃用**，因为在React异步渲染机制下，**该方法可能被多次调用**。
  - render，**返回JSX结构**，用于描述具体的渲染内容。
    - 并没有真正渲染组件，渲染是React操作JSX的结构完成的。
    - 纯函数，每次渲染都会被调用。setState每次都会触发，不可以调用，会死循环；也不可以调用事件，因为会频繁注册事件。
  - `componentDidMount`，挂载完成时调用，**会在组件挂载后（插入 DOM 树中）立即调用**
    - 比如发起网络请求或者绑定事件
    - 在**render之后调用** 。除了在浏览器环境，并不是真实dom绘制完成后调用。
- 更新阶段，指外部props传入或者state发送变化时。
  - UNSAFE_componentWillReceiveProps，被getDerivedStateFromProps替代
  - `static getDerivedStateFromProps(props, state)`，会在调用 render 方法之前调用，并且在初始挂载及后续更新时都会被调用。
  - shouldComponentUpdate() ，返回true或false判断是否触发新的渲染，性能优化的重点
  - UNSAFE_componentWillUpdate()，后续React异步方案可能会出现暂停更新渲染的情况
  - render，和挂载时一致
  - getSnapshotBeforeUpdate()，配合新异步机制的新周期。返回值作为componentDidUpdate()第三个参数使用
- 卸载阶段
  - componentWillUnmount()，清理事件绑定和消除计时器

#### 什么情况下会触发重新渲染

- 函数组件
  - 任何情况下都会重新渲染，没有生命周期，官方提供React.memo优化
- React.Component,不实现`shouldComponentUpdate` 下，有两种情况触发重新渲染
  - 当state发送变化时
  - 当父级组件props传入时，**无论是否变化**
- React.pureComponent，仅在props和state进行浅比较后，**确认变更才触发重新渲染**



#### 错误边界

这种组件可以捕获并打印发生在其子组件树任何位置的 JavaScript 错误，并且，它会渲染出备用 UI，而不是渲染那些崩溃了的子组件树。当抛出错误后，请使用 static getDerivedStateFromError() 渲染备用 UI ，使用 componentDidCatch() 打印错误信息。
