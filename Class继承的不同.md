### ES5与ES6的js继承不同点

1. > ES5里的构造函数就是一个普通的函数，可以使用new调用，也可以直接调用，而ES6的class不能当做普通函数直接调用，必须使用new操作符调用 

   - 原因是因为class在编译后，会有一个`_classCallCheck` 函数，判断实例是否instanceof构造函数

   - ```js
     function _classCallCheck(instance, Constructor) {
       if (!(instance instanceof Constructor)) {
         throw new TypeError("Cannot call a class as a function");
       }
     }
     ```

2. > ES5的原型方法和静态方法默认是可枚举的，而class的默认不可枚举，如果想要获取不可枚举的属性可以使用Object.getOwnPropertyNames方法

   - class编译后，会有一个`_defineProperties` 函数，该函数使用`Object.defineProperty()` 去定义class里的方法和静态方法

   - ```js
     function _defineProperties(target, props) {
       for (var i = 0; i < props.length; i++) {
         var descriptor = props[i];
         // 设置该属性是否可枚举，设为false则for..in、Object.keys遍历不到该属性
         descriptor.enumerable = descriptor.enumerable || false;
         // 默认可配置，即能修改和删除该属性
         descriptor.configurable = true;
         // 设为true时该属性的值能被赋值运算符改变
         if ("value" in descriptor) descriptor.writable = true;
         Object.defineProperty(target, descriptor.key, descriptor);
       }
     }
     ```

3. > 子类可以直接通过__proto__找到父类，而ES5是指向Function.prototype：
   >
   > ES6：`Sub.__proto__=== Sup`
   >
   > ES5：`Sub.__proto__ === Function.prototype`

   - 因为编译后，`_inherits`函数会把父类和子类的构造函数绑定在一起

   - ```js
     if (superClass) _setPrototypeOf(subClass, superClass);
     function _setPrototypeOf(o, p) {
         _setPrototypeOf =
             Object.setPrototypeOf ||
             function _setPrototypeOf(o, p) {
                 o.__proto__ = p;
                 return o;
             };
         return _setPrototypeOf(o, p);
     }
     ```

4. > ES5继承是：先建子类实例对象（new Sub()），再执行父类构造函数（`Super.call(this)`）继承属性；然后通过修改`Sub.prototype = Object.create(Super.prototype, { constructor: Sub})` 继承方法
   >
   > ES6继承是：先通过`_inherits`将父类的属性和方法继承（通过`__proto__，连接Sub.prototype 和Super.prototype`），再通过父类创建子类实例对象（通过`Relect.construct或者Super.apply`），Sub构造函数会返回一个基于Super创建的对象（该对象的`__proto__` 是指向`Sub.prototype`）

5. > class不存在变量提升，父类必须定义在子类之前

