### JavaScript面向对象-继承

#### 原型链继承

通过 **对象实例** 的`__proto__` 属性，可以获取到原型链上的 **原型对象** ，进而得到继承关系的属性和方法

```js
function Class() {};
const instance = new Class();
instance.__proto__ === Class.prototype // true 
```

通过将子类的prototype用字面量赋值方式修改，就可以通过原型链达到继承。

```js
function SuperType() {
  this.property = true;
}
SuperType.prototype.getSuperValue = function() {
  return this.property;
};
function SubType() {
  this.subproperty = false;
}
// 继承SuperType
SubType.prototype = new SuperType();

const subInstance = new SubType();
console.log(subInstance.getSuperValue()); // true
```

上述代码，subType继承了superType原型上的方法。因为subType的原型对象已经是SuperType的实例了，

```js
new SuperType() === superInstance; //true
superInstance.__proto__ === SuperType.prototype; //true
subInstance.__proto__.__proto__.getSuperValue === superInstance.__proto__.getSuperValue === SuperType.prototype.getSuperValue // 通过两个原型链查到getSuperValue
```

##### 缺点

- 原型中包含引用值时，会在【全部实例】上共享引用，导致意料之外的修改
- 子类在实例化时不能给父类的构造函数传参

##### 总结：原型链继承方式基本不会被单独使用



#### 盗用构造函数 （经典继承）

基本思路很简单:在子类构造函数中【调用父类构造函数】。因为毕竟函数就是在特定上下文中执行代码的简单对象，所以可以使用 apply()和 call()方法以新创建的对象为上下文执行构造函数。

```js
function SuperType(name) {
	this.colors = ["red", "blue", "green"];
  this.name = name
}
function SubType() {
	SuperType.call(this, 'Neo');
}
let instance1 = new SubType();
instance1.colors.push("black");
console.log(instance1.colors); // "red,blue,green,black"
let instance2 = new SubType();
console.log(instance2.colors); // "red,blue,green"
```

通过使用 call() /apply()方法，SuperType 构造函数在为 SubType 的实例创建的新对象的上下文中执行了。这相当于新的 SubType 对象上运行了 SuperType()函数中的所有初始化代码。结果就是每个实例都会有自己的 colors 属性。

##### 优点

- 传递参数，在子类调用父类构造函数时，可以把参数加入到需要继承的父类中
- 每个实例拥有独立的引用值属性

##### 缺点

- 使用构造函数模式自定义类型的问题：必须在构造函数中定义方法，因此函数不能重用。
- 子类也不能访问父类原型上**定义的方法**，因此所有类型只能使用构造函数模式



#### 组合继承 （伪经典继承）

> 综合了原型链和盗用构造函数，将两者的优点集中了起来

基本的思路是使用原型链**继承原型上的属性和方法**，而通过盗用构造函数**继承实例属性**。

```js
function SuperType(name){
  this.name = name;
  this.colors = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function() {
  console.log(this.name);
};
function SubType(name, age){
  // 继承属性
  SuperType.call(this, name);
	this.age = age;
}
// 继承方法
SubType.prototype = new SuperType();
SubType.prototype.constructor = SubType; // 修正构造函数

SubType.prototype.sayAge = function() { // 添加子类新方法
  console.log(this.age);
};
let instance1 = new SubType("Nicholas", 29);
instance1.colors.push("black");
console.log(instance1.colors); // "red,blue,green,black"
instance1.sayName(); // "Nicholas";
instance1.sayAge(); // 29
let instance2 = new SubType("Greg", 27);
console.log(instance2.colors); // "red,blue,green"
instance2.sayName(); // "Greg";
instance2.sayAge(); // 27
```

##### 总结

组合继承弥补了原型链和盗用构造函数的不足，是 JavaScript 中使用最多的继承模式。而且组合继 承也保留了 instanceof 操作符和 isPrototypeOf()方法识别合成对象的能力。



#### 原型式继承 && 寄生式继承

```js
function object(o) {
  function F() {};
  F.prototype = o;
  return new F();
}

function createAnother(original){
  let clone = object(original); // 通过调用函数创建一个新对象
  clone.sayHi = function() { // 以某种方式增强这个对象
    console.log("hi"); 
  };
  return clone; // 返回这个对象 
}
let person = {
  name: "Nicholas",
  friends: ["Shelby", "Court", "Van"]
};
let anotherPerson = createAnother(person);
anotherPerson.sayHi(); // "hi"
```

这个例子基于 person 对象返回了一个新对象。新返回的 anotherPerson 对象具有 person 的所 有属性和方法，还有一个新方法叫 sayHi()。



### 寄生式组合继承 (目前ES6 class的实现方式)

**组合继承**其实也存在效率问题。最主要的效率问题就是父类构造函数始终会被调用两次:

- 一次在是 修改子类原型时调用
- 另一次是在子类构造函数中调用。

其中，第一次的` new SuperType()`，会出现一个问题：子类的原型上，会出现父类构造函数上的属性，且为undefined

```js
function SuperType(name){
  this.name = name;
  this.colors = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function() {
  console.log(this.name);
};
function SubType(name, age){
  // 继承属性
  SuperType.call(this, name);
	this.age = age;
}
// 继承方法
SubType.prototype = new SuperType();
SubType.prototype.constructor = SubType;

let instance = new SubType("Nicholas", 29);
```

这里由于`SubType.prototype = new SuperType();` 这段代码的作用，导致在 `SubType.prototype.name === undefined` 。虽然在 instance 上有一个`name = 29`属性，会优先被引擎查询到，但这始终是影响了子类的原型。



寄生组合继承结合了【寄生式继承】和【组合继承】，其实就是把`SubType.prototype = new SuperType();` 这段代码，改为一个对象**newObj**，**这个newObj需要符合** `newObj.__proto__ = superType.prototype`

```js
function SuperType(name){
  this.name = name;
  this.colors = ["red", "blue", "green"];
}
SuperType.prototype.sayName = function() {
  console.log(this.name);
};
function SubType(name, age){
	this.age = age;
}

// 继承方法
function inheritPrototype(subType, superType){
    var newObj = Object.create(superType.prototype); // 创建了一个以superType.prototype为__proto__的对象
    newObj.constructor = subType;             // 修正原型的构造函数
    subType.prototype = newObj;               // 将子类的原型替换为这个新对象
}
inheritPrototype(SubType, SuperType);

let instance = new SubType("Nicholas", 29);
```



附上babel的实现

```js
function _inherits(subClass, superClass) {
  // step1: 判断需要继承的父类，要么是构造函数，要么是null
  if (typeof superClass !== "function" && superClass !== null) {
    throw new TypeError("Super expression must either be null or a function");
  }
  // step2: 子类原型指向【以父类原型为__proto__的新对象】,并修复子类原型的构造函数
  subClass.prototype = Object.create(superClass && superClass.prototype, {
    constructor: { value: subClass, writable: true, configurable: true },
  });
  // step3: 子类原型的对象属性为不可再写
  Object.defineProperty(subClass, "prototype", { writable: false });
  // step4: 如果父类不是null，修改子类构造函数的原型链为父类构造函数，继承静态属性和方法
  if (superClass) _setPrototypeOf(subClass, superClass);
}

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

