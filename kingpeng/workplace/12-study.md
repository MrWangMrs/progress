## Styled-Components

[官方文档](https://www.styled-components.com/docs)

> Styled-components 是目前 React 样式方案中最受关注的一种，它既具备了 css-in-js 的模块化与参数化优点，又完全使用CSS的书写习惯，不会引起额外的学习成本。本文是 styled-components 作者之一 Max Stoiber 所写，首先总结了前端组件化样式中的最佳实践原则，然后在此基础上介绍了 styled-components 的设计理念：“强制你使用组件化样式的最佳实践”。本文对于前端组件、样式的实践原则等很有启发性，同时本人也十分推荐 styled-components 这一方案。

### 一、基本

#### 1、安装

```
npm install --save styled-components
除了npm安装使用模块化加载包之外，也支持UMD格式直接加载脚本文件。

```

#### 2、入门

##### styled-components使用标签模板来对组件进行样式化。

它移除了组件和样式之间的映射。这意味着，当你定义你的样式时，你实际上创造了一个正常的React组件，你的样式也附在它上面。

这个例子创建了两个简单的组件，一个容器和一个标题，并附加了一些样式。

```

// Create a Title component that'll render an <h1> tag with some styles
const Title = styled.h1`
  font-size: 1.5em;
  text-align: center;
  color: palevioletred;
`;

// Create a Wrapper component that'll render a <section> tag with some styles
const Wrapper = styled.section`
  padding: 4em;
  background: papayawhip;
`;

// Use Title and Wrapper like any other React component – except they're styled!
render(
  <Wrapper>
    <Title>
      Hello World, this is my first styled component!
    </Title>
  </Wrapper>
);

```
** 注意

CSS规则会自动添加浏览器厂商前缀，我们不必考虑它。
透传props
styled-components会透传所有的props属性。

```

// Create an Input component that'll render an <input> tag with some styles
const Input = styled.input`
  padding: 0.5em;
  margin: 0.5em;
  color: palevioletred;
  background: papayawhip;
  border: none;
  border-radius: 3px;
`;

// Render a styled text input with a placeholder of "@mxstbr", and one with a value of "@geelen"
render(
  <div>
    <Input placeholder="@mxstbr" type="text" />
    <Input value="@geelen" type="text" />
  </div>
);

```

##### 基于props做样式判断

模板标签的函数插值能拿到样式组件的props，可以据此调整我们的样式规则。

```

const Button = styled.button`
  /* Adapt the colours based on primary prop */
  background: ${props => props.primary ? 'palevioletred' : 'white'};
  color: ${props => props.primary ? 'white' : 'palevioletred'};

  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid palevioletred;
  border-radius: 3px;
`;

render(
  <div>
    <Button>Normal</Button>
    <Button primary>Primary</Button>
  </div>
);

```

##### 样式化任意组件

```
// This could be react-router's Link for example
const Link = ({ className, children }) => (
  <a className={className}>
    {children}
  </a>
)

const StyledLink = styled(Link)`
  color: palevioletred;
  font-weight: bold;
`;

render(
  <div>
    <Link>Unstyled, boring Link</Link>
    <br />
    <StyledLink>Styled, exciting Link</StyledLink>
  </div>
);

```

##### 扩展样式

我们有时候需要在我们的样式组件上做一点扩展，添加一些额外的样式：

需要注意的是.extend在对样式组件有效，如果是其他的React组件，需要用styled样式化一下。

```

// The Button from the last section without the interpolations
const Button = styled.button`
  color: palevioletred;
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid palevioletred;
  border-radius: 3px;
`;

// We're extending Button with some extra styles
const TomatoButton = Button.extend`
  color: tomato;
  border-color: tomato;
`;

render(
  <div>
    <Button>Normal Button</Button>
    <TomatoButton>Tomato Button</TomatoButton>
  </div>
);

```

##### 在极少特殊情况下，我们可能需要更改样式组件的标签类型。我们有一个特别的API，withComponent可以扩展样式和替换标签：

```

const Button = styled.button`
  display: inline-block;
  color: palevioletred;
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border: 2px solid palevioletred;
  border-radius: 3px;
`;

// We're replacing the <button> tag with an <a> tag, but reuse all the same styles
const Link = Button.withComponent('a')

// Use .withComponent together with .extend to both change the tag and use additional styles
const TomatoLink = Link.extend`
  color: tomato;
  border-color: tomato;
`;

render(
  <div>
    <Button>Normal Button</Button>
    <Link>Normal Link</Link>
    <TomatoLink>Tomato Link</TomatoLink>
  </div>
);

```

##### 添加attr

我们可以使用attrsAPI来为样式组件添加一些attr属性，它们也可以通过标签模板插值函数拿到props传值。

```

const Input = styled.input.attrs({
  // we can define static props
  type: 'password',

  // or we can define dynamic ones
  margin: props => props.size || '1em',
  padding: props => props.size || '1em'
})`
  color: palevioletred;
  font-size: 1em;
  border: 2px solid palevioletred;
  border-radius: 3px;

  /* here we use the dynamically computed props */
  margin: ${props => props.margin};
  padding: ${props => props.padding};
`;

render(
  <div>
    <Input placeholder="A small text input" size="1em" />
    <br />
    <Input placeholder="A bigger text input" size="2em" />
  </div>
);

```

##### 动画

带有@keyframes的CSS animations，一般来说会产生复用。styled-components暴露了一个keyframes的API，我们使用它产生一个可以复用的变量。这样，我们在书写css样式的时候使用JavaScript的功能，为CSS附能，并且避免了名称冲突。

```

// keyframes returns a unique name based on a hash of the contents of the keyframes
const rotate360 = keyframes`
  from {
    transform: rotate(0deg);
  }

  to {
    transform: rotate(360deg);
  }
`;

// Here we create a component that will rotate everything we pass in over two seconds
const Rotate = styled.div`
  display: inline-block;
  animation: ${rotate360} 2s linear infinite;
  padding: 2rem 1rem;
  font-size: 1.2rem;
`;

render(
  <Rotate>&lt; 💅 &gt;</Rotate>
);

##### 支持 React Native

高级特性: Theming
styled-components暴露了一个<ThemeProvider>容器组件，提供了设置默认主题样式的功能，他类似于react-rudux的顶层组件Provider，通过context实现了从顶层到底层所有样式组件的默认主题共用。

const Button = styled.button`
  font-size: 1em;
  margin: 1em;
  padding: 0.25em 1em;
  border-radius: 3px;
  
  /* Color the border and text with theme.main */
  color: ${props => props.theme.main};
  border: 2px solid ${props => props.theme.main};
`;

Button.defaultProps = {
  theme: {
    main: 'palevioletred'
  }
}
// Define what props.theme will look like
const theme = {
  main: 'mediumseagreen'
};

render(
  <div>
    <Button>Normal</Button>
    <ThemeProvider theme={theme}>
      <Button>Themed</Button>
    </ThemeProvider>
  </div>
);

```

##### Refs

通常我们在给一个非原生样式组件添加ref属性的时候，其指向都是该组件实例的索引，我们通过用innerRef可以直接拿到里面的DOM节点。

```

const AutoFocusInput = styled.input`
  background: papayawhip;
  border: none;
`;

class Form extends React.Component {
  render() {
    return (
      <AutoFocusInput
        placeholder="Hover here..."
        innerRef={x => { this.input = x }}
        onMouseEnter={() => this.input.focus()}
      />
    );
  }
}

```

#####  Security

因为styled-components允许我们使用任意输入作为CSS属性值，一旦意识到这一点，我们马上明白要对输入做安全性校验了，因为使用用户外部的输入样式可以导致用户的浏览器被CSS注入攻击。CSS注入攻击可能不明显，但是我们还是得小心一点，某些IE浏览器版本甚至允许在URL声明中执行任意的JS。

这个例子告诉我们外部的输入甚至可能在CSS内调用一个API网络请求。

```

// Oh no! The user has given us a bad URL!
const userInput = '/api/withdraw-funds';

const ArbitraryComponent = styled.div`
  background: url(${userInput});
  /* More styles here... */
`;

```

CSS.escape这个未来API标准可净化JS中的CSS的问题。但是浏览器兼容性目前还不是太好，所以我们建议在项目中使用polyfill by Mathias Bynens。

##### CSS共存

如果我们打算把styled-components和现有的css共存的话，我们需要注意两个实现的细节问题：

styled-components也会生成真实的样式表，并通过className属性链接生成的样式表内容。在JS运行时，他会生成一份真实的style节点插入到document的head内。

注意的一个小地方：

```

// MyComponent.js
const MyComponent = styled.div`background-color: green;`;

// my-component.css
.red-bg {
  background-color: red;
}

// For some reason this component still has a green background,
// even though you're trying to override it with the "red-bg" class!
<MyComponent className="red-bg" />
我们styled-components生成的style样式表一般是在head头部的最底下，同等CSS优先级条件下是会覆盖默认前者css文件的样式的。这个插入顺序使用webpack来调整是比较难得。所以，我们一般都这样通过调整css优先级来改变显示：

/* my-component.css */
.red-bg.red-bg {
  background-color: red;
}

```

##### Media Templates

媒体查询是开发响应式web应用不可或缺的存在，这是一个简单的例子：

```

const Content = styled.div`
  background: papayawhip;
  height: 3em;
  width: 3em;

  @media (max-width: 700px) {
    background: palevioletred;
  }
`;

render(
  <Content />
);

```

因为媒体查询语句很长，并且经常在整个应用程序中重复使用，所以为此创建一些模板来复用是很有必要的。

使用JS的功能特性，我们可以轻松定义一份可配置的语句，包装媒体查询和样式。

```

const sizes = {
  desktop: 992,
  tablet: 768,
  phone: 376
}

// Iterate through the sizes and create a media template
const media = Object.keys(sizes).reduce((acc, label) => {
  acc[label] = (...args) => css`
    @media (max-width: ${sizes[label] / 16}em) {
      ${css(...args)}
    }
  `

  return acc
}, {})

const Content = styled.div`
  height: 3em;
  width: 3em;
  background: papayawhip;

  /* Now we have our methods on media and can use them instead of raw queries */
  ${media.desktop`background: dodgerblue;`}
  ${media.tablet`background: mediumseagreen;`}
  ${media.phone`background: palevioletred;`}
`;

render(
  <Content />
);

```

##### Tagged Template Literals

标签模板是ES6的一个新特性，这是我们styled-components创建样式组件的方式和规则。

```

const aVar = 'good';

// These are equivalent:
fn`this is a ${aVar} day`;
fn([ 'this is a ', ' day' ], aVar);

```

这看起来有点麻烦，但是这意味着我们可以在styled-components生成样式组件中接受变量、函数、minxins，并将其变为纯css。

这篇文章可以了解更多：The magic behind 💅 styled-components

Server Side Rendering

styled-components很好地支持SSR。

一个例子：

```

import { renderToString } from 'react-dom/server'
import { ServerStyleSheet } from 'styled-components'

const sheet = new ServerStyleSheet()
const html = renderToString(sheet.collectStyles(<YourApp />))
const styleTags = sheet.getStyleTags() // or sheet.getStyleElement()
也可以这样组件化包裹，只要在客户端不这么使用：

import { renderToString } from 'react-dom/server'
import { ServerStyleSheet, StyleSheetManager } from 'styled-components'

const sheet = new ServerStyleSheet()
const html = renderToString(
  <StyleSheetManager sheet={sheet.instance}>
    <YourApp />
  </StyleSheetManager>
)

const styleTags = sheet.getStyleTags() // or sheet.getStyleElement()
sheet.getStyleTags()返回一个style标签数组。具体styled-components关于SSR更深入的操作，不在这里继续讨论了，还可以告知他兼容Next.js关于SSR的解决方案。

##### Referring to other components

styled-components提供了component selector组件选择器模式来代替我们以往对class名的依赖，解决得很干净。这下我们不必为命名和选择器冲突而苦恼了。

const Link = styled.a`
  display: flex;
  align-items: center;
  padding: 5px 10px;
  background: papayawhip;
  color: palevioletred;
`;

const Icon = styled.svg`
  transition: fill 0.25s;
  width: 48px;
  height: 48px;

  ${Link}:hover & {
    fill: rebeccapurple;
  }
`;

const Label = styled.span`
  display: flex;
  align-items: center;
  line-height: 1.2;

  &::before {
    content: '◀';
    margin: 0 10px;
  }
`;

render(
  <Link href="#">
    <Icon viewBox="0 0 20 20">
      <path d="M10 15h8c1 0 2-1 2-2V3c0-1-1-2-2-2H2C1 1 0 2 0 3v10c0 1 1 2 2 2h4v4l4-4zM5 7h2v2H5V7zm4 0h2v2H9V7zm4 0h2v2h-2V7z"/>
    </Icon>
    <Label>Hovering my parent changes my style!</Label>
  </Link>
);

注意：

class A extends React.Component {
  render() {
    return <div />;
  }
}

const B = styled.div`
  ${A} {
  }
`;
这个例子是不可以的，因为A继承ReactComponent，不是被styled构造过的。我们的组件选择器只支持在Styled Components创建的样式组件。

class A extends React.Component {
  render() {
    return <div className={this.props.className} />;
  }
}

const StyledA = styled(A)``;

const B = styled.div`
  ${StyledA} {
  }
`;

```


在样式组件中，我们支持所有CSS加嵌套。因为我们生成一个真实的stylesheet而不是内联样式，所以CSS中的任何工作都在样式组件中工作！

（&）被我们所生成的、唯一的类名替换给样式组件，使其具有复杂的逻辑变得容易。

支持flow和typescript
更多工具
Babel Plugin
Test Utilities
Jest Styled Components，基于jest，可对styled-components做单元测试


Stylelint
使用stylelint 检查我们的styled-components样式书写规范。

Styled Theming 语法高亮显示
在模板文本中写入CSS时丢失的一个东西是语法高亮显示。我们正在努力在所有编辑器中实现正确的语法高亮显示。支持大部分编辑器包括Visual Studio Code、WebStorm。

总结

下面简单总结一下 styled-components 在开发中的表现：

* 提出了 container 和 components 的概念，移除了组件和样式之间的映射关系，符合关注度分离的模式；
* 可以在样式定义中直接引用到 js 变量，共享变量，非常便利，利用js的特性为css附能，帅毙了！
* 支持组件之间继承，方便代码复用，提升可维护性；
* 兼容现有的 className 方式，升级无痛；
* 这下写CSS也乐趣十足了。
* styled-components的最基本思想就是通过移除样式和组件之间的映射来执行最佳实践
* 一个让styled-components很容易被接受的特性：当他被怀疑的时候，你同样可以使用你熟悉的方法去使用它！
* 当然，styled-components 还有一些优秀的特性，比如服务端渲染和 React Native 的支持。

题外：styled-components的魔法
如果你从来没看见过styled-components，下面是一个简单的样式组件的例子：

```

const Button = styled.button`
  background-color: papayawhip;
  border-radius: 3px;
  color: palevioletred;
`
```
现在可以像使用普通React组件一样渲染使用。
```
<Button>Hi Dad!</Button>
```
那么，这是怎么工作的呢？这个过程中到底发生了什么魔法？

标签模板
实际上， style.button` `是JavaScript的新语法特性，属于ES6的标签模板功能。

本质上， styled.button` 和styled.button()`是一样的。他们的差异只在传递参数时就变得可见了。

styled-components利用模板字符串的用处在于可以给内部props赋值。

```

const Button = styled.button`
  font-size: ${props => props.primary ? '2em' : '1em'};
`
// font-size: 2em;
<Button primary />

```


##### Example：

PageLayout.tsx
  
```

import { SFC } from 'react'
import styled from 'styled-components'


const Header = styled.header`
    text-align: center;
    color: #979797;
    font-size: 18px; 
    margin-bottom: 25vh;
    margin-top: 30px;
`


const PageLayout: SFC<{ header: string }> = (props) => (
    <div style={{ margin: '0 30px' }} className="root">
        <Header>{props.header}</Header>
        {props.children}
    </div>
)

export default PageLayout


```
