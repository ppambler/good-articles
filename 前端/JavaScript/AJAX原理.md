# AJAX 原理

> 教程：[AJAX原理 · 语雀](https://www.yuque.com/woozyzzz/ybz8i1/niki6t_xcotw5#WvXAF)、[Demo](https://github.com/ppambler/demo/tree/main/01-07-2020-12-02/01/server)

AJAX（Async JavaScript And XML）是指异步 JS 与 XML 技术。AJAX 的全部内容实质上就是**用 JS 发送请求和接收响应**。

## ★背景

* AJAX 是浏览器提供的功能（最早由 IE5 推出，之后 FireFox 和 Chrome 跟进）
  + 浏览器可以 **发送请求，接收响应** 
  + 浏览器在 `window` 上添加了一个 `XMLHttpRequest` 函数
  + 用这个构造函数（类）可以构造出一个对象（变量 `request`）
  + JS 通过这个对象来实现 **发送请求，接收响应**
* 准备一个服务器
  + 复制 [server.js](https://github.com/FrankFang/nodejs-test/blob/master/server.js) 中代码，并将其作为我们的服务器
  + 使用 `node server.js 8888` 启动
  + 添加 `index.html` 与 `main.js` 两个路由（判断字符串`path`的值）
* 细节
  + 可以安装 node-dev 自动重启 `server.js` 便于开发
  + 运行 `yarn global add node-dev` 安装工具，安装后执行 `node-dev server.js 8888` 启动（建议用`npm`安装）
  + server.js 中是 node.js 代码（Python、Java、PHP 等都能作为后端代码）
  + 在 index.html 路由内容中添加对 main.js 的引用即可在 `localhost：8888/index.html` 中看到 JS 代码
  + node.js 服务器读取的内容本质就是字符串，因此可以创建 `public` 目录新建 index.html 和 main.js 并填写内容，路由代码使用 `response.write(fs.readFileSync('./public/index.html'))`

完整的 `server.js` ：

``` js
var http = require("http");
var fs = require("fs");
var url = require("url");
var port = process.argv[2];

if (!port) {
  console.log("请指定端口号好不啦？\nnode server.js 8888 这样不会吗？");
  process.exit(1);
}

var server = http.createServer(function(request, response) {
  var parsedUrl = url.parse(request.url, true);
  var pathWithQuery = request.url;
  var queryString = "";
  if (pathWithQuery.indexOf("?") >= 0) {
    queryString = pathWithQuery.substring(pathWithQuery.indexOf("?"));
  }
  var path = parsedUrl.pathname;
  var query = parsedUrl.query;
  var method = request.method;

  /******** 从这里开始看，上面不要看 ************/

  console.log("有个傻子发请求过来啦！路径（带查询参数）为：" + pathWithQuery);

  if (path === "/" || path === "/index.html") {
    response.statusCode = 200;
    response.setHeader("Content-Type", "text/html;charset=utf-8");
    response.write(fs.readFileSync('./public/index.html'));
    response.end();
  } else if (path === "/scripts/main.js") {
    response.statusCode = 200;
    response.setHeader("Content-Type", "text/javaScript;charset=utf-8");
    response.write(fs.readFileSync('./scripts/main.js'));
    response.end();
  } else {
    response.statusCode = 404;
    response.setHeader("Content-Type", "text/html;charset=utf-8");
    response.write(`你输入的路径不存在对应的内容`);
    response.end();
  }

  /******** 代码结束，下面不要看 ************/
});

server.listen(port);
console.log(
  "监听 " +
  port +
  " 成功、n 请用在空中转体 720 度然后用电饭煲打开 http://localhost:" +
  port
);
```

测试效果：

![image-20201202133539927](https://gitee.com/ppambler/blog-images/raw/master/images/image-20201202133539927.png)

## ★加载 CSS

* 以前用 `<link rel="stylesheet" href="/1.css" />`

* 这次用 AJAX 加载 CSS

### 1）四个步骤

1. 创建 `HttpRequest` 对象（全称是 `XMLHttpRequset`）
2. 调用对象的 `open` 方法
3. 监听对象的 `onload` 和 `onerror` 事件（专业前端会改用 `onreadystatechange` 事件），在事件中操作 CSS 内容
4. 调用对象的 `send` 方法（发送请求）

### 2）实施

在 server.js 中创建 `/styles/main.css` 路由，创建 `styles/main.css` 文件，在 index.html 中创建 `id` 为 `getCSS` 的按钮，在 main.js 中监听按钮的点击事件

``` js
let btn = document.querySelector('#getCSS')

btn.onclick = () => {
  console.log(1)
  const request = new XMLHttpRequest()
  request.open('GET', '/styles/main.css')
  request.onload = () => {
    console.log('GetCSS 成功！')
    const style = document.createElement('style')
    style.textContent = request.response
    document.head.appendChild(style)
  }
  request.onerror = () => {
    console.log('GetCSS 失败！')
  }
  request.send()
}
```

测试：

![image-20201202141425785](https://gitee.com/ppambler/blog-images/raw/master/images/image-20201202141425785.png)

但实际上当我们输入地址出现错误时，如 `request.open('GET','/styles/mian.css')` ， `onerror` 事件却没有触发，触发的是 `server.js` 中的 `404` 。这是由于 `onerror` 是发明的，因此没有很好的匹配 AJAX，简单来说， `onerror` 会在请求失败的时候才会触发，即便返回一个 404 也是一次成功的响应！

![image-20201202142327451](https://gitee.com/ppambler/blog-images/raw/master/images/image-20201202142327451.png)

专业前端会使用 [onreadystatechange](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/onreadystatechange) 事件，一个请求的一生可以用它的 [readyState](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest/readyState) 来表示：

![image-20201202143506542](https://gitee.com/ppambler/blog-images/raw/master/images/image-20201202143506542.png)

``` js
var xhr = new XMLHttpRequest();
console.log('UNSENT', xhr.readyState); // readyState will be 0

xhr.open('GET', '/api', true);
console.log('OPENED', xhr.readyState); // readyState will be 1

xhr.onprogress = function() {
  console.log('LOADING', xhr.readyState); // readyState will be 3
};

xhr.onload = function() {
  console.log('DONE', xhr.readyState); // readyState will be 4
};

xhr.send(null);
```

* 0：`const request = new XMLHttpRequest()`

* 1：`request.open()`

* 2：`request.send()`

* 3：当第一个信息出现在浏览器中时（即开始下载时）
* 4：下载完成

在浏览器中 `F12` 打开开发者工具 -> 点击 Network -> 点击 Online 改为 Slow 3G（模拟慢速 3G 网络）测试，只有在 `request.readyState===4` 时，获取到的 `request.response` 才是完整的。

如果地址写错了， `readyState` 仍然会达到 `4` ，但此时 `request.status` 的值为 `404` （以 2 开头的状态码都表示成功）。

下面是改写后的 main.js 代码：

``` js
btn.onclick = () => {
  const request = new XMLHttpRequest();
  console.log(request.readyState) //0
  request.open("GET", "/styles/main.css");
  console.log(request.readyState) //1
  request.onreadystatechange = () => {
    console.log(request);
    // console.log(request.getAllResponseHeaders()); //获取所有的响应头消息
    console.log(request.readyState, request.status);
    if (request.readyState === 4 && request.status === 200) {
      const style = document.createElement("style");
      style.textContent = request.response;
      document.head.appendChild(style);
    }
  };
  // 异步代码，执行 cb 时才会拿到 readyState 为 2 的状态
  // 注意，send 必须放在 open 之后执行
  request.send();
  console.log(request.readyState) // 1
};
```

测试效果：

![image-20201202145248068](https://gitee.com/ppambler/blog-images/raw/master/images/image-20201202145248068.png)

## ★加载 JS

* 以前用 `<script src="2.js"></script>`

* 这次用 AJAX 加载 JS

### 1）四个步骤

1. 创建 HttpRequest 对象（全称是 XMLHttpRequest）
2. 调用对象的 open 方法
3. 监听对象的 onreadystatechange 事件（在事件中操作 JS 内容）
4. 调用对象的 send 方法

在 server.js 中创建 `/scripts/getjs.js` 路由，创建 `/scripts/getjs.js` 文件，在 index.html 中创建 id 为 getJS 的按钮，在 main.js 中监听按钮的点击事件。

``` js
const btn1 = document.querySelector('#getJS')

btn1.onclick = () => {
  const xhr = new XMLHttpRequest()
  xhr.open('GET', '/scripts/getjs.js')
  xhr.onreadystatechange = () => {
    if (xhr.readyState === 4 && xhr.status === 200) {
      const script = document.createElement('script')
      script.textContent = xhr.response
      document.body.appendChild(script)
    }
  }
  xhr.send()
}
```

> 点击一个「GetJS」按钮就会发送一个 AJAX 请求，请求 url，即后端配置的路由地址，我们拿到 `getjs.js` 里边的内容字符串，把这个内容响应回给浏览器，前端拿到这个数据，就会把该数据组装成一个 `script` 标签，然后插入到 `body` 标签的最后边……

关于 `node-dev` 的启动目录：

![image-20201202153538854](https://gitee.com/ppambler/blog-images/raw/master/images/image-20201202153538854.png)

## ★加载 HTML

* 以前在地址栏输入 url 加载 HTML
* 这次用 AJAX 加载 HTML

### 1）四个步骤

1. 创建 HttpRequest 对象（全称是 XMLHttpRequest）
2. 调用对象的 open 方法
3. 监听对象的 onreadystatechange 事件（在事件中操作 HTML 内容）
4. 调用对象的 send 方法

在 server.js 中创建 `3.html` 路由，创建 `public/3.html` 文件，文件内容为 `<div style="width: 200px; height: 300px; border: 1px solid blueviolet;">我是 3.html</div>` ，在 index.html 中创建 id 为 getHTML 的按钮，在 main.js 中监听按钮的点击事件。

``` js
const btn2 = document.querySelector('#getHTML')

btn2.onclick = () => {
  const xhr = new XMLHttpRequest()
  xhr.open('GET', '/public/3.html')
  xhr.onreadystatechange = () => {
    if (xhr.readyState === 4 && xhr.status === 200) {
      const div = document.createElement('div')
      div.innerHTML = xhr.response
      document.body.appendChild(div)
    }
  }
  xhr.send()
}
```

## ★加载 XML

* 以前不会加载`4.xml`
* 在 JOSN 出生前，用 XML 传递数据

### 1）四个步骤

1. 创建 HttpRequest 对象（全称是 XMLHttpRequest）
2. 调用对象的 open 方法
3. 监听对象的 onreadystatechange 事件（在事件中使用 `responseXML` 获取 XML 内容，DOM 不仅能操作 HTML 标签也能操作 XML 标签）
4. 调用对象的 send 方法

在 server.js 中创建 `4.xml` 路由，创建 `public/4.xml` 文件，文件内容如下：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<message>
  <warning>
      Hello World
  </warning>
</message>
```

在 index.html 中创建 id 为 `getXML` 的按钮，在 main.js 中监听按钮的点击事件：

``` js
const btn3 = document.querySelector('#getXML')

btn3.onclick = () => {
  const xhr = new XMLHttpRequest()
  xhr.open('GET', '/4.xml')
  xhr.onreadystatechange = () => {
    if (xhr.readyState === 4 && xhr.status === 200) {
      const dom = xhr.responseXML
      // 这个 dom 相当于是 HTML 文档的 document
      console.log(dom)
      const text = dom.getElementsByTagName('warning')[0].textContent
      alert(text.trim())
    }
  }
  xhr.send()
}
```

## ★小结

* HTTP 是个框，什么都能往里装
  + 可以装 HTML、CSS、JS、XML……
  + 设置正确的 Content-Type 是好习惯
* 解析方法
  + 得到 CSS 后生成 style 标签
  + 得到 JS 后生成 script 标签
  + 得到 HTML 后使用 innerHTML 和 DOM API
  + 得到 XML 后使用 responseXML 和 DOM API
  + 不同类型的数据有不同的解析办法，对应不同的使用方法

![image-20201202165205582](https://gitee.com/ppambler/blog-images/raw/master/images/image-20201202165205582.png)

## ★JSON

JSON（JavaScript Object Notation）与 HTML、XML、Markdown 一样是一门独立的标记语言（非编程语言），它被用于展示数据。点击 [JSON 中文官网](http://json.org/json-zh.html)，浏览网页中的铁轨图可以快速学习它的语法。

* 支持的数据类型（六种）
  + string：只支持双引号，不支持单引号和无引号
  + number：支持科学计数法
  + bool：true 和 false
  + null：没有 undefined
  + object：不支持函数，不支持变量
  + array

JSON 之父道格拉斯曾写过一本《JS 语言精粹》，也被叫做蝴蝶书（现已过时），道格拉斯根据 JS 发明了 JSON 这门标记语言，他比 JS 之父布莱登年长，也曾向布莱登吐槽过 JS 这门语言。

## ★加载 JSON

用 AJAX 加载 JSON

### 1）四个步骤

1. 创建 HttpRequest 对象（全称是 XMLHttpRequest）
2. 调用对象的 open 方法
3. 监听对象的 onreadystatechange 事件（在事件中使用 `JSON.parse` 将 `request.response` 字符串转化为 JS 数据类型）
4. 调用对象的 send 方法

在 server.js 中创建 `5.json` 路由，JSON 的 `Content-Type` 可以是 `text/json` 也可以是 `application/json` ，创建 `public/5.json` 文件，在 index.html 中创建 id 为 `getJSON` 的按钮，在 main.js 中监听按钮的点击事件

``` js
const btn4 = document.querySelector('#getJSON')

btn4.onclick = () => {
  const xhr = new XMLHttpRequest()
  xhr.open('GET', '/5.json')
  xhr.onreadystatechange = () => {
    if (xhr.readyState === 4 && xhr.status === 200) {
      let obj
      try {
        obj = JSON.parse(xhr.response)
      } catch (e) {
        console.log('转换对象失败')
        console.log(e)
        obj = {
          name: 'noName'
        }
      }
      console.log(obj)
    }
  }
  xhr.send()
}
```

### 2）window. JSON

* JSON.parse
  + 将符合 JSON 语法的字符串转换成 JS 对应类型的数据
  + JSON 字符串 -> JS 数据
  + 由于 JSON 只有六种数据类型，因此转换成的数据也只有六种
  + 若不符合 JSON 语法，则会抛出一个 Error 对象
  + 一般用 try catch 捕捉错误
* JSON.stringify
  + 是 JSON.parse 的逆运算
  + JS 数据 -> JSON 字符串
  + 由于 JS 的数据类型比 JSON 多，因此不一定成功
  + 失败时也会抛出一个 Error 对象

> 我测试了一下 `JSON.stringify` ，传入了一个函数，结果啥事也没发生……

## ★综合应用（加载分页）

### 1）加载列表

* 需求
  + 用户打开页面，看到第一页数据
  + 用户点击下一页，看到第二页数据
  + 用户点击下一页，看到第三页数据
  + 用户点击下一页，提示没有更多了
* 优化点
  + 在点击第三页时，禁用下一页按钮

图示整个需求：

![image-20201202212242335](https://gitee.com/ppambler/blog-images/raw/master/images/image-20201202212242335.png)

> 第一次透过 `url` 请求页面 `index.html` ，所拿到的数据是前后端不分离的，第二次透过点击按钮发送 AJAX 请求，才是前后端分离的，即前端代码和后端代码是分开的，即交给前端代码去构建 DOM 节点，而不是交给后端去构建！

### 2）实现

1. 在 index.html 中创建一对 div 标签，其包含的内容为 🟡🟡page🟡🟡
2. 在 server.js 中创建 page1.json、page2.json 和 page3.json 路由
3. 修改 index.html 路由内容

`index.html`：

``` html
<button id="prePage" disabled><s>prePage</s></button>
<button id="nextPage">nextPage</button>
<div>🟡🟡page🟡🟡</div>
```

`/` or `/index.html`路由：

``` js
if (path === "/" || path === "/index.html") {
  response.statusCode = 200;
  console.log("他请求访问/index.html");
  response.setHeader("Content-Type", "text/html;charset=utf-8");
  let string = fs.readFileSync(`public/index.html`);
  console.log(`fs.readFileSync('public/index.html') 是一个${typeof string}`);
  console.log(string) //Buffer 实例
  string = string.toString();
  console.log(string) // index.html 里边的字符串化内容
  const page1 = fs.readFileSync(`db/page1.json`);
  console.log(page1)
  const array = JSON.parse(page1);
  console.log(array)
  const result = array.map((item) => `<li>${item.id}</li>`).join("");
  // 前后端不分离的渲染技术
  string = string.replace(`{{page}}`, `<ul>${result}</ul>`);
  console.log(result)
  response.write(string);
  response.end();
}
```

数据库：

- `page1.json`：`[{"id":1},{"id":2},{"id":3},{"id":4},{"id":5}]`
- `page2.json`：`[{"id":6},{"id":7},{"id":8},{"id":9},{"id":10}]`
- `page3.json`：`[{"id":11},{"id":12},{"id":13},{"id":14},{"id":15}]`

AJAX 请求上一页和下一页：

``` js
// 第 n 页，就是第 n 页数据，如 n 为 1，那就是第一页数据，
// n 为 3，nextPage 这个按钮不可点击
// n 为 1，prePage 这个按钮不可点击
let n = 1;
nextPage.onclick = () => {
  console.log(typeof prePage.innerHTML) // string
  const string = prePage.innerHTML.replace("</s>", "").replace("<s>", "");
  prePage.innerHTML = string;
  prePage.removeAttribute("disabled");
  const request = new XMLHttpRequest();
  request.open("GET", `/page${n + 1}.json`);
  request.onreadystatechange = () => {
    if (request.readyState === 4 && request.status === 200) {
      console.log(`nextPage 成功，读取/page${n + 1}.json`);
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
      console.log(`prePage 成功，读取/page${n - 1}.json`);
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

至此，关于 AJAX 的分页加载可以说比较完备了

## ★前后端分离

好处：

- 代码分离：前端代码 & 后端代码，代码更好看……
- 节约用人成本：全栈`2w`，前端`8k`，后端`8k` -> 学习成本低

弊端：

- 前后端沟通困难
- 不利于开发者自行去创业

➹：[Web 前后端分离的意义大吗？ - 知乎](https://www.zhihu.com/question/28207685)

## ★了解更多

- [ajax 获取 Response Headers 响应头信息 - 未来往事](https://www.fity.cn/post/671.html)

## ★总结

- 我们往`style`标签和`script`标签里边写的内容，都是字符串！ -> 从`script.textContent = xhr.response`这行代码可以看出来，`xhr.response`是`string`类型的数据！我把请求脚本的响应头改成是`text/plain`也不影响 JS 内容的解析执行！

## ★Q&A

### 1）`text/html` vs `text/plain`？

1. `text/html` 的意思是将文件的 `content-type` 设置为 `text/html` 的形式，浏览器在获取到这种文件时会自动调用 html 的解析器对文件进行相应的处理。
2. `text/plain` 的意思是将文件设置为纯文本的形式，浏览器在获取到这种文件时并不会对其进行处理

➹：[text/html和text/plain的区别_空白_回忆的博客-CSDN博客](https://blog.csdn.net/qq_26291823/article/details/50479093)

### 2）`application/xml` vs `text/xml`？

推荐用前者，因为 `application/xml` 实体默认用 `UTF-8` 字符集。

➹：[MIME 类型中，application/xml 与 text/xml 的区别_kikajack的博客-CSDN博客](https://blog.csdn.net/kikajack/article/details/79233017)
