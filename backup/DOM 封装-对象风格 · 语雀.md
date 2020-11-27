> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.yuque.com](https://www.yuque.com/woozyzzz/ybz8i1/yo9rg6)

DOM 封装-对象风格
===========

经过了对 DOM 操作的学习，我发现原生 DOM API 不大方便使用，因此我尝试采用对象风格封装一个 DOM 库。本文记录了库中代码封装的主要实现思想，同时可以作为参考文档供使用者查阅。

[GitHub 源码地址](https://github.com/Woozyzzz/dom-library-object)    [Gitee 源码地址](https://gitee.com/Woozyzzz/dom-library-object)    [Gitee 预览地址](http://woozyzzz.gitee.io/dom-library-object/dist/)

  

对象风格也称作命名空间风格，我在项目创建 dom.js 文件，在该文件中创建了全局对象  `window.dom = {}` ，将对元素增删改查的功能作为其属性。

查
=

获取节点：dom.find
-------------

返回与指定容器中的选择器组匹配的文档中的节点列表，返回的对象是 NodeList。

*   语法

```
const elementList = dom.find(selectors, container);
```

*   参数

*   `selectors` String，一个包含单个或多个匹配的选择器的字符串
*   `container` Element，一个表示节点获取范围的元素对象

*   实现

*   基于 `querySelectorAll` 实现，将 `document` 作为不指定容器 `container` 时的保底值

```
dom.find = function (selectors, container) {
  return (container || document).querySelectorAll(selectors)
}
```

获取父节点：dom.parent
----------------

返回指定节点的父节点。

*   语法

```
const parentNode = dom.parent(node)
```

*   实现

```
dom.parent = function (node) {
  return node.parentNode
}
```

获取子节点：dom.children
------------------

返回指定节点的所有子节点列表，返回的对象是 HTMLCollection。

*   语法

```
const children = dom.children(node)
```

*   实现

```
dom.children = function (node) {
  return node.children
}
```

获取同级节点：dom.siblings
-------------------

返回指定节点的所有同级节点列表，返回的对象是 Array。

*   语法

```
const siblings = dom.siblings(node)
```

*   实现

*   先获取指定节点的父节点，再获取其所有的子节点并从中过滤掉指定节点

```
dom.siblings = function (node) {
  return Array.from(dom.children(dom.parent(node))).filter(item => item !== node)
}
```

获取前一个同级节点：dom.previous
----------------------

返回指定节点的前一个同级非文本节点。

*   语法

```
const previousNode = dom.previous(node)
```

*   实现

*   若指定节点的前一个同级节点为文本节点，则继续查找

```
dom.previous = function (node) {
  let p = node.previousSilbing
  while (p && p.nodeType === 3) {
    p = p.previousSibling
  }
  return p
}
```

获取后一个同级节点：dom.next
------------------

返回指定节点的后一个同级非文本节点。

*   语法

```
const nextNode = dom.next(node)
```

*   实现

*   若指定节点的后一个同级节点为文本节点，则继续查找

```
dom.next = function (node) {
  let n = node.nextSilbing
  while (n && n.nodeType === 3) {
    n = n.nextSibling
  }
  return n
}
```

遍历节点：dom.travel
---------------

对节点列表中的每个节点执行一次给定的函数。

*   语法

```
dom.travel(nodeList, callback(currentNode))
```

*   参数

*   `nodeList` 需要遍历的节点列表
*   `callback` 节点列表中每个节点执行的函数，该函数接收一个参数

*   `currentNode` 节点列表中正在处理的当前节点

*   实现

```
dom.each = function (nodeList, callback) {
  for (let i = 0; i < nodeList.length; i++) {
    callback.call(null, nodeList[i])
  }
}
```

获取节点索引：dom.index
----------------

返回指定节点在其父节点中的索引值。

*   语法

```
const index = dom.index(node)
```

*   实现

*   获取并遍历指定节点的父节点的所有子节点列表

```
dom.index = function (node) {
  const nodeList = dom.children(dom.parent(node))
  for (let i = 0; i < nodeList.length; i++) {
    if (nodeList[i] === node) {
      return i
    }
  }
}
```

增
=

创建节点：dom.create
---------------

创建并返回一个指定 HTML 内容的节点。

*   语法

```
const element = dom.create(string)
```

*   实现

*   基于 `createElement` 与 `innerHTML` 实现，将接收的 HTML 内容作为 `template` 元素的后代并返回 `template` 元素的首个子节点

```
dom.create = function (string) {
  const template = document.createElement("template")
  template.innerHTML = string.trim()
  return template.content.firstChild
}
```

向前插入节点：dom.before
-----------------

将指定节点作为参考节点的前一个节点。

*   语法

```
dom.before(node, referenceNode)
```

*   实现

*   基于 `insertBefore` 实现，获取参考节点的父节点后，将指定节点插入到参考节点之前

```
dom.before = function (node, referenceNode) {
  dom.parent(referenceNode).insertBefore(node, referenceNode)
}
```

向后插入节点：dom.after
----------------

将指定节点作为参考节点的后一个节点。

*   语法

```
dom.after(node, referenceNode)
```

*   实现

*   获取参考节点的父节点后，将指定节点插入到参考节点的后一个同级节点之前

```
dom.after = function (node, referenceNode) {
  dom.parent(referenceNode).insertBefore(node, dom.next(referenceNode))
}
```

插入子节点：dom.append
----------------

将指定节点作为另一个节点的末位子节点。

*   语法

```
dom.append(node, parentNode)
```

*   实现

```
dom.append = function (node, parentNode) {
  parentNode.appendChild(node)
}
```

插入父节点：dom.wrap
--------------

将指定节点作为另一个节点的父节点。

*   语法

```
dom.wrap(node, childNode)
```

*   实现

*   先将指定节点作为另一个节点的前一个同级节点，再将另一个节点作为指定节点的末位子节点

```
dom.wrap = function (node, childNode) {
  dom.before(node, childNode)
  dom.append(childNode, node)
}
```

删
=

删除节点：dom.remove
---------------

删除并返回指定节点。

*   语法

```
const element = dom.remove(node)
```

*   实现

```
dom.remove = function (node) {
  node.remove()
  return node
}
```

清空子节点：dom.empty
---------------

删除并返回指定节点的所有子节点。

*   语法

```
const children = dom.empty(node)
```

*   实现

```
dom.empty = function (node) {
  const children = dom.children(node)
  let array = []
  for (let i = 0; i < children.length; i++) {
    array.push(dom.remove(children[i]))
  }
  return array
}
```

改
=

读写特性：dom.attribute
------------------

返回或修改指定节点的某项特性值。

*   语法

```
let attribute = dom.attribute(node, name)
dom.attribute(node, name, value)
```

*   参数

*   `node` Element，需要操作的节点
*   `name` String，需要获取或修改的特性名称的字符串
*   `value` String，需要修改的特性新值的字符串

*   实现

*   重载：当传入两个参数时，读取指定节点的指定特性值；当传入三个参数时，修改指定节点的指定特性值

```
dom.attribute = function (node, name, value) {
  if (arguments.length === 2) {
    return node.getAttribute(name)
  } else if (arguments.length === 3) {
    node.setAttribute(name, value)
  }
}
```

读写 style 属性：dom.style
---------------------

返回或修改指定节点的某项样式属性值。

*   语法

```
let value = dom.style(node, name)
dom.style(node, name, value)
```

*   参数

*   `node` Element，需要操作的节点
*   `name` String | Object，需要获取或修改的样式属性名称的字符串或者需要修改的样式属性的对象
*   `value` String，需要修改的样式属性新值的字符串

*   实现

*   重载：当传入两个参数时，读取指定节点的指定特性值；当传入三个参数时，修改指定节点的指定特性值

```
dom.style = function (node, name, value) {
  if (arguments.length === 2) {
    if (typeof name === "string") {
      return node.style[name]
    } else if (name instanceof Object) {
      for (let key in name) {
        node.style[key] = name[key]
      }
    }
  } else if (arguments.length === 3) {
    node.style[name] = value
  }
}
```

修改 class 属性：dom.class
---------------------

*   方法

*   `dom.class.has` 判断指定节点的 class 属性是否包含相应的字符串，返回值为布尔值
*   `dom.class.add` 为指定节点添加 class 属性
*   `dom.class.remove` 为指定节点删除 class 属性

*   语法

```
let bool = dom.class.has(node, className)
dom.class.add(node, className)
dom.class.remove(node, className)
```

*   实现

```
dom.class = {
  has(node, className) {
    return node.classList.contains(className)
  },
  add(node, className) {
    node.classList.add(className)
  },
  remove(node, className) {
    node.classList.remove(className)
  },
}
```

读写文本内容：dom.text
---------------

返回或修改指定节点的文本内容。

*   语法

```
let text = dom.text(node)
dom.text(node, string)
```

*   实现

*   重载：当传入一个参数时，读取指定节点的文本内容；当传入两个参数时，修改指定节点的文本内容
*   适配：基于 `document.all` 实现，兼容 IE 浏览器

```
dom.text(node, string) {
  if (arguments.length === 1) {
    return document.all ? node.innerText : node.textContent
  } else if (arguments.length === 2) {
    document.all ? (node.innerText = string) : (node.textContent = string)
  }
}
```

读写 HTML 内容：dom.html
-------------------

返回或修改指定节点的 HTML 内容。

*   语法

```
let content = dom.html(node)
dom.html(node, string)
```

*   实现

*   重载：当传入一个参数时，读取指定节点的 HTML 内容；当传入两个参数时，修改指定节点的 HTML 内容

```
dom.html(node, string) {
  if (arguments.length === 1) {
    return node.innerHTML
  } else if (arguments.length === 2) {
    node.innerHTML = string
  }
}
```

事件监听：dom.on / dom.off
---------------------

设置/删除指定节点的监听事件。

*   语法

```
dom.on(node, eventType, listener, useCapture)
dom.off(node, eventType, listener, useCapture)
```

*   实现

```
dom.on = function (node, eventType, listener, useCapture = false) {
  node.addEventListener(eventType, listener, useCapture)
}
dom.off = function (node, eventType, listener, useCapture = false) {
  node.removeEventListener(eventType, listener, useCapture)
}
```

参考文章
====

1.  [MDN DOM概述](https://developer.mozilla.org/zh-CN/docs/Web/API/Document_Object_Model/Introduction#Core_Interfaces_in_the_DOM)
2.  [MDN Element](https://developer.mozilla.org/zh-CN/docs/Web/API/Element)
3.  [MDN <template>：内容模板元素](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/template)

若有收获，就点个赞吧