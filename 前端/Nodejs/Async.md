# Async

## 1. 异步

- 所谓"异步"，简单说就是一个任务分成两段，先执行第一段，然后转而执行其他任务，等做好了准备，再回过头执行第二段，比如，有一个任务是读取文件进行处理，异步的执行过程就是下面这样。

![image-20210331111505336](https://gitee.com/ppambler/blog-images/raw/master/images/20210331111505.png)

这种不连续的执行，就叫做异步。相应地，连续的执行，就叫做同步。

![image-20210331111626915](https://gitee.com/ppambler/blog-images/raw/master/images/20210331111627.png)

## 2. 高阶函数

函数作为一等公民，可以作为参数和返回值

### 2.1 可以用于批量生成函数

```js
let toString = Object.prototype.toString;
let isString = function (obj) {
  return toString.call(obj) == `[object String]`;
}
let isFunction = function (obj) {
  return toString.call(obj) == `[object Function]`;
}
let isType = function (type) {
  return function (obj) {
    return toString.call(obj) == `[object ${type}]`;
  }
}
```

如：`let isArray = isType('Array')` -> 用返回一个函数的姿势来批量生成函数

### 2.2 可以用于需要调用多次才执行的函数

```js
let after = function(times,task){
  return function(){
    if(times--==1){
      return task.apply(this,arguments);
    }
  }
}
// 在函数被调用三次后再执行任务
let fn = after(3,function(){ console.log(3); });
fn();
fn(); // 3
```

## 3. 异步编程的语法目标，就是怎样让它更像同步编程，有以下几种

* 回调函数实现
* 事件监听
* 发布订阅
* Promise/A+ 和生成器函数
* async/await

## 4. 回调

所谓回调函数，就是把任务的第二段单独写在一个函数里面，等到重新执行这个任务的时候，就直接调用这个函数

```js
fs.readFile('某个文件', function (err, data) {
  if (err) throw err;
  console.log(data);
});
```

这是一个错误优先的回调函数 (error-first callbacks), 这也是 Node.js 本身的特点之一。

> 没想到「回调」这个概念是这样理解的（这是异步回调的理解姿势），之前的理解是什么打电话之类的！其它理解：一个函数被传入之后又被调用就叫回调函数，注意，回调和异步没有必然联系，上边的代码给了我们一种，说到回调就关联起了异步这个概念 -> 同步回调，调用一个函数，直接在函数体里边执行 `callback`

➹：[「每日一题」Callback（回调）是什么？ - 知乎](https://zhuanlan.zhihu.com/p/22677687)

## 5. 回调的问题

### 5.1 异常处理

```js
try{
  //xxx
}catch(e){
	//TODO
}
```

异步代码时`try catch`不再生效

```js
let async = function(callback){
  try{
    setTimeout(function(){
      callback();
    },1000)
  }catch(e){
    console.log('捕获错误',e);
  }
}

async(function(){
  console.log(t); // Uncaught ReferenceError: t is not defined
});
```

因为这个回调函数被存放了起来，直到下一个事件环的时候才会取出，try 只能捕获当前循环内的异常，对 callback 异步无能为力。

Node 在处理异常有一个约定，将异常作为回调的第一个实参传回，如果为空表示没有出错。

```js
async(function(err,callback){
  if(err){
    console.log(err);
  }
});
```

异步方法也要遵循两个原则

- 必须在异步之后调用传入的回调函数

- 如果出错了要向回调函数传入异常供调用者判断

  ```js
  let async = function (callback) {
    try {
      setTimeout(function () {
        // 异步之后  
        if (success) callback(null);
        else callback('错误');
      }, 1000);
    } catch (e) {
      console.log('捕获错误', e);
    }
  };
  ```

### 5.2 回调地狱

异步多级依赖的情况下嵌套非常深，代码难以阅读的维护

```js
let fs = require('fs');
fs.readFile('template.txt', 'utf8', function (err, template) {
  fs.readFile('data.txt', 'utf8', function (err, data) {
    console.log(template + ' ' + data);
  });
});
```

> 异步`A`的`callback`嵌套着一个异步`B`，异步`B`的`callback`嵌套着一个异步`C`……

## 6. 异步流程解决方案

### 6.1 事件发布/订阅模型

订阅事件实现了一个事件与多个回调函数的关联

```js
let fs = require('fs');
let EventEmitter = require('events');
let eve = new EventEmitter();
let html = {};
eve.on('ready',function(key,value){
  html[key] = value;
  if(Object.keys(html).length==2){
    console.log(html);
  }
});
function render(){
  fs.readFile('template.txt','utf8',function(err,template){
    eve.emit('ready','template',template);
  })
  fs.readFile('data.txt','utf8',function(err,data){
    eve.emit('ready','data',data);
  })
}
render();
```

### 6.2 哨兵变量

```js
let fs = require("fs");

let after = function (times, callback) {
  let result = {};
  return function (key, value) {
    result[key] = value;
    if (Object.keys(result).length == times) {
      callback(result);
    }
  };
};
let done = after(2, function (result) {
  console.log(result);
});

function render() {
  fs.readFile("template.txt", "utf8", function (err, template) {
    console.log(1);
    done("template", template);
  });
  fs.readFile("data.txt", "utf8", function (err, data) {
    console.log(2);
    done("data", data);
  });
}
render();
// 1
// 2
// { template: xxx, data: xxx }
```

> 哨兵：顾名思义，指站岗、放哨、巡逻、稽查的士兵。在 `while` 循环里边，我们用 `i` 控制最终的循环次数，如 `let i = 0; while(i < 10) { ++i }` -> 具有这个作用的变量有时也称为哨兵变量（Sentinel variable） -> `2`即`times`就是哨兵变量

### 6.3 Promise/Deferred 模式

promise/deferred 模式是，根据 promise/A 或者它的增强修改版 promise/A+ 规范 实现的 promise 异步操作的一种实现方式。

> 什么是规范，规范其实就相当于制定的规则，但却没有在代码层面上有默认的具体实现

promise/deferred 模式包含两部分：Promise 和 Deferred。

- Deferred 主要是用于内部，来维护异步模型的状态。
- Promise 只要用于外部，通过`then()`方法，暴露给外部调用，以添加业务逻辑和业务的组装。

promise 和 deferred 的关系图：

![图片描述](https://gitee.com/ppambler/blog-images/raw/master/images/20210331143625.png)

从图中可以看到：

```txt
1. deferred 对象通过 resolve 方法，改变自身状态为执行态，并触发 then() 方法的 onfulfilled 回调函数
2. deferred 对象通过 reject 方法，改变自身状态为拒绝态，并触发 then() 方法的 onrejected 回调函数
```

➹：[promise/deferred 模式原理分析和实现 - SegmentFault 思否](https://segmentfault.com/a/1190000010167155)

### 6.4 生成器 Generators/ yield

- 当你在执行一个函数的时候，你可以在某个点暂停函数的执行，并且做一些其他工作，然后再返回这个函数继续执行， 甚至是携带一些新的值，然后继续执行。
- 上面描述的场景正是 JavaScript 生成器函数所致力于解决的问题。当我们调用一个生成器函数的时候，它并不会立即执行， 而是需要我们手动的去执行迭代操作（`next`方法）。也就是说，你调用生成器函数，它会返回给你一个迭代器。迭代器会遍历每个中断点。
- `next` 方法返回值的 `value` 属性，是 Generator 函数向外输出数据；`next` 方法还可以接受参数，这是向 Generator 函数体内输入数据

#### 6.4.1 生成器的使用

```js
function* foo() {
  var index = 0;
  while (index < 2) {
    // yield 后的返回值，就是 value 的值
    yield index++; // 暂停函数执行，并执行 yield 后的操作
  }
}
var bar = foo(); // 返回的其实是一个迭代器

console.log(bar.next()); // { value: 0, done: false }
console.log(bar.next()); // { value: 1, done: false }
// foo return 后，意味着 bar.next() 的意义结束了，毕竟再这么执行，都是返回一样的结果
console.log(bar.next()); // { value: undefined, done: true }
```

#### 6.4.2 Co

`co`是一个为`Node.js`和浏览器打造的基于生成器的流程控制工具，借助于 Promise，你可以使用更加优雅的方式编写非阻塞代码。

```js
let fs = require("fs");
function readFile(filename) {
  return new Promise(function (resolve, reject) {
    fs.readFile(filename, function (err, data) {
      if (err) reject(err);
      else resolve(data);
    });
  });
}
function* read() {
  // template.txt -> <div></div>
  let template = yield readFile("./template.txt");
  // data.txt -> frank
  let data = yield readFile("./data.txt");
  return template + "+" + data;
}
co(read).then(
  function (data) {
    console.log(data);
  },
  function (err) {
    console.log(err);
  }
);
```

```js
function co(gen) {
  let it = gen();
  return new Promise(function (resolve, reject) {
    !(function next(lastVal) {
      let { value, done } = it.next(lastVal);
      if (done) {
        resolve(value);
      } else {
        value.then(next, (reason) => reject(reason));
      }
    })();
  });
}
```

![image-20210331145026037](https://gitee.com/ppambler/blog-images/raw/master/images/20210331145026.png)

## 6.5 Async/ await

使用`async`关键字，你可以轻松地达成之前使用生成器和 co 函数所做到的工作

### 6.5.1 Async 的优点

- 内置执行器
- 更好的语义
- 更广的适用性

```js
let fs = require("fs");
function readFile(filename) {
  return new Promise(function (resolve, reject) {
    fs.readFile(filename, "utf8", function (err, data) {
      if (err) reject(err);
      else resolve(data);
    });
  });
}

async function read() {
  console.log(1);
  let template = await readFile("./template.txt");
  console.log(4);
  let data = await readFile("./data.txt");
  console.log(5);
  return template + "+" + data;
}
let result = read(); // 在 await readFile 的结果时，就执行下边的代码了
console.log(result); // Promise { <pending> }
console.log(2);
result.then((data) => console.log(data)); // <div></div>+frank
console.log(3);
```

![image-20210331150113049](https://gitee.com/ppambler/blog-images/raw/master/images/20210331150113.png)

### 6.5.2 async 函数的实现

```js
async function read() {
  let template = await readFile("./template.txt");
  let data = await readFile("./data.txt");
  return template + "+" + data;
}

// 等同于
function read() {
  return co(function* () {
    let template = yield readFile("./template.txt");
    let data = yield readFile("./data.txt");
    return template + "+" + data;
  });
}
```

参考：

- [async 函数](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/async_function)
- [generator](https://www.ruanyifeng.com/blog/2015/04/generator.html)
