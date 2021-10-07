# CSS in JS: styled-components 实践

@[TOC](文章目录)

<!-- TOC -->

- [CSS in JS: styled-components 实践](#css-in-js-styled-components-实践)
- [正文](#正文)
  - [0. 演进](#0-演进)
  - [1. 特性：原生标签样式](#1-特性原生标签样式)
  - [2. 特性：为组件加上样式](#2-特性为组件加上样式)
  - [3. 特性：嵌套、占位符](#3-特性嵌套占位符)
  - [4. 特性：组件属性赋值](#4-特性组件属性赋值)
  - [5. 特性：动画 & keyframes](#5-特性动画--keyframes)
  - [6. 特性：布景主题](#6-特性布景主题)
  - [7. 总结：用法、特点](#7-总结用法特点)
- [其他资源](#其他资源)
  - [参考连接](#参考连接)
  - [完整代码示例](#完整代码示例)

<!-- /TOC -->

# 正文

## 0. 演进

- pure CSS
- Sass、Less (CSS preprocess)
- BEM = Block Element Modifier (prefix as namespace)
- CSS Module (Auto BEM)
- styled-components (CSS in JS)

styled-components 可以说是 CSS in JS 的预演，目标在于将 CSS 完全融合到 JS 当中。不过当前 styled-components 还是强依赖于 React 框架的实现

## 1. 特性：原生标签样式

原生标签加上样式生成可复用的 React 组件

- 基本语法

```ts
import styled from 'styled-components';

// 生成组件
const StyledDiv = styled.div`
    // your css
`

// 使用 组件
render(
    <StyledDiv></StyledDiv>
)
```

- `/src/layouts/ShowCase1.tsx`

```ts
import React from 'react';
import styled from 'styled-components';

// 1.1 原生标签
const Wrapper = styled.section`
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 2em;
  background: papayawhip;
`;

const Title = styled.h1`
  font-size: 1.5em;
  text-align: center;
  color: palevioletred;
`;

interface IButtonProps {
  primary?: boolean;
}

// 1.2 原生标签带参数
const Button = styled.button<IButtonProps>`
  background: ${(props) => (props.primary ? 'palevioletred' : 'white')};
  color: ${(props) => (props.primary ? 'white' : 'palevioletred')};

  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid palevioletred;
  border-radius: 3px;
`;

const ShowCase1 = () => {
  return (
    <Wrapper>
      <Title>Hello World!</Title>
      <div>
        <Button>Normal</Button>
        <Button primary>Primary</Button>
      </div>
    </Wrapper>
  );
};

export default ShowCase1;
```

- `styled` + 原生标签名来创建组件，后面跟上组件的样式，使用的是 ES6 的模版字符串的特性
- 模版字符串内部还可以写成函数表达式，接受组件的 props 作为参数

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_styled_components_1.png)

## 2. 特性：为组件加上样式

除了原生标签还能为组件也加上样式

- `/src/layouts/ShowCase2.tsx`

```ts
import React, { FC } from 'react';
import styled from 'styled-components';

const Button = styled.button`
  color: palevioletred;
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid palevioletred;
  border-radius: 3px;
`;

// 2.1 组件样式覆盖
const TomatoButton = styled(Button)`
  color: tomato;
  border-color: tomato;
`;

// 2.2 修饰函数组件
const Link: FC<{ className?: string }> = ({ className, children }) => <a className={className}>{children}</a>;

const StyledLink = styled(Link)`
  color: palevioletred;
  font-weight: bold;
`;

const ShowCase2 = () => {
  return (
    <div style={{ textAlign: 'center' }}>
      <div>
        <Button>Normal Button</Button>
        <TomatoButton>Tomato Button</TomatoButton>
      </div>
      <div>
        <Link>Unstyled, boring Link</Link> <StyledLink>Styled, exciting Link</StyledLink>
      </div>
    </div>
  );
};

export default ShowCase2;
```

- `styled(Component)` 来创建组件的样式

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_styled_components_2.png)

## 3. 特性：嵌套、占位符

与 Sass、Less 类似，我们也可以在样式内部使用嵌套 CSS 样式、父级选择器占位符 `&`

- `/src/layouts/ShowCase3.tsx`

```ts
import React from 'react';
import styled from 'styled-components';

// 3. 伪元素占位符（父类选择器占位符）
const Thing = styled.div.attrs((/* props */) => ({ tabIndex: 0 }))`
  color: blue;
  cursor: pointer;

  &:hover {
    color: red; // <Thing> when hovered
  }

  & ~ & {
    background: tomato; // <Thing> as a sibling of <Thing>, but maybe not directly next to it
  }

  & + & {
    background: lime; // <Thing> next to <Thing>
  }

  &.something {
    background: orange; // <Thing> tagged with an additional CSS class ".something"
  }

  .something-else & {
    border: 1px solid; // <Thing> inside another element labeled ".something-else"
  }
`;

const ShowCase3 = () => {
  return (
    <div>
      <Thing>Hello world!</Thing>
      <Thing>How ya doing?</Thing>
      <Thing className="something">The sun is shining...</Thing>
      <div>Pretty nice day today.</div>
      <Thing>Don't you think?</Thing>
      <div className="something-else">
        <Thing>Splendid.</Thing>
      </div>
    </div>
  );
};

export default ShowCase3;
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_styled_components_3.png)

## 4. 特性：组件属性赋值

除了决定 style 之外，还能够提前定制标签原生属性，如 `<input>` 的 `type` 属性等

- `/src/layouts/ShowCase4.tsx`

```ts
import React from 'react';
import styled from 'styled-components';

interface IInputProps {
  size?: string;
}

// 4. 传递额外参数
const Input = styled.input.attrs((props: IInputProps) => ({
  // we can define static props
  type: 'text',

  // or we can define dynamic ones
  size: props.size || '1em',
}))<IInputProps>`
  color: palevioletred;
  font-size: 1em;
  border: 2px solid palevioletred;
  border-radius: 3px;

  /* here we use the dynamically computed prop */
  margin: ${(props) => props.size};
  padding: ${(props) => props.size};
`;

const ShowCase4 = () => {
  return (
    <div>
      <Input placeholder="A small text input" />
      <Input placeholder="A bigger text input" size="2em" />
    </div>
  );
};

export default ShowCase4;
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_styled_components_4.png)

## 5. 特性：动画 & keyframes

除了简单的 CSS 样式，动画也是 CSS 非常重要的一环

- `/src/layouts/ShowCase5.tsx`

```ts
import React from 'react';
import styled, { keyframes } from 'styled-components';

// 5.1 动画片段
const rotate = keyframes`
  from {
    transform: rotate(0deg);
  }

  to {
    transform: rotate(360deg);
  }
`;

// 5.2 带动画元素
const Rotate = styled.div`
  display: inline-block;
  animation: ${rotate} 2s linear infinite;
  padding: 2rem 1rem;
  font-size: 1.2rem;
`;

const ShowCase5 = () => {
  return (
    <div style={{ height: '7em' }}>
      <Rotate>&lt; 💅🏾 &gt;</Rotate>
    </div>
  );
};

export default ShowCase5;
```

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_styled_components_5.png)

## 6. 特性：布景主题

除了平常的 CSS 之外，styled-components 还实现了布景主题(Theme)，透过全局的 `ThemeProvider, ThemeContext`，使用 styled-components 创建的组件能自动的获取到 Context 上的 theme 变量

- `/src/layouts/ShowCase6.tsx`

```ts
import React, { useContext } from 'react';
import styled, { ThemeProvider, ThemeContext } from 'styled-components';

// 6.1 按布景主题实现样式
const Button = styled.button`
  color: ${(props) => props.theme.fg};
  border: 2px solid ${(props) => props.theme.fg};
  background: ${(props) => props.theme.bg};

  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border-radius: 3px;
`;

// 6.2 主题 context
const theme = {
  fg: 'palevioletred',
  bg: 'white',
};

// 6.3 主题 context（函数形式）
const invertTheme = ({ fg, bg }) => ({
  fg: bg,
  bg: fg,
});

// 6.4 ThemeContext 消费主题
const Inner = () => {
  const themeCtx = useContext(ThemeContext);

  console.log('[Sample15] theme', themeCtx);

  return (
    <div>
      <Button>Default Theme</Button>
      <ThemeProvider theme={invertTheme}>
        <Button>Inverted Theme</Button>
      </ThemeProvider>
    </div>
  );
};

// 6.5 ThemeProvider 挂载主题
const ShowCase6 = () => {
  return (
    <ThemeProvider theme={theme}>
      <Inner />
    </ThemeProvider>
  );
};

export default ShowCase6;
```

- 看到 6.3 我们知道 theme 除了传入简单的对象，还能够传入一个参数，接受外层的 theme 做参数返回新的 theme

![](https://picures.oss-cn-beijing.aliyuncs.com/img/css_styled_components_6.png)

## 7. 总结：用法、特点

- 用法
  - `styled.<tagName> + Template Literal` 创建带样式的原生组件
  - `styled(Component) + Template Literal` 创建带样式的自定义组件
  - `css + Template Literal` 创建可服用的样式片段
  - `keyframes + Template Literal` 创建动画帧片段 
  - `createGlobalStyle + Template Literal` 创建全局 CSS 样式
  - `<ThemeProvider>` 创建样式 Context 生产者
  - `useContext(ThemeContext)` 在 JS 代码中消费样式的 Context

# 其他资源

## 参考连接

| Title                                                                | Link                                                                                                                                                                                                                             |
| -------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| styled-components docs                                               | [https://styled-components.com/docs](https://styled-components.com/docs)                                                                                                                                                         |
| The magic behind 💅 styled-components                                 | [https://mxstbr.blog/2016/11/styled-components-magic-explained/](https://mxstbr.blog/2016/11/styled-components-magic-explained/)                                                                                                 |
| styled-components 运行原理                                           | [https://www.jianshu.com/p/dcd91a5c2a56](https://www.jianshu.com/p/dcd91a5c2a56)                                                                                                                                                 |
| CSS Evolution: From CSS, SASS, BEM, CSS Modules to Styled Components | [https://medium.com/@perezpriego7/css-evolution-from-css-sass-bem-css-modules-to-styled-components-d4c1da3a659b](https://medium.com/@perezpriego7/css-evolution-from-css-sass-bem-css-modules-to-styled-components-d4c1da3a659b) |

## 完整代码示例

[https://github.com/superfreeeee/Blog-code/tree/main/front_end/css/css_styled_components](https://github.com/superfreeeee/Blog-code/tree/main/front_end/css/css_styled_components)
