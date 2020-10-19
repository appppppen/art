### 跨域背景

同域
同域：协议、域名、端口号均相同

跨域：
跨域是指一个域下的文档或脚本试图去请求另一个域下的资源。

同源策略：
同源策略由 Netscape 公司 1995 年引入浏览器，协助浏览器免受到 XSS、CSFR 等攻击。所谓同源是指"协议+域名+端口"三者相同，即便两个不同的域名指向同一个 ip 地址，也非同源。

```
URL                                      说明                    是否允许通信
http://www.demo.com/a.js
http://www.demo.com/b.js         同一域名，不同文件或路径           允许
http://www.demo.com/lab/c.js

http://www.demo.com:8000/a.js
http://www.demo.com/b.js         同一域名，不同端口                不允许

http://www.demo.com/a.js
https://www.demo.com/b.js        同一域名，不同协议                不允许

http://www.demo.com/a.js
http://127.0.0.1/b.js           域名和域名对应相同ip              不允许

http://www.demo.com/a.js
http://x.demo.com/b.js           主域相同，子域不同                不允许
http://demo.com/c.js

http://www.demo1.com/a.js
http://www.demo2.com/b.js        不同域名                         不允许
```

### 跨域行为

ajax 请求 XMLHttpRequest；

注意：浏览器并不会限制跨域请求的发出，即服务端再未做其他限制的情况下仍可收到跨域请求，浏览器只是限制了 ajax 请求返回信息的读取、cookie 写入等操作
扩展：服务端如何限制仅接收指定域名请求? refer?

- fetch； \*

非同域窗体之间（比如 iframe 引入的情况的父子窗体）不能直接进行数据通讯或共享
Web 字体 (CSS 中通过 @font-face 使用跨域字体资源)

WebGL 贴图；

使用 drawImage 将 Images/video 画面绘制到 canvas

跨域报错：

```javascript
No 'Access-Control-Allow-Origin' header is present on the requested resource
// 并且The response had HTTP status code 404
```

### 解决方案

1. 通过 jsonp 跨域

JSONP 的原理很简单，就是利用 script 标签没有跨域限制的漏洞。通过 script 标签指向一个需要访问的地址并提供一个回调函数来接收数据当需要通讯时。JSONP 使用简单且兼容性不错，但是只限于 get 请求，不可能支持 post 请求。在开发中可能会遇到多个 JSONP 请求的回调函数名是相同的，这时候就需要自己封装一个 JSONP，以下是简单实现：

```javascript
function jsonp(url, jsonpCallback, success) {
  let script = document.createElement("script");
  script.src = url;
  script.async = true;
  script.type = "text/javascript";
  window[jsonpCallback] = function (data) {
    success & success(data);
  };
  document.body.appendChild(script);
}
jsonp("http://xxx", "callback", function (value) {
  console.log(value);
});
```

应用场景：淘宝和天猫通过 jsonp 跨域共享 cookie，在淘宝(www.taobao.com)登录后，切换到天猫(www.tmall.com)，会看到顶栏已经有登录用户信息。打开控制台，刷新 tmall 页面，可以看到如下 jsonp 请求。

```javascript
// Jquery jsonp
$.ajax({
  url: "http://www.domain2.com:8080/login",
  type: "get",
  dataType: "jsonp", // 请求方式为jsonp
  jsonpCallback: "onBack", // 自定义回调函数名
  data: {},
});

// vue 中jsonp用法
this.$http
  .jsonp("http://www.domain2.com:8080/login", {
    params: {},
    jsonp: "onBack",
  })
  .then((res) => {
    console.log(res);
  });
```

优点：简单（不推荐此方法）

缺点：如果动态脚本插入有效，就执行调用；如果无效，就静默失败：不能从服务器捕捉到 404 错误，也不能取消或重新开始请求。（自行设置定时器，超时后没有进入回调即判定为请求失败 ）。

2. CORS

服务端设置 Access-Control-Allow-Origin 即可，前端无须设置，若要带 cookie 请求，前后端都需要设置，CORS 需要浏览器和后端同时支持。IE 8 和 9 需要通过 XDomainRequest 来实现。浏览器会自动进行 CORS 通信，实现 CORS 通信的关键是后端。只要后端实现了 CORS，就实现了跨域。服务端设置 Access-Control-Allow-Origin 就可以开启 CORS。 该属性表示哪些域名可以访问资源，如果设置通配符则表示所有网站都可以访问资源。

