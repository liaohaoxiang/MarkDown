# JavaScript基础语法总结

### 类型转换

![js类型转换](./type.png)

- 加法的特殊性

  - 当一侧为String类型，被识别为字符串拼接，并会优先将另一侧转换为字符串类型。 `1+'3' = '13'`
  - 当一侧为Number类型，另一侧为原始类型，则将原始类型转换为Number类型。 `1 + true = 2`
  - 当一侧为Number类型，另一侧为引用类型，将引用类型和Number类型转换成字符串后拼接。 `1 + {} = 1[object Object]`

- 使用 == 比较中的5条规则

  - NaN和其他任何类型比较永远返回false  `NaN == ? //false`
  - Boolean 和其他任何类型比较，Boolean 首先被转换为 Number 类型  `1==true //true `
  - String和Number比较，先将String转换为Number类型。 `1 == '1' //true` `1 == 'haox' // false`
  - null == undefined比较结果是true，除此之外，null、undefined和其他任何结果的比较值都为false
  - 原始类型和引用类型做比较时，引用类型会依照ToPrimitive规则(先用valueOf,再用toString)转换为原始类型

  转换优先级(由高到底): NaN->引用类型->布尔值->字符串->数值

如 'haox' == true,先转换布尔值 true = 1, 变成'haox' == 1,然后转换Number('haox') = NaN, 即NaN == 1 ,结果为false

#### Symbol

无字面量语法，构造使用 Symbol(description) 

```ts
let name = Symbol.for('foo'); //全局symbol注册, 即没有foo这个symbol时才会创建，否则重用
Symbol.keyFor(name) // 查询全局symbol

let obj = {
  [name]: 'foo value'
}
```



### 位操作符

> ECMAScript中所有 **数值** 都是以 IEEE 754的64位格式存储，但确是把值转换成32位整数操作。
>
> 其中第32位（即左边第一位）是符号位，0为整数 1为负数

#### 正负数转换 （题目：求证-13 >> 2 === -4）

负值以 **二补数** 存储 （二进制补码），步骤以下

1. 找出绝对值的二进制表示 （-13而言，就是确定13的二进制数）

   13 === 0000 … 0000 1101

2. 找到数值的**反码**，即 **将1变0，0变1**

   0000 … 0000 1101 => 1111 …  1111 0010 (32位)

3. 最后将**反码+1** 

   -13 === 1111 … 1111 0011

#### 位左移和右移

> 正数：左移 数变大，右移 数变小；负数相反

题目 -13 >> 2 表示将-13右移两位，上面求出-13的二补数后，可以进行右移

```1111 … 1111 0011 >> 2  === 1111 … 1111 1100  ```

这个数是一个负数，我们得找到它的正数，逆运算上面的方法（正负数转换）

1. 先减一

   得到 1111 … 1111 1011

2. 取反码

   0000 … 0000 0100

3. 得正数

   0100 = 4

那么这个负数其实是-4，求证完成



### 其他操作符

|           | 逻辑非（！）                  |
| --------- | ----------------------------- |
| 对象      | false                         |
| 字符串    | 空字符串 = true, 其他 = false |
| 整形      | 0 = true，其他false           |
| null      | true                          |
| NaN       | true                          |
| undefined | true                          |



#### 乘法操作符

- 超过数值不能计算，返回Infinity或者-Infinity

- 有NaN，返回NaN

- ```js
  Infinity * 0 = NaN
  Infinity * 正数 = Infinity，反之
  Infinity * Infinity = Infinity
  ```

- 不是整形，先使用Number()转型，再运用上述规则

#### 位操作

- 按位非（~），对一个数值进行操作

  返回数值的**补数** （即1变0，0变1）, 所有数字结果都是 -(x + 1)

- 按位与（&），两个数值运算

  对比两个数的32位二进制数，只有在**都有1的情况才变成1，其余变0**

- 按位或（|）





### 运算法则

```&& 和 || ```

"与" 和 "或" 运算,优先级 ```&&  >  ||``` 



&& 在等于false时候返回, 如果

```js
var a = 1 && 2 // 返回2
```

因为 第一个条件 是true ,所以无论第二个条件是true 还是 false 都返回 第二个



如果

```js
var b = 0 && 1 // 返回0
```

