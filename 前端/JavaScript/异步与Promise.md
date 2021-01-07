# 异步与 Promise

## ★同步与异步

- 同步（能直接拿到结果）
  - 死等，不拿到结果不会离开
  - 例如：医院挂号
- 异步（不能直接拿到结果）
  - 取号后可以离开，不影响拿结果
  - 每隔十分钟去问问店家（轮询）
  - 店家电话通知（回调）

> 同步：等结果；异步：不等结果，之后透过回调拿到结果！

### 1）异步举例

JS 中请求到响应的时间大约几百毫秒到 1~2 秒

* 以 AJAX 为例
  * `request.send()` 之后，并不能直接得到 `response`
  * 必须等到 `readyState` 变为 `4` 后（下载完成），浏览器回调 `request.onreadystatechange` 函数
  * 才能得到 `request.response`

这就好比，你在餐厅等位，不能马上入座，就先去逛街，等到餐厅有座服务员电话通知你，才能就座入席。

> 你要「餐厅位置」这个结果，但餐厅不能立刻给你，毕竟目前是满座的状态，于是你就继续去玩，玩够了，就等着餐厅打电话 call 你说「先生，你的位置在 `xxx` 号！」 -> 为什么餐厅会知道你的电话号码？那是你留下来给餐厅的！

