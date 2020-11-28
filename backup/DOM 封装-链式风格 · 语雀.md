> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.yuque.com](https://www.yuque.com/woozyzzz/ybz8i1/dh10q6)

DOM 封装-链式风格
===========

为了进一步理解 jQuery 的设计思想，我尝试采用链式风格封装一个 DOM 库。本文记录了库中代码封装的主要实现思想，同时可以作为参考文档供使用者查阅。

[GitHub 源码地址](https://github.com/Woozyzzz/dom-library-jquery)    [Gitee 源码地址](https://gitee.com/Woozyzzz/dom-library-jquery)    [Gitee 预览地址](http://woozyzzz.gitee.io/dom-library-jquery/dist/)

  

链式风格也称作 jQuery 风格，与对象风格不同的是，我在项目中创建 jquery.js 文件，在文件中创建了 `window.jQuery = function(){}` 作为全局函数而非全局对象，并在 jQuery 函数中构建对元素增删改查的功能。

链式思想
====

命名风格
----

由于全局函数 `window.jQuery` 的拼写不够方便，因此将 `window.$` 作为其别名。

```
window.$ = window.jQuery = function () {}
```

特殊的函数
-----

为了实现链式操作，在使用 `jQuery("选择器")` 或 `$("选择器")` 选择页面元素时，应当返回一个可操作对应元素的对象，而不是元素本身。我们称这个对象为由 jQuery 函数构造的对象，简称为 **jQuery 对象**。

在 jQuery 对象中添加 `print` 属性，属性值为一个用于打印出选择元素的函数，函数的返回值为 this，指向调用它的对象，即作为 jQuery 函数返回值的 jQuery 对象。函数与选择的元素构成闭包结构，这样我们就可以继续对元素进行操作，从而形成链式风格。

```
window.$ = window.jQuery = function (selectors) {
  let elements = document.querySelectorAll(selectors)
  return {
    print() {
      console.log(elements)
      return this
    }
  }
}

// 选择 class 属性为 test 的所有元素并将它们打印出来
$(".test").print()
```

后续：内存优化
-------

由于每次调用 jQuery 对象中的函数都会新占用一块地址，从而导致内存浪费，因此可以把 jQuery 对象的共有属性（函数）全部放在 `jQuery.prototype` 上，并对其使用别名 `jQuery.fn = jQuery.prototype` ，再将 jQuery 对象的原型指向 `jQuery.fn` 。可以使用 `Object.assign` 为 jQuery 对象添加多个属性，使用 `constructor` 设置 jQuery 对象的构造函数 `jQuery` 。

查
=

选择元素：$(selectors)
-----------------

根据指定的 CSS 选择器选择所有元素，返回 jQuery 对象。

*   语法

```
const $api = $(selectors)
```

*   参数

*   `selectors` String，一个包含单个或多个匹配的选择器的字符串

*   实现

```
window.$ = window.jQuery = function (selectorsOrArray) {
  let elements = document.querySelectorAll(selectorsOrArray);
  return {}
}
```

遍历元素：.each(fn)
--------------

遍历元素列表，对每个元素执行一次指定的函数，返回 jQuery 对象。

*   语法

```
const $api = $(selectors).each(fn(element, index))
```

*   实现

```
window.$ = window.jQuery = function (selectors) {
  let elements = document.querySelectorAll(selectorsOrArray)
  return {
    each(fn) {
      for (let index = 0; index < elements.length; index++) {
        const element = elements[index]
        fn.call(this, element, index)
      }
      return this
    }
  }
}
```

查找元素：.find(selectors)
---------------------

遍历元素列表，根据指定的 CSS 选择器选择元素，返回 jQuery 对象。

*   语法

```
const $api = $(selectorsOrArray).find(selectors)
```

*   参数

*   `selectorsOrArray` String | Array，一个包含单个或多个匹配的选择器的字符串或者 `.find()` 方法中处理的数组
*   `selectors` String，一个包含单个或多个匹配的选择器的字符串

*   实现

*   重载：当传入参数为 String 类型时，元素列表为 CSS 选择器指定元素；当传入参数为 Array 类型时，元素列表为 `.find()` 方法查找的指定元素

```
window.$ = window.jQuery = function (selectorsOrArray) {
  let elements
  if (typeof selectorsOrArray === "string") {
    elements = document.querySelectorAll(selectorsOrArray)
  } else if (selectorsOrArray instanceof Array) {
    elements = selectorsOrArray
  }
  return { 
    find(selectors) {
      const array = []
      this.each((element) => {
        array.push(...element.querySelectorAll(selectors))
      })
      return jQuery(array)
    }
  }
}

```

元素回退：.end()
-----------

终止当前链的操作，返回上一层的 jQuery 对象。

*   语法

```
const $api = $(selectorsOrArray).end()
const $oldApi = $(selectorsOrArray).find(selectors).end()
```

*   实现

*   在 `.find()` 方法中将当前 jQuery 对象作为数组 `array` 的 `oldApi` 属性值，使得新的 jQuery 对象可以通过 `oldApi` 得到上一层的 jQuery 对象
*   若没有上层 jQuery 对象， `.end()` 方法返回当前 jQuery 对象作为保底值

```
window.$ = window.jQuery = function (selectorsOrArray) {
  let elements
  if (typeof selectorsOrArray === "string") {
    elements = document.querySelectorAll(selectorsOrArray)
  } else if (selectorsOrArray instanceof Array) {
    elements = selectorsOrArray
  }
  return {
    oldApi: selectorsOrArray.oldApi,
    find(selectors) {
      const array = []
      this.each((element) => {
        array.push(...element.querySelectorAll(selectors))
      })
      array.oldApi = this
      return jQuery(array)
    },
    end() {
      return this.oldApi || this
    }
  }
}
```

获取节点：.get(index)
----------------

通过 jQuery 对象获取指定索引的 DOM 节点。

*   语法

```
const elNode = $(selectorsOrArray).get(index)
```

*   参数

*   `index` Number，指定的索引值，若值为负数，则从元素列表中的最后一个元素开始倒数

*   实现

*   若 `index` 值为负数，则倒数元素列表，若值过小，则返回第一个元素，若值过大，则返回最后一个元素

```
window.$ = window.jQuery = function (selectorsOrArray) {
  let elements
  if (typeof selectorsOrArray === "string") {
    elements = document.querySelectorAll(selectorsOrArray)
  } else if (selectorsOrArray instanceof Array) {
    elements = selectorsOrArray
  }
  return {
    oldApi: selectorsOrArray.oldApi,
    get(index) {
      const length = elements.length
      if (index < 0) {
        index = index + length < 0 ? 0 : index + length
      } else if (index >= length) {
        index = length - 1
      }
      return elements[index]
    }
  }
}
```

获取单个元素：.eq(index)
-----------------

通过 jQuery 对象获取指定索引的元素，返回 jQuery 对象。

*   语法

```
const $api = $(selectorsOrArray).eq(index)
```

*   参数

*   `index` Number，指定的索引值，若值为负数，则从元素列表中的最后一个元素开始倒数

*   实现

*   若 `index` 值为负数，则倒数元素列表，若值过小，则返回第一个元素，若值过大，则返回最后一个元素

```
window.$ = window.jQuery = function (selectorsOrArray) {
  let elements
  if (typeof selectorsOrArray === "string") {
    elements = document.querySelectorAll(selectorsOrArray)
  } else if (selectorsOrArray instanceof Array) {
    elements = selectorsOrArray
  }
  return {
    oldApi: selectorsOrArray.oldApi,
    eq(index) {
      const length = elements.length
      if (index < 0) {
        index = index + length < 0 ? 0 : index + length
      } else if (index >= length) {
        index = length - 1
      }
      return jQuery([elements[index]])
    }
  }
}
```

选择父元素：.parent()
---------------

遍历元素列表，选择每一个元素的父元素，返回 jQuery 对象。

*   语法

```
const $api = $(selectorsOrArray).parent()
```

*   实现

*   遍历时判断当前元素的父节点是否已存在于数组 `array` 中，若存在，则跳过

```
window.$ = window.jQuery = function (selectorsOrArray) {
  let elements
  if (typeof selectorsOrArray === "string") {
    elements = document.querySelectorAll(selectorsOrArray)
  } else if (selectorsOrArray instanceof Array) {
    elements = selectorsOrArray
  }
  return {
    oldApi: selectorsOrArray.oldApi,
    parent() {
      const array = []
      this.each((element) => {
        const parentElement = element.parentNode
        if (array.indexOf(parentElement) === -1) {
          array.push(parentElement)
        }
      });
      array.oldApi = this
      return jQuery(array)
    }
  }
}
```

选择子元素：.children()
-----------------

遍历元素列表，选择每一个元素的子元素，返回 jQuery 对象。

*   语法

```
const $api = $(selectorsOrArray).children()
```

*   实现

```
window.$ = window.jQuery = function (selectorsOrArray) {
  let elements
  if (typeof selectorsOrArray === "string") {
    elements = document.querySelectorAll(selectorsOrArray)
  } else if (selectorsOrArray instanceof Array) {
    elements = selectorsOrArray
  }
  return {
    oldApi: selectorsOrArray.oldApi,
    children() {
      const array = []
      this.each((element) => {
        array.push(...element.children)
      })
      array.oldApi = this
      return jQuery(array)
    }
  }
}
```

选择同级元素：.siblings()
------------------

遍历元素列表，选择每一个元素的同级元素，返回 jQuery 对象。

*   语法

```
const $api = $(selectorsOrArray).siblings()
```

*   实现

*   先选择当前元素的父元素，再获取其所有的子元素并从中过滤掉当前元素

```
window.$ = window.jQuery = function (selectorsOrArray) {
  let elements
  if (typeof selectorsOrArray === "string") {
    elements = document.querySelectorAll(selectorsOrArray)
  } else if (selectorsOrArray instanceof Array) {
    elements = selectorsOrArray
  }
  return {
    oldApi: selectorsOrArray.oldApi,
    siblings() {
      const array = []
      this.each((element) => {
        const siblingsElements = Array.from(element.parentNode.children).filter(
          (item) => item !== element
        )
        array.push(...siblingsElements)
      })
      array.oldApi = this
      return jQuery(array)
    }
  }
}
```

选择前一个同级元素：.prev()
-----------------

遍历元素列表，选择每一个元素的前一个同级非文本元素，返回 jQuery 对象。

*   语法

```
const $api = $(selectorsOrArray).prev()
```

*   实现

```
window.$ = window.jQuery = function (selectorsOrArray) {
  let elements
  if (typeof selectorsOrArray === "string") {
    elements = document.querySelectorAll(selectorsOrArray)
  } else if (selectorsOrArray instanceof Array) {
    elements = selectorsOrArray
  }
  return {
    oldApi: selectorsOrArray.oldApi,
    prev() {
      const array = []
      this.each((element) => {
        let p = element.previousSibling
        while (p && p.nodeType === 3) {
          p = p.previousSibling
        }
        array.push(p)
      })
      array.oldApi = this
      return jQuery(array)
    }
  }
}
```

选择后一个同级元素：.next()
-----------------

遍历元素列表，选择每一个元素的后一个同级非文本元素，返回 jQuery 对象。

*   语法

```
const $api = $(selectorsOrArray).next()
```

*   实现

```
window.$ = window.jQuery = function (selectorsOrArray) {
  let elements
  if (typeof selectorsOrArray === "string") {
    elements = document.querySelectorAll(selectorsOrArray)
  } else if (selectorsOrArray instanceof Array) {
    elements = selectorsOrArray
  }
  return {
    oldApi: selectorsOrArray.oldApi,
    next() {
      const array = []
      this.each((element) => {
        let n = element.nextSibling
        while (n && n.nodeType === 3) {
          n = n.nextSibling
        }
        array.push(n)
      })
      array.oldApi = this
      return jQuery(array)
    }
  }
}
```

增
=

创建元素：$(template)
----------------

根据指定的模板创建并选择元素，返回 jQuery 对象。

*   语法

```
const $api = $(selectorsOrArrayOrTemplate)
```

*   参数

*   `selectorsOrArrayOrTemplate` String，用于创建元素的模板字符串

*   实现

*   重载：当传入参数为 String 类型时，若参数为模板字符串，则元素列表为 `create` 函数创建的元素，若参数不为模板字符串，则元素列表为 CSS 选择器指定元素；当传入参数为 Array 类型时，元素列表为 `.find()` 方法查找的指定元素
*   基于 `createElement` 与 `innerHTML` 实现，将接收的 HTML 内容作为 `template` 元素的后代并返回 `template` 元素的首个子节点

```
window.$ = window.jQuery = function (selectorsOrArrayOrTemplate) {
  let elements
  if (typeof selectorsOrArrayOrTemplate === "string") {
    if (selectorsOrArrayOrTemplate[0] === "<") {
      elements = [create(selectorsOrArrayOrTemplate)]
    } else {
      elements = document.querySelectorAll(selectorsOrArrayOrTemplate)
    }
  } else if (selectorsOrArrayOrTemplate instanceof Array) {
    elements = selectorsOrArrayOrTemplate
  }
  function create(string) {
    const template = document.createElement("template")
    template.innerHTML = string.trim()
    return template.content.firstChild
  }
  return {
    oldApi: selectorsOrArrayOrTemplate.oldApi,
    elements: elements
  }
}
```

向前插入元素：.before(jQueryOrNode)
----------------------------

将指定元素作为所选择元素的前一个节点。

*   语法

```
const $api = $(selectorsOrArrayOrTemplate).before(jQueryOrNode)
```

*   参数

*   `jQueryOrArray` jQuery 对象 | Node，需要插入的元素或者 DOM 节点

*   实现

*   将 `jquery` 属性添加至 jQuery 对象中，用于区分 jQuery 对象与 DOM 节点
*   基于 `insertBefore` 实现，获取元素列表中当前节点的父节点后，将指定节点插入到当前节点之前

```
window.$ = window.jQuery = function (selectorsOrArrayOrTemplate) {
  let elements
  if (typeof selectorsOrArrayOrTemplate === "string") {
    if (selectorsOrArrayOrTemplate[0] === "<") {
      elements = [create(selectorsOrArrayOrTemplate)]
    } else {
      elements = document.querySelectorAll(selectorsOrArrayOrTemplate)
    }
  } else if (selectorsOrArrayOrTemplate instanceof Array) {
    elements = selectorsOrArrayOrTemplate
  }
  function create(string) {
    const template = document.createElement("template")
    template.innerHTML = string.trim()
    return template.content.firstChild
  }
  return {
    oldApi: selectorsOrArray.oldApi,
    elements: elements,
    jquery: true,
    before(jQueryOrNode) {
      if (jQueryOrNode.jquery) {
        const element = elements[0]
        element.parentNode.insertBefore(jQueryOrNode.get(0), element)
      } else if (jQueryOrNode instanceof Node) {
        element.parentNode.insertBefore(jQueryOrNode, element)
      }
      return this
    }
  }
}
```

向后插入元素：.after(jQueryOrNode)
---------------------------

将指定元素作为所选择元素的后一个节点。

*   语法

```
const $api = $(selectorsOrArrayOrTemplate).after(jQueryOrNode)
```

*   实现

*   基于 `insertBefore` 实现，获取元素列表中当前节点的父节点后，将指定节点插入到当前节点后一个同级节点之后

```
window.$ = window.jQuery = function (selectorsOrArrayOrTemplate) {
  let elements
  if (typeof selectorsOrArrayOrTemplate === "string") {
    if (selectorsOrArrayOrTemplate[0] === "<") {
      elements = [create(selectorsOrArrayOrTemplate)]
    } else {
      elements = document.querySelectorAll(selectorsOrArrayOrTemplate)
    }
  } else if (selectorsOrArrayOrTemplate instanceof Array) {
    elements = selectorsOrArrayOrTemplate
  }
  function create(string) {
    const template = document.createElement("template")
    template.innerHTML = string.trim()
    return template.content.firstChild
  }
  return {
    oldApi: selectorsOrArray.oldApi,
    elements: elements,
    jquery: true,
    after(jQueryOrNode) {
      const element = elements[0]
      if (jQueryOrNode.jquery) {
        element.parentNode.insertBefore(
          jQueryOrNode.get(0),
          jQuery([element]).next().get(0)
        )
      } else if (jQueryOrNode instanceof Node) {
        element.parentNode.insertBefore(
          jQueryOrNode,
          jQuery([element]).next().get(0)
        )
      }
      return this
    }
  }
}
```

插入子元素：.append(jQueryOrNode)
---------------------------

将指定元素作为所选择元素的末位子节点。

*   语法

```
const $api = $(selectorsOrArrayOrTemplate).append(jQueryOrNode)
```

*   实现

```
window.$ = window.jQuery = function (selectorsOrArrayOrTemplate) {
  let elements
  if (typeof selectorsOrArrayOrTemplate === "string") {
    if (selectorsOrArrayOrTemplate[0] === "<") {
      elements = [create(selectorsOrArrayOrTemplate)]
    } else {
      elements = document.querySelectorAll(selectorsOrArrayOrTemplate)
    }
  } else if (selectorsOrArrayOrTemplate instanceof Array) {
    elements = selectorsOrArrayOrTemplate
  }
  function create(string) {
    const template = document.createElement("template")
    template.innerHTML = string.trim()
    return template.content.firstChild
  }
  return {
    oldApi: selectorsOrArray.oldApi,
    elements: elements,
    jquery: true,
    append(jQueryOrNode) {
      const element = this.get(0)
      if (jQueryOrNode.jquery) {
        element.appendChild(jQueryOrNode.get(0))
      } else if (jQueryOrNode instanceof Node) {
        element.appendChild(jQueryOrNode)
      }
      return this
    }
  }
}
```

插入子元素（逆）：.appendTo(jQueryOrNode)
--------------------------------

将所选择元素作为指定元素的末位子节点。

*   语法

```
const $api = $(selectorsOrArrayOrTemplate).appendTo(jQueryOrNode)
```

*   实现

```
window.$ = window.jQuery = function (selectorsOrArrayOrTemplate) {
  let elements
  if (typeof selectorsOrArrayOrTemplate === "string") {
    if (selectorsOrArrayOrTemplate[0] === "<") {
      elements = [create(selectorsOrArrayOrTemplate)]
    } else {
      elements = document.querySelectorAll(selectorsOrArrayOrTemplate)
    }
  } else if (selectorsOrArrayOrTemplate instanceof Array) {
    elements = selectorsOrArrayOrTemplate
  }
  function create(string) {
    const template = document.createElement("template")
    template.innerHTML = string.trim()
    return template.content.firstChild
  }
  return {
    oldApi: selectorsOrArray.oldApi,
    elements: elements,
    jquery: true,
    appendTo(jQueryOrNode) {
      const element = this.get(0)
      if (jQueryOrNode.jquery) {
        jQueryOrNode.get(0).appendChild(element)
      } else if (jQueryOrNode instanceof Node) {
        jQueryOrNode.appendChild(element)
      }
      return this
    }
  }
}
```

删
=

删除元素：.remove()
--------------

遍历元素列表并删除其中所有节点，返回 jQuery 对象。

*   语法

```
const $api = $(selectorsOrArrayOrTemplate).remove()
```

*   实现

```
window.$ = window.jQuery = function (selectorsOrArrayOrTemplate) {
  let elements
  if (typeof selectorsOrArrayOrTemplate === "string") {
    if (selectorsOrArrayOrTemplate[0] === "<") {
      elements = [create(selectorsOrArrayOrTemplate)]
    } else {
      elements = document.querySelectorAll(selectorsOrArrayOrTemplate)
    }
  } else if (selectorsOrArrayOrTemplate instanceof Array) {
    elements = selectorsOrArrayOrTemplate
  }
  function create(string) {
    const template = document.createElement("template")
    template.innerHTML = string.trim()
    return template.content.firstChild
  }
  return {
    oldApi: selectorsOrArray.oldApi,
    elements: elements,
    jquery: true,
    remove() {
      this.each((element) => {
        element.remove
      })
      return this
    }
  }
}
```

清空子元素：.empty()
--------------

遍历元素列表并删除其中所有元素的子节点，返回 jQuery 对象。

*   语法

```
const $api = $(selectorsOrArrayOrTemplate).empty()
```

*   实现

```
window.$ = window.jQuery = function (selectorsOrArrayOrTemplate) {
  let elements
  if (typeof selectorsOrArrayOrTemplate === "string") {
    if (selectorsOrArrayOrTemplate[0] === "<") {
      elements = [create(selectorsOrArrayOrTemplate)]
    } else {
      elements = document.querySelectorAll(selectorsOrArrayOrTemplate)
    }
  } else if (selectorsOrArrayOrTemplate instanceof Array) {
    elements = selectorsOrArrayOrTemplate
  }
  function create(string) {
    const template = document.createElement("template")
    template.innerHTML = string.trim()
    return template.content.firstChild
  }
  return {
    oldApi: selectorsOrArray.oldApi,
    elements: elements,
    jquery: true,
    empty() {
      this.each((element) => {
        const children = element.children()
        for (let i = 0; i < children.length; i++) {
          children[i].remove
        }
      })
      return this
    }
  }
}
```

改
=

读写特性：attr(name, value)
----------------------

返回或修改所选元素的指定特性值。

*   语法

```
const attributeValue = $(selectorsOrArrayOrTemplate).attr(name)
const $api = $(selectorsOrArrayOrTemplate).attr(name, value)
```

*   参数

*   `name` String，需要获取或修改的特性名称的字符串
*   `value` String，需要修改的特性新值的字符串

*   实现

*   重载：当传入一个参数时，读取元素列表中第一个元素的指定特性值；当传入两个参数时，修改元素列表中每一个元素指定特性值

```
window.$ = window.jQuery = function (selectorsOrArrayOrTemplate) {
  let elements
  if (typeof selectorsOrArrayOrTemplate === "string") {
    if (selectorsOrArrayOrTemplate[0] === "<") {
      elements = [create(selectorsOrArrayOrTemplate)]
    } else {
      elements = document.querySelectorAll(selectorsOrArrayOrTemplate)
    }
  } else if (selectorsOrArrayOrTemplate instanceof Array) {
    elements = selectorsOrArrayOrTemplate
  }
  function create(string) {
    const template = document.createElement("template")
    template.innerHTML = string.trim()
    return template.content.firstChild
  }
  return {
    oldApi: selectorsOrArray.oldApi,
    elements: elements,
    jquery: true,
    attr(name, value) {
      if (arguments.length === 1) {
        return this.get(0).getAttribute(name)
      } else if (arguments.length === 2) {
        this.each((element) => {
          element.setAttribute(name, value)
        })
      }
      return this
    }
  }
}
```

读写 style 属性：.css(name, value)
-----------------------------

返回或修改所选元素的指定样式属性值。

*   语法

```
const styleValue = $(selectorsOrArrayOrTemplate).css(name)
const $api = $(selectorsOrArrayOrTemplate).css(name, value)
```

*   参数

*   `name` String | Object，需要获取或修改的样式属性名称的字符串或者需要修改的样式属性的对象
*   `value` String，需要修改的样式属性新值的字符串

*   实现

*   重载：当传入一个参数时，读取元素列表中第一个元素的指定特性值；当传入两个参数时，修改元素列表中每一个元素指定特性值

```
window.$ = window.jQuery = function (selectorsOrArrayOrTemplate) {
  let elements
  if (typeof selectorsOrArrayOrTemplate === "string") {
    if (selectorsOrArrayOrTemplate[0] === "<") {
      elements = [create(selectorsOrArrayOrTemplate)]
    } else {
      elements = document.querySelectorAll(selectorsOrArrayOrTemplate)
    }
  } else if (selectorsOrArrayOrTemplate instanceof Array) {
    elements = selectorsOrArrayOrTemplate
  }
  function create(string) {
    const template = document.createElement("template")
    template.innerHTML = string.trim()
    return template.content.firstChild
  }
  return {
    oldApi: selectorsOrArray.oldApi,
    elements: elements,
    jquery: true,
    css(name, value) {
      if (arguments.length === 1) {
        if (typeof name === "string") {
          return this.get(0).style[name]
        } else if (name instanceof Object) {
          this.each((element) => {
            for (let key in name) {
              element.style[key] = name[key]
            }
          })
        }
      } else if (arguments.length === 2) {
        this.each((element) => {
          element.style[name] = value
        })
      }
      return this
    }
  }
}
```

读写 class 属性
-----------

### 判断 class 属性：.hasClass(className)

判断元素列表中的第一个元素的 class 属性是否包含相应的字符串，返回布尔值。

### 添加 class 属性：.addClass(className)

遍历元素列表，为其中每一个元素添加 class 属性。

### 删除 class 属性：.removeClass(className)

遍历元素列表，为其中每一个元素删除 class 属性。

*   语法

```
let bool = $(selectorsOrArrayOrTemplate).hasClass(className)
let $api = $(selectorsOrArrayOrTemplate).addClass(className)
let $api = $(selectorsOrArrayOrTemplate).removeClass(className)
```

*   实现

```
window.$ = window.jQuery = function (selectorsOrArrayOrTemplate) {
  let elements
  if (typeof selectorsOrArrayOrTemplate === "string") {
    if (selectorsOrArrayOrTemplate[0] === "<") {
      elements = [create(selectorsOrArrayOrTemplate)]
    } else {
      elements = document.querySelectorAll(selectorsOrArrayOrTemplate)
    }
  } else if (selectorsOrArrayOrTemplate instanceof Array) {
    elements = selectorsOrArrayOrTemplate
  }
  function create(string) {
    const template = document.createElement("template")
    template.innerHTML = string.trim()
    return template.content.firstChild
  }
  return {
    oldApi: selectorsOrArray.oldApi,
    elements: elements,
    jquery: true,
    hasClass(className) {
      return this.get(0).classList.contains(className)
    },
    addClass(className) {
      this.each((element) => {
        element.classList.add(className)
      });
      return this
    },
    removeClass(className) {
      this.each((element) => {
        element.classList.remove(className)
      })
      return this
    }
  }
}
```

读写文本内容：.text(textString)
------------------------

返回或修改所选元素的文本内容。

*   语法

```
let text = $(selectorsOrArrayOrTemplate).text()
const $api = $(selectorsOrArrayOrTemplate).text(textString)
```

*   实现

*   重载：当不传入参数时，读元素列表中每个元素的文本内容结合；当传入一个参数时，修改元素列表中每个元素的文本内容
*   适配：基于 `document.all` 实现，兼容 IE 浏览器

```
window.$ = window.jQuery = function (selectorsOrArrayOrTemplate) {
  let elements
  if (typeof selectorsOrArrayOrTemplate === "string") {
    if (selectorsOrArrayOrTemplate[0] === "<") {
      elements = [create(selectorsOrArrayOrTemplate)]
    } else {
      elements = document.querySelectorAll(selectorsOrArrayOrTemplate)
    }
  } else if (selectorsOrArrayOrTemplate instanceof Array) {
    elements = selectorsOrArrayOrTemplate
  }
  function create(string) {
    const template = document.createElement("template")
    template.innerHTML = string.trim()
    return template.content.firstChild
  }
  return {
    oldApi: selectorsOrArray.oldApi,
    elements: elements,
    jquery: true,
    text(textString) {
      if (arguments.length === 0) {
        let string = ""
        this.each((element) => {
          string += document.all ? element.innerText : element.textContent
        });
        return string
      } else if (arguments.length === 1) {
        this.each((element) => {
          document.all
            ? (element.innerText = textString)
            : (element.textContent = textString)
        })
      }
      return this
    }
  }
}
```

读写 HTML 内容：.html(htmlString)
----------------------------

返回或修改所选元素的 HTML 内容。

*   语法

```
let html = $(selectorsOrArrayOrTemplate).html()
const $api = $(selectorsOrArrayOrTemplate).html(htmlString)
```

*   实现

*   重载：当不传入参数时，读取元素列表中第一个元素的 HTML 内容；当传入一个参数时，修改元素列表中所有元素的 HTML 内容

```
window.$ = window.jQuery = function (selectorsOrArrayOrTemplate) {
  let elements
  if (typeof selectorsOrArrayOrTemplate === "string") {
    if (selectorsOrArrayOrTemplate[0] === "<") {
      elements = [create(selectorsOrArrayOrTemplate)]
    } else {
      elements = document.querySelectorAll(selectorsOrArrayOrTemplate)
    }
  } else if (selectorsOrArrayOrTemplate instanceof Array) {
    elements = selectorsOrArrayOrTemplate
  }
  function create(string) {
    const template = document.createElement("template")
    template.innerHTML = string.trim()
    return template.content.firstChild
  }
  return {
    oldApi: selectorsOrArray.oldApi,
    elements: elements,
    jquery: true,
    html(htmlString) {
      if (arguments.length === 0) {
        return this.get(0).innerHTML
      } else if (arguments.length === 1) {
        this.each((element) => {
          element.innerHTML = htmlString
        })
      }
      return this
    }
  }
}
```

事件监听
----

### 绑定事件监听：.on(eventType, listener, useCapture)

遍历元素列表，为其中每一个元素绑定事件监听函数。

### 删除事件监听：.off(eventType, listener, useCapture)

遍历元素列表，为其中每一个元素删除事件监听函数。

*   语法

```
let $api = $(selectorsOrArrayOrTemplate).on(eventType, listener, useCapture)
let $api = $(selectorsOrArrayOrTemplate).off(eventType, listener, useCapture)
```

*   实现

```
window.$ = window.jQuery = function (selectorsOrArrayOrTemplate) {
  let elements
  if (typeof selectorsOrArrayOrTemplate === "string") {
    if (selectorsOrArrayOrTemplate[0] === "<") {
      elements = [create(selectorsOrArrayOrTemplate)]
    } else {
      elements = document.querySelectorAll(selectorsOrArrayOrTemplate)
    }
  } else if (selectorsOrArrayOrTemplate instanceof Array) {
    elements = selectorsOrArrayOrTemplate
  }
  function create(string) {
    const template = document.createElement("template")
    template.innerHTML = string.trim()
    return template.content.firstChild
  }
  return {
    oldApi: selectorsOrArray.oldApi,
    elements: elements,
    jquery: true,
    on(eventType, listener, useCapture = false) {
      this.each((element) => {
        element.addEventListener(eventType, listener, useCapture)
      })
      return this
    },
    off(eventType, listener, useCapture = false) {
      this.each((element) => {
        element.removeEventListener(eventType, listener, useCapture)
      })
      return this
    }
  }
}
```

参考文章
====

1.  [jQuery API 中文文档](https://www.jquery123.com/)
2.  [阮一峰 jQuery 设计思想](http://www.ruanyifeng.com/blog/2011/07/jquery_fundamentals.html)
3.  [方应杭 jQuery 都过时了，那我还学它干嘛？](https://fangyinghang.com/why-still-jquery/)
4.  [Woozyzzz DOM 封装-对象风格](https://www.yuque.com/woozyzzz/ybz8i1/yo9rg6)

若有收获，就点个赞吧