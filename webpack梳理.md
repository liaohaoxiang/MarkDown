## 基本配置

1. 配置webpack.common.js 公共配置

   1. 使用`webpack-merge` 合并production和dev的配置

2. webpack-dev-serve

   1. ```js
      devServer: {
          port: 8080,
          progress: true, // 打包条进度
          contentBase: './dist', // 告诉服务器从哪里提供内容
          open: true, // 自动打开浏览器
          hot: true, // 开启热更新
          compress: true, // 开启gzip压缩
          proxy: { // 跨域问题解决方案
            '/api': {
              target: 'http://localhost:3000',
              pathRewrite: { // 重写路径为空
                '^/api': ''
              },
            },
          },
        }
      ```

3. ES6语法转换。 module.rules 里配置处理多种文件的对象数组，这个对象包含 `test = 正则表达式/通配符` ，`use = 使用哪个loader去加载, 预设哪个选项` 

   1. 使用`babel-loader`

   2. ```js
      module: {
          rules: [
            {
              test: /\.js$/,
              exclude: /node_modules/,
              use: {
                loader: 'babel-loader',
                options: {
                  presets: ['@babel/preset-env']
                }
            	},
          	}
          ]
        },
      ```

4. 处理样式和图片

   1. ```js
      {
        test: /\.css$/,
        use: ['style-loader', 'css-loader', 'postcss-loader'], // loader执行顺序是从后往前
      }
      ```

   2. 图片

   3. ```js
      {
        test: /\.(png|jpg|gif)$/,
          use: {
            loader: 'url-loader',
              options: {
               limit: 5 * 1024, // 小于5kb就用base64，否则用file-loader产出url
               name: '[hash:16].[ext]', // 别名
               outputPath: 'assets/', // 打包到哪个目录
               publicPath: 'cdn.com', // 图片cdn地址
              }
          }
      }
      ```

   

## 高级配置

1. 多入口

   1. ```js
      entry: {
        index: path.resolve(__dirname, 'index.js'),
        other: path.resolve(__dirname, 'other.js'),
      }
      output: {
        filename: '[name].[contentHash:8.js]' // 多入口用占位符去区分产出文件
      }
      plugins: [
        // 多入口生成index.html
        new HtmlWebpackPlugin({
          template: 'public/index.html',
          filename: 'index.html',
          chunks: ['index'] // 只引用index.js到html里
        }),
        // 多入口生成other.html
        new HtmlWebpackPlugin({
          template: 'public/other.html'
          filename: 'other.html',
          chunks: ['other'] // 只引用other.js到html里
        }),
      ]
      ```

   2. 抽离压缩js和css

      1. ```js
         // 仅生产环境下有效
         const TerserPlugin = require('terser-webpack-plugin'); // 简洁js
         const MiniCssExtractPlugin = require("mini-css-extract-plugin"); // 提取css成单独文件
         const CssMinimizerPlugin = require('css-minimizer-webpack-plugin'); // 压缩css
         
         {
           // ...
           module: {
             rules: [
               {
                 test: /\.css$/,
                 exclude: /node_modules/,
                 // use: ['style-loader', 'css-loader', 'postcss-loader'],
                 // 使用mini-css-extract-plugin插件，将css提取到单独的css文件中
                 use: [MiniCssExtractPlugin.loader, 'css-loader', 'postcss-loader'],
               }
             ],
             plugins: {
               // 抽离css文件
               new MiniCssExtractPlugin({
                 filename: 'css/[contenthash:8].css', // 产出到dist的文件夹和名字
               }),
             },
             optimization: {
               minimizer: [
                 new TerserPlugin(), // 启用js压缩优化
                 new CssMinimizerPlugin() //启用css压缩优化
               ],
             },
           }
         }
         
         ```

   3. 抽离公共代码-chunk

      1. ```js
         optimization: { 
         	splitChunks: {
                 chunks: 'all',
                 /**
                  * initial 入口初始化时分割, 不对异步导入文件处理
                  * async 异步导入时分割, 对异步导入文件处理
                  * all 全部分割, 对所有文件都处理
                  */
         
                 // 缓存分组
                 cacheGroups: {
                   // 第三方库
                   vendors: {
                     name: 'chunk-vendors', // chunk名称
                     test: /[\\/]node_modules[\\/]/, // 正则匹配
                     priority: 1, // 优先级
                     minSize: 0, // 小于多少字节的文件不生成chunk
                     minChunks: 1, // 引用次数
                   },
                   // 公共模块
                   common: {
                     name: 'chunk-common',
                     priority: 0,
                     minSize: 0,
                     minChunks: 2,
                   }
                 }
               }
         }
         ```
         
      2. plugins里需要改动
   
         1. ```js
            plugins :[
                // 多入口生成index.html
                new HtmlWebpackPlugin({
                  template: 'public/index.html',
                  filename: 'index.html',
                  chunks: ['index', 'chunk-vendors', 'chunk-common'] // 引用index.js,第三方库chunk和公共文件chunk
                }),
                // 多入口生成other.html
                new HtmlWebpackPlugin({
                  template: 'public/other.html',
                  filename: 'other.html',
                  chunks: ['other', 'chunk-common'] // 只引用other.js和公共chunk
                }),
              ],
            ```

   4. 懒加载（异步加载js）
   
      1. ```js
         // index.js
         setTimeout(() => {
           import('./async.js').then(console.log(res.name))
         }, 1500) // 1.5s后加载模块async,同时产生一个chunk，浏览器里可以观察到新的js文件
         
         // async.js
         export const name = 'neo'
         ```

