---

@Author: Neo Holk

@Date: 2021-04-27 13:54:16

@LastEditors: Neo Holk

@LastEditTime: 2021-04-27 17:45:41

@Description:

---

## 在不同项目下安装 tailwindcss

### vanillaJS 中

1. 使用`mkdir [filename]` 新建文件夹
2.  使用`npm init -y ` 初始化 package.json
3. 安装必要的 npm 包 `npm install -D tailwindcss postcss-cli autoprefixer`
4. 初始化 tailwind 的文件 `npx tailwindcss init -p`
5. 新增 css 文件夹, 目录新增 style.css 载入 tailwindcss 核心文件

```
@tailwind base;
@tailwind components;
@tailwind utilities;
```

6. 新增 script `"watch": "postcss css/style.css -o dist/style.css --watch"` 运行 npm run watch , tailwind 的源码会放在 dist/style.css 下（一共 18K+行）

### Vue 中(使用vite)
1. npm init @vitejs/app '你的项目名称'
2. 同上 ```3,4步骤``` 
3. 在src里新增 ```index.css```,写入上面```步骤5``` 的代码
4. ```tailwind.config.js``` 里的purge写入
```
purge: [
    './index.html',
    './src/**/*.{vue,js,ts,jsx,tsx}'
  ],
```
5. 在```./src/main.js/``` 加入```import './index.css'```
6. 运行```npm run dev```







> postcss 是用于处理 tailwindcss 的输出工具;

> autoprefixer 是用于对不同样式在不同浏览器中的支持程度，如-wbekit-前缀,如不需要，可以在`postcss.config.js` 文件中去掉

### purge CSS 设置

该功能是可以按需引入 tailwindcss，从而减少打包出来的代码

1. 打开 tailwind.config.js 在 purge 里输入需要使用到 tailwind 的文件目录
   `'./dist/**/**.html'`
2. 执行新 script 命令`"build": "NODE_ENV=production postcss css/style.css -o dist/style.css"`
3. 在./dist/style.css 看到代码行缩小,使用终端命令 `ls -lah dist/`可以看到输出 style.css 文件只有 11KB

### 打包自己的 css 模块

1. 把一个元素上的所有 tailwind class 起一个别名(alias), 如`text-9xl text-center bg-black text-white` ,直接换成 `header`
2. 在引入@tailwind 的 style 文件下方，建立`.header`,使用`@apply`关键字把头部所有用到的 tailwind 类都放进去
3. 重新打包后发现有一个`.header` 类，它总结了我们刚刚输入的 tailwind 类名,并返回给我们的别名 header 来为使用
