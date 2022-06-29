### Next.js学习

1. 手脚架搭建

   1. ```shell
      npx create-next-app@latest --typescript .
      ```

   2. 在gtihub拉取资源库到本地后，使用上述命令。（后面的一个点代表是在git拉取的文件夹中生成项目）

1. 配置 eslint stylelint prettier
   1. eslint在项目中已经自带，在`.eslintrc` 中加入 `"extends": ["next/core-web-vitals", "eslint:recommended"]`
   2. styleint: 在vscode中安装插件；执行命令`yarn add stylelint stylelint-config-standard-scss -D`
   3. 