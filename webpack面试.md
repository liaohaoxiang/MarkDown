- Webpack面试题

  ## 配置

  - 基本配置

    - ```js
      // webpack.common.js
      const HtmlWebpackPlugin = require('html-webpack-plugin')
      const { CleanWebpackPlugin } = require('clean-webpack-plugin')
      const path = require('path')
      
      module.exports = {
        // 入口文件
        entry: {
          main: './src/index.js',
          common: './src/common.js', // 公用代码打包入口，可以通过chunckhash确认不同打包后的bundle
        },
        // 模块定义
        module: {
          // 规则定义
          rules:[{
            test: /\.(jpg|png|gif)$/, //遇到的类型时，使用什么规则的loader解析
            //使用loader处理 
            use: {
              loader: 'url-loader',
              options: {
                //placeholder 占位符
                name: '[name].[ext]',
                //输出路径
                outputPath: 'images/',
                //如果小于25k，已base64打包到打包文件里
                limit: 25 * 1024
              }
            }
          }]
        },
        // 插件定义
        plugins :[
          new HtmlWebpackPlugin({
              template: 'src/index.html'
            }),
            //打包前删除之前的目录下的文件(新版不需要指定路径)
          new CleanWebpackPlugin()
        ],
        // 优化设置
        optimization: {
          //把类库代码分割(code splitting) 有效分散js文件避免重新加载
          //SplitChunksPlugin 插件可以将公共的依赖模块提取到已有的入口 chunk 中，
          //或者提取到一个新生成的 chunk
          splitChunks: {
            chunks: 'all'
          }
        },
        // 打包后的代码处理
        output: {
          publicPath: 'http://cdn.com.cn', //固定前缀地址，常用于静态资源
          filename: '[chunckhash:16].js',
          //使用[name]占位符，可以保证不会输出同名js
          //占位符：[hash], [chunckhash], [name], [id], [query], [function]
          path: path.resolve(__dirname, '../dist'), // 打包后生成路径
          library: 'myLib', // 打包生成一个库，定义库的名称
          libraryTarget: 'this', // 打包库生成出来的规范，默认是var，保险起见可以用this
          // 如果是var， 只能用<script>标签引入打包出来的库
          // 如果是common，只能按照 commonjs 规范引入
          // 如果是amd，只能按照amd规范引入
          // 如果是umd，可以用上述三个规范引入
        },
        // 外部依赖，用于库打包较多，可以在打包后bundle的代码把依赖（import）的库排除
        externals: {
          jquery: 'jQuery',
        },
      }
      ```

    - 

  - 高级配置

  - 优化打包效率

  - 优化产出代码

  - 构建流程概述

  - babel

  

  ### webpack构建过程

  webpack运行是一个 **串行** 的过程，从启动到结束需要经历

  - 初始化：从配置文件和shell语句中读取与merge参数

  - 准备阶段：用上一步得到的参数初始化 `Compiler` 对象 ，注册加载所有插件，给对应的Webpack生命周期绑定Hook，

  - 开始编译：执行 `Compiler` 对象的 run 方法开始执行编译compiler.run 方法调用 compiler.compile，在compile 内实例化一个Compilation 对象，Compilation是做构建打包的事情，主要事情包括:

    - 查找入口:根据 entry 配置，找出全部的入口文件;
    - 编译模块: 从入口文件出发，根据文件类型和 loader 配置，使用对应 loader 对文件进行转换处理; 
    - 解析文件的 AST 语法树，找出文件依赖关系 ;
    - 递归编译依赖的模块。

    递归完后得到每个文件的最终结果，根据 entry 配置生成代码块 chunk;

  - 

  ###  为什么前端需要打包和构建

  代码层面

  - 体积更小(Tree-Shaking, 压缩,合并),加载更快
  - 编译高级语言和语法(TS, ES6+, 模块化, scss/less)
  - 兼容性和错误检查(Polyfill, postcss, eslint)

  开发层面

  - 统一、高效的开发环境
  - 统一的构建流程和产出标准
  - 集成公司构建规范（提测、上线）

  

  ### module chunk bundle的区别

  - module - 各个源码文件， webpack中一切皆模块
  - chunk - 多模块合并成的，根据不同入口文件进行依赖文件的解析，构建出来的chunk
  - bundle - 最终输出文件

  

  ### loader和plugin 的区别

  - loader - 模块转换器， 如 less -> css
  - plugin - 扩展插件， 如HtmlWebpackPlugin ，把js和css插入到html的插件

  

  ### 常用的loader和plugin

  loader： babel-loader, style-loader, url-loader， css-loader

  plugin：HotModuleReplacementPlugin（模块热替换插件），HtmlWebpackPlugin

  

  ### babel和webpack区别

  - babel - js新语法**编译工具** ，不关心模块化
  - webpack - **打包构建工具** ，是多个loader、plugin的集合

  

  ### babel-polyfill和babel-runtime 的区别

  - babel-polyfill **会污染全局**
  - babel-runtime **不会污染全局**
  - 产出第三方lib，使用babel-runtime

  

  ### webpack如何实现懒加载

  - import()  如

    - ```js
      const element = document.createElement('div');
      import('lodash')
      	.then(({ default: _ }) => { //当调用 ES6 模块的 import() 方法（引入模块）时，必须指向模块的 .default 值
        	element.innerHTML = _.join(['Hello', 'webpack'], ' ');
        	return element;
      	})
        .catch((error) => 'An error occurred while loading the component');
      
      // 或者
      const { default: _ } = await import('lodash');
      element.innerHTML = _.join(['Hello', 'webpack'], ' ');
      // 或者
      import(/* webpackChunkName: "lodash" */ 'lodash').then({ default: _ } => {const a = _.join(['Hello'], ' ')});
      ```

    - 

  - 结合Vue 或React异步组件

  - 结合router库异步加载路由

  ### 为何Proxy不能被Polyfill

  Proxy无法被Object.defineProperty模拟，无法降级

  ### webpack 优化构建速度 （用于生产环境）

  - 优化babel-loader
  - ignorePlugin
  - noParse
  - happyPack
  - PareallelUglifyPlugin

  ### webpack 优化构建速度 （不用于生产环境）

  - 自动刷新
  - 热更新HMR
  - DllPlugin

  ### webpack优化产出代码

  - 小图片base64编码
  - bundle加hash
  - 懒加载
  - 提取公共代码
  - 使用CDN加速
  - IgnorePlugin
  - 使用production
  - Scope Hosting