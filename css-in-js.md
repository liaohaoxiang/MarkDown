## css模块化方案

- emotion css-in-js的模块化方案

  - ```ts
    import styled from '@emotion/styled'
    export const Button = styled.buttom`
    	color: turquoise;
    `
    ```

- 作用域问题

  - 传统css只有一个全局作用域，css-in-js 可以通过生成独特的选择符，实现作用域的效果

    - ```css
      .css-1c4ktv6 >* {
        margin-top: 0!important;
      }
      ```

    - 实现

      - ```js
        const css = styleBlock => {
          const className = someHash(styleBlock); // 生成唯一的hash名
          const styleEl = document.createElement('style');
          styleEl.textContent = `   // 将style传入className中
        		.${className} {
        			${styleBlock}
        		}
        	`;
          document.head.appendChild(styleEl);
          return className;
        }
        // 使用
        const className = css(`
        	color: red;
        	padding: 20px;	
        `)
        ```

- 变量问题

  - 传统css规则没有变量，css-in-js可以方便控制变量

    - ```css
      const Container = styled.div(props => ({
        display: 'flex',
        flexDirection: props.column && 'column'
      }))
      ```



### Emotion

- rem 和 em
  - em 是相对于父元素的
  - rem 是相对于根元素 （通常是html的 font-szie：62.5%，因为浏览器默认是16px，1rem 就 = 10px）