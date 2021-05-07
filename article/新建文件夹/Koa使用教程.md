## 0. 准备

检查 Node 版本，使用 koa 需要使用 7.6 以上的版本

```
node -v
```

复制代码

## 一. 基本使用

### 1.1 搭建 http 服务器

使用 koa 搭建 http 服务器很简单，只需要如下三步，即可

```javascript
//demo01.js
const Koa = require("koa");
const app = new Koa();

app.listen(3000);
```

然后使用 node 命令运行该文件即可

```
node buildHttp.js
```

打开浏览器，我们输入 http://loaclhost:3000 访问，页面显示 Not Found，这是因为我们并没有告诉 Koa 应该显示什么内容

### 1.2 Context 对象

Koa 提供一个 Context 对象，表示一次对话的上下文（包括 http 的请求对象和响应对象），通过操作这个对象，就可以控制返回给用户的内容
`Context.response.body` 属性就是发送给用户的内容

```javascript
//demo02.js
const Koa = require("koa");
const app = new Koa();

var main = (ctx) => {
  ctx.response.body = "hello world";
};

app.use(main);
app.listen(3000);
```

其中 main 方法就是用来设置响应内容，然后通过 app.use 方法使用该函数
使用 node 命令运行该文件，再次访问浏览器

### 1.3 http Response 的类型

Koa 的默认返回类型是 `text/plain`，我们可以通过 `ctx.request.accepts` 判断一下客户希望接收什么类型的数据（根据 HTTP 请求中的 Accept 字段），然后通过 `ctx.response.type` 指定返回值类型

```javascript
// demo03.js

const Koa = require("koa");
const app = new Koa();

const main = (ctx) => {
  if (ctx.request.accepts("xml")) {
    ctx.response.type = "xml";
    ctx.response.body = "<data>Hello World</data>";
  } else if (ctx.request.accepts("json")) {
    ctx.response.type = "json";
    ctx.response.body = { data: "Hello World" };
  } else if (ctx.request.accepts("html")) {
    ctx.response.type = "html";
    ctx.response.body = "<p>Hello World</p>";
  } else {
    ctx.response.type = "text";
    ctx.response.body = "Hello World";
  }
};

app.use(main);
app.listen(3000);
```

### 1.4 网页模版

实际开发过程中，返回给用户网页一般都是写成模版文件，可以让 koa 先获取模版，然后将模版返回给用户

```javascript
// demo04.js
const Koa = require("koa");
const app = new Koa();
const fs = require("fs");

var main = (ctx) => {
  ctx.response.type = "html";
  ctx.response.body = fs.createReadStream("./views/template.html");
};

app.use(main);
app.listen(3000);
```

## 二. 路由

### 2.1 原生路由

原生路由即我们手写的路由，通过对 URL 的判断，返回給用户不同的内容，具体实现如下

```javascript
// demo05.js
const Koa = require("koa");
const app = new Koa();

var main = (ctx) => {
  if (ctx.request.path == "/") {
    ctx.response.type = "html";
    ctx.response.body = "home page";
  } else if (ctx.request.path === "/about") {
    ctx.response.type = "html";
    ctx.response.body = "<h1>about page</h1>";
  } else {
    ctx.response.type = "html";
    ctx.response.body = "<h1>no set url resource</h1>";
  }
};

app.use(main);

app.listen(3000);
```

2.2 koa-router 模块

原生路由用起来不太方便，于是我们封装了 koa-route 模版

```javascript
// demo06.js

const Koa = require("koa");
const app = new Koa();
const route = require("koa-route");

var home = (ctx) => {
  ctx.response.type = "html";
  ctx.response.body = `<a href='#'>Home page</a>`;
};

var about = (ctx) => {
  ctx.response.type = "html";
  ctx.response.body = `<a href='#'>About page</a>`;
};

app.use(route.get("/", home));
app.use(route.get("/about", about));

app.listen(3000);
```

上面代码中，/路径的处理函数是 home，/about 路径的处理函数是 about

### 2.3 静态资源

如果网站提供了静态资源（图片，字体，样式，脚本），为他们一个个写路由就很麻烦，也没必要
koa-static 模块封装了这部分的请求

