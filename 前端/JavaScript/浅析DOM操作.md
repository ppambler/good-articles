# 浅析 DOM 操作

> 原文：[浅析 DOM 操作 · 语雀](https://www.yuque.com/woozyzzz/ybz8i1/gtaozk)

<!-- TOC -->

- [浅析 DOM 操作](#浅析-dom-操作)
  - [网页是一棵树](#网页是一棵树)
  - [获取 DOM 元素](#获取-dom-元素)
    - [常规元素](#常规元素)
    - [特定元素](#特定元素)
  - [节点的增删改查](#节点的增删改查)
    - [增](#增)
    - [删](#删)
    - [改](#改)
      - [改属性](#改属性)
      - [改内容](#改内容)
      - [改事件处理函数](#改事件处理函数)
    - [查](#查)
  - [关于属性](#关于属性)
    - [Attribute 与 Property 的区别](#attribute-与-property-的区别)
    - [属性同步](#属性同步)
  - [参考文章](#参考文章)

<!-- /TOC -->

## 网页是一棵树

> 文档对象模型 (DOM) 将 web 页面与到脚本或编程语言连接起来。DOM 模型用一个逻辑树来表示一个文档，树的每个分支的终点都是一个节点 (node)，每个节点都包含着对象 (objects)。DOM 的方法 (methods) 让你可以用特定方式操作这个树，用这些方法你可以改变文档的结构、样式或者内容。

![image-20201126171232389](https://gitee.com/ppambler/blog-images/raw/master/images/image-20201126171232389.png)

HTML 文档会被浏览器解析为一棵 DOM 树，浏览器为开发者提供 `window.document` API 使其能够使用 JS 改变 DOM 树的结构。可是实际上，DOM 操作的设计不够方便，导致前端使用 jQuery 操作 DOM，之后又使用 Vue 和 React。

- 线程各司其职
  - JS 引擎只能操作 JS，不能操作页面
  - 渲染引擎只能操作页面，不能操作 JS
- 跨线程通信
  - 当浏览器发现 JS 在 body 中添加了一个 div 对象时
  - 浏览器就会通知渲染引擎在页面中也添加一个 div
  - 对同一个对象的多次操作可能会不断重新渲染或者合并为一次操作渲染

## 获取 DOM 元素

在使用 JS 操作 DOM 树中的元素（节点）之前，需要先获取到这个元素。而获取到的 DOM 元素是一个对象，它的原型链超级长，以 div 元素为例：

1. `HTMLDivElement.prototype` 中是 div 的共有属性
2. `HTMLElement.prototype` 中是所有 HTML 元素的共有属性
3. `Element.prototype` 中是所有 XML、HTML 元素的共有属性
4. `Node.prototype` 中是所有节点的共有属性，节点包括 XML 与 HTML 元素、文本、注释等等
5. `EventTarget.prototype` 中的 addEventListener 事件监听属性最为重要
6. `Object.prototype`

节点 Node 包括很多种，通过 `xxx.nodeType` 可以得到一个对应不同种类节点的数字，其中 1 表示元素、3 表示文本、8 表示注释，详情可见 [ MDN 完整描述](https://developer.mozilla.org/zh-CN/docs/Web/API/Node/nodeType)

### 常规元素

- 通常使用选择器语法与 querySelector 或 querySelectorAll 
  - `document.querySelector("选择器")`
  - `document.querySelectorAll("选择器")[0]`
  - `window.id 名称` 或 直接 `id 名称`
  - 需要兼容 IE 时再使用 `document.getElementByxxx`

### 特定元素

- `document.documentElement` 获取 html 元素
- `document.head` 获取 head 元素
- `document.body` 获取 body 元素
- `window` 获取包含 DOM 文档的窗口（不是元素）
- `document.all` 获取所有元素
  - IE 发明的 API，其它浏览器都不支持
  - 早期被当作表达式用于检测浏览器是否为 IE（第六个 falsy）
  - `if(document.all)console.log("IE");`

## 节点的增删改查

### 增

创建节点

```js
// 创建一个元素节点
const div1 = document.createElement("div")
// 创建一个文本节点
const text1 = document.createTextNode("您好")
```

插入节点

```js
// 为元素节点添加子节点
div1.appendChild(text1)
// 元素节点中插入文本节点
div1.innerText = "您好" // 性能差，但兼容 IE，不推荐
div1.textContent = "您好"
```

将节点插入页面中

- 新创建的节点默认在 JS 线程中
- 必须将其插入到 head 或 body 中才能生效
- `已在页面中的元素.appendChild(节点)`

```js
const test1 = document.querySelector("#test1")
const test2 = document.querySelectorAll("#test2")
// 新创建的节点以页面中元素的最后子元素出现
document.body.appendChild(test1)
test1.appendChild(test2)
```

### 删

- 旧方法：`div.parentNode.removeChild(div)`
- 新方法：`div.remove()` 不支持 IE
- 节点被删除后，还可以使用 appendChild 添加回来

### 改

#### 改属性

- 读取标准属性
  - class 属性
    - `div.className`
    - `div.classList.value`
    - `div.getAttribute("class")`
  - a 标签属性
     - `a.href`（浏览器可能会自动补全网址）
     - `a.getAttribute("href")`

- 修改标准属性
  - class 属性
    - `div.className = "red round"` 全覆盖
    - `div.classList.add("blue")` 添加
    - `div.classList.remove("red")` 删除
  - style 属性
    - `div.style = "width: 100px;color: blue;"` 注意分号
    - `div.style.width = "100px"` 部分修改
    - `div.style.backgroundColor = "red"` 注意大小写
  - 自定义数据属性（`data-*`）
    - `div.dataset.x = "frank"`

#### 改内容

- 文本内容
  - `div.innerText = "xxx"`
  - `div.textContent = "xxx"` 推荐
- HTML 内容
  - `div.innerHTML = "<strong>重要内容</strong>"`
  - 添加内容过多，会导致浏览器卡死
- 元素内容（先清空再写内容）
  - `div.innerHTML = ""`
  - `div.appendChild(div2)`
- 更换父节点
  - `divParent.appendChild(div1)`

#### 改事件处理函数

- `div.addEventListener`
- `div.onclick` 默认值为 `null`

### 查

- 查找父节点
  - `node.parentElement`
  - `node.parentNode` 推荐
- 查找父节点的父节点
  - `node.parentNode.parentNode`
- 查找子节点
  - `node.childNodes`（子节点们）
  - `node.children` （子元素们）推荐
  - 查老大：`node.firstChild`
  - 查老幺：`node.lastChild`
- 查找同级节点
  - 需要遍历排除自身
    - `node.parentNode.childNodes`
    - `node.parentNode.children` 推荐
  - 查找上一个同级节点
    - `node.previousSibling`
  - 查找下一个同级节点
    - `node.nextSibling`
- 遍历一个 `div` 中的所有节点

```js
const div = document.querySelector(".red")
const travel = (node, fn) => {
  fn(node)
  if (node.children) {
    for (let i = 0; i < node.children.length; i++) {
      travel(node.children[i], fn)
    }
  }
}
travel(div, node => console.log(node))
```

## 关于属性

### Attribute 与 Property 的区别

在 HTML 中元素的属性分为：**内容属性** 和 **IDL**（接口描述语言）属性。

内容属性通常称为特性（attribute），是渲染引擎中元素对应标签的属性，需要在内容（HTML 代码）中设置，在 JS 中可以通过 `element.getAttribute("特性名")` 获取特性或者 `element.setAttribute("特性名", "特性值")` 设置特性，`attribute` 的值只支持字符串。

IDL 属性通常称为属性（property），是 JS 引擎中元素的所有属性，同名属性初始值与 attribute 相等，需要在 JS 中设置，可以通过类似 JS 对象属性的方法 `element. 属性名` 获取或设置，`property` 的值支持字符串、布尔等类型。

### 属性同步

- 标准属性
  - 如 `id`、`className`、`title` 等，详情可见 [ MDN 属性列表](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Attributes#属性列表)
  - 对标准属性的修改，会被浏览器同步到页面中
- 非标准属性
  - 对非标准属性的修改，只会停留在 JS 线程中，不会同步到页面中
- 自定义数据属性（`data-*`）
  - 自定义数据属性的修改，会被浏览器同步到页面中
  - 可以将 `data-` 作为希望同步到页面的自定义数据的前缀

## 参考文章

1. [MDN 文档对象模型（DOM）](https://developer.mozilla.org/zh-CN/docs/Web/API/Document_Object_Model)
2. [张鑫旭 小 tips: JS DOM innerText 和 textContent 的区别](https://www.zhangxinxu.com/wordpress/2019/09/js-dom-innertext-textcontent/)
3. [MDN HTML 属性参考](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Attributes#%E5%86%85%E5%AE%B9%E5%B1%9E%E6%80%A7%E5%92%8C_IDL%E5%B1%9E%E6%80%A7)
4. [MDN 全局属性](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Global_attributes/data-*)

