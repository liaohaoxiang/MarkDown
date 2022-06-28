### 垂直水平居中CSS总结

1. 不需要知道子元素大小的

   - **`absolute`  + `transform`** (不支持IE10以下)

     - ```css
       .container {
         position: relative;
       }
       
       .children {
         position: absolute;
         top: 50%;
         left: 50%;
         transform: translate(-50%, -50%);
       }
       ```

   - **`flex`**

     - ```css
       .container {
         display: flex;
         justify-content: center;
         align-items: center;
       }
       
       .children {
         background-color: orangered;
       }
       
       ```

   - **`grid`**

     - ```css
       .container {
         display: grid;
         justify-content: center;
         align-content: center;
       }
       
       .children {
          background-color: orangered;
       }
       ```

     - ```css
       .container {
         display: grid;
       }
       
       .children {
         margin: auto;
       }
       ```

     - ```css
       .children {
         justify-self: center; /* 水平居中 */
         align-self: center; /* 垂直居中 */
       }
       ```

     - 

2. 知道子元素大小

   - **`absolute`  + `margin负值`**

     - ```css
       .children {
         position: absolute;
         width: 100px;
         height: 100px;
         top: 50%;
         left: 50%;
         margin-top: -50px;
         margin-left: -50px;
       }
       ```

   - **`absolute` + `margin: auto`**

     - 该方法的副作用：

       - `left: 0; right: 0;` 相当于 `width: 100%;`
       - `top: 0; bottom: 0;` 相当于 `height: 100%;`

     - ```css
       .children {
         position: absolute;
         width: 100px;
         height: 100px;
         top: 0;
         right: 0;
         bottom: 0;
         left: 0;
         margin: auto;
       }
       ```

     