## 优化打包效率

1. 优化babel-loader

   1. 开启缓存

      1. ```js
          {
            test: /\.(js|jsx)$/,
            exclude: /node_modules/,
                use: {
                  loader: 'babel-loader',
                  options: {
                    presets: ['@babel/preset-env'],
                      cacheDirectory: true // 开启后，之后的构建会尝试读取缓存
                  }
                },
          },
         ```

         

   2. include或者exclude

2. IgnorePlugin

   1. 用于在import或require时，只生成正则表达式的模块。比如moment库，只用中文

   2. ```js
      /**
      *	requestRegExp: 匹配资源请求路径的正则表达式
      * [contentRegExp]: 可选，匹配资源上下文(目录)的正则表达式
      **/
      new webpack.IgnorePlugin(/^\.\/locale$/, /moment$/)
      ```

3. noParse

4. happyPack-多进程打包

   1. 

5. ParallelUglifyPlugin

6. 自动刷新

7. 热更新HMR

8. DllPlugin

## 优化产出代码



## 构建流程概述

## babel



### Tree-shaking

作用：摇掉没有引用的代码

开启需要

1. 使用`ES6规范`的import/export, 引用的第三方库也需要是 es 模块
3. 在`package.json`中配置`sideEffects = false`。为了避免有些副作用代码，而导致生产环境代码被误删除，所以有了这个配置
4. 注意`babel` 是否开启了配置 `modules` ，转变成其他规范
4. development 模式下中，需要开启`optimization.minimize` 和`optimization.minimize.minimizer = [new TerserWebpackPlugin()]`。product模式下默认启动

1. lodash
类似 import { throttle } from 'lodash' 就属于有副作用的引用，会将整个 lodash 文件进行打包。
优化方式是使用` import { throttle } from 'lodash-es' `代替` import { throttle } from 'lodash'`，lodash-es 将 Lodash 库导出为 ES 模块，支持基于 ES modules 的 tree shaking，实现按需引入。
2. ant-design
  ant-design 默认支持基于 ES modules 的 tree shaking，对于 js 部分，直接引入 import { Button } from 'antd' 就会有按需加载的效果。
  假如项目中仅引入少部分组件，import { Button } from 'antd' 也属于有副作用，webpack不能把其他组件进行tree-shaking。这时可以缩小引用范围，将引入方式修改为 import { Button } from 'antd/lib/button' 来进一步优化。
  链接：https://juejin.cn/post/6996816316875161637

