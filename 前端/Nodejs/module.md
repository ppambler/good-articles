# module

## 1. 模块化

模块化是指把一个复杂的系统分解到多个模块以方便编码。

> 把一块大豆腐切成多块小豆腐以便烹煮

### 1.1 命名空间

开发网页要通过命名空间的方式来组织代码

```js
<script src="jquery.js">
```

- 命名空间冲突，两个库可能会使用同一个名称
- 无法合理地管理项目的依赖和版本；
- 无法方便地控制依赖的加载顺序。

> 需要包管理工具、需要像`import/export`这样的语法

### 1.2 CommonJS

CommonJS 是一种使用广泛的`JavaScript`模块化规范，核心思想是通过`require`方法来**同步**地加载依赖的其他模块，通过 `module.exports` 导出需要暴露的接口。

#### 1.2.1 用法

采用 CommonJS 导入及导出时的代码如下：

```js
// 导入
const someFun= require('./moduleA');
someFun();

// 导出
module.exports = someFunc;
```

#### 1.2.2 原理

a.js

```js
let fs = require("fs");
let path = require("path");
let b = req("./b.js");
function req(mod) {
  // __dirname -> e:\blogdemo\demo\02-珠峰架构、module
  // filename -> e:\blogdemo\demo\02-珠峰架构、module\b.js
  let filename = path.join(__dirname, mod);
  // console.log('bbb');
  // exports.name = 'zfpx';
  let content = fs.readFileSync(filename, "utf8");
  let fn = new Function(
    "exports",
    "require",
    "module",
    "__filename",
    "__dirname",
    content + "\n return module.exports;"
  );
  let module = {
    exports: {},
  };

  return fn(module.exports, req, module, __filename, __dirname);
}
```

b.js

```js
console.log('bbb');
exports.name = 'b';
```