遇到false就返回



口诀: **前真就输出后 ,前假会中断 输出前**



|| 在等于ture的时候返回

```js
var c = 0 || 1 //返回1
```

如果第一个条件是 false 那么无论第二个条件是啥都返回第二个



```js
var d = 1 || 0
```

如果第一个条件是true 不会向下执行,直接返回



口诀: **前真为真,前假强制出后**



## 时间

```js
/**
 * 格式化时间
 * @param {date} dt 时间
 * @param {string} format 时间格式
 */
export function formatDate(dt, format) {
    var date = {
        "M+": dt.getMonth() + 1,
        "d+": dt.getDate(),
        "h+": dt.getHours(),
        "m+": dt.getMinutes(),
        "s+": dt.getSeconds(),
        "q+": Math.floor((dt.getMonth() + 3) / 3),
        "S+": dt.getMilliseconds()
    };
    if (/(y+)/i.test(format)) {
        format = format.replace(RegExp.$1, (dt.getFullYear() + '').substr(4 - RegExp.$1.length));
    }
    for (var k in date) {
        if (new RegExp("(" + k + ")").test(format)) {
            format = format.replace(RegExp.$1, RegExp.$1.length == 1
                ? date[k] : ("00" + date[k]).substr(("" + date[k]).length));
        }
    }
    return format;
}

formatDate(new Date(), "YYYY-MM-dd")
```



### 类型转换

```js
/**
	* Boolean
	**/
// 强制转换为Boolean 用 !!
var bool = !!"c";
console.log(typeof bool); // boolean

/**
	* Number
	**/
// 强制转换为Number 用 +   (弃用)
//数字类型转换推荐用 Number() 或者 parseInt() 函数
var num = +"1234";
console.log(typeof num); // number

/**
	* String
	**/
// 强制转换为String 用 ""+
var str = ""+ 1234;
console.log(typeof str); // string

/**
	* 数组
	**/
// 使用数组展开符 ... 来拷贝数组
// good
const itemsCopy = [...items];

/**
	* 对象
	**/
使用对象扩展操作符（spread operator）浅拷贝对象
// good
const original = { a: 1, b: 2 };
const copy = { ...original, c: 3 }; // copy => { a: 1, b: 2, c: 3 }

const { a, ...noA } = copy; // noA => { b: 2, c: 3 }

```



### 拆解URL参数中queryString

```js
const queryString = (str)=>{
    const obj = {}
    str.replace(/([^?&=]+)=([^&]+)/g, (_, k, v) => (obj[k] = v))
    return obj
}

const dismantle = (url) => {
     const aimUrl = url.split('?').pop().split('#').shift().split('&');
     const res = {};
     aimUrl.forEach(item => {
          const [key, val] = item.split('=');
          res[key] = val;
     });
     return res;
}
```



### 运算符优先级