```javascript
// 前端设置
// JS
var xhr = new XMLHttpRequest(); // IE8/9需用window.XDomainRequest兼容

// 前端设置是否带cookie
xhr.withCredentials = true;

xhr.open('post', 'http://www.domain2.com:8080/login', true);
xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded');
xhr.send('user=admin');

xhr.onreadystatechange = function() {
    if (xhr.readyState == 4 && xhr.status == 200) {
        alert(xhr.responseText);
    }
};

// 或 jquery
$.ajax({
    ...
   xhrFields: {
       withCredentials: true    // 前端设置是否带cookie
   },
   crossDomain: true,   // 会让请求头中包含跨域的额外信息，但不会含cookie
    ...
});

// 或vue.axios
axios.defaults.withCredentials = true

// 或vue-resource
Vue.http.options.credentials = true

// 服务端设置
// java
/*
 * 导入包：import javax.servlet.http.HttpServletResponse;
 * 接口参数中定义：HttpServletResponse response
 */

// 允许跨域访问的域名：若有端口需写全（协议+域名+端口），若没有端口末尾不用加'/'
response.setHeader("Access-Control-Allow-Origin", "http://www.domain1.com");

// 允许前端带认证cookie：启用此项后，上面的域名不能为'*'，必须指定具体的域名，否则浏览器会提示
response.setHeader("Access-Control-Allow-Credentials", "true");

// 提示OPTIONS预检时，后端需要设置的两个常用自定义头
response.setHeader("Access-Control-Allow-Headers", "Content-Type,X-Requested-With");

// 或node 后端
var http = require('http');
var server = http.createServer();
var qs = require('querystring');

server.on('request', function(req, res) {
    var postData = '';

    // 数据块接收中
    req.addListener('data', function(chunk) {
        postData += chunk;
    });

    // 数据接收完毕
    req.addListener('end', function() {
        postData = qs.parse(postData);

        // 跨域后台设置
        res.writeHead(200, {
            'Access-Control-Allow-Credentials': 'true',     // 后端允许发送Cookie
            'Access-Control-Allow-Origin': 'http://www.domain1.com',    // 允许访问的域（协议+域名+端口）
            /*
             * 此处设置的cookie还是domain2的而非domain1，因为后端也不能跨域写cookie(nginx反向代理可以实现)，
             * 但只要domain2中写入一次cookie认证，后面的跨域接口都能从domain2中获取cookie，从而实现所有的接口都能跨域访问
             */
            'Set-Cookie': 'l=a123456;Path=/;Domain=www.domain2.com;HttpOnly'  // HttpOnly的作用是让js无法读取cookie
        });

        res.write(JSON.stringify(postData));
        res.end();
    });
});

server.listen('8080');
console.log('Server is running at port 8080...');
```

优点：所有浏览器都支持该功能(IE8+：IE8/9 需要使用 XDomainRequest 对象来支持 CORS）)，CORS 也已经成为主流的跨域解决方案。

缺点: CORS 可以使用 XDomainRequest 来支持 IE8/9。但是，并不支持带上 cookies。所以在需要兼容 IE8/9 的情况下，CORS 用不了。

3. document.domain + iframe 跨域
   该方式只能用于二级域名相同的情况下，比如 a.test.com 和 b.test.com 适用于该方式。只需要给页面添加 document.domain = 'test.com' 表示二级域名都相同就可以实现跨域。

```javascript
// 父级(http://www.domain.com/a.html)
<iframe id="iframe" src="http://child.domain.com/b.html"></iframe>
<script>
    document.domain = 'domain.com';
    var user = 'admin';
</script>

// 子级(http://child.domain.com/b.html)
<script>
    document.domain = 'domain.com';
    // 获取父窗口中变量
    alert('get js data from parent ---> ' + window.parent.user);
</script>
```

优点：同类业务场景内跨域方便快捷。

缺点: 仅限主域相同，子域不同的跨域应用场景，适用范围小。

4. location.hash + iframe

该方案是依赖消息通信机制实现，原理是在目标服务器放置一个代理文件(proxy_frame.html)，通过加载该代理文件和服务端进行数据交互（同域请求），返回数据通过消息通讯(如 post message)返回给上层应用以实现跨域数据交互 a.b.com 域页面。实际上是利用窗体之间通讯方式 将跨域请求转化为同域请求。

```javascript
// 父级(http://www.domain1.com/a.html)
<iframe id="iframe" src="http://www.domain2.com/b.html" style="display:none;"></iframe>
<script>
    var iframe = document.getElementById('iframe');

    // 向b.html传hash值
    setTimeout(function() {
        iframe.src = iframe.src + '#user=admin';
    }, 1000);

    // 开放给同域c.html的回调方法
    function onCallback(res) {
        alert('data from c.html ---> ' + res);
    }
</script>

// 子级(http://www.domain2.com/b.html)
<iframe id="iframe" src="http://www.domain1.com/c.html" style="display:none;"></iframe>
<script>
    var iframe = document.getElementById('iframe');

    // 监听a.html传来的hash值，再传给c.html
    window.onhashchange = function () {
        iframe.src = iframe.src + location.hash;
    };
</script>
```

优点：利用 iframe 的 location.hash 传值，相同域之间直接 js 访问来通信。

缺点：适用范围小。

5. window.name + iframe 跨域

该方案也是依赖消息通信机制实现，window.name 属性的神奇之处在于 name 值在不同的页面（甚至不同域名）加载后依旧存在（如果没修改则值不会变化），并且可以支持非常长的 name 值（2MB）。

```javascript
/ a 页面(http://www.domain1.com/a.html)
var proxy = function(url, callback) {
    var state = 0;
    var iframe = document.createElement('iframe');

    // 加载跨域页面
    iframe.src = url;

    // onload事件会触发2次，第1次加载跨域页，并留存数据于window.name
    iframe.onload = function() {
        if (state === 1) {
            // 第2次onload(同域proxy页)成功后，读取同域window.name中数据
            callback(iframe.contentWindow.name);
            destoryFrame();

        } else if (state === 0) {
            // 第1次onload(跨域页)成功后，切换到同域代理页面
            iframe.contentWindow.location = 'http://www.domain1.com/proxy.html';
            state = 1;
        }
    };

    document.body.appendChild(iframe);

    // 获取数据以后销毁这个iframe，释放内存；这也保证了安全（不被其他域frame js访问）
    function destoryFrame() {
        iframe.contentWindow.document.write('');
        iframe.contentWindow.close();
        document.body.removeChild(iframe);
    }
};

// 请求跨域b页面数据
proxy('http://www.domain2.com/b.html', function(data){
    alert(data);
});

// proxy 页面(http://www.domain1.com/proxy....
// 中间代理页，与a.html同域，内容为空即可
// proxy代理页面可以没有这个文件，会报404但是不影响功能（但是路径一定要和index页面同源）。

// b 页面(http://www.domain2.com/b.html)
<script>
    window.name = 'This is domain2 data!';
</script>
```

优点：通过 iframe 的 src 属性由外域转向本地域，跨域数据即由 iframe 的 window.name 从外域传递到本地域。这个就巧妙地绕过了浏览器的跨域访问限制，但同时它又是安全操作。

缺点：iframe 在现实中的表现是一直不停地刷新，每次触发 onload 时间后，重置 src，相当于重新载入页面，又触发 onload 事件，于是就不停地刷新了。另外还有 IE 兼容问题。

6. postMessage 跨域

该方案也是依赖消息通信机制实现，这种方式通常用于获取嵌入页面中的第三方页面数据。一个页面发送消息，另一个页面判断来源并接收消息。

```javascript
// 发送消息端
window.parent.postMessage("message", "http://test.com");
// 接收消息端
var mc = new MessageChannel();
mc.addEventListener("message", (event) => {
  var origin = event.origin || event.originalEvent.origin;
  if (origin === "http://test.com") {
    console.log("验证通过");
  }
});
```

优点：postMessage 是 HTML5 XMLHttpRequest Level 2 中的 API，且是为数不多可以跨域操作的 window 属性之一。

缺点: 部分浏览器只支持字符串，所以传参时最好用 JSON.stringify()序列化。

