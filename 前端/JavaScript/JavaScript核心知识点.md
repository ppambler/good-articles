# JavaScript 核心知识点

## 1. 栈

`栈`者，存储货物或供旅客住宿的地方，可引申为仓库

**栈**（类似好多的计算机系统中的概念）是计算机科学工作者抽象出来的概念（客观世界中只有内存这个东西），所谓科学，其实是从认识和实践客观事物中抽象出一定规律，并在这个规律中进行演绎，最后反过来指导认识和实践客观事物的过程。

> 所谓科学，即类似于取之于民，用之于民

### 1.1 数据结构中的栈

- 栈是一组数据的存放方式，特点是先进后出，后进先出

![instack](https://gitee.com/ppambler/blog-images/raw/master/fe/VZpCelM6cbwSEHX.png)

![outstack](https://gitee.com/ppambler/blog-images/raw/master/fe/20210516203728.png)

> 想想羽毛球桶 -> 拿羽毛球的规则是怎样的……

| 方法名 | 操作                                 |
| :----- | :----------------------------------- |
| push() | 添加新元素到栈顶                     |
| pop()  | 移除栈顶的元素，同时返回被移除的元素 |

```ts
class Stack {
    private items: number[] = [];
    // 添加元素到栈顶，也就是栈的末尾
    push(element: number) {
        this.items.push(element);
    }
    // 栈的后进先出原则，从栈顶出栈
    pop(): number | undefined {
        return this.items.pop();
    }
}

let stack = new Stack();
// stack.push(1);
// stack.push(2);
// stack.push(3);
console.log(stack.pop());
// console.log(stack.items) // items 只能在 class 里边被访问
```

### 1.2 代码的运行方式 

- 表示函数的一层层调用

![image-20210516205937505](https://gitee.com/ppambler/blog-images/raw/master/fe/20210516205937.png)

> 代码执行完，意味着 `anonymous` 也就消失了 -> 这 TM 就是 Call Stack

### 1.3 内存区域

- 栈也是存放数据的一种内存区域
- 程序运行的时候，需要内存空间存放数据。一般来说，系统会划分出两种不同的内存空间：一种叫做 stack（栈），另一种叫做 heap（堆）
  - stack 是有结构的，每个区块按照一定次序存放，可以明确知道每个区块的大小
  - heap 是没有结构的，数据可以任意存放。因此，stack 的寻址速度要快于 heap
- 只要是局部的、占用空间确定的数据，一般都存放在 stack 里面，否则就放在 heap 里面，所有的对象都存放在 heap

> **内存对于 cpu 来说就是一段连续的数组（整个内存容量）** -> 栈就在这个连续的数组中的某一段

```js
function task() {
    var a = 1;
    var b = 2;
    var c = {
        name: 'zhufeng',
        age: 10
    }
}
task();
```

![memoryposition](https://gitee.com/ppambler/blog-images/raw/master/fe/20210516210436.jpeg)

> 如果一行就是一个数组单元，那么整个仓库就是个内存条 -> 如果一行就是一个堆 -> 如果一行就是一个对象，像 `A1` 就是它的内存地址，那么整个仓库就是一个堆

## 2. 队列

- 队列是一种操作受限制的线性表
- 特殊之处在于它只允许在表的前端进行删除操作，而在表的后端进行插入操作
- 进行插入操作的端称为队尾，进行删除操作的端称为队头
- 因为队列只允许在一端插入，在另一端删除，所以只有最早进入队列的元素才能最先从队列中删除，故队列又称为先进先出线性表

![inqueue.png](https://gitee.com/ppambler/blog-images/raw/master/fe/20210516220129.png)

![outqueue.png](https://gitee.com/ppambler/blog-images/raw/master/fe/20210516220143.png)

```tsx
class Queue {
    private items: number[] = [];
    // 添加元素到栈顶，也就是栈的末尾
    enqueue(element: number) {
        this.items.push(element);
    }
    dequeue() {
        return this.items.shift();
    }
}

let queue = new Queue();
queue.enqueue(1);
queue.enqueue(2);
queue.enqueue(3);
console.log(queue.dequeue());//1
```

## 3. 数据类型

JS 中有七种基本数据类型

- 六种基本数据类型 Boolean Number String Null Undefined Symbol
- 一种引用类型 Object -> `{} [] /^$/ new Date() Math`

| 类型      | 值          |
| :-------- | :---------- |
| Boolean   | true 或 false |
| Null      | null        |
| Undefined | undefined   |
| Number    | 数字        |
| String    | 字符串      |
| Symbol    | 符号类型    |

## 4. 执行上下文

### 4.1 如何存储

- 当函数运行时，会创建一个执行环境，这个执行环境就叫执行上下文 (Execution Context)
- 执行上下文中会创建一个对象叫作变量对象 (Value Object), 基础数据类型都保存在变量对象中
- 引用数据类型的值保存在堆里，我们通过操作对象的引用地址来操作对象

![image-20210516234530226](https://gitee.com/ppambler/blog-images/raw/master/fe/20210516234530.png)

![valueobject](https://gitee.com/ppambler/blog-images/raw/master/fe/20210517000413.png)

> ECStack -> EC(Global) -> EC(Local)
>
> 代码在执行前，浏览器会创建一个可执行环境，即 `ECStack`（执行环境栈，分配一块栈内存给程序使用） -> `script`里边的代码执行，在执行前会初始化这件事情：`GO`、`VO`、`[[scope]] -> VO` ，执行开始会确定 `scopeChain -> 上一级的空 VO`
>
> 函数的创建，会初始化`[[scope]]`（确定自己是哪个作用域内被创建的） -> 而函数的执行，会确定 `scopeChain -> 上一级的 VO `、`VO` 、`this`
>
> 闭包 -> 保护作用（外界不能访问函数内部的 `VO`）、保存作用（函数执行完，EC 会被释放，如果 `VO`里边有东西 `xxx` 被全局某个东西间接使用着，那么这个 `xxx` 是会被保存下来的）

### 4.2 如何复制

#### 4.2.1 基本数据

- 基本数据类型复制的是值本身

```js
var a = 1;
var b = a;
b = 2;
console.log(a);
```

```js
var ExecuteContext = {
    VO: { a: 1 }
};

ExecuteContext.VO.b = ExecuteContext.VO.a;
ExecuteContext.VO.b = 2;
console.log(ExecuteContext.VO.a);
```

![changebasictype](https://gitee.com/ppambler/blog-images/raw/master/fe/20210517090439.png)

> 叫你去用这个东西 -> 你得自己买

#### 4.2.2 引用数据

- 引用数据类型复制的是引用地址指针

```js
var m = { a: 1, b: 2 };
var n = m;
n.a = 10;
console.log(m.a);
```

```js
var ExecuteContext = {
    VO: { m: { a: 1, b: 2 } }
};

ExecuteContext.VO.n = ExecuteContext.VO.m;
ExecuteContext.VO.n.a = 10;
console.log(ExecuteContext.VO.m.a);
```

![copyrefer](https://gitee.com/ppambler/blog-images/raw/master/fe/20210517091150.png)

> 一起用这个东西

## 5. 多个执行上下文栈

### 5.1 执行上下文分类

- JS 代码在执行的时候会进入一个执行上下文，可以理解为当前代码的运行环境
- 在 JS 中运行环境主要分为全局执行上下文环境和函数环执行上下文环境
  - 全局执行上下文只有一个，在客户端中一般由浏览器创建，也就是我们熟知的 window 对象，我们能通过 this 直接访问到它
  - window 对象还是 var 声明的全局变量的载体。我们通过 var 创建的全局对象，都可以通过 window 直接访问

> 全局执行上下文真得是 `window` 对象吗？那 `window.this === undefined`怎么说？ -> `this`与 `window`是同级的啊！

### 5.2 多个执行上下文

- 在 JS 执行过程会产出多个执行上下文，JS 引擎会有栈来管理这些执行上下文
- 执行上下文栈（下文简称执行栈）也叫调用栈，执行栈用于存储代码执行期间创建的所有上下文，具有 LIFO（Last In First Out 后进先出，也就是先进后出）的特性
- 栈底永远是全局上下文，栈顶为当前正在执行的上下文
- 当开启一个函数执行时会生成一个新的执行上下文并放入调用栈，执行完毕后会自动出栈

![image-20210517104521839](https://gitee.com/ppambler/blog-images/raw/master/fe/20210517104521.png)

```js
var globalExecuteContext = {
    VO: { setTimeout: 'setTimeout' }
}
var executeContextStack = [globalExecuteContext];
var oneExecuteContext = {
    VO: { a: 1 }
}
executeContextStack.push(oneExecuteContext);
var twoExecuteContext = {
    VO: { b: 2 }
}
executeContextStack.push(twoExecuteContext);
var threeExecuteContext = {
    VO: { c: 3 }
}
executeContextStack.push(threeExecuteContext);
console.log(executeContextStack);

executeContextStack.pop();
```

> Web APIs 难道不是 `window` 旗下的吗？ -> 所谓的 GO 就是 VO 吗？ -> GO、VO、window、this、`[[scope]]` -> `window`指向`GO`，`VO`里边存储的是被声明的变量，`let/const`声明的变量不会放到 `GO` 旗下，`var` 声明的变量则会放到 `GO` 旗下，所以 `window`旗下会有 `var` 声明的变量，但不管怎样，全局声明的变量，都是全局变量！

## 6. 执行上下文生命周期

### 6.1 生命周期有两个阶段

- 一个新的执行上下文的生命周期有两个阶段
  - 创建阶段
    - 创建变量对象
    - 确定作用域链
    - 确定`this`指向
  - 执行阶段
    - 变量赋值
    - 函数赋值
    - 代码执行

### 6.2 变量对象

- 变量对象会保存变量声明 (var)、函数参数 (arguments)、函数定义 (function)
  - 变量对象会首先获得函数的参数变量和值
  - 获取所有用`function`进行的函数声明，函数名为变量对象的属性名，值为函数对象，如果属性已经存在，值会用新值覆盖
  - 再依次所有的`var`关键字进行的变量声明，每找到一个变量声明，就会在变量对象上建一个属性，值为`undefined`, 如果变量名已经存在，则会跳过，并不会修改原属性值，`let`声明的变量并不会在此阶段进行处理
- 函数声明优先级更高，同名的函数会覆盖函数和变量，但同名`var`变量并不会覆盖函数。执行阶段重新赋值可以改变原有的值

#### 6.2.1 基本类型

```js
console.log(a);
var a = 1;
```

```js
var a = undefined; // 变量提升  -> 创建并初始化
console.log(a); 
a = 1;
```

#### 6.2.2 变量提升

- 正常编写

```js
var a = 1;
function fn(m) { console.log('fn'); }
function fn(m) { console.log('new_fn'); }
function a() { console.log('fn_a'); }
console.log(a);
fn(1);
var fn = 'var_fn';
console.log(fn);
// 1
// new_fn
// var_fn
```

- 真正执行

![image-20210517130038956](https://gitee.com/ppambler/blog-images/raw/master/fe/20210517130039.png)

- 上下文

```js
// 创建阶段
var globalEC = {
    VO: {
        ...arguments,
        a: () => { console.log('fn_a'); },
        fn: () => { console.log('new_fn'); }
    }
}
var ECStack = [globalEC];

//执行阶段
globalEC.VO.a = 1;
console.log(globalEC.VO.a);
globalEC.VO.fn();
globalEC.VO.fn = 'var_fn';
console.log(globalEC.VO.fn);
```

#### 6.2.3 激活对象

- 在函数的调用栈中，如果当前执行上下文处于函数调用栈的顶端，则意味着当前上下文处于激活状态，此时变量对象称为活动对象 (AO,Activation Object) `VO -> AO`
- 活动变量包含变量对象所有的属性，并有包含`this`指针

```js
function one(m) {
    function two() {
        console.log('two');
    }
}
one(1);

// 执行阶段 VO => AO
let VO = AO = {
    m:1,
    two: () => { console.log('two'); },

}
let oneEC={
    VO,
    this: window,
    scopeChain:[VO,globalVO]
}
```

#### 6.2.4 全局上下文的变量对象

- 在浏览器里，全局对象为`window`
- 全局上下文的变量对象为`window`, 而且这个变量对象不能激活变成活动对象
- 只在窗口打开，全局上下文会一直存在，所有的上下文都可以直接访问全局上下文变量对象上的属性
- 只有全局上下文的变量对象允许通过 VO 的属性名称来间接访问，在函数上下文中是不能直接访问 VO 对象的
- 未进入执行阶段前，变量对象中的属性都不能访问！但是**进入到执行阶段之后，变量对象转变成了活动对象**，里面的属性都能被访问了，对于函数上下文来讲，活动对象与变量对象其实都是同一个对象，**只是处于执行上下文的不同生命周期**

## 7. 作用域

### 7.1 作用域

- 在 JS 中，作用域是用来规定变量访问范围的规则

```js
function one() {
    var a = 1;
}
console.log(a);
```

### 7.2 作用域链

- 作用域链是由当前执行环境与上层执行环境的一系列变量对象组成的，它保证了当前执行环境对符合访问权限的变量和函数的**有序访问**

#### 7.2.1 例 1

```js
function one() {
    var a = 1;
    function two() {
        var b = 2;
        function three() {
            var c = 3;
            console.log(a, b, c);
        }
        three();
    }
    two();
}
one(); 
```

![image-20210517191640607](https://gitee.com/ppambler/blog-images/raw/master/fe/20210517191859.png)

```js

// 1. 创建全局上下文
var globalExecuteContextVO = { one: `()=>{var a = 1;}` }
var globalExecuteContext = {
    VO: globalExecuteContextVO,
    scopeChain: [globalExecuteContextVO]
}
var executeContextStack = [globalExecuteContext];
//2. 执行 one，创建 one 执行上下文
var oneExecuteContextVO = {
    a: 1,
    two: `()=>{var b = 2 ;}`
}
var oneExecuteContext = {
    VO: oneExecuteContextVO,
    scopeChain: [oneExecuteContextVO, globalExecuteContext.VO]
}
//2. 执行 two，创建 two 执行上下文
var twoExecuteContextVO = {
    b: 2,
    three: `()=>{var c = 3 ;}`
}
var twoExecuteContext = {
    VO: twoExecuteContextVO,
    scopeChain: [twoExecuteContextVO, oneExecuteContext.VO, globalExecuteContext.VO]
}
//3. 执行 three，创建 three 执行上下文
var threeExecuteContextVO = {
    c: 3
}
var threeExecuteContext = {
    VO: threeExecuteContextVO,
    scopeChain: [threeExecuteContextVO, twoExecuteContext.VO, oneExecuteContext.VO, globalExecuteContext.VO]
}

// 遍历作用域链，从自身开始，一直往上一级走……
function getValue(varName) {
    for (let i = 0; i < threeExecuteContext.scopeChain.length; i++) {
        if (varName in threeExecuteContext.scopeChain[i]) {
            return threeExecuteContext.scopeChain[i][varName];
        }
    }
}
//console.log(a, b, c);
console.log(
    getValue('a'),
    getValue('b'),
    getValue('c'),
);
```

> 在执行某个函数的时候，会创建执行上下文，此时会有个创建阶段，用于初始化 `VO` 和 `scopeChain`，等到真正执行这个函数的时候，`VO`也就成了`AO`，表示此时的 `Call Stack` 的栈顶是这个函数的执行上下文！

#### 7.2.2 例 2

- `scopeChain`其实是在创建函数的时候确定的

```js
function one() {
    var a = 1;
    function two() {
        console.log(a);
    }
    return two;
}
var a = 2;
var two = one();
two(); // 1
```

 

```js

// 1. 创建全局上下文
var globalExecuteContextVO = { one: `()=>{var a = 1;}`, a: undefined, two: undefined }
var globalExecuteContext = {
    VO: globalExecuteContextVO,
    scopeChain: [globalExecuteContextVO]
}
//2. 开始执行
globalExecuteContextVO.a = 2;
//3. 开始执行 one
var oneExecuteContextVO = { a: undefined, two: `()=>{console.log(a)}` }
var oneExecuteContext = {
    VO: oneExecuteContextVO,
    scopeChain: [oneExecuteContextVO, globalExecuteContextVO]
}
oneExecuteContextVO.a = 1;
//4. 给 two 赋值
globalExecuteContextVO.two = oneExecuteContextVO.two;
//5. 执行 two
var twoExecuteContextVO = {}
var twoExecuteContext = {
    VO: twoExecuteContextVO,
    //scopeChain 是在创建此函数据的时候就决定了，跟在哪里执行无关
    scopeChain: [twoExecuteContextVO, oneExecuteContextVO, globalExecuteContextVO]
}
```

> 当你执行 `two` 函数的时候，它在找 `a` 的值，是从作用域链开始往上找的，而作用域链的初始化，是在函数创建时确定的！

## 8. 闭包

- 闭包有两部分组成，一个是当前的执行上下文 A，一个是在该执行上下文中创建的函数 B
- 当 B 执行的时候引用了当前执行上下文 A 中的变量就会产出闭包
- 当一个值失去引用的时候就会会标记，被垃圾收集回收机回收并释放空间
- 闭包的本质就是在函数外部保持内部变量的引用，从而阻止垃圾回收
- 调用栈的并不会影响作用域链，函数调用栈是在执行时才确定，而作用域规则是在代码编译阶段就已经确定了
- MDN 定义：闭包是指这样的作用域`foo`, 它包含了一个函数`fn`，这个函数`fn1`可以调用被这个作用域所封闭的变量`a`、函数等内容

### 8.1 闭包

- `Call Stack`为当前的函数调用栈
- `Scope`为当前正在被执行函数的作用域链
- `Local`为当前的活动对象

```js
function one() {
    var a = 1;
    var b = 2;
    function two() {
        var c = 3;
        debugger;
        console.log(a,c);
    }
    return two;
}
let two = one();
two();
```

![image-20210517203041358](https://gitee.com/ppambler/blog-images/raw/master/fe/20210517203041.png)

```js
function one() {
    var a = 1;
    var b = 2;
    function two() {
        debugger;
        console.log(a);
    }
    two();
}
one();
```

![image-20210517204327489](https://gitee.com/ppambler/blog-images/raw/master/fe/20210517204327.png)

关于 `Scope` 旗下的 `Script` 测试：

![image-20210517204940503](https://gitee.com/ppambler/blog-images/raw/master/fe/20210517204940.png)

> 函数 `one` 也在 `Global` 里边 -> `Script`存储的是用 `let`、`const` 声明的变量

`Closure`是特殊的，它旗下的值是可以被保存下来的，相当于在 `two` 的作用链途中留下了版本记录或者说痕迹

```js
function one() {
    var a = 1;
    var b = 2;
    function two() {
        var c = 3;
        debugger;
        console.log(++a);
    }
    return two;
}
let x = 666
var y = 777
const z = 888
let two = one();
two(); // 2
// 第二次执行 two，two 中的 a 会找到上一次执行 two 所记录的 a 值
two(); // 3
```

### 8.2 闭包优化

- 中间没用到的变量闭包会被忽略

```js
function one() {
    var a = 1;
    function two() {
        var b = 2;
        function three() {
            var c = 3;
            debugger;
            console.log(a, b, c);
        }
        three();
    }
    two();
}
one();
```

![image-20210517205700523](https://gitee.com/ppambler/blog-images/raw/master/fe/20210517205700.png)

```js
function one() {
    var a = 1;
    function two() {
        var b = 2;
        function three() {
            var c = 3;
            debugger;
            console.log(a, c);
        }
        three();
    }
    two();
}
one();
```

![image-20210517205924886](https://gitee.com/ppambler/blog-images/raw/master/fe/20210517205924.png)

### 8.3 arguments

```js
function log(a, b) {
    debugger;
    console.log(a, b);
}
log(1, 2);
```

![image-20210517210318858](https://gitee.com/ppambler/blog-images/raw/master/fe/20210517210318.png)

## ★了解更多

➹：[栈究竟在哪里？ - FanMo 的回答 - 知乎](https://www.zhihu.com/question/319926787/answer/651578913) 

➹：[TypeScript: 游乐场 - 一个用于 TypeScript 和 JavaScript 的在线编辑器](https://www.typescriptlang.org/zh/play#)

➹：[04-复习上节课的作业题「详细讲解：VO、AO、GO 以及一些其它细节知识点」 - zf-fe](https://ppambler.github.io/zf-fe/02-JS/04.html)

➹：[闭包 - JavaScript - MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures)

➹：[JavaScript Scope and Closures | CSS-Tricks](https://css-tricks.com/javascript-scope-closures/)

➹：[backbone.js - Chrome javascript tools: Global, Closure and Local Scope - Stack Overflow](https://stackoverflow.com/questions/13002072/chrome-javascript-tools-global-closure-and-local-scope)

➹：[function - How do JavaScript closures work? - Stack Overflow](https://stackoverflow.com/questions/111102/how-do-javascript-closures-work)