![image-20201203121251641](https://gitee.com/ppambler/blog-images/raw/master/images/image-20201203121251641.png)

* 回调 (callback)
  * 写给自己用的函数，不是回调
  * 写给别人用的函数，才是回调
  * 比如预留电话这个函数，不是给自己留的，是给店家留的
  * `request.onreadystatechange` 就是写给浏览器调用的
  * 告诉浏览器回头调用一下这个函数

写了自己不调用，来给别人（回头）调用的函数，就是回调函数。

> 如果回调函数是 `call`这个函数，那么这个函数会等到有「餐位」这个结果之后，才会`push`到任务队列里边去，等`stack`清空之后，事件循环才会按顺序`push`这个`call`函数到`stack`里边交给 JS 引擎去解析执行！ -> 说白了，`call`能被叫做回调函数，那是因为它经历了一段特殊的历程，才会被调用！ -> 再简单来说，我走了，又回来了才会被执行，而不是我不走，我就待着执行就好了！

### 2）回调举例

把函数 1 给另一个函数 2

``` js
function f1(){}
function f2(fn){fn()}
f2(f1)
```

分析：

- f1 没有被直接调用
- f1 被传给了 f2
- f2 调用了 f1
- 因此 f1 就是我给 f2 写的函数，即回调函数

## ★异步和回调的关系

* 关联
  * 异步任务需要在得到结果时通知 JS 来拿结果
  * 可以让 JS 留一个函数地址（电话号码）给浏览器
  * 异步任务完成时浏览器调用该函数地址即可（拨打电话）
  * 同时把结果作为参数传给该函数（电话里说有座位了过来吧）
  * 该函数是我写给浏览器调用的，因此是回调函数
* 区别
  * 异步任务需要用回调函数来通知结果
  * 异步任务常用回调，但不一定用回调，还可以用轮询
  * 回调函数也不一定只用在异步任务里，也可以用在同步任务
  * `array.forEach(n=>console.log(n))`就是同步回调

> 什么叫任务？完成一件事就是你的任务！

## ★如何判断同步异步

* 若一个函数的返回值处于以下三项内部，则该函数就是异步函数：
  * setTimeout
  * AJAX（即 XMLHttpRequest）
  * addEventListener
  * 其他异步 API 另行说明

虽然 AJAX 可以设置为同步（open 第三个参数），但是这样页面会在请求期间卡住（智障才同步）。

> 文字，还是比较抽象的，看代码比较好理解 -> 什么是色情？不知道，但我看了这个东西就知道这是不是色情了……

### 1）举例分析

``` js
function 摇骰子 (){
  setTimeout(()=>{
    return Math.floor(Math.random()*6)+1
  }, 1000)
  // return underfined
}
```

`Math.random` 用于生成一个 `[0, 1)` 的随机数，通过与 `6` 相乘得到一个范围在 `[0, 1*6)` 的随机数，`Math.floor` 向下取整后加 `1`，即可得到范围 `[1,6]` 的随机整数

分析：

* 若函数中没有写返回值，则默认返回值就是`undefined`
* 摇骰子函数调用的箭头函数的返回值是真正的结果
* 因此这是一个异步任务

> 我是第一次学到这样分析一个函数是否是异步任务的！
> 
> 如果把箭头函数抽出来为一个具名函数，作为`摇骰子`这个函数的参数传入，那么`摇骰子`就更像是一个异步任务了呀！

💡：如何拿到异步的结果？

我们这样做：

``` js
function 摇骰子 (){
  setTimeout(()=>{
    return Math.floor(Math.random()*6)+1
  }, 1000)
  // return underfined
}
const n = 摇骰子 ()
console.log(n) // undefined
```

显然无法拿到

但我们可以用回调完成这件事。写个回调函数 f1，将 f1 作为参数传给摇骰子，然后在摇骰子函数得到结果后，将结果作为参数传给 f1：

``` js
function 摇骰子 (fn){
  setTimeout(()=>{
    fn(Math.floor(Math.random()*6)+1)
  }, 1000)
}
function f1(x){console.log(x)}

摇骰子 (f1)
```

简化为箭头函数：

> 由于 f1 声明后只用了一次，因此可以用匿名函数或箭头函数简化

``` js
function 摇骰子 (fn){
  setTimeout(()={
    fn(Math.floor(Math.random()*6)+1)
  }, 1000)
}
摇骰子 (x=>console.log(x))
// 再简化为
摇骰子 (console.log)
// 如果参数不相等，就不能这样简化
```

## ★小结

* 异步任务拿不到结果
* 于是传一个回调函数给异步任务
* 在异步任务完成时调用回调函数
* 调用时将完成结果作为回调函数的参数

![image-20201203164114630](https://gitee.com/ppambler/blog-images/raw/master/images/image-20201203164114630.png)

> `饭店`函数是异步任务，我们传了一个`callMe`给异步任务，异步任务完成时，也就是`seTimeout`时间到了，会把传给定时器的箭头函数扔到任务队列里边去，等 Stack 空空如也的时候，会把箭头函数 `push` 到 `callStack`里边执行 -> 执行`callMe`

一个疑问：所谓的异步任务完成，指的是已经把箭头函数推送到调用栈执行了？还是说把箭头函数扔到任务队列里边去就算完成了？

按照个人的理解：箭头函数到任务队列里边了，意味着异步任务已经完成了，相当于是「饭店已经有空座位了，只是还没打电话，因为不太确定客人是否有空，比如客人还在做着某些重要的事儿……」，客人通知任务队列说「我此刻无事可干」，于是饭店就会打电话给客人说「有座位了！」

> `Stack`清空后，浏览器会轮询任务队列说「任务完成了吗？」，如果完成了，会`push`一个`callback`到`Stack`里边去执行！

## ★如果异步的结果是成功或失败

### 1）方法一：回调函数接收两个参数

``` js
fs.readFileSync('./1.txt', (error, data)=>{
  if(error){
    console.log('失败')
  }else{
    console.log(data.toString())
  }
})
```

> 把第二个参数看成是`callMe`，`readFileSync`内部的实现会这样调用的：`callMe(error,data)`

### 2）方法二：使用两个回调函数

``` js
// 接收一个成功回调函数，再接收一个失败回调函数
ajax('GET', '/1.json', data=>{}, error=>{})
// 接收一个对象，对象中有两个 key 分别表示成功和失败的回调函数
ajax('GET', '/1.json', {success:()=>{}, fail:()=>{}})
```

### 3）不足之处

面试常考：**为什么用 Promise？** （三点）

1. 不规范，成功和失败回调函数的名称五花八门（success+error, success+fail, done+fail）
2. 容易出现回调地狱，代码可读性差
3. 难以进行错误处理

如同波动拳 (Hadoken) 一般的回调地狱（Callback Hell）：

![image-20201203172015154](https://gitee.com/ppambler/blog-images/raw/master/images/image-20201203172015154.png)

💡：如何解决？

* 规范回调函数的名称和顺序
* 拒绝回调地狱
* 让捕获错误更方便

为此，前端程序员开始翻书查资料借鉴后端知识：

1976 年，Daniel P. Friedman 和 David Wise 提出了 Promise 思想，后人基于此发明了 Future、Delay、Deferred 等。前端结合 Promise 和 JS，制订了 [Promise/A+规范](https://www.ituring.com.cn/article/66566)，该规范中详细描述了 Promise 的原理和使用方法。

## ★Promise：以 AJAX 封装为例

### 1）老 jQuery 写法

``` js
ajax = (method, url, options)=>{
  const {success, fail} = options // 析构赋值
  // const success = options.success
  // const fail = options.fail
  const request = new XMLHttpRequest()
  request.open(method, url)
  request.onreadystatechange = ()=>{
    if(request.readyState===4){
      if(request.status < 400){
        success.call(null, request.response)
      }else if(request.status >= 400){
        fail.call(null, request, request.status)
      }
    }
  }
  request.send()
}
ajax('GET', '/xxx', {success(response){}, fail:(request, status)=>{}}) // 两种不同的函数缩写
```

### 2）Promise 写法

``` js
ajax = (method, url, options)=>{
  return new Promise((resolve, reject)=>{
    const request = new XMLHttpRequest()
    request.open(method, url)
    request.onreadystatechange = ()=>{
      if(request.readyState===4){
        if(request.status < 400){
          resolve.call(null, request.response)
        }else if(request.status >= 400){
          reject.call(null, request)
        }
      }
    }
    request.send()
  })
}

ajax('GET', '/xxx').then((response)=>{}, (request)=>{})
```

Promise 姿势在调用方面：

* `ajax()` 返回了一个含有`.then()`方法的对象
* `.then()`方法接收两个参数，第一个是成功回调函数，第二个是失败回调函数，也都分别只能传一个参数
* `return new Promise((resolve, reject)=>{})` 背下来

## ★小结

* `return new Promise((resolve, reject)=>{})`
* 任务成功则调用`resolve(result)`
* 任务失败则调用`reject(error)`
* resolve 和 reject 会再调用 success 和 fail 函数
* 使用`.then(success, fail)`传入成功和失败函数
* [Promise MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)
* Promise 高级用法以后说

目前我们封装的 ajax 还有一些缺点：

* POST 无法上传数据，上传的数据应作为 `request.send()` 的参数
* 不能设置请求头，`request.setRequestHeader(key, value)`

解决方法：

* 使用 `jQuery.ajax`
* 使用 axios（比 jQuery 更有逼格）
* 花时间将 ajax 写到完美（精进）

## ★jQuery.ajax

* 已经非常完美
  * 查看 [中文文档](https://www.jquery123.com/jQuery.ajax/) / [英文文档](https://api.jquery123.com/jQuery.ajax/)，搜索 ajax，找到 `jQuery.ajax`
  * 查看参数说明，阅读代码示例
  * 看到 jQuery 的封装，便会感叹自己的封装是辣鸡
* 封装优点
  * 支持更多形式的参数
  * 支持 Promise
  * 支持超多功能
* 无需掌握 jQuery.ajax
  * 现在的专业前端都用 axios
  * 写一篇博客罗列一下功能，然后就可以忘掉 jQuery 了

## ★Axios

* 目前最新的 AJAX 库
  * 抄袭了 jQuery 的封装思路
  * 查看 [文档](http://www.axios-js.com/zh-cn/docs/) 和方方的 [博客](https://juejin.im/post/5a9cddb46fb9a028bc2d3c2f)，以便快速了解 axios 的用法
  * 推荐也通过写博客的方式来学习库
* axios 高级用法
  * JSON 自动处理
    * axios 如果发现响应的 Content-Type 是 json
    * 就会自动调用 JSON.parse
    * 因此正确设置 Content-Type 是好习惯
  * [请求拦截器](https://github.com/axios/axios#interceptors)
    * 可以在所有请求里加些东西，比如加查询参数
  * 响应拦截器
    * 可以在所有响应里加些东西，甚至修改内容
  * 可生成不同实例（对象）
    * 不同实例可以设置不同配置，以用于复杂场景
  * Promise 不可以取消请求，axios 通过其他方式可以取消

## ★总结

- 异步是什么：不能直接拿到结果
- 异步为什么用回调（或轮询）：为了拿到不能直接拿到的结果
- 回调的三个问题：参数名、地狱、错误处理
- Promise 是什么：1976 年的一种设计模式
- Promise 怎么用：`return new Promise((resolve, reject)=>{})`
- Axios 怎么用：bootcdn 引用后发请求试试

初级学接口，中级学封装，高级造轮子。

Promise 是前端解决异步问题的统一解决方案，因此面试一定会问！

---

- 关于异步
  - 如果 JS 不能直接拿到一个函数的结果，可以先去执行其他代码，等到有结果了再去取结果，这就是异步
  - 异步的结果可以通过轮询获取，轮询就是定时去询问是否出结果
  - 异步的结果可以通过回调获取，一般结果会被作为回调的第一个参数
  - 异步的好处是可以把用来等待的时间拿去做别的事
- 关于回调
  - 满足某些条件的函数才能被称为回调，比如写一个函数 A，传给另一个函数 B 调用，那么函数 A 就是回调
  - 回调不一定用于异步任务，也可以用于同步任务
  - 有时也可以将回调传给一个对象，如 `request.onreadystatechange`，等待浏览器来调用
- 关于 Promise
  - Promise 不是前端发明的
  - Promise 是目前前端解决异步问题的统一方案
  - `window.Promise` 是一个用于构造 Promise 对象的全局函数
  - 使用 `return new Promise((resolve, reject)=>{})` 即可构造一个 Promise 对象
  - 构造出来的 Promise 对象含有一个`.then()` 函数属性
  - `resovle` 和 `reject` 可以改成任何其他名字而不影响使用，但一般都叫这个名字
  - 异步任务成功时调用 resolve，失败时调用 reject
  - resolve 和 reject 都只接收一个参数
  - resolve 和 reject 并不是`.then(success(){},fail(){})` 里边的 success 和 fail，resolve 会调用 success，reject 会调用 fail -> 类似传给`setTimeout`的回调函数！
- 关于 Axios
  - Axios 是一个专门用于操作 AJAX 的库
  - `axios.get('/xxx')`：返回一个 Promise 对象
  - `axios.get('/xxx').then(s,f)`：在请求成功时调用 `s`，失败时调用 `f`

## ★了解更多

➹：[【Node 系列】回调地狱和异步编程 - 飞鹰走码](https://lichangwei.github.io/2017/06/30/callback-hell-and-async-program/)

## ★总结

- 文章的作者也是看看芳芳的视频学习，我也是看看芳芳的视频学习，但他总结的比我要好，比较简练，说白了，就是你得知道自己要明白什么……

## ★Q&A

### 1）为什么不用原生 AJAX 发请求？

因为不同的浏览器平台对 AJAX 的实现并不相同，所以你用原生 AJAX 发送请求的话，就不得不考虑代码的兼容性问题，而写出这样的代码，显然并不容易啊！

而 jQuery 团队为我们解决了这个难题，我们只需要一行简单的代码，就可以实现 AJAX 功能。

➹：[jQuery AJAX 简介 - jQuery 基础教程 - 简单教程，简单编程](https://www.twle.cn/l/yufei/jquery/jquery-basic-ajax-intro.html)

➹：[【jQuery】(8)---jquery Ajax - 雨点的名字 - 博客园](https://www.cnblogs.com/qdhxhz/p/12337724.html)

➹：[「每日一题」AJAX 是什么？ - 知乎](https://zhuanlan.zhihu.com/p/22564745)
