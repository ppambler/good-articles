# React 深入系列１：React 中的元素、组件、实例和节点

> 原文：[React 深入系列１：React 中的元素、组件、实例和节点](https://juejin.cn/post/6844903587445735437)

> React 深入系列，深入讲解了 React 中的重点概念、特性和模式等，旨在帮助大家加深对 React 的理解，以及在项目中更加灵活地使用 React。

React 中的元素、组件、实例和节点，是 React 中关系密切的 4 个概念，也是很容易让 React 初学者迷惑的 4 个概念。现在，老干部就来详细地介绍这 4 个概念，以及它们之间的联系和区别，满足喜欢咬文嚼字、刨根问底的同学（老干部就是其中一员）的好奇心。

![image-20210310130053286](https://gitee.com/ppambler/blog-images/raw/master/images/image-20210310130053286.png)

## 元素 (Element)

**React 元素其实就是一个简单 JavaScript 对象，一个 React 元素和界面上的一部分 DOM 对应，描述了这部分 DOM 的结构及渲染效果**。一般我们通过 JSX 语法创建 React 元素，例如：

```jsx
const element = <h1 className='greeting'>Hello, world</h1>;
```

element 是一个 React 元素。在编译环节，JSX 语法会被编译成对`React.createElement()`的调用，从这个函数名上也可以看出，JSX 语法返回的是一个 React 元素。上面的例子编译后的结果为：

```jsx
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```

最终，element 的值是类似下面的一个简单 JavaScript 对象：

```js
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world'
  }
}
```

React 元素可以分为两类：**DOM 类型的元素和组件类型的元素**。DOM 类型的元素使用像`h1、div、p`等 DOM 节点创建 React 元素，前面的例子就是一个 DOM 类型的元素；组件类型的元素使用 React 组件创建 React 元素，例如：

```jsx
const buttonElement = <Button color='red'>OK</Button>;
```

buttonElement 就是一个组件类型的元素，它的值是：

```js
const buttonElement = {
  type: 'Button',
  props: {
    color: 'red',
    children: 'OK'
  }
}
```

对于 DOM 类型的元素，因为和页面的 DOM 节点直接对应，所以 React 知道如何进行渲染。但是对于组件类型的元素，如 buttonElement，React 是无法直接知道应该把 buttonElement 渲染成哪种结构的页面 DOM，这时就需要组件自身提供 React 能够识别的 DOM 节点信息，具体实现方式在介绍组件时会详细介绍。

有了 React 元素，我们应该如何使用它呢？其实，绝大多数情况下，我们都不会直接使用 React 元素，React 内部会自动根据 React 元素，渲染出最终的页面 DOM。**更确切地说，React 元素描述的是 React 虚拟 DOM 的结构，React 会根据虚拟 DOM 渲染出页面的真实 DOM**。

![image-20210310132436014](https://gitee.com/ppambler/blog-images/raw/master/images/image-20210310132436014.png)

## 组件 (Component)

React 组件，应该是大家最熟悉的 React 中的概念。React 通过组件的思想，将界面拆分成一个个可以复用的模块，每一个模块就是一个 React 组件。一个 React 应用由若干组件组合而成，一个复杂组件也可以由若干简单组件组合而成。

React 组件和 React 元素关系密切，**React 组件最核心的作用是返回 React 元素**。这里你也许会有疑问：React 元素不应该是由 `React.createElement()` 返回的吗？但 `React.createElement()` 的调用本身也是需要有“人”负责的，React 组件正是这个“责任人”。React 组件负责调用 `React.createElement()`，返回 React 元素，供 React 内部将其渲染成最终的页面 DOM。

既然组件的核心作用是返回 React 元素，那么最简单的组件就是一个返回 React 元素的函数：

```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```

Welcome 是一个用函数定义的组件。如果使用类（class）定义组件，返回 React 元素的工作具体就由组件的 render 方法承担，例如：

```jsx
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

其实，使用类定义的组件，render 方法是唯一必需的方法，其他组件的生命周期方法都只不过是为 render 服务而已，都不是必需的。

现在来考虑下面这个例子：

```jsx
class Home extends React.Component {
  render() {
    return (
      <div>
        <Welcome name='老干部' />
        <p>Anything you like</p>
      </div>
    )
  }
}
```

Home 组件使用了 Welcome 组件，返回的 React 元素为：

```js
{
  type: 'div',
  props: {
    children: [
      {
        type: 'Welcome',
        props: {
          name: '老干部'
        }
      },
      {
        type: 'p',
        props: {
          children: 'Anything you like'
        }
      }，
    ]
  }
}
```

对于这个结构，React 知道如何渲染 `type = 'div'` 和 `type = 'p'` 的节点，但不知道如何渲染 `type='Welcome'`的节点，当 React 发现 Welcome 是一个 React 组件时（**判断依据是 Welcome 首字母为大写**），会根据 Welcome 组件返回的 React 元素决定如何渲染 Welcome 节点。Welcome 组件返回的 React 元素为：

```js
{
  type: 'h1',
  props: {
  	children: 'Hello, 老干部'
  }
}
```

这个结构中只包含 DOM 节点，React 是知道如何渲染的。如果这个结构中还包含其他组件节点，React 会重复上面的过程，继续解析对应组件返回的 React 元素，直到返回的 React 元素中只包含 DOM 节点为止。这样的递归过程，让 React 获取到页面的完整 DOM 结构信息，渲染的工作自然就水到渠成了。

另外，如果仔细思考的话，可以发现，**React 组件的复用，本质上是为了复用这个组件返回的 React 元素，React 元素是 React 应用的最基础组成单位**。

![image-20210310151233073](https://gitee.com/ppambler/blog-images/raw/master/images/image-20210310151233073.png)

> HTML 元素 vs React 元素 -> React 元素的构成组合：HTML`{0,}`（DOM 类型的元素）、HTML+组件（混合类型的元素）、组件`{1,}`（组件类型的元素）

## 实例 (Instance）

这里的实例特指 React 组件的实例。React 组件是一个函数或类，实际工作时，发挥作用的是 React 组件的实例对象。只有组件实例化后，每一个组件实例才有了自己的 `props` 和 `state`，才持有对它的 DOM 节点和子组件实例的引用。在传统的面向对象的开发方式中，实例化的工作是由开发者自己手动完成的，但**在 React 中，组件的实例化工作是由 React 自动完成的，组件实例也是直接由 React 管理的**。换句话说，开发者完全不必关心组件实例的创建、更新和销毁。

![image-20210310152829941](https://gitee.com/ppambler/blog-images/raw/master/images/image-20210310152829941.png)

## 节点 (Node)

在使用 PropTypes 校验组件属性时，有这样一种类型：

```js
MyComponent.propTypes = { 
  optionalNode: PropTypes.node,
}
```

PropTypes.node 又是什么类型呢？这表明 optionalNode 是一个 React 节点。React 节点是指可以被 React 渲染的数据类型，包括数字、字符串、React 元素，或者是一个包含这些类型数据的数组。例如：

```jsx
// 数字类型的节点
function MyComponent(props) {
  return 1;
}

// 字符串类型的节点
function MyComponent(props) {
  return 'MyComponent';
}

// React 元素类型的节点
function MyComponent(props) {
  return <div>React Element</div>;
}

// 数组类型的节点，数组的元素只能是其他合法的 React 节点
function MyComponent(props) {
  const element = <div>React Element</div>;
  const arr = [1, 'MyComponent', element];
  return arr;
}

// 错误，不是合法的 React 节点
function MyComponent(props) {
  const obj = { a : 1}
  return obj;
}
```

最后总结一下，React 元素和组件的概念最重要，也最容易混淆；React 组件实例的概念大家了解即可，几乎使用不到；React 节点有一定使用场景，但看过本文后应该也就不存在理解问题了。

![image-20210310164709861](https://gitee.com/ppambler/blog-images/raw/master/images/image-20210310164709861.png)

> 我们可以写个组件标签作为某个组件实例的`props`值 -> 这个组件标签可以返回各种类型的节点 -> 如数值类型的、字符串类型的、元素类型的（React 元素类型的）、数组类型的、文档碎片（这个不知道怎么测试，返回一个文档碎片显然是不行的，毕竟这是文档碎片节点呀 -> 话说，fragment 是指文档碎片节点吗？）

## 了解更多

➹：[Typechecking With PropTypes – React](https://reactjs.org/docs/typechecking-with-proptypes.html)