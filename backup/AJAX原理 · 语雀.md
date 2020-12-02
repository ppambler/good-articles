> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [www.yuque.com](https://www.yuque.com/woozyzzz/ybz8i1/niki6t_xcotw5#WvXAF)

AJAX原理
======

AJAX（Async JavaScript And XML）是指异步JS与XML技术。AJAX的全部内容实质上就是**用JS发送请求和接收响应**。

背景
==

*   AJAX是浏览器提供的功能（最早由IE5推出，之后FireFox和Chrome跟进）

*   浏览器可以 **发送请求，接收响应** 
*   浏览器在window上添加了一个 XMLHttpRequest 函数
*   用这个构造函数（类）可以构造出一个对象（变量request）
*   JS通过这个对象来实现 **发送请求，接收响应** 

*   准备一个服务器

*   复制 [server.js](https://github.com/FrankFang/nodejs-test/blob/master/server.js) 中代码，并将其作为我们的服务器
*   使用 `node server.js 8888` 启动
*   添加 index.html 与 main.js 两个路由（判断字符串path的值）

*   细节

*   可以安装 node-dev 自动重启 server.js 便于开发
*   运行 `yarn global add node-dev` 安装工具，安装后执行 `node-dev server.js 8888` 启动
*   server.js中是node.js代码（Python、Java、PHP等都能作为后端代码）
*   在index.html路由内容中添加对main.js的引用即可在localhost：8888/index.html中看到JS代码
*   node.js服务器读取的内容本质就是字符串，因此可以创建public目录新建index.html和main.js并填写内容，路由代码使用 ``response.write(fs.readFileSync(`./public/index.html`))`` 

```
console.log('有人发请求过来啦！路径（带查询参数）为：' + pathWithQuery)

  if(path === '/index.html'){
    response.statusCode = 200
    response.setHeader('Content-Type', 'text/html;charset=utf-8')
    response.write(`<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <meta  />
    <title>AJAX demo</title>
  </head>
  <body>
    <script src="/main.js"></script>
  </body>
</html>`)
    response.end()
  } else if(path === '/main.js'){
    response.statusCode = 200
    response.setHeader('Content-Type', 'text/javaScript;charset=utf-8')
    response.write(`console.log('我是main.js')`)
    response.end()
  } else {
    response.statusCode = 404
    response.setHeader('Content-Type', 'text/html;charset=utf-8')
    response.write(`你输入的路径不存在对应的内容`)
    response.end()
  }
```

加载CSS
=====

*   以前用 `<link rel="stylesheet" href="/1.css" />` 
*   这次用AJAX加载CSS

四个步骤
----

1.  创建HttpRequest对象（全称是XMLHttpRequset）
2.  调用对象的open方法
3.  监听对象的onload和onerror事件（专业前端会改用onreadystatechange事件），在事件中操作CSS内容
4.  调用对象的send方法（发送请求）

实施
--

在server.js中创建1.css路由，创建public/1.css文件，在index.html中创建id为getCSS的按钮，在main.js中监听按钮的点击事件。

```
// main.js
getCSS.onclick = () =>{
  const request = new XMLHttpRequst()
  request.open('GET', `/1.css`)
  request.onload = ()=>{
    console.log(`getCSS成功`)
    const style = document.createElement('style')
    style.textContent = request.response
    document.head.appendChild(style)
  }
  request.onerror = ()=>{console.log(`getCSS失败`)}
  request.send()
}
```

但实际上当我们输入地址出现错误时，onerror事件却没有触发，出发的是server.js中的404。这是由于onerror是发明的，因此没有很好的匹配AJAX。

专业前端会使用[onreadystatechange](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/onreadystatechange)事件，一个请求的一生可以用它的[readyState](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/readyState)来表示： ![image.png](https://cdn.nlark.com/yuque/0/2020/png/1112664/1588166015246-c8e6ec8a-a88f-4958-ad5e-3876d1007021.png "image.png")

值状态

描述

代理被创建,但尚未调用opent方法.

UNSENT

open()方法已经被调用.

OPENED

2

HEADERSRECEIVED

sendO方法已经被调用,并且头部和状态已经可获得.

下载中:responseText属性已经包含部分数据.

LOADING

下载操作已完成.

DONE

*   0： `const request = new XMLHttpRequest()` 
*   1： `request.open()` 
*   2： `request.send()` 
*   3： 当第一个信息出现在浏览器中时（开始下载时）
*   4：下载完成

可在浏览器中F12打开开发者工具->点击Network->点击Online改为Slow 3G（模拟慢速3G网络）测试，只有在request.readyState===4时，获取到的request.response才是完整的。

如果地址写错了，readyState仍然会达到4，但此时request.status的值为404（以2开头的状态码都表示成功）。下面是改写后的main.js代码。

```
getCSS.onclick = ()=>{
  const request = new XMLHttpRequest()
  request.open('GET', '/1.css')
  request.onreadystatechange = ()=>{
    if(request.readyState===4 && request.status===200){
      const style = document.createElement('style')
      style.textContent = request.response
      document.head.appendChild(style)
    }
  }
  request.send()
}
```

加载JS
====

*   以前用 `<script src="2.js"></script>` 
*   这次用AJAX加载JS

四个步骤
----

1.  创建HttpRequest对象（全称是XMLHttpRequest）
2.  调用对象的open方法
3.  监听对象的onreadystatechange事件（在事件中操作JS内容）
4.  调用对象的send方法

在server.js中创建2.js路由，创建public/2.js文件，在index.html中创建id为getJS的按钮，在main.js中监听按钮的点击事件。

```
getJS.onclick = ()=>{
  const request = new XMLHttpRequest()
  request.open('GET', '/2.js')
  request.onreadystatechange = ()=>{
  if(request.readyState===4 && request.status===200){
    const script = document.createElement('script')
    script.textContent = request.response
    document.body.appendChild(script)
  }
  request.send()
}
```

加载HTML
======

*   以前不会加载3.html

四个步骤
----

1.  创建HttpRequest对象（全称是XMLHttpRequest）
2.  调用对象的open方法
3.  监听对象的onreadystatechange事件（在事件中操作HTML内容）
4.  调用对象的send方法

在server.js中创建3.html路由，创建public/3.html文件，文件内容为 `<div style="width: 200px; height: 300px; border: 1px solid blueviolet;">我是3.html</div>` ，在index.html中创建id为getHTML的按钮，在main.js中监听按钮的点击事件。

```
getHTML.onclick = ()=>{
  const request = new XMLHttpRequest
  request.open('GET', '/3.html')
  request.onreadystate = ()=>{
    if(request.readyState===4 && request.status===200){
      const div = document.createElement('div')
      div.innerHTML = request.response
      document.body.appendChild(div)
    }
  }
  request.send()
}
```

加载XML
=====

*   以前不会加载4.xml
*   在JSON出生前，用XML传递数据

四个步骤
----

1.  创建HttpRequest对象（全称是XMLHttpRequest）
2.  调用对象的open方法
3.  监听对象的onreadystatechange事件（在事件中使用responseXML获取XML内容，DOM不仅能操作HTML标签也能操作XML标签）
4.  调用对象的send方法

在server.js中创建4.xml路由，创建public/4.xml文件，文件内容如下：

```
<?xml version="1.0" encoding="UTF-8"?>
<message>
    <warning>
         Hello World
    </warning>
</message>
```

在index.html中创建id为getXML的按钮，在main.js中监听按钮的点击事件。

```
getXML.onclick = ()=>{
  const request = new XMLHttpRequest()
  request.open('GET', '/4.xml')
  request.onreadystatechange = ()=>{
    if(request.readyState===4 && request.status===200){
      const dom = request.responseXML
      cosnt text = dom.getElementsByTagName('warning')[0].textContent
      console.log(text.trim())
    }
  }
  request.send()
}
```

小结
==

*   HTTP是个框，什么都能往里装

*   可以装HTML、CSS、JS、XML……
*   设置正确的Content-Type是好习惯

*   解析方法

*   得到CSS后生成style标签
*   得到JS后生成script标签
*   得到HTML后使用innerHTML和DOM API
*   得到XML后使用responseXML和DOM API
*   不同类型的数据有不同的解析办法，对应不同使用方法

JSON
====

JSON(JavaScript Object Notation)与HTML、XML、Markdown一样是一门独立的标记语言（非编程语言），它被用于展示数据。点击[JSON中文官网](http://json.org/json-zh.html)，浏览网页中的铁轨图可以快速学习它的语法。

*   支持的数据类型（六种）

*   string：只支持双引号，不支持单引号和无引号
*   number：支持科学计数法
*   bool：true和false
*   null：没有undefined
*   object：不支持函数，不支持变量
*   array

JSON之父道格拉斯曾写过一本《JS语言精粹》，也被叫做蝴蝶书（现已过时），道格拉斯根据JS发明了JSON这门标记语言，他比JS之父布莱登年长，也曾向布莱登吐槽过JS这门语言。

加载JSON
======

*   用AJAX加载JSON

四个步骤
----

1.  创建HttpRequest对象（全称是XMLHttpRequest）
2.  调用对象的open方法
3.  监听对象的onreadystatechange事件（在事件中使用JSON.parse将request.response字符串转化为JS数据类型）
4.  调用对象的send方法

在server.js中创建5.json路由，JSON的Content-Type可以是text/json也可以是application/json，创建public/5.json文件，在index.html中创建id为getJSON的按钮，在main.js中监听按钮的点击事件。

```
getJSON.onclick = ()=>{
  const request = new XMLHttpRequest()
  request.open('GET', '/5.json')
  request.onreadystatechange = ()=>{
    if(request.readyState===4 && request.status){
      let obj
      try{
        obj = JSON.parse(request.response)
      }catch(error){
        console.log(`转换对象失败`)
        console.log(error)
        obj = {name:'noName'}
      }
      console.log(obj)
    }
  }
  request.send()
}
```

window.JSON
-----------

*   JSON.parse

*   将符合JSON语法的字符串转换成JS对应类型的数据
*   JSON字符串=>JS数据
*   由于JSON只有六种数据类型，因此转换成的数据也只有六种
*   若不符合JSON语法，则会抛出一个Error对象
*   一般用try catch捕捉错误

*   JSON.stringify

*   是JSON.parse的逆运算
*   JS数据=>JSON字符串
*   由于JS的数据类型比JSON多，因此不一定成功
*   失败时也会抛出一个Error对象

综合应用（加载分页）
==========

加载列表
----

*   需求

*   用户打开页面，看到第一页数据
*   用户点击下一页，看到第二页数据
*   用户点击下一页，看到第三页数据
*   用户点击下一页，提示没有更多了

*   优化点

*   在点击第三页时，禁用下一页按钮

在index.html中创建一对div标签，其包含的内容为{{page}}，在server.js中创建page1.json、page2.json和page3.json路由，修改index.html路由内容如下：

```
if(path==='/index.html'){
  response.setHeader('Content-Type', 'application/json;charset=utf-8')
  let string = fs.readFileSync(`./public/index.html`).toString()
  const arr = JSON.parse(fs.readFileSync(`./db/page1.json`))
  const result = arr.map(item=>`<li>${item.id}</li>`).join('')
  // 前后端不分离的渲染技术
  string = string.replace('{{page}}', `<ul>${result}</ul>`)
  response.write(string)
  response.end()
}
```

创建db/page1.json、db/page2.json和db/page3.json文件，文件内容为如下数组（注意格式数字递增）：

```
[{"id":1},{"id":2},{"id":3},{"id":4},{"id":5}]
```

在index.html中创建id为getPage的按钮，在main.js中监听按钮的点击事件。

```
getPage.onclick = ()=>{
  const request = new XMLHttpRequest()
  request.open('GET', '/page2.json')
  request.onreadystatechange = ()=>{
    if(request.readyState===4 && request.status===200){
      const ul = document.getElementsByTagName('ul')[0]
      ul.innerHTML = ''
      const arr = JSON.parse(request.response)
      arr.forEach(item=>{
        const li = document.createElement('li')
        li.textContent = item.id
        ul.appendChild(li)
      })
    }
  }
  request.send()
}
```

优化后代码如下：

```
let n = 1;
getPage.onclick = () => {
  if (n < 3) {
    console.log(n);

    const request = new XMLHttpRequest();
    request.open("GET", `/page${n + 1}.json`);
    request.onreadystatechange = () => {
      if (request.readyState === 4 && request.status === 200) {
        const ul = document.getElementsByTagName("ul")[0];
        ul.innerHTML = "";
        console.log(`getPage成功，读取/page${n + 1}.json`);
        const array = JSON.parse(request.response);
        array.forEach((element) => {
          const li = document.createElement("li");
          li.textContent = element.id;
          ul.appendChild(li);
        });
        n += 1;
      }
    };
    request.send();
  } else { // 还需再点一次才变disabled
    getPage.innerHTML = `<s>${getPage.textContent}</s>`;
    getPage.setAttribute("disabled", "");
  }
};
```

再优化，增加上一页按钮（默认disabled不可选中）：

```
<!DOCTYPE html>
<html lang="zh-CN">
  <head>
    <meta charset="UTF-8" />
    <meta  />
    <title>AJAX demo</title>
  </head>
  <body>
    <h1>人类的赞歌是勇气的赞歌！</h1>
    <p>
      <button id="getCSS">getCSS</button>
      <button id="getJS">getJS</button>
      <button id="getHTML">getHTML</button>
      <button id="getXML">getXML</button>
      <button id="getJSON">getJSON</button>
      <button id="prePage" disabled><s>prePage</s></button>
      <button id="nextPage">nextPage</button>
    </p>
    <div>{{page}}</div>
    <script src="/main.js"></script>
  </body>
</html>

```

```
let n = 1;
nextPage.onclick = () => {
  const string = prePage.innerHTML.replace("</s>", "").replace("<s>", "");
  prePage.innerHTML = string;
  prePage.removeAttribute("disabled");
  const request = new XMLHttpRequest();
  request.open("GET", `/page${n + 1}.json`);
  request.onreadystatechange = () => {
    if (request.readyState === 4 && request.status === 200) {
      console.log(`nextPage成功，读取/page${n + 1}.json`);
      const ul = document.getElementsByTagName("ul")[0];
      ul.innerHTML = "";
      const array = JSON.parse(request.response);
      array.forEach((element) => {
        const li = document.createElement("li");
        li.textContent = element.id;
        ul.appendChild(li);
      });
      n += 1;
      if (n >= 3) {
        nextPage.innerHTML = `<s>${nextPage.textContent}</s>`;
        nextPage.setAttribute("disabled", "");
      }
    }
  };
  request.send();
};

prePage.onclick = () => {
  const string = nextPage.innerHTML.replace("</s>", "").replace("<s>", "");
  nextPage.innerHTML = string;
  nextPage.removeAttribute("disabled");
  const request = new XMLHttpRequest();
  request.open("GET", `page${n - 1}.json`);
  request.onreadystatechange = () => {
    if (request.readyState === 4 && request.status === 200) {
      console.log(`prePage成功，读取/page${n - 1}.json`);
      const ul = document.querySelector("ul");
      ul.innerHTML = "";
      const arr = JSON.parse(request.response);
      arr.forEach((item) => {
        const li = document.createElement("li");
        li.textContent = item.id;
        ul.appendChild(li);
      });
      n -= 1;
      if (n <= 1) {
        prePage.innerHTML = `<s>${prePage.textContent}</s>`;
        prePage.setAttribute("disabled", "");
      }
    }
  };
  request.send();
};
```

至此AJAX的加载可以说比较完备了。[源码链接](https://github.com/Woozyzzz/ajax-1)

p.s.关于前后端分离的意义，我简单[搜了一下](https://www.zhihu.com/question/28207685)，目前还不太理解。

![image.png](https://cdn.nlark.com/yuque/0/2020/png/1112664/1588188923724-789ded67-0e75-46cb-81dc-d31e4c982470.png "image.png")

Web前后端分离的意义大吗?

廖雪峰

业余马拉松选手www.liaoxuefeng.com

240人赞同了该回答

前后端要不要分,怎么分,是由具体业务决定的.

需要搜索引擎带流量的,必须由服务器端染.

需要用户登录且不能由搜索引擎抓取,前后端分离是鼓励的.

需要App和后端交互,必须分离.

但是分了就表示架构合理?不一定.设计一套合理/可升级/客户端友好的AP也不容易.

要想分离好,前端开发要了解后端架的抱,后端开发要虚心学习Javascript,双方如果互相都视,分

了也白搭.

编辑于2017-09-21

52条评论

赞同240

分享

收藏

喜欢

若有收获，就点个赞吧