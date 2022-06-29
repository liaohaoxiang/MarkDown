- `typeof` 和 `instanceof`

  - typeof 对于**原始类型的判断是准确的**, 但是对于引用类型如：null，function是不准确的

    - ```js
      typeof 'neo'; // string
      typeof 123; // number
      typeof true; // boolean
      typeof undefined; // undefined
      typeof null; // object 【null也是object】
      typeof new Object(); // object
      typeof new Array(); // object 【其他object类型除了function都是object】
      typeof (()=>''); // function
      ```

  - instanceof 对于**引用类型判断是准确的**, 返回布尔值。对于引用值，他们都在Obejct原型链上，所以`instanceof Object` 必然为true。

    只能判断某一个实例是不是某一个构造函数的实例

    - ```js
      // object instanceof constructor
      new Array() instanceof Array; // true
      new RegExp() instanceof RegExp; // true
      ```

  - 准确判断类型：**使用Object.prototype.toString.call**

    - 返回一个字符串 `[object Type]`

- **内存泄漏问题** （*红宝书第四版 电子版 849页）*

  - **意外声明全局变量**，即没有用任何关键字声明变量

    - ```js
      function setName() {
        name = 'neo';
      }
      ```

    - 这里会被解析为 `window.name = 'neo'` （浏览器上） ，只要window对象不被删除，那么就会一直存在。window对象是浏览器里的`全局上下文` ，只有在关闭或者销毁时才会清除。

  - **定时器** ，定时器通过闭包引用外部变量

    - ```js
      let name = 'holk';
      setInterval(() => {
        console.log(name); // 通过 上下文获取到全局变量 name，导致定时器一直运行，就会一直占用内存
      }, 100)
      ```

  - **闭包**

    - ```js
      let outer = function() {
        let name = 'holk';
        return function(){ // 闭包
          return name;  // 一直引用着outer函数上下文里的name，导致outer函数执行完成后无法销毁上下文，形成闭包
        }
      }
      outer(); // 只要执行，就会形成闭包
      ```

- - - -