![image-20210406012535965](https://gitee.com/ppambler/blog-images/raw/master/images/20210406012536.png)

### 1.3 AMD

> 文档：[Why Web Modules?](https://requirejs.org/docs/why.html)

AMD（Asynchronous Module Definition） 也是一种 JavaScript 模块化规范，与 CommonJS 最大的不同在于它采用**异步**的方式去加载依赖的模块。 AMD 规范主要是为了解决针对浏览器环境的模块化问题，最具代表性的实现是 requirejs。

> 在浏览器中同步加载脚本会影响性能，而在服务端，由于脚本是本地的，所以同步加载对性能的影响并不大！

AMD 的优点：

- 可在不转换代码的情况下直接在浏览器中运行
- 可加载多个依赖
- 代码可运行在浏览器环境和 Node.js 环境下

AMD 的缺点：

- JavaScript 运行环境没有原生支持 AMD，需要先导入实现了 AMD 的库后才能正常使用。

#### 1.3.1 用法

安装：

```bash
yarn add requirejs
```

在浏览器端使用：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta http-equiv="X-UA-Compatible" content="IE=edge">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Document</title>
  <script data-main="main" src="/node_modules/requirejs/require.js"></script>
</head>
<body>
  <h1>Test AMD</h1>
</body>
</html>
```

```js
// main.js
require(["a", "b"], function () {
  // a、b 都是全局变量
  a();
  b();
});
console.log(1);

// a.js
function a(){
  console.log("aaaaaaaaaaaaaaaaaaaaaaaaaaa");
}

// b.js
function b(){
  console.log("bbbbbbbbbbbbbbbbbbbbb");
}
```

![image-20210406100841501](https://gitee.com/ppambler/blog-images/raw/master/images/20210406100841.png)

> 浏览器端也是用`define`来定义一个模块

在服务端使用：

```js
// main.js -> 执行它
var requirejs = require("requirejs");

requirejs.config({
  //Pass the top-level main.js/index.js require
  //function to requirejs so that node modules
  //are loaded relative to the top-level JS file.
  nodeRequire: require,
});

// foo -> a 模块的返回值，同理，bar 是 b 模块的返回值，c 是 c 模块的返回值
requirejs(["a", "b", "c"], function (foo, bar, c) {
  //foo and bar are loaded according to requirejs
  //config, but if not found, then node's require
  //is used to load the module.
  console.log(foo);
  console.log(bar);
  console.log(c)
});

// a.js
// 定义一个模块
define('a', [], function () {
  return 'a';
});

// b.js
define('b', ['a'], function (a) {
  return a + 'b';
});

// c.js
// require 已经不是 CommonJS 规范那个 require 了
// 导入和使用
require(["a"],(a)=>{
 console.log(a)
})
// 定义一个模块
define('c',[],()=>{
  return 'c'
})

// 输出结果 -> a ab c a
```

➹：[RequireJS 异步模块加载 - 粉丝日志](http://blog.fens.me/nodejs-requirejs/)

➹：[RequireJS in Node](https://requirejs.org/docs/node.html)

#### 1.3.2 原理

```js
let factories = {};
function define(modName, dependencies, factory) {
  factory.dependencies = dependencies;
  factories[modName] = factory;
}
function require(modNames, callback) {
  let loadedModNames = modNames.map(function (modName) {
    let factory = factories[modName];
    let dependencies = factory.dependencies;
    let exports;
    require(dependencies, function (...dependencyMods) {
      exports = factory.apply(null, dependencyMods);
    });
    return exports;
  });
  callback.apply(null, loadedModNames);
}
```

> 嗯……看不懂

### 1.4 ES6 模块化

ES6 模块化是`ECMA`提出的`JavaScript`模块化规范，它在语言的层面上实现了模块化。浏览器厂商和`Node.js` 都宣布要原生支持该规范。它将逐渐取代`CommonJS`和`AMD`规范，成为浏览器和服务器通用的模块解决方案。 采用 ES6 模块化导入及导出时的代码如下：

```js
// 导入
import { name } from './person.js';

// 导出
export const name = 'Jack';
```

ES6 模块虽然是终极模块化方案，但它的缺点在于目前无法直接运行在大部分 JavaScript 运行环境下，必须通过工具转换成标准的 ES5 后才能正常运行。

## 2. 自动化构建

构建就是做这件事情，把源代码转换成发布到线上的可执行 JavaScrip、CSS、HTML 代码，包括如下内容。

- 代码转换：ES6 编译成 ES5、LESS 编译成 CSS 等。
- 文件优化：压缩 JavaScript、CSS、HTML 代码，压缩合并图片等。
- 代码分割：提取多个页面的公共代码、提取首屏不需要执行部分的代码让其异步加载。
- 模块合并：在采用模块化的项目里会有很多个模块和文件，需要构建功能把模块分类合并成一个文件。
- 自动刷新：监听本地源代码的变化，自动重新构建、刷新浏览器。
- 代码校验：在代码被提交到仓库前需要校验代码是否符合规范，以及单元测试是否通过。
- 自动发布：更新完代码后，自动构建出线上发布代码并传输给发布系统。

> 你做了这件事儿，就被称之为「xxx」

## 3. Webpack

Webpack 是一个打包模块化 JavaScript 的工具，在 Webpack 里一切文件皆模块，通过 Loader 转换文件，通过 Plugin 注入钩子，最后输出由多个模块组合成的文件。Webpack 专注于构建模块化项目。

一切文件：JavaScript、CSS、SCSS、图片、模板，在 Webpack 眼中都是一个个模块，这样的好处是能清晰的描述出各个模块之间的依赖关系，以方便 Webpack 对模块进行组合和打包。 经过 Webpack 的处理，最终会输出浏览器能使用的**静态资源**。

### 3.1 安装 Webpack

在用 Webpack 执行构建任务时需要通过「webpack 可执行文件」去启动构建任务，所以需要安装 「webpack 可执行文件」

#### 3.1.1 安装 Webpack 到本项目

> 文档：[安装 - webpack 中文文档](https://webpack.docschina.org/guides/installation/)

```bash
# 查看 webpack 所有版本
npm view webpack versions

# 查看 webpack 的最行版本
npm view webpack version

# 查看 webpack 的详细信息 -> 告诉你如果安装 webpack2、webpack3、webpack4 会安装哪个版本
npm info webpack
 
# 安装最新稳定版
npm i -D webpack

# 安装指定版本
npm i -D webpack@<version>

# 安装最新体验版本 -> 可能仍然包含 bug，因此不应该用于生产环境
npm i -D webpack@next

# 或特定的 tag/分支
npm i -D webpack/webpack#<tagname/branchname>
```

```json
{
  "scripts": {
  	"build": "webpack --config webpack.config.js"
  }
}
```

💡：`-D`的意思？

npm i -D 是 `npm install --save-dev` 的简写，是指安装模块并保存到 `package.json` 的 `devDependencies`

- `"dependencies"`: Packages required by your application in production.
- `"devDependencies"`: Packages that are only needed for local development and testing.

➹：[Specifying dependencies and devDependencies in a package.json file - npm Docs](https://docs.npmjs.com/specifying-dependencies-and-devdependencies-in-a-package-json-file)

#### 3.1.2 安装 Webpack 到全局

安装到全局后你可以在任何地方共用一个 Webpack 可执行文件，而不用各个项目重复安装

```bash
npm i -g webpack
```

> 推荐安装到当前项目，原因是可防止不同项目依赖不同版本的 Webpack 而导致冲突

其实，官方 **不推荐** 全局安装 webpack。因为这会将你项目中的 webpack 锁定到指定版本，并且在使用不同的 webpack 版本的项目中， 可能会导致构建失败。

### 3.2 使用 Webpack

> 没看懂以下这些代码是在干嘛 -> 难道这就是 webpack 打包的本质？

```js
(function (modules) {
  function require(moduleId) {
    var module = {
      exports: {},
    };
    modules[moduleId].call(module.exports, module, module.exports, require);
    return module.exports;
  }
  return require("./index.js");
})({
  "./index.js": function (module, exports, require) {
    eval("console.log('hello');\n\n");
  },
});
```

```js
#! /usr/bin/env node
const pathLib = require("path");
const fs = require("fs");
let ejs = require("ejs");
let cwd = process.cwd();
let {
  entry,
  output: { filename, path },
} = require(pathLib.join(cwd, "./webpack.config.js"));
let script = fs.readFileSync(entry, "utf8");
let bundle = `
(function (modules) {
    function require(moduleId) {
      var module = {
          exports: {}
      };
      modules[moduleId].call(module.exports, module, module.exports, require);
      return module.exports;

    }
    return require("<%-entry%>");
})
    ({
      "<%-entry%>":
        (function (module, exports, require) {
            eval("<%-script%>");
        })
    });
`;
let bundlejs = ejs.render(bundle, {
  entry,
  script,
});
try {
  fs.writeFileSync(pathLib.join(path, filename), bundlejs);
} catch (e) {
  console.error("编译失败 ", e);
}
console.log("compile sucessfully!");
```

#### 3.2.1 依赖其它模块

```js
#! /usr/bin/env node
const pathLib = require("path");
const fs = require("fs");
let ejs = require("ejs");
let cwd = process.cwd();
let {
  entry,
  output: { filename, path },
} = require(pathLib.join(cwd, "./webpack.config.js"));
let script = fs.readFileSync(entry, "utf8");
let modules = [];
script.replace(/require\(['"](.+?)['"]\)/g, function () {
  let name = arguments[1];
  let script = fs.readFileSync(name, "utf8");
  modules.push({
    name,
    script,
  });
});
let bundle = `
(function (modules) {
    function require(moduleId) {
      var module = {
          exports: {}
      };
      modules[moduleId].call(module.exports, module, module.exports, require);
      return module.exports;
    }
    return require("<%-entry%>");
})
    ({
      "<%-entry%>":
          (function (module, exports, require) {
              eval(\`<%-script%>\`);
          })
      <%if(modules.length>0){%>,<%}%>
      <%for(let i=0;i<modules.length;i++){
          let module = modules[i];%>   
          "<%-module.name%>":
          (function (module, exports, require) {
              eval(\`<%-module.script%>\`);
          })
      <% }%>    
    });
`;
let bundlejs = ejs.render(bundle, {
  entry,
  script,
  modules,
});
try {
  fs.writeFileSync(pathLib.join(path, filename), bundlejs);
} catch (e) {
  console.error("编译失败 ", e);
}
console.log("compile sucessfully!");
```
