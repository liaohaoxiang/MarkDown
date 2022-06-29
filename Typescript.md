# TypeScript



### interface和type（类型别名）的区别

- 相同点

  - 都可以用来描述对象或函数的类型

    - ```tsx
      interface Point {
        x: number;
        y: number;
      }
      
      interface SetPoint {
        (x: number, y: number): void;
      }
      
      type Point = {
        x: number;
        y: number;
      };
      
      type SetPoint = (x: number, y: number) => void;
      ```

- 不同点

  - interface**无法声明** 基本类型，联合类型，交叉类型（| 和 &），还有使用typeof获取实例类型（type T = typeof div）
  - interface 能够**extend 扩展声明** ，即声明两个**同名**类型，使用时会结合两个声明里的类型

- 文档描述最大不同点 [类型别名](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-aliases)

  - type cannot be re-opened to add new properties  interface is always extendable.

  - 类型别名不可以修改添加属性（可以用交叉类型实现，但是出来的是新的类型别名）， 接口是一直可以

  - ```tsx
    interface Window {
      title: string
    }
    
    interface Window {
      ts: TypeScriptAPI
    }
    // it's ok
    
    type Window = {
      title: string
    }
    
    type Window = {
      ts: TypeScriptAPI
    }
    // Error: Duplicate identifier 'Window'.
    ```

  - 

## Utility Types（工具类型）

定义：用泛型传入一个类型，**然后工具类型** 进行某种操作

### `Parameters<T>`

- 作用
- 

### `Partial<T>`

- 作用： 把传入的泛型，变成全部为可选属性

- 实现：

  - ```tsx
    type Partial<T> = {
      [P in keyof T]?:T[P]
    }
    // keyof 获取类型里的全部key值，用|隔开，变为联合类型
    // in 遍历联合类型
    // 返回是跟传入T泛型是一致的类型，但是多了个问号，达到全部属性可选
    ```

  - 

### `Omit<T, K>`

- 作用:把第一个类型中的第二个类型删除掉,可以通过联合类型删除多个

- 例子

  - ```tsx
    type Person = {
      name: string,
      age: number,
      born: string
    };
    const alien: Omit<Person, 'born' | 'age'> = {name: 'Wall E'}
    ```

- 实现：

  - ```tsx
    type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>
    // Exclude 遍历传入Person类型的键值为'name' | 'age' | 'born'，过滤'born' | 'age'键值的类型，返回
    // 形成Pick<Person, name>，那么alien类型为 {name: string}
    ```

  - Omit传入一个类型，一个键值或联合类型，返回的是类型

### `Pick<T,K>`

- 作用： 挑选T类型里的键值K，形成新类型

- 实现：

  - ```tsx
    type Pick<T, K extends keyof T> = {
      [P in K]: T[P]
    }
    // K需要是扩展自 T 的键值
    // 循环K类型，作为新类型键值，值是对应T类型里的P键里的值
    ```

  - Pick传入一个类型，一个键值或联合类型，返回的是类型

### `Exclude<T,U>`

- 作用：对联合类型T里，删除带有U类型再返回新类型

  - ```tsx
    type time = string | number;
    type age = Exclude<time, string> // type age = number
    ```

- 实现

  - ```tsx
    	type Exclude<T, U> = T extends U ? never : T;
    ```

  - Exclude传入的是键值或联合类型，第二个参数是键值或联合类型，返回的是一个键值

