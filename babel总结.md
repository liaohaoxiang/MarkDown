### 什么是Babel

> 一个`JavaScript` 编译器，可以把最新版的 JavaScript编译成当下可以执行的版本。在低版本浏览器中，我们可以通过Babel将项目中高级的语法和一些尚在实验性的语法编译到浏览器支持的ES版本



### 运行原理

![](/Users/diu/Desktop/MarkDownFile/面试题/babel.png)

- 解析 （Parse）

  - 接收代码通过一个叫`babylon`的js词法解析器并输出 `AST` 

  - 这里分成两个阶段，词法分析和语法分析

    - 词法分析：将整个代码字符串分割成语法单元数组（token）也叫`分词`

      - ```js
        function add(x,y){
            return x + y
        }
        // => token
        [
            {"type": "Keyword","value": "function"}, // Keyword = 关键字
            {"type": "Identifier","value": "add"},
            {"type": "Punctuator","value": "("}, // Punctuator = 标点符号
            {"type": "Identifier","value": "x"}, // Identifier = 标识符
            {"type": "Punctuator","value": ","},
            {"type": "Identifier","value": "y"},
            {"type": "Punctuator","value": ")"},
            {"type": "Punctuator","value": "{"},
            {"type": "Keyword","value": "return"},
            {"type": "Identifier","value": "x"},
            {"type": "Punctuator","value": "+"},
            {"type": "Identifier","value": "y"},
            {"type": "Punctuator","value": "}"}
        ]
        ```

      - 语法分析：在token结果的基础上分析语法单元之间的关系。

        - 主要概念：`语句(statement)`和`表达式(expression)`
        - 语法分析的过程是对`语句`和`表达式`的**递归**识别，babel在过程中设置一个暂存器，**暂存**当前读取到的语法单元，**如果错误，会返回暂存点**，**如果成功，会销毁暂存点**，递归执行，直到生成AST

      ```js
      // 对于上面的token 翻译成的AST如下
      {
        "type": "Program",
        "body": [{
            "type": "FunctionDeclaration",
            "id": {"type": "Identifier","name": "add"},
            "params": [{"type": "Identifier","name": "x"},{"type": "Identifier","name": "y"}],
            "body": {
              "type": "BlockStatement",
              "body": [{
                  "type": "ReturnStatement",
                  "argument": {
                    "type": "BinaryExpression",
                    "operator": "+",
                    "left": {
                      "type": "Identifier",
                      "name": "x"
                    },
                    "right": {
                      "type": "Identifier",
                      "name": "y"
                    }
                  }
                }]
            },
            "generator": false,
            "expression": false,
            "async": false
          }
        ],
        "sourceType": "script"
      }
      ```

      

      - 

- 转换（Transform）

  - 对生成的`AST` 通过`babel-traverse` 进行遍历，在此过程中进行添加、更新及移除等操作。
  - plugins 在这个阶段进行，插件会自动使用对应的词法插件

-  生成 （Generate）

  - 将变换后的` AST` 再转换为不同版本ES的 JS 代码, 使用到的模块是` babel-generator`



### Babel常用包

核心包

- babel-core:
  - **babel转译器**，提供了转译API如babel.transform，用于对代码进行转译。像webpack的babel-loader和plugins就是调用这些API来完成转译过程的。
- babylon (babel7后为@babel/parser)
  - js的词法解析器
- babel-traverse
  - 用于对AST（抽象语法树）的遍历，主要给plugin用
- babel-generator
  - 根据AST生成代码



功能包