| 优先级                                                       | 运算类型                                                     | 关联性           | 运算符        |
| :----------------------------------------------------------- | :----------------------------------------------------------- | :--------------- | :------------ |
| 21                                                           | [`圆括号`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Grouping) | n/a（不相关）    | `( … )`       |
| 20                                                           | [`成员访问`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Property_Accessors#点符号表示法) | 从左到右         | `… . …`       |
|                                                              | [`需计算的成员访问`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Property_Accessors#括号表示法) | 从左到右         | `… [ … ]`     |
|                                                              | [`new`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new) (带参数列表) | n/a              | `new … ( … )` |
|                                                              | [函数调用](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Functions) | 从左到右         | `… ( … )`     |
|                                                              | [可选链（Optional chaining）](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Optional_chaining) | 从左到右         | `?.`          |
| 19                                                           | [new](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/new) (无参数列表) | 从右到左         | `new …`       |
| 18                                                           | [后置递增](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Increment)(运算符在后) | n/a              | `… ++`        |
|                                                              | [后置递减](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Decrement)(运算符在后) |                  | `… --`        |
| 17                                                           | [逻辑非](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Logical_Operators#Logical_NOT) | 从右到左         | `! …`         |
|                                                              | [按位非](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators#Bitwise_NOT) | `~ …`            |               |
|                                                              | [一元加法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Unary_plus) | `+ …`            |               |
|                                                              | [一元减法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Unary_negation) | `- …`            |               |
|                                                              | [前置递增](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Increment) | `++ …`           |               |
|                                                              | [前置递减](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Decrement) | `-- …`           |               |
|                                                              | [typeof](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/typeof) | `typeof …`       |               |
|                                                              | [void](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/void) | `void …`         |               |
|                                                              | [delete](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/delete) | `delete …`       |               |
|                                                              | [await](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await) | `await …`        |               |
| 16                                                           | [幂](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Exponentiation) | 从右到左         | `… ** …`      |
| 15                                                           | [乘法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Multiplication) | 从左到右         | `… * …`       |
|                                                              | [除法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Division) | `… / …`          |               |
|                                                              | [取模](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Remainder) | `… % …`          |               |
| 14                                                           | [加法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Addition) | 从左到右         | `… + …`       |
|                                                              | [减法](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Arithmetic_Operators#Subtraction) | `… - …`          |               |
| 13                                                           | [按位左移](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators) | 从左到右         | `… << …`      |
|                                                              | [按位右移](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators) | `… >> …`         |               |
|                                                              | [无符号右移](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators) | `… >>> …`        |               |
| 12                                                           | [小于](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Comparison_Operators#Less_than_operator) | 从左到右         | `… < …`       |
|                                                              | [小于等于](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Comparison_Operators#Less_than__or_equal_operator) | `… <= …`         |               |
|                                                              | [大于](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Comparison_Operators#Greater_than_operator) | `… > …`          |               |
|                                                              | [大于等于](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Comparison_Operators#Greater_than_or_equal_operator) | `… >= …`         |               |
|                                                              | [in](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/in) | `… in …`         |               |
|                                                              | [instanceof](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/instanceof) | `… instanceof …` |               |
| 11                                                           | [等号](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Comparison_Operators#Equality) | 从左到右         | `… == …`      |
|                                                              | [非等号](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Comparison_Operators#Inequality) | `… != …`         |               |
|                                                              | [全等号](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Comparison_Operators#Identity) | `… === …`        |               |
|                                                              | [非全等号](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Comparison_Operators#Nonidentity) | `… !== …`        |               |
| 10                                                           | [按位与](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators#Bitwise_AND) | 从左到右         | `… & …`       |
| 9                                                            | [按位异或](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators#Bitwise_XOR) | 从左到右         | `… ^ …`       |
| 8                                                            | [按位或](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Bitwise_Operators#Bitwise_OR) | 从左到右         | `… | …`       |
| 7                                                            | [逻辑与](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Logical_Operators#Logical_AND) | 从左到右         | `… && …`      |
| 6                                                            | [逻辑或](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Logical_Operators#Logical_OR) | 从左到右         | `… || …`      |
| 5                                                            | [空值合并](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing_operator) | 从左到右         | `… ?? …`      |
| 4                                                            | [条件运算符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Conditional_Operator) | 从右到左         | `… ? … : …`   |
| 3                                                            | [赋值](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Assignment_Operators) | 从右到左         | `… = …`       |
| `… += …`                                                     |                                                              |                  |               |
| `… -= …`                                                     |                                                              |                  |               |
| `… **= …`                                                    |                                                              |                  |               |
| `… *= …`                                                     |                                                              |                  |               |
| `… /= …`                                                     |                                                              |                  |               |
| `… %= …`                                                     |                                                              |                  |               |
| `… <<= …`                                                    |                                                              |                  |               |
| `… >>= …`                                                    |                                                              |                  |               |
| `… >>>= …`                                                   |                                                              |                  |               |
| `… &= …`                                                     |                                                              |                  |               |
| `… ^= …`                                                     |                                                              |                  |               |
| `… |= …`                                                     |                                                              |                  |               |
| `… &&= …`                                                    |                                                              |                  |               |
| `… ||= …`                                                    |                                                              |                  |               |
| `… ??= …`                                                    |                                                              |                  |               |
| 2                                                            | [yield](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/yield) | 从右到左         | `yield …`     |
| [yield*](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/yield*) | `yield* …`                                                   |                  |               |
| 1                                                            | [展开运算符](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Spread_operator) | n/a              | `...` …       |
| 0                                                            | [逗号](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Operators/Comma_Operator) | 从左到右         | `… , …`       |