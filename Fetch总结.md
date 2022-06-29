# Fetch API用法



## 定制HTTP请求

`fetch()`的第一个参数是 URL，还可以接受第二个参数，作为配置对象，定制发出的 HTTP 请求。

```js
window.fetch(url, optionObj)
```

参数列表如下

```js
const response = fetch(url, {
  method: "GET",
  headers: {
    "Content-Type": "text/plain;charset=UTF-8"
  },
  body: undefined,
  referrer: "about:client",
  referrerPolicy: "no-referrer-when-downgrade",
  mode: "cors | same-origin | no-cors",  // 指定请求的模式, no-cors只能使用有限的几个简单标头，不能添加跨域的复杂标头
  credentials: "same-origin", // 指定是否发送 Cookie, include：一律发送 Cookie , omit：一律不发送
  cache: "default", // 指定如何处理缓存
  redirect: "follow", // 指定 HTTP 跳转的处理方法
  integrity: "",
  keepalive: false, // 用于页面卸载时，告诉浏览器在后台保持连接，继续发送数据
  signal: undefined
});
```

- POST请求

  - 提交json数据

    - ```js
      const user =  { name:  'John', surname:  'Wick'  };
      const response = await fetch(url, {
        method: 'POST', // HTTP 请求的方法
        headers: {
          'Content-Type': 'application/json;charset=utf-8',
        }, // 一个对象，用来定制 HTTP 请求的标头
        body: JSON.stringify(user), // 请求的数据体
      });
      
      const json = await response.json();
      ```

  - 提交表单

    - ```js
      const form = document.querySelector('form');
      
      const response = await fetch('/users', {
        method: 'POST',
        body: new FormData(form)
      })
      ```