- babel-types 

  - 用于检验、构建和改变AST树的节点

  - ```js
    const t = require('babel-types');
    module.exports = function() {
      return {
       //访问者
        visitor: {
         //我们需要操作的访问者方法(节点)
          VariableDeclaration(path) {
            console.log('var转化的node',path.node)
            //该路径对应的节点
            const node = path.node;
            //判断节点kind属性是let或者const,转化为var
            ['let', 'const'].includes(node.kind) && (node.kind = 'var');
          },
          //箭头函数对应的访问者方法(节点)
          ArrowFunctionExpression(path) {
            //该路径对应的节点信息  
            let { id, params, body, generator, async } = path.node;
            //进行节点替换 (arrowFunctionExpression->functionExpression)
            path.replaceWith(t.functionExpression(id, params, body, generator, async));
          }
        }
      };
    };
    
    ```

  - 





### Polyfill和runtime如何选择

当实际环境中（旧浏览器）没有新的语法，我们需要用到`polyfill` 去做一个 '垫片'的操作，即是把一些已经实现好的新语法注入到代码中，从而达到打上补丁的效果。



`@babel/polyfill` 这个包可以解决这个问题，需要在install后引入到文件中 `improt @babel/polyfill` 

> polyfill本身就是stable(稳定)版本的core-js和regenerator-runtime的合集

虽然 `@babel/polyfill` 解决了模拟浏览器不存在对象方法的事情，但是还是存在两个缺点

-  直接修改内置的原型，造成全局污染
- 无法按需引入，webpack打包时，会把所有Polyfill都加载进来，导致打包文件过大



#### Babel runtime

为了解决 `polyfill` 的问题， 出现了`@babel/runtime` 的方案，它不再修改原型，而是采用替换的方式。

比如我们用Promise，`@babel/polyfill`会产生一个`window.Promise`的全局对象，而`@babel/runtime` 则会生成一个`_Promise`来替换代码中用到的Promise。

`@babel/runtime` 支持按需引入，可以控制打包体积

示例：

- 安装 `npm i @babel/runtime` 和 `npm i -D @babel/plugin-transform-runtime` 作为插件



#### Babel7的解决方案Babel7后，把polyfill都整合了。我们可以用`@babel/preset-env`去配置polyfill。

`@babel/preset-env` 配置 `useBuiltIns` ，控制`@babel/preset-env` 处理polyfill

- 

  ```json
  {
    "presets": [
        [
          "@babel/preset-env", {
            "useBuiltIns": "usage",
            "corejs": 3 // 一定要有corejs版本，需要安装core-js@3
          }
      ]
    ]
  }
  ```

- `useBuiltIns: 'entry'` 有历史变更，` @babel/polyfill` 从 `Babel`7.4.0 版本开始就被弃用了

  | Version  | Changes                                                      |
  | :------: | ------------------------------------------------------------ |
  | `v7.4.0` | 取代`"core-js/stable"` 和`"regenerator-runtime/runtime"`入口 |
  | `v7.0.0` | 取代`"@babel/polyfill"` 入口                                 |

  - 使用这种`entry`配置需要在你的业务代码当中注入

  - ```js
    import 'core-js/stable'
    import 'regenerator-runtime/runtime'
    ```

    

- `useBuiltIns: 'usage'` 可以实现按需导入





### @babel/plugin-transform-runtime

该插件可以做到：把重复的polyfill引入抽离出来



#### 最佳实践 (babel > v7.4)

```js
 // .babelrc
{
  "plugins": [
    [
      "@babel/plugin-transform-runtime", {
      "corejs": false, // 默认值，可以不写
      "helpers": true, // 默认，可以不写
      "regenerator": false, // 通过 preset-env 已经使用了全局的 regeneratorRuntime, 不再需要 transform-runtime 提供
      的 不污染全局的 regeneratorRuntime
      "useESModules": true // 使用 es modules helpers, 减少 commonJS 语法代码
    	}
    ], 
   "presets": [
    [
      "@babel/preset-env", {
      "targets": {}, // 这里是targets的配置，根据实际browserslist设置 "corejs": 3, // 添加core-js版本
      "modules": false, // 模块使用 es modules ，不使用 commonJS 规范 "useBuiltIns": "usage" // 默认 auto
    }
	]
}
```

