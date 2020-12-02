> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.yuque.com](https://www.yuque.com/woozyzzz/ybz8i1/zn4l59)

浅析 jQuery 操作
============

jQuery 诞生于2006年，是目前前端领域使用最广泛、最长寿的 JS 库，目前全球仍有80%以上的网站在使用它。

选择元素
====

使用 jQuery 选择元素，需要将一个选择表达式作为传入 jQuery 函数的参数，即可得到被选的元素。jQuery 函数可以简写为 $，其中的选择表达式既可以是 CSS 选择器，也可以是 jQuery [特有的表达式](https://api.jquery.com/category/selectors/)。

```
// CSS 选择器
$(document) // 选择整个文档对象
$("#project") // 选择 id 为 project 的元素
$("p.myClass") // 选择 class 为 myClass 的 p 元素
$("input[name=code]") // 选择 name 属性为 code 的input元素

// jQuery 选择器
$("img:first") // 选择网页中的首个 img 元素
$("tr:odd") // 选择表格的奇数行
$("#myForm:input") // 选择表单中的 input，textarea，select 和 button 元素
$("div:visible") // 选择可见的 div 元素
$("div:gt(3)") // 选择前4个以外的 div 元素
$("div:animated") // 选择当前处于动画状态的 div 元素
```

过滤操作
====

jQuery 提供了各种用于筛选选择元素的[过滤器](https://api.jquery.com/category/traversing/filtering/)。

```
$("div").eq(2) // 选择第3个 div 元素
$("div").not("#tab") // 选择 id 属性不为 tab 的 div 元素
$("div").filter("#tab") // 选择 id 属性为 tab 的 div 元素
$("div").first() // 选择第1个 div 元素
$("div").has("span") // 选择包含 span 元素的 div 元素
```

链式操作
====

链式操作是 jQuery 最典型的特点。由于 jQuery 的每一步操作都返回一个 jQuery 对象，因此在使用 jQuery 选择元素后，可以将对元素的各种操作像链条一样连续写出来。

jQuery 提供的 `.end()` 方法可以结束当前链中的最新过滤操作，并将匹配的元素集返回到其先前状态。

```
$("div.round") // 选择 class 为 round 的 div 元素
  .find("h2") // 选择其中的 h2 元素
  .eq(3) // 选择其中第4个元素
  .html("ready") // 修改该元素的 HTML 内容为 ready
  .end() // 结束过滤并返回选中 h2 元素步骤
  .eq(1) // 选择其中的第2个元素
  .html("go")// 修改该元素的 HTML 内容为 go
```

元素的增删改查
=======

增
-

### 新增元素

*   创建元素：将表示新元素的字符串传入 jQuery 函数

```
$("<span>Hi</span>")
$("<div class='red'>new</div>")
$("ul").append("<li>data</li>")
```

*   复制元素：`.clone()`

### 移动元素

*   以所选元素为基准，移动指定元素

*   `.after()` 返回所选元素并在其后面插入指定元素
*   `.before()` 返回所选元素并在其前面插入指定元素
*   `.append()` 返回所选元素并在其内部最后一位插入指定元素
*   `.prepend()` 返回所选元素并在其内部第一位插入指定元素

*   将所选元素移动到指定元素的相对位置

*   `.insertAfter()` 、`.insertBefore()` 、`.appendTo()` 和 `.prependTo()` 区别于上边的方法，操作的视角不同

删
-

*   `.detach()` 删除所选元素同时保留元素上的事件及 jQuery 数据一边再次添加
*   `.remove()` 删除所选元素同时删除元素上的事件及 jQuery 数据
*   `.empty()` 删除所选元素的全部子元素

改
-

### 文本与属性

jQuery 中大量使用了重载的设计模式，使用同一个函数，来完成取值（getter）和赋值（setter），具体是取值还是赋值，由函数的参数决定。

*   `.text()` 读取或修改所选元素的文本内容
*   `.html()` 读取或修改所选元素的 HTML 内容
*   `.attr()` 读取或修改所选元素指定特性的值
*   `.prop()` 读取或修改所选元素指定属性的值
*   `.width()` 读取或修改所选元素的宽度
*   `.height()` 读取或修改所选元素的高度
*   `.val()` 读取所选表单元素的值

值得注意的是，在选择多个元素时，对所选元素的修改，将作用于其中所有的元素；而对所选元素的读取，将只读取第一个元素的值，如果需要全部读取需要借助工具方法 `$.each()` ， `.text()` 将读取所有元素的文本内容）。

### 事件操作

jQuery 提供在网页元素上绑定[事件](https://api.jquery.com/category/events/)的方法。

*   `.bind()` 为所选元素绑定一个事件
*   `.unbind()` 为所选元素解除绑定一个事件
*   `.on()` 为所选元素绑定单个或多个事件
*   `.off()` 为所选元素解除绑定单个或多个事件
*   `.one()` 为所选元素绑定单个或多个单次执行事件

查
-

```
$("div").parent() // 选择 div 元素的父元素
$("div").children() // 选择 div 的所有子元素
$("div").siblings() // 选择 div 的同级元素
$("div").closest("ul") // 选择与 div 元素最近的 form 父元素
$("div").prev("span") // 选择 div 元素前面第一个 span 元素
$("div").next("span") // 选择 div 元素后面第一个 span 元素
```

工具方法
====

jQuery 除了操作选中的元素外，还可以使用工具方法进行一些与元素无关的操作。

```
$.trim() // 去除字符串两端的空格
$.each() // 遍历一个数组或对象
$.extend() // 将对个对象合并到第一个对象
$.type() // 判断对象的类别
$.isEmptyObject() // 判断对象是否为空
```

参考文章
====

1.  [jQuery API 文档](https://api.jquery.com/)
2.  [jQuery API 中文文档](https://www.jquery123.com/)
3.  [阮一峰 jQuery 设计思想](http://www.ruanyifeng.com/blog/2011/07/jquery_fundamentals.html)

若有收获，就点个赞吧