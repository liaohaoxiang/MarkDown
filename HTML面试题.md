# HTML5面试题



### src和href的区别

**href(Hypertext reference 超文本引用)** 是建立与外部资源的关系,用于与**本页面关联的引用** . 用作**连接前往**

如 link a

```html
<link href="style.css" rel="stylesheet"/>
<a href="google.com" />
```

**src(Source 来源)** 是外部资源的url路径,**用作拿取资源**

```html
<img src="image.jpg"/>
<script src="jquery.js"/>
```



### DOCTYPE的作用, HTML5为什么只写```<!DOCTYPE html>```

DOCTYPE 是用来**声明文档类型和 DTD 规范(Document Type Definition)**的,这个声明位于HTML 文档中的第一行,**不是HTML标签**

告诉浏览器使用哪个版本的**HTML规范来渲染文档** ,不存在这个DOCTYPE或者不正确会导致HTML**以混杂模式** 呈现

*标准模式(Standards mode)*以**浏览器支持的最高标准运行** ; *混杂模式(Quirks mode)*中页面会向后兼容方式运行



HTML5不基于 标准通用标记语言(SGML),因此不需要对DTD规范进行引用,但是**需要DOCTYPE来规范浏览器行为** 



### 行内(内联)元素, 块级元素, 空元素

*行内元素* 

`span, a, b, i, em, br, input, textarea, big, strong, small, br, img, script, button, label, select ` 

- 本身属性`display: inline`
- 和其他 **行内元素** 从左到右 一行显示,**不可以控制宽度,高度 等其他css属性** , 可以设置内外边距左右(margin padding)

*块级元素* 

```
address(联系方式), article(文章内容),aside(伴随内容), audio(音频),blockquote(块级引用), fieldset(表单元素),footer(页尾)
header(页头), hr(水平分割线), output(表单输出), p(行) ,pre(预格式化文本), section(一个页面区段), table(表格),video(视频)
canvas, dd, div, dl,form , h1-h6, noscript, ol, ul
```

- 本身属性 `display: block`
- 独占一行, 每个块级元素会从新的一行重新开始,**从上到下排列**, **可以控制宽高**等属性

*空元素*

- 开始标签中关闭,没有闭合标签 `如<br>, <hr>, <link>, <meta>, <img> , <input>,`



### async和defer的作用

- 如果脚本没有 `async`  或 `defer` ，浏览器会立即加载并执行指定的脚本。
- `defer` 属性表示延迟执行引入的script，当整个document解析完成后在再执行脚本文件，在DomContentLoaded事件触发前完成，多个脚本按顺序执行
- `async` 属性表示异步执行引入的script，与defer的区别是如果已经加载好，就会立刻执行，阻塞文档解析，但是加载的过程不阻塞，多个脚本无法保证执行顺序



### 回流（重排）与重绘

- ![浏览器渲染](/Users/diu/Desktop/MarkDownFile/面试题/浏览器渲染.png)
- 两者关系： **回流必将引起重绘，而重绘不一定会引起回流。**
- 回流（重排， **reflow/layout**）
  - 回流当render tree中的一部分或全部**因为元素的规模尺寸、布局、隐藏等改变时**，浏览器需要**重新计算**各节点和css具体的大小和位置，**重新渲染**部分DOM或全部DOM的过程。回流也被称为重排，其实从字面上来看，重排更容易让人形象易懂（即重新排版整个页面）。
- 重绘（**painting/repaint**）
  - 当页面元素样式改变**不影响元素在文档流中的位置**时（如background-color，border-color，visibility），浏览器只会将新样式赋予元素并进行重新绘制操作。
- **减少回流和重绘**
  - 合并样式修改
  - 不用table布局，修改table会回流整个table布局
  - 批量操作DOM，在**脱离标准流（也叫DOM离线）**后，对元素进行的多次操作，**不会触发回流**，等操作完成后，再将元素放回标准流。
    - 脱离标准流的操作： 1.隐藏元素 2.使用文档碎片（`document.createDocumentFragment`） 3.拷贝节点 (`cloneNode`)
  - 避免多次触发（强制回流）
    - offsetTop，scrollTop，clientTop，getComputedStyle() 这些属性都会触发



### DOMContentLoaded和Load 之间的区别

- DOMContentLoaded：在初始HTML文档被**完全加载和解析**完成后，触发DOMContentLoaded，无需等待样式表，图像和子框架
- Load： 当所有资源加载完成后触发



### 浏览器内多个标签之间通信

- 原理：实现多标签通信，本质上都是通过中间人的模式实现
  - 1. `websocket`协议，`websocket`可以实现服务器推送，所以可以用这个来做中介，由服务器向其他标签页转发
    
    2. `ShareWorket` 方式，H5的新特性**共享worker**，会在页面存在的生命周期内创建一个`唯一线程` ，通过共享线程来实现数据交换
    
       1. ```js
          // main.js
          const myWorker = new SharedWorker('worker.js');
          myWorker.port.start(); // open port
          
          myWorker.port.postMessage(123)
          ```
    
       2. ```js
          // worker.js
          
          onconnect = function(e) {
            const port = e.ports[0];
            port.onmessage = function(e) {
              const workerResult = e.data[0] + 1
               port.postMessage(workerResult); // 传过来的123会自增1返回到worker
            }
          }
          ```
    
    3. `localStorage` 方式， 可以对`storage或者onStorage`变化事件进行监听，仅当另外的标签页修改数据，监听事件获取到数据



### 主流浏览器内核以及私有属性CSS前缀

```
mozilla内核 (firebox,flock) 前缀：-moz
webkit内核  (safari, chrome) 前缀：-webkit
opera内核  (opera) 前缀-o
trident内核 (ie浏览器) 前缀： -ms
```



### 前端性能优化

- 页面内容方面
  - 文件合并，css雪碧图，使用base64，等方式来减少HTTP请求数，避免过多的请求
  - 使用DNS缓存等机制减少DNS查询次数
  - 设置缓存策略，对常用不变的资源进行缓存
  - 设置延迟加载，减少页面首屏加载时需要请求的资源
  - 设置预加载
- 服务器方面
  - 使用CDN方式
  - 启用Gzip等方式对传输资源进行压缩，减少文件体积
  - 减少cookie大小
- css和js
  - 样式表放在head标签，减少页面首次渲染时间
  - 避免使用@import标签
  - 尽量把js脚本放在页面底部，使用defer或async属性，避免脚本加载和执行阻塞页面的渲染