7. nginx 代理跨域

将本域服务端配置成 需要跨域获取的资源的 反向代理服务器，比如：使用 Nginx 配置请求转发：proxy_pass。此时可在 nginx 的静态资源服务器中加入以下配置：

```javascript
location / {
  add_header Access-Control-Allow-Origin *;
}
```

nginx 反向代理接口跨域，通过 nginx 配置一个代理服务器（域名与 domain1 相同，端口不同）做跳板机，反向代理访问 domain2 接口，并且可以顺便修改 cookie 中 domain 信息，方便当前域 cookie 写入，实现跨域登录。

```javascript
// nginx 配置
#proxy服务器
server {
    listen       81;
    server_name  www.domain1.com;

    location / {
        proxy_pass   http://www.domain2.com:8080;  #反向代理
        proxy_cookie_domain www.domain2.com www.domain1.com; #修改cookie里域名
        index  index.html index.htm;

        # 当用webpack-dev-server等中间件代理接口访问nignx时，此时无浏览器参与，故没有同源限制，下面的跨域配置可不启用
        add_header Access-Control-Allow-Origin http://www.domain1.com;  #当前端只跨域不带cookie时，可为*
        add_header Access-Control-Allow-Credentials true;
    }
}

// 前端代码
var xhr = new XMLHttpRequest();

// 前端开关：浏览器是否读写cookie
xhr.withCredentials = true;

// 访问nginx中的代理服务器
xhr.open('get', 'http://www.domain1.com:81/?user=admin', true);
xhr.send();

// node 代码
var http = require('http');
var server = http.createServer();
var qs = require('querystring');

server.on('request', function(req, res) {
    var params = qs.parse(req.url.substring(2));

    // 向前台写cookie
    res.writeHead(200, {
        'Set-Cookie': 'l=a123456;Path=/;Domain=www.domain2.com;HttpOnly'   // HttpOnly:脚本无法读取
    });

    res.write(JSON.stringify(params));
    res.end();
});

server.listen('8080');
console.log('Server is running at port 8080...');

```

应用场景：nginx 反向代理代理的是服务器，正向代理代理的是客户端。

优点：反向代理，负载均衡，占有内存少，并发能力强。

缺点：无。

8. nodejs 中间件代理跨域

node 中间件实现跨域代理，原理大致与 nginx 相同，都是通过启一个代理服务器，实现数据的转发，也可以通过设置 cookieDomainRewrite 参数修改响应头中 cookie 中域名，实现当前域的 cookie 写入，方便接口登录认证。

非 vue 2 次跨域
利用 node + express + http-proxy-middleware 搭建一个 proxy 服务器。

```javascript
// 前端代码
var xhr = new XMLHttpRequest();

// 前端开关：浏览器是否读写cookie
xhr.withCredentials = true;

// 访问http-proxy-middleware代理服务器
xhr.open("get", "http://www.domain1.com:3000/login?user=admin", true);
xhr.send();

// 中间件服务器
var express = require("express");
var proxy = require("http-proxy-middleware");
var app = express();

app.use(
  "/",
  proxy({
    // 代理跨域目标接口
    target: "http://www.domain2.com:8080",
    changeOrigin: true,

    // 修改响应头信息，实现跨域并允许带cookie
    onProxyRes: function (proxyRes, req, res) {
      res.header("Access-Control-Allow-Origin", "http://www.domain1.com");
      res.header("Access-Control-Allow-Credentials", "true");
    },

    // 修改响应信息中的cookie域名
    cookieDomainRewrite: "www.domain1.com", // 可以为false，表示不修改
  })
);

app.listen(3000);
console.log("Proxy server is listen at port 3000...");
```

_vue 框架 1 次跨域_

利用 node + webpack + webpack-dev-server 代理接口跨域。在开发环境下，由于 vue 渲染服务和接口代理服务都是 webpack-dev-server 同一个，所以页面与代理接口之间不再跨域，无须设置 headers 跨域信息了。

```javascript
// webpack.config.js
module.exports = {
    entry: {},
    module: {},
    ...
    devServer: {
        historyApiFallback: true,
        proxy: [{
            context: '/login',
            target: 'http://www.domain2.com:8080',  // 代理跨域目标接口
            changeOrigin: true,
            secure: false,  // 当代理某些https服务报错时用
            cookieDomainRewrite: 'www.domain1.com'  // 可以为false，表示不修改
        }],
        noInfo: true
    }
}
```

9. WebSocket 协议跨域

WebSocket protocol 是 HTML5 一种新的协议。它实现了浏览器与服务器全双工通信，同时允许跨域通讯，但建立 socket 长连接，需要验证，本质上可以视为安全，不存在跨域限制,由于资源消耗较大，除了一些特殊场景，一般不使用。

```javascript
// 前端代码
<div>user input：<input type="text"></div>
<script src="./socket.io.js"></script>
<script>
var socket = io('http://www.domain2.com:8080');

// 连接成功处理
socket.on('connect', function() {
    // 监听服务端消息
    socket.on('message', function(msg) {
        console.log('data from server: ---> ' + msg);
    });

    // 监听服务端关闭
    socket.on('disconnect', function() {
        console.log('Server socket has closed.');
    });
});

document.getElementsByTagName('input')[0].onblur = function() {
    socket.send(this.value);
};
</script>

// Nodejs socket
var http = require('http');
var socket = require('socket.io');

// 启http服务
var server = http.createServer(function(req, res) {
    res.writeHead(200, {
        'Content-type': 'text/html'
    });
    res.end();
});

server.listen('8080');
console.log('Server is running at port 8080...');

// 监听socket连接
socket.listen(server).on('connection', function(client) {
    // 接收信息
    client.on('message', function(msg) {
        client.send('hello：' + msg);
        console.log('data from client: ---> ' + msg);
    });

    // 断开处理
    client.on('disconnect', function() {
        console.log('Client socket has closed.');
    });
});
```

10. Flash 代理跨域

与 frame 代理模式类似，请求通过 Flash 来发送 ( proxy_flash.swf 放置在同源站)，利用 Flash 的策略文件 crossdomain.xml 来控制资源的共享权限，获取目标服务器请求返回数据---相当于把 iframe 改成 flash 。

前提条件：你必须有权限管理所跨域的那片域上的文件，因为需要放置一个通行文件。如果你是跨域调图片、视频一类的，可以用通行文件的方法。通行文件制作方法，请将以下代码存为 crossdomain.xml ，并放到要跨域的目标站点根目录下面。就是说，你的 FLASH 在 a.com ，你要访问 b.com 上的资源，你就要确定 http://b.com/crossdomain.xml 能访问到。

```xml
<?xml version="1.0"?>
<cross-domain-policy>
<allow-access-from domain="*" />
</cross-domain-policy>
```

上面是允许全部网站调用，如果要限制某个网站可以用下面的：

```xml
<?xml version="1.0"?>
<!DOCTYPE cross-domain-policy SYSTEM "http://www.macromedia.com/xml/dtds/cross-domain-policy.dtd">
<cross-domain-policy>
<allow-access-from domain="www.mayax.net" />
<allow-access-from domain="*.vkcms.com" />
</cross-domain-policy>

```

上面代码中，允许了 www.mayax.net 进行跨域调用。但 bbs.mayax.net 不行；同时也允许了 vkcms.com 进行跨域， bbs.vkcms.com 也可以， vkflash.vkcms.com 也行！但，别的如 zhi.in 、 www.zhoujingsong.com 都不能跨域访问了。
　　如果你跨域访问 JS，或给跨域的 JS 传数据，那就要在通行文件的基础上，再作如下处理。如果你的 FLASH 文件要给 JS 文件传数据，调用 Flash 的 SWF 文件的 HTML 代码，要加上这一行参数：

```xml
<param name="allowScriptAccess" value="always" />
```

参数说明：always 允许随时执行脚本操作；never 禁止所有脚本执行操作。如果你的 FLASH 是读取 JS 发送的数据，那就在 FLASH 的 AS 代码中，加上：

```javascript
System.security.allowDomain("*");
```

还有一个常见问题，提示 import flash.display.BitmapData 失效，把下面这行代码加在 AS 里面就可以了：

```javascript
System.security.loadPolicyFile(http://yoursite.com/crossdomain.xml);
```

crossdomain.xml 的内容同上。