//demo07.js
const Koa = require('koa')
const app = new Koa()
const serve = require('koa-static')
const path = require('path')

const main = serve(path.join(\_\_dirname))

app.use(main)

app.listen(3000)
复制代码
设置之后可以直接在 URL 地址中输入静态资源文件名进行访问

### 2.4 重定向

有些场合，服务器需要重定向，
`ctx.response.redirect()` 方法可以发出一个 302 跳转，将用户导向另一个路由

```javascript
// demo08.js
const Koa = require("koa");
const app = new Koa();
const route = require("koa-route");

var home = (ctx) => {
  ctx.response.type = "html";
  ctx.response.body = `<a href='#'>Home page</a>`;
};

var about = (ctx) => {
  ctx.response.type = "html";
  ctx.response.body = `<a href='#'>About page</a>`;
};

var redict = (ctx) => {
  ctx.response.redirect("/");
};

app.use(route.get("/", home));
app.use(route.get("/about", about));
app.use(route.get("/redict", redict));

app.listen(3000);
```

## 三，中间件

### 3.1 logger 功能

koa 的最大特色，也是最重要的一个设计，就是中间件。
来看如下一个案例：实现打印日志的功能，

```javascript
// demo09.js
const Koa = require("koa");
const app = new Koa();

var main = (ctx) => {
  console.log(`${Date.now()}--${ctx.request.method}--${ctx.request.url}`);
  ctx.response.body = "hello world";
};

app.use(main);
app.listen(3000);
```

### 3.2 中间件的概念

在如上的案例基础上，我们可以把日志功能抽离出来，行成一个单独的函数
抽离出来的单独的函数，就叫做中间件
**含义：**处于 Http Request 和 Http Response 之间，用来实现某种中间功能，然后使用 app.use 方法来使用中间件
**参数：**每个中间件默认接受两个参数

- ctx：上下文对象
- next：第二个参数是 next 函数。只要调用 next 函数，就可以把执行权交给下一个中间件

```javascript
const logger = (ctx, next) => {
  console.log(`${Date.now()}--${ctx.request.method}--${ctx.request.url}`);
  next();
};

app.use(logger);
```

### 3.3 中间件栈

> 多个中间件会行成一个栈结构，以先进后出的顺序执行

执行顺序如下

```javascript
1. 最外层的中间件首先执行。
2. 调用next函数，把执行权交给下一个中间件
3. ...
4. 最内层的中间件最后执行
5. 执行结束后，把执行权交回上一层的中间件
6. ...
7. 最外层的中间件收回执行权之后，执行next函数之后的代码
```

示例

```javascript
// demo10.js
const Koa = require("koa");
const app = new Koa();

const one = (ctx, next) => {
  console.log(">> one");
  next();
  console.log("<< one");
};

const two = (ctx, next) => {
  console.log(">> two");
  next();
  console.log("<< two");
};

const three = (ctx, next) => {
  console.log(">> three");
  next();
  console.log("<< three");
};

app.use(one);
app.use(two);
app.use(three);

app.listen(3000);
```

运行结果

```javascript
>> one
>> two
>> three
<< three
<< two
<< one
```

### 3.4 异步中间栈

如果有异步操作，中间件就必须写成 async 函数

```javascript
// demo11.js
const Koa = require("koa");
const app = new Koa();
const fs = require("fs");

var main = async function (ctx, next) {
  ctx.response.type = "html";
  ctx.response.body = await fs.readFileSync("./views/template.html", "utf-8");
};

app.use(main);

app.listen(3000);
```

### 3.5 中间件的合成

> koa-compose 模块可以将多个中间件合成为一个

```javascript
// demo12.js
const Koa = require("koa");
const app = new Koa();
const compose = require("koa-compose");

const logger = (ctx, next) => {
  console.log(`${Date.now()}${ctx.request.method}${ctx.request.url}`);
  next();
};

const main = (ctx, next) => {
  ctx.response.body = "hello world";
};

const middleWares = compose([logger, main]);
app.use(middleWares);
app.listen(3000);
```

**注：本文参考阮一峰：**[koa 框架教程](http://www.ruanyifeng.com/blog/2017/08/koa.html)
