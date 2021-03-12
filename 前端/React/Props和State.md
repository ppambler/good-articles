# React 深入系列 3：Props 和 State

> 原文：[React 深入系列３：Props 和 State](https://juejin.cn/post/6844903591845724167)

> React 深入系列，深入讲解了 React 中的重点概念、特性和模式等，旨在帮助大家加深对 React 的理解，以及在项目中更加灵活地使用 React。

React 的核心思想是组件化的思想，而 React 组件的定义可以通过下面的公式描述：

```js
UI = Component(props, state)
```

组件根据 `props` 和 `state` 两个参数，计算得到对应界面的 UI。可见，`props` 和 `state` 是组件的两个重要数据源。

![New values for props and states can change the UI](https://gitee.com/ppambler/blog-images/raw/master/images/1487237596props-and-state.png)

**本篇文章不是对 props 和 state 基本用法的介绍，而是尝试从更深层次解释 props 和 state，并且归纳使用它们时的注意事项。**

## Props 和 State 本质

![image-20210311145710782](https://gitee.com/ppambler/blog-images/raw/master/images/image-20210311145710782.png)

**一句话概括，props 是组件对外的接口，state 是组件对内的接口**。组件内可以引用其他组件，组件之间的引用形成了一个树状结构（组件树），如果下层组件需要使用上层组件的数据或方法，上层组件就可以通过下层组件的 props 属性进行传递，因此 props 是组件对外的接口。组件除了使用上层组件传递的数据外，自身也可能需要维护管理数据，这就是组件对内的接口 state。根据对外接口 props 和对内接口 state，组件计算出对应界面的 UI。

组件的 props 和 state 都和组件最终渲染出的 UI 直接相关。两者的主要区别是：state 是可变的，是组件内部维护的一组用于反映组件 UI 变化的状态集合；而 props 是组件的只读属性，组件内部不能直接修改 props，要想修改 props，只能在该组件的上层组件中修改。在组件**状态上移**的场景中，父组件正是通过子组件的 props，传递给子组件其所需要的状态。

![Cover image for Master the art of React state and props in 5 minutes](https://gitee.com/ppambler/blog-images/raw/master/images/react-state-vs-props.jpg)

> 个人理解的状态上移：A 组件不用 `title` 这个 `state`，而是用来自 `props` 的 `title`

## 如何定义 State

定义一个合适的 state，是正确创建组件的第一步。state 必须能代表一个组件 UI 呈现的**完整状态集**，即组件对应 UI 的任何改变，都可以从 state 的变化中反映出来；同时，state 还必须是代表一个组件 UI 呈现的**最小状态集**，即 state 中的所有状态都是用于反映组件 UI 的变化，没有任何多余的状态，也不需要通过其他状态计算而来的中间状态。

![image-20210311151905136](https://gitee.com/ppambler/blog-images/raw/master/images/image-20210311151905136.png)

组件中用到的一个变量是不是应该作为组件 `state`，可以通过下面的 4 条依据进行判断：

1. 这个变量是否是通过 props 从父组件中获取？如果是，那么它不是一个状态。
2. 这个变量是否在组件的整个生命周期中都保持不变？如果是，那么它不是一个状态。
3. 这个变量是否可以通过 state 或 props 中的已有数据计算得到？如果是，那么它不是一个状态。
4. 这个变量是否在组件的 render 方法中使用？如果**不是**，那么它不是一个状态。这种情况下，这个变量更适合定义为组件的一个**普通属性**（除了 props 和 state 以外的组件属性 ），例如组件中用到的定时器，就应该直接定义为 `this.timer`，而不是 `this.state.timer`。

**请务必牢记，并不是组件中用到的所有变量都是组件的状态！当存在多个组件共同依赖同一个状态时，一般的做法是状态上移**，将这个状态放到这几个组件的公共父组件中。

## 如何正确修改 State

### 1. 不能直接修改 State。

直接修改 state，组件并不会重新重发 render。例如：

```js
// 错误
this.state.title = 'React';
```

正确的修改方式是使用`setState()`:

```js
// 正确
this.setState({title: 'React'});
```

### 2. State 的更新是异步的。

调用`setState`，组件的 state 并不会立即改变，`setState`只是把要修改的状态放入一个队列中，React 会优化真正的执行时机，并且 React 会出于性能原因，可能会将多次`setState`的状态修改合并成一次状态修改。所以不能依赖当前的 state，计算下个 state。当真正执行状态修改时，依赖的 this.state 并不能保证是最新的 state，因为 React 会把多次 state 的修改合并成一次，这时，this.state 还是等于这几次修改发生前的 state。另外需要注意的是，同样不能依赖当前的 props 计算下个 state，因为 props 的更新也是异步的。

举个例子，对于一个电商类应用，在我们的购物车中，当点击一次购买按钮，购买的数量就会加 1，如果我们连续点击了两次按钮，就会连续调用两次`this.setState({quantity: this.state.quantity + 1})`，在 React 合并多次修改为一次的情况下，相当于等价执行了如下代码：

```js
Object.assign(
  previousState,
  {quantity: this.state.quantity + 1},
  {quantity: this.state.quantity + 1}
)
```

于是乎，后面的操作覆盖掉了前面的操作，最终购买的数量只增加了 1 个。

如果你真的有这样的需求，可以使用另一个接收一个函数作为参数的`setState`，这个函数有两个参数，第一个参数是组件的前一个 state（本次组件状态修改成功前的 state），第二个参数是组件当前最新的 props。如下所示：

```js
// 正确
this.setState((preState, props) => ({
  counter: preState.quantity + 1; 
}))
```

> 我连续 click 一个 button -> 为啥会多次执行 `setState` 呢？ -> 我 click 一次，就执行一次 `add` 方法，click 两次就搞两次 `add` 咯 -> 难道 `this.setState` 这个异步操作比两次事件入栈还要慢吗？ -> 如果你连续执行 setState ，那么下一次的 preState 就是最新的 state 了，第二个参数并不常用啊！

### 3. State 的更新是一个浅合并（Shallow Merge）的过程。

当调用`setState`修改组件状态时，只需要传入发生改变的状态变量，而不是组件完整的 state，因为组件 state 的更新是一个浅合并（Shallow Merge）的过程。例如，一个组件的 state 为：

```js
this.state = {
  title : 'React',
  content : 'React is an wonderful JS library!'
}
```

当只需要修改状态`title`时，只需要将修改后的`title`传给`setState`：

```js
this.setState({title: 'Reactjs'});
```

React 会合并新的`title`到原来的组件 state 中，同时保留原有的状态`content`，合并后的 state 为：

```js
{
  title : 'Reactjs',
  content : 'React is an wonderful JS library!'
}
```

## State 与 Immutable

React 官方建议把 state 当作不可变对象，一方面是如果直接修改 `this.state`，组件并不会重新 render；另一方面 state 中包含的所有状态都应该是不可变对象。当 state 中的某个状态发生变化，我们应该重新创建一个新状态，而不是直接修改原来的状态。那么，当状态发生变化时，如何创建新的状态呢？根据状态的类型，可以分成三种情况：

### 1. 状态的类型是不可变类型（数字，字符串，布尔值，null， undefined）

这种情况最简单，因为状态是不可变类型，直接给要修改的状态赋一个新值即可。如要修改 count（数字类型）、title（字符串类型）、success（布尔类型）三个状态：

```js
this.setState({
  count: 1,
  title: 'Redux',
  success: true
})
```

### 2. 状态的类型是数组

如有一个数组类型的状态 books，当向 books 中增加一本书时，使用数组的 concat 方法或 ES6 的数组扩展语法（spread syntax）：

```js
// 方法一：使用 preState、concat 创建新数组
this.setState(preState => ({
  books: preState.books.concat(['React Guide']);
}))

// 方法二：ES6 spread syntax
this.setState(preState => ({
  books: [...preState.books, 'React Guide'];
}))
```

当从 books 中截取部分元素作为新状态时，使用数组的 slice 方法：

```js
// 使用 preState、slice 创建新数组
this.setState(preState => ({
  books: preState.books.slice(1,3);
}))
```

当从 books 中过滤部分元素后，作为新状态时，使用数组的 filter 方法：

```js
// 使用 preState、filter 创建新数组
this.setState(preState => ({
  books: preState.books.filter(item => {
    return item != 'React'; 
  });
}))
```

注意不要使用 push、pop、shift、unshift、splice 等方法修改数组类型的状态，因为这些方法都是在原数组的基础上修改，而 concat、slice、filter 会返回一个新的数组。

### 3. 状态的类型是简单对象 (Plain Object)

如 state 中有一个状态 owner，结构如下：

```js
this.state = {
  owner: {
    name: '老干部',
    age: 30
  }  
}
```

当修改 state 时，有如下两种方式：

**1） 使用 ES6 的 Object.assgin 方法**

```js
this.setState(preState => ({
  owner: Object.assign({}, preState.owner, {name: 'Jason'});
}))
```

**2） 使用对象扩展语法（[object spread properties](https://github.com/sebmarkbage/ecmascript-rest-spread)）**

```js
this.setState(preState => ({
  owner: {...preState.owner, name: 'Jason'};
}))
```

总结一下，创建新的状态的关键是，避免使用会直接修改原对象的方法，而是使用可以返回一个新对象的方法。当然，也可以使用一些 Immutable 的 JS 库，如 [Immutable.js](https://github.com/facebook/immutable-js)，实现类似的效果。

那么，为什么 React 推荐组件的状态是不可变对象呢？

一方面是因为不可变对象方便管理和调试，了解更多可 [参考这里](http://redux.js.org/docs/faq/ImmutableData.html#benefits-of-immutability)；另一方面是出于性能考虑，当组件状态都是不可变对象时，我们在组件的`shouldComponentUpdate`方法中，仅需要比较状态的引用就可以判断状态是否真的改变，从而避免不必要的`render`方法的调用。当我们使用 React 提供的`PureComponent`时，更是要保证组件状态是不可变对象，否则在组件的`shouldComponentUpdate`方法中，状态比较就可能出现错误。

## 了解更多

➹：[React入门 Part6_秋名山山妖的博客-CSDN博客](https://blog.csdn.net/qq_40462579/article/details/114032810)


## Q&A

### 1）浅合并是什么？

合并的意思：

1. v：把几个事物合成一个事物 -> 精简机构，合并科室
2. v：由一种疾病引发另一种疾病；（多种病）同时发作。 -> 合并症

简单来说，把 5 班的同学合并到 3 班去，这样 5 班就不存在了！

对于两个对象而言，所谓浅合并就是把第一层的键和值进行合并和替换：

1. 合并指的是 -> obj2 有 obj1 没有的属性，那就给 obj1
2. 替换指的是 -> obj2、obj1 都有的属性，那 obj2 的属性值就会替换掉 obj1 的

``` js
obj1 === Object.assign(obj1, obj2) // true
```

> 如果 obj1 不是对象，如 5、'6'这样，那么这些基本类型值会被包装成对象，再进行合并，如 `'6'` 会包装成字符串对象，即它的构造器是 `String`

同理，深合并就是，第二层、第三层等也会进行合并和替换

➹：[web 前端高级 JavaScript - 对象的深合并与浅合并_lixiaosenlin 的专栏-CSDN 博客](https://blog.csdn.net/lixiaosenlin/article/details/109785219)

### 2）什么叫不可变对象？

不可变的原理很简单，就是**不修改原有对象**，而是通过产生新的来代替原来的，作用也非常单一，就是零副作用。听起来有点别扭，举个例子，你在一个循环里面遍历一个数组，遍历过程中还修改了原数组的数据，这就会带来一些副作用，比如死循环之类的。

immer 和 immutable.js 都只是基于此做了一些封装，让“不可变”写起来更爽而已。

> 简单来说，不要修改对象里边的属性

➹：[immer.js 不可变对象的用途是什么呢？什么情况需要使用呢？ - SegmentFault 思否](https://segmentfault.com/q/1010000022867342)

➹：[JavaScript 浅析 -- 可变对象和不可变对象 - 简书](https://www.jianshu.com/p/327de4c87991)

➹：[不可变数据结构（immutable data） · Issue #33 · sunyongjian/blog](https://github.com/sunyongjian/blog/issues/33)

➹：[从 JS 对象开始，谈一谈前端“不可变数据”和函数式编程](https://juejin.cn/post/6844903470718255118)

### 3）对象里边出现等号是什么神仙语法？

``` js
{
  owner = {
    name: '老干部',
    age: 30
  }
}

// {name:'老干部',age:30}
```

最外层的这个`{}`是块级作用域哈！

文章里边是这样的：

``` js
this.state = {
  owner = {
    name: '老干部',
    age: 30
  }  
}
```

显然是`owner:{}`

➹：[Object initializer - JavaScript - MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer)

### 4）`PureComponent`是什么？

React15.3 中新加了一个 `PureComponent` 类，顾名思义， `pure` 是纯的意思，`PureComponent` 也就是纯组件，取代其前身 `PureRenderMixin` ,`PureComponent` 是优化 `React` 应用程序最重要的方法之一，易于实施，只要把继承类从 `Component` 换成 `PureComponent` 即可，可以减少不必要的 `render` 操作的次数，从而提高性能，而且可以少写 `shouldComponentUpdate` 函数，节省了点代码。

➹：[React PureComponent 使用指南](https://juejin.cn/post/6844903480369512455)

➹：[可靠 React 组件设计的 7 个准则之纯组件](https://juejin.cn/post/6844903912663678990)

➹：[React 组件 纯组件 函数组件 高阶组件](https://juejin.cn/post/6844904099679305741)