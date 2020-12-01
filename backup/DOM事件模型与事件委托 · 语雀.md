> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.yuque.com](https://www.yuque.com/woozyzzz/ybz8i1/cwy46p)

DOM事件模型与事件委托
============

点击事件
====

```
<div id="grandpa">
  <div id="papa">
    <div id="son">
      click here
    </div>
  </div>
</div>
```

```
grandpa.addEventListener("click",()=>{
  console.log("grandpa")
})
papa.addEventListener("click",()=>{
  console.log("papa")
})
son.addEventListener("click",()=>{
  console.log("son")
})
```

*   提问1

*   点击文字内容，算点击爷爷、爸爸还是儿子？
*   答：都算

*   提问2

*   点击文字内容，先调用哪个函数？
*   答：都行
*   IE5 认为先调用儿子，网景认为先调用爷爷，闹到上了 W3C

*   2002年，W3C发布标准

*   文档名为 DOM Level 2 Events Specification
*   规定浏览器应该同时支持两种调用顺序
*   首先按照 爷爷=>爸爸=>儿子 顺序看有没有函数监听
*   然后按照 儿子=>爸爸=>爷爷 顺序看有没有函数监听
*   有函数监听就调用并提供事件信息，没有就跳过

*   术语

*   从外向内 找监听函数，叫 事件捕获
*   从内向外 找监听函数，叫 事件冒泡
*   由开发者自己选择把监听函数放在 捕获阶段 还是放在 冒泡阶段

addEventListener
================

*   事件绑定 API

*   IE5：`baba.attachEvent("onclick",fn)` 冒泡
*   网景：`baba.addEventListener("click",fn)` 捕获
*   W3C：`baba.addEventListener("click",fn,bool)` 

*   若 bool 为 falsy 或不传，则

*   让 fn 走冒泡，即当浏览器在冒泡阶段发现有对应监听函数，就调用 fn ，并提供事件信息。

*   若 bool 为 true，则

*   让 fn 走捕获，即当浏览器在捕获阶段发现有对应监听函数，就调用 fn ，并提供事件信息。

小结
==

*   捕获与冒泡

*   捕获时先调用爸爸的监听函数
*   冒泡时先调用儿子的监听函数

*   W3C事件模型

*   先捕获（爸爸=>儿子）再冒泡（儿子=>爸爸）
*   注意 e 对象被传给所有监听函数
*   事件结束后，e 对象就不存在了

target 与 currentTarget
======================

*   区别

*   e.target：用户在操作的元素
*   e.currentTarget：程序员监听的元素
*   this 是 e.currentTarget（不推荐）

*   举例

*   div > span{文字}，用户点击文字
*   e.target 就是 span
*   e.currentTarget 就是 div

特例
==

执行顺序
----

*   背景

*   只有一个 div 被监听（不考虑父子同时被监听）
*   fn 分别在捕获阶段和冒泡阶段监听 click 事件
*   用户点击的元素就是开发者监听的

*   代码

*   `div.addEventListener("click",f1)` 
*   `div.addEventListener("click",f2,true)` 
*   问：谁先执行？
*   答：谁先监听谁先执行

取消冒泡
----

*   捕获不可取消，但冒泡可以

*   使用 e.stopPropagation() 可以中止冒泡，浏览器不再向上执行监听函数
*   一般用于封装某些独立的组件

不可取消冒泡
------

*   有些事件不可以取消冒泡

*   MDN 搜 scroll event，看到 Bubbles 和 Cancelable
*   Bubbles 指该事件是否冒泡
*   Cancelable 指开发者是否可以取消冒泡
*   推荐看英文版，中文版内容不全

*   如何阻止滚动

*   scroll 事件不可取消冒泡
*   scroll 事件阻止默认动作也没用，因为先滚动才有滚动事件
*   要阻止滚动，可以阻止 wheel 和 touchstart 的默认动作
*   CSS 修改滚动条宽度 `::webkit-scrollbar{width:0 !important}` 
*   CSS 隐藏滚动条 `overflow:hidden;` 也可以，但仍然可以通过JS修改 scrollTop

自定义事件
=====

*   浏览器自带事件

*   一共100多种，[MDN](https://developer.mozilla.org/zh-CN/docs/Web/Events)

*   提问

*   在自带时间外，自定义事件
*   使用 [CustomEvent](https://developer.mozilla.org/zh-CN/docs/Web/API/CustomEvent) 构造函数创建一个自定义事件

```
<div id=div1>
  <button id=button1>
    点击触发 haha
  </button>
</div>
```

```
button1.addEventListener('click', () => {
  const event = new CustomEvent("haha", {
    "detail": {
      name: 'woozyzzz',
      age: 18
    },
    bubbles:true
  })
  button1.dispatchEvent(event)
})

div1.addEventListener('haha', (e) => {
  console.log('haha')
  console.log(e)
})
```

事件委托
====

*   场景一

*   需求：监听100个按钮的点击事件
*   办法：监听这100个按钮的祖先，在冒泡时判断 target 是否为其中的按钮

```
div1.addEventListener("click",(e)={
  if(e.target.tagName.toLowerCase==="button"){
    console.log(e.target.textContent)
  }
})
```

*   场景二

*   需求：监听目前不存在的事件
*   办法：监听祖先，等事件发生时看 target 是不是想要监听的目标元素

*   优点

*   节省监听代码（内存）
*   可以监听动态元素

封装事件委托
------

*   要求

*   写出一个函数 `on("click", "#testDiv", "li", fn)` 
*   当用户点击 #testDiv 里的 li 元素时，调用 fn 函数
*   要求用事件委托

*   答案1

*   监听祖先元素
*   判断 target 是否匹配 "li"

*   使用 [Element.matches(selectorString)](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/matches)
*   判断 Element 元素是否匹配 selectorString 选择器字符串

*   面试可以但实际有bug

*   答案2

*   递归判断 target/target的祖先

```
on("click", "#div1", "button", ()=>{
  console.log("button 被点击了")
})

function on(eventType, element, selector, fn){
  if(!(element instanceof Element)){
    element = document.querySelector(element)
  }
  element.addEventListener(eventType, e=>{
    let el = e.target
    while(!el.matches(selector)){
      if(element===el){
        el = null
        break
      }
      el = el.parentNode
    }
    el && fn.call(el, e, el)
  })
  return element
}
```

*   整合进 jQuery

*   `$("#xxx").on("click","li",fn)` 

JS支持也不支持事件
==========

*   DOM 事件不属于 JS 的功能

*   DOM 事件与 JS 都是浏览器提供的功能
*   它们之间的关系是平行的
*   JS 只是调用了 DOM 提供的 addEventListener API

*   面试可能会问：手写一个事件系统

*   使用队列

* * *

面试题
===

*   简述 DOM 事件模型或 DOM事件机制（描述捕获与冒泡）

*   浏览器中的 DOM 事件模型包括事件捕获和事件冒泡两个阶段，在浏览器对事件监听过程需要先后遍历两边 DOM 树，先从外层元素到内层元素捕获，再从内层元素到外层元素冒泡。使用 Element.addEventListener(event，fn, bool) 监听事件，其中 bool 默认为 false，表示监听冒泡阶段，true 表示监听捕获阶段。

*   简述事件委托

*   在监听多个元素或者监听尚未动态创建的元素时可以使用事件绑定，即监听目标元素的外层元素并判断用户操作的元素 target ，达到节省内存的目的。

若有收获，就点个赞吧