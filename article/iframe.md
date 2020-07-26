原文链接：[segmentfault.com](https://segmentfault.com/a/1190000004502619)

### iframe, 我们来谈一谈

某大咖说: "iframe 是能耗最高的一个元素，请尽量减少使用"
某大牛说: "iframe 安全性太差，请尽量减少使用"
...
wtf, 你们知不知道你们这样浇灭了多少孩纸学习 iframe 的热情和决心。 虽然，你们这样说的我竟无法反驳，但是 iframe 强大功能是不容忽视的。 可以看看各大邮箱网站是否还在使用 iframe，查查知乎[iframe](https://www.zhihu.com/question/20653055). iframe 不死，我心不灭。现在给大家安利一下 iframe.

### iframe 基本内涵

通常我们使用 iframe 直接直接在页面嵌套 iframe 标签指定 src 就可以了。

```xml
<iframe src="demo_iframe_sandbox.htm"></iframe>
```

但是，有追求的我们，并不是想要这么 low 的 iframe. 我们来看看在 iframe 中还可以设置些什么属性

iframe 常用属性:
1.frameborder:是否显示边框，1(yes),0(no)

2.height:框架作为一个普通元素的高度，建议在使用 css 设置。

3.width:框架作为一个普通元素的宽度，建议使用 css 设置。

4.name:框架的名称，window.frames[name]时专用的属性。

5.scrolling:框架的是否滚动。yes,no,auto。

6.src：内框架的地址，可以使页面地址，也可以是图片的地址。
7.srcdoc , 用来替代原来 HTML body 里面的内容。但是 IE 不支持, 不过也没什么卵用
8.sandbox: 对 iframe 进行一些列限制，IE10+支持

上面一些 tag，会在下文进行穿插说明，单个不好说。
我们通常使用 iframe 最基本的特性，就是能自由操作 iframe 和父框架的内容(DOM). 但前提条件是同域. 如果跨域顶多只能实现页面跳转 window.location.href.
那什么是同域/ 什么是跨域呢?
就是判断你的 url 首部是否一样，下面会有讲解，这里只是提及。
同域不同域的问题:

```xml
A:<iframe id="mainIframe" name="mainIframe" src="/main.html" frameborder="0" scrolling="auto" ></iframe>

B:<iframe id="mainIframe" name="mainIframe" src="http://www.baidu.com" frameborder="0" scrolling="auto" ></iframe>
```

使用 A 时，因为同域，父页面可以对子页面进行改写,反之亦然。
使用 B 时，不同域，父页面没有权限改动子页面,但可以实现页面的跳转
这里，我们先从简单的开始，当主页面和 iframe 同域时，我们可以干 些什么。

### 获取 iframe 里的内容

主要的两个 API 就是 contentWindow,和 contentDocument
iframe.contentWindow, 获取 iframe 的 window 对象
iframe.contentDocument, 获取 iframe 的 document 对象
这两个 API 只是 DOM 节点提供的方式(即 getELement 系列对象)

```javascript
var iframe = document.getElementById("iframe1");
var iwindow = iframe.contentWindow;
var idoc = iwindow.document;
console.log("window", iwindow); //获取iframe的window对象
console.log("document", idoc); //获取iframe的document
console.log("html", idoc.documentElement); //获取iframe的html
console.log("head", idoc.head); //获取head
console.log("body", idoc.body); //获取body
```

实际情况如:
![](https://user-gold-cdn.xitu.io/2017/2/10/c7f5019177486e2f22ce829f4e314a62?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

另外更简单的方式是，结合 Name 属性，通过 window 提供的 frames 获取.

```xml
<iframe src ="/index.html" id="ifr1" name="ifr1" scrolling="yes">
  <p>Your browser does not support iframes.</p>
</iframe>
<script type="text/javascript">
    console.log(window.frames['ifr1'].window);
console.dir(document.getElementById("ifr1").contentWindow);
</script>
```

其实 window.frames['ifr1']返回的就是 window 对象，即

```javascript
window.frames["ifr1"] === window;
```

这里就看你想用哪一种方式获取 window 对象，两者都行，不过本人更倾向于第二种使用 frames[xxx].因为，字母少啊喂~ 然后，你就可以操控 iframe 里面的 DOM 内容。

### 在 iframe 中获取父级内容

同理，在同域下，父页面可以获取子 iframe 的内容，那么子 iframe 同样也能操作父页面内容。在 iframe 中，可以通过在 window 上挂载的几个 API 进行获取.

window.parent 获取上一级的 window 对象，如果还是 iframe 则是该 iframe 的 window 对象
window.top 获取最顶级容器的 window 对象，即，就是你打开页面的文档
window.self 返回自身 window 的引用。可以理解 window===window.self(脑残)

如图:

![](https://user-gold-cdn.xitu.io/2017/2/10/ce7c43f401f2a731c288d87dd94a74b8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

ok, 获取了之后，我们就可以进行相关操作了。 在同域的 iframe 中，我们可以巧妙的使用 iframe 的黑科技来实现一些 trick.

### iframe 的轮询

话说在很久很久以前，我们实现异步发送请求是使用 iframe 实现的~!
怎么可能!!!
真的史料为证(自行 google), 那时候为了不跳转页面，提交表单时是使用 iframe 提交的。现在，前端发展尼玛真快，websocket,SSE,ajax 等，逆天 skill 的出现，颠覆了 iframe, 现在基本上只能活在 IE8,9 的浏览器内了。 但是，宝宝以为这样就可以不用了解 iframe 了,而现实就是这么残酷，我们目前还需要兼容 IE8+。所以，iframe 实现长轮询和长连接的 trick 我们还是需要涉猎滴。

#### iframe 长轮询

如果写过 ajax 的童鞋，应该知道，长轮询就是在 ajax 的 readyState = 4 的时，再次执行原函数即可。 这里使用 iframe 也是一样，异步创建 iframe，然后 reload, 和后台协商好, 看后台哥哥们将返回的信息放在,然后获取里面信息即可. 这里是直接放在 body 里.

```javascript
var iframeCon = docuemnt.querySelector('#container'),
        text; //传递的信息
    var iframe = document.createElement('iframe'),
        iframe.id = "frame",
        iframe.style = "display:none;",
        iframe.name="polling",
        iframe.src="target.html";
    iframeCon.appendChild(iframe);
    iframe.onload= function(){
        var iloc = iframe.contentWindow.location,
            idoc  = iframe.contentDocument;
        setTimeout(function(){
            text = idoc.getElementsByTagName('body')[0].textContent;
            console.log(text);
            iloc.reload(); //刷新页面,再次获取信息，并且会触发onload函数
        },2000);
    }
```

这样就可以实现 ajax 的长轮询的效果。 当然，这里只是使用 reload 进行获取，你也可以添加 iframe 和删除 iframe 的方式，进行发送信息，这些都是根据具体场景应用的。另外在 iframe 中还可以实现异步加载 js 文件，不过，iframe 和主页是共享连接池的，所以还是很蛋疼的，现在基本上都被 XHR 和 hard calllback 取缔了，这里也不过多介绍了。

### 自适应 iframe 之蜜汁广告

网页为了赚钱，引入广告是很正常的事了。通常的做法就是使用 iframe，引入广告地址就可以了，然后根据广告内容设置相应的显示框。但是，为什么是使用 iframe 来进行设置，而不是在某个 div 下嵌套就行了呢？

要知道，广告是与原文无关的，这样硬编码进去，会造成网页布局的紊乱,而且,这样势必需要引入额外的 css 和 js 文件，极大的降低了网页的安全性。 这些所有的弊端，都可以使用 iframe 进行解决。

**我们通常可以将 iframe 理解为一个沙盒，里面的内容能够被 top window 完全控制，而且，主页的 css 样式是不会入侵 iframe 里面的样式，** 这些都给 iframe 的广告命运埋下伏笔。可以看一下各大站点是否都在某处放了些广告，以维持生计比如: [W3School](http://www.w3school.com.cn/js/js_htmldom_elements.asp)
但，默认情况下，iframe 是不适合做展示信息的，所以我们需要对其进行 decorate.

#### 自适应 iframe

默认情况下，iframe 会自带滚动条，不会全屏.如果你想自适应 iframe 的话:
第一步：去掉滚动条

```xml
<iframe src="./iframe1.html" id="iframe1" scrolling="no"></iframe>
```

第二步,设置 iframe 的高为 body 的高

```java
var iwindow = iframe.contentWindow;
var idoc = iwindow.document;
iframe.height = idoc.body.offsetHeight;

```

另外,还可以添加其它的装饰属性:
属性| 效果
--|--
allowtransparency |true or false 是否允许 iframe 设置为透明，默认为 false
allowfullscreen |true or false 是否允许 iframe 全屏，默认为 false
看个例子:

```xml
<iframe id="google_ads_frame2" name="google_ads_frame2" width="160" height="600" frameborder="0" src="target.html" marginwidth="0" marginheight="0" vspace="0" hspace="0" allowtransparency="true" scrolling="no" allowfullscreen="true"></iframe>
```

你可以直接写在内联里面，也可以在 css 里面定义，不过对于广告 iframe 来说，样式写在属性中，是 best pratice.

### iframe 安全性探索

iframe 出现安全性有两个方面，一个是你的网页被别人 iframe,一个是你 iframe 别人的网页。 当你的网页被别人 iframe 时，比较蛋疼的是被钓鱼网站利用，然后 victim+s 留言逼逼你。真.简直了。 所以为了防止页面被一些不法分子利用，我们需要做好安全性措施。

### 防嵌套网页

比如，最出名的 clickhacking 就是使用 iframe 来 拦截 click 事件。因为 iframe 享有着 click 的最优先权，当有人在伪造的主页中进行点击的话，如果点在 iframe 上，则会默认是在操作 iframe 的页面。 所以，钓鱼网站就是使用这个技术，通过诱导用户进行点击，比如，设计一个"妹妹寂寞了"等之类的网页，诱导用户点击，但实际结果，你看到的不是"妹妹",而是被恶意微博吸粉。
所以，为了防止网站被钓鱼，可以使用 window.top 来防止你的网页被 iframe.

```javascript
//iframe2.html
if (window != window.top) {
  window.top.location.href = correctURL;
}
```

这段代码的主要用途是限定你的网页不能嵌套在任意网页内。
如果你想引用同域的框架的话，可以判断域名。

```javascript
if (top.location.host != window.location.host) {
  top.location.href = window.location.href;
}
```

当然，如果你网页不同域名的话，上述就会报错。
所以，这里可以使用 try...catch...进行错误捕获。如果发生错误，则说明不同域，表示你的页面被盗用了。可能有些浏览器这样写是不会报错，所以需要降级处理。
这时候再进行跳转即可.

```javascript
try {
  top.location.hostname; //检测是否出错 //如果没有出错，则降级处理
  if (top.location.hostname != window.location.hostname) {
    top.location.href = window.location.href;
  }
} catch (e) {
  top.location.href = window.location.href;
}
```

这只是浏览器端，对 iframe 页面的权限做出相关的设置。
我们还可以在服务器上，对使用 iframe 的权限进行设置.

#### X-Frame-Options

X-Frame-Options 是一个相应头，主要是描述服务器的网页资源的 iframe 权限。目前的支持度是 IE8+(已经很好了啊喂)
有 3 个选项:

> DENY：当前页面不能被嵌套 iframe 里，即便是在相同域名的页面中嵌套也不允许,也不允许网页中有嵌套 iframe
> SAMEORIGIN：iframe 页面的地址只能为同源域名下的页面
> ALLOW-FROM：可以在指定的 origin url 的 iframe 中加载

简单实例:

> X-Frame-Options: DENY
> 拒绝任何 iframe 的嵌套请求

> X-Frame-Options: SAMEORIGIN
> 只允许同源请求，例如网页为 foo.com/123.php，則 foo.com 底下的所有网页可以嵌入此网页，但是 foo.com 以外的网页不能嵌入

> X-Frame-Options: ALLOW-FROM http://s3131212.com
> 只允许指定网页的 iframe 请求，不过兼容性较差 Chrome 不支持

X-Frame-Options 其实就是将前端 js 对 iframe 的把控交给服务器来进行处理。

```js
//js
if(window != window.top){
    window.top.location.href = window.location.href;
}
//等价于
X-Frame-Options: DENY

//js
if (top.location.hostname != window.location.hostname) {
　　　　top.location.href =window.location.href;
}
//等价于
X-Frame-Options: SAMEORIGIN
```

该属性是对页面的 iframe 进行一个主要限制，不过，涉及 iframe 的 header 可不止这一个，另外还有一个 Content Security Policy, 他同样也可以对 iframe 进行限制，而且，他应该是以后网页安全防护的主流。

#### CSP 之页面防护

和 X-Frames-Options 一样，都需要在服务器端设置好相关的 Header. CSP 的作用， 真的是太大了，他能够极大的防止你的页面被 XSS 攻击，而且可以制定 js,css,img 等相关资源的 origin，防止被恶意注入。不过他的兼容性，也是渣的一逼。目前支持 Edge12+ 以及 IE10+。
而且目前市面上，流行的是 3 种 CSP 头，以及各种浏览器的兼容性

![](https://user-gold-cdn.xitu.io/2017/2/10/c9f323b161a6269451040eb12161d6fc?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

使用主要是在后端服务器上配置，在前端，我们可以观察 Response Header 里是否有这样的一个 Header:

> Content-Security-Policy: default-src 'self'

这就表明，你的网页是启用 CSP 的。
通常我们可以在 CSP 后配置各种指定资源路径，有

> default-src,
> script-src,
> style-src,
> img-src,
> connect-src,
> font-src,
> object-src,
> media-src,
> sandbox,
> child-src,
> ...

如果你未指定的话，则是使用 default-src 规定的加载策略.
默认配置就是同域: default-src "self".
这里和 iframe 有一点瓜葛的就是 child-src 和 sandbox.
child-src 就是用来指定 iframe 的有效加载路径。其实和 X-Frame-Options 中配置 allow-origin 是一个道理。不过,allow-origin 没有得到厂商们的支持。
而，sandbox 其实就和 iframe 的 sandbox 属性（下文介绍）,是一样一样的，他可以规定来源能够带有什么权限.
来个 demo:

> Content-Security-Policy: child-src 'self' http://example.com; sandbox allow-forms allow-same-origin

此时，iframe 的 src 就只能加载同域和 example.com 页面。 最后再补充一点: 使用 CSP 能够很好的防止 XSS 攻击，原理就是 CSP 会默认 escape 掉内联样式和脚本，以及 eval 执行。但是，你可以使用 srcipt-src 进行降低限制.

> Content-Security-Policy: script-src 'unsafe-inline'

如果想更深入的了解 CSP,可以参阅:[CSP](http://content-security-policy.com/),[中文 CSP](https://imququ.com/post/content-security-policy-reference.html),[H5rock 之 CSP](http://www.html5rocks.com/en/tutorials/security/content-security-policy/)

ok, 上面基本上就是防止自己页面被嵌套而做的一些安全防护工作。 当然，我们面临的安全问题还有一个，就是当 iframe 别人的页面时，我们需要对其进行安全设限，虽然，跨域时 iframe 的安全性会大很多，但是，互联网是没有安全的地方。在以前，我们会进行各种 trick 来防止自己的页面被污染，现在 h5 提供的一个新属性 sandbox 可以很好的解决这个问题。

#### sandbox

sandbox 就是用来给指定 iframe 设置一个沙盒模型限制 iframe 的更多权限.
sandbox 是 h5 的一个新属性,IE10+支持(md~).
启用方式就是使用 sandbox 属性:

```xml
<iframe sandbox src=”...”></iframe>
```

这样会对 iframe 页面进行一系列的限制:

1. script 脚本不能执行
2. 不能发送 ajax 请求
3. 不能使用本地存储，即 localStorage,cookie 等
4. 不能创建新的弹窗和 window
5. 不能发送表单
6. 不能加载额外插件比如 flash 等

看到这里，我也是醉了。 好好的一个 iframe，你这样是不是有点过分了。 不过，你可以放宽一点权限。在 sandbox 里面进行一些简单设置

```xml
<iframe sandbox=”allow-same-origin” src=”...”></iframe>
```

常用的配置项有:
配置|效果
--|--
allow-forms| 允许进行提交表单
allow-scripts |运行执行脚本
allow-same-origin |允许同域请求,比如 ajax,storage
allow-top-navigation |允许 iframe 能够主导 window.top 进行页面跳转
allow-popups |允许 iframe 中弹出新窗口,比如,window.open,target="\_blank"
allow-pointer-lock| 在 iframe 中可以锁定鼠标，主要和鼠标锁定有关

可以通过在 sandbox 里，添加允许进行的权限.

```xml
<iframe sandbox=”allow-forms allow-same-origin allow-scripts” src=”...”></iframe>
```

这样，就可以保证 js 脚本的执行，但是禁止 iframe 里的 javascript 执行 top.location = self.location。
哎，其实，iframe 的安全问题还是超级有的。比如[location 劫持](http://drops.wooyun.org/papers/104)，[Refers 检查](http://drops.wooyun.org/papers/104)等。 不过目前而言，知道怎么简单的做一些安全措施就 over 了，白帽子们会帮我们测试的。

### resolve iframe 跨域

iframe 就是一个隔离沙盒，相当于我们在一个页面内可以操控很多个标签页一样。如果踩坑的童鞋应该知道，iframe 的解决跨域也是很有套套的。
首先我们需要明确什么是跨域。
浏览器判断你跨没跨域，主要根据两个点。 一个是你网页的协议(protocol)，一个就是你的 host 是否相同，即，就是 url 的首部:

```js
window.location.protocol + window.location.host;
```

![](https://user-gold-cdn.xitu.io/2017/2/10/aff862d2f3b1528907f492f508bfe536?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

需要强调的是，url 首部必须一样，比如:二级域名或者 IP 地址，都算是跨域.

```
//域名和域名对应ip, 跨域
http://www.a.com/a.js
http://70.32.92.74/b.js

//统一域名，不同二级域名。 跨域
http://www.a.com/a.js
http://a.com/b.js
```

对于第二种方式的跨域（主域相同而子域不同）,可以使用 iframe 进行解决。
在两个不同子域下(某一方使用 iframe 嵌套在另一方)，
即:
http: //www.foo.com/a.html 和 http: //script.foo.com/b.html
两个文件中分别加上 document.domain = ‘foo.com’,指定相同的主域，然后,两个文档就可以进行交互。

```js
//b.html是以iframe的形式嵌套在a.html中

//www.foo.com上的a.html

document.domain = "foo.com";
var ifr = document.createElement("iframe");
ifr.src = "http://script.foo.com/b.html";
ifr.style.display = "none";
document.body.appendChild(ifr);
ifr.onload = function () {
  var doc = ifr.contentDocument || ifr.contentWindow.document;
  // 在这里操纵b.html
  alert(doc.getElementsByTagName("h1")[0].childNodes[0].nodeValue);
};
//script.foo.com上的b.html

document.domain = "foo.com";
```

默认情况下 document.domain 是指 window.location.hostname. 你可以手动更改，但是最多只能设置为主域名。 通常，主域名就是指不带 www 的 hostname, 比如: foo.com , baidu.com 。 如果，带上 www 或者其他的前缀，就是二级域名或者多级域名。通过上述设置，相同的 domain 之后，就可以进行同域的相关操作。

另外还可以使用 iframe 和 location.hash，不过由于技术 out 了，这里就不做介绍了。

### H5 的 CDM 跨域与 iframe

如果你设置的 iframe 的域名和你 top window 的域名完全不同。 则可以使用 CDM(cross document messaging)进行跨域消息的传递。该 API 的兼容性较好 ie8+都支持.
发送消息: 使用 postmessage 方法
接受消息: 监听 message 事件

#### postmessage

该方法挂载到 window 对象上，即，使用 window.postmessage()调用.
该方法接受两个参数:postMessage(message, targetOrigin):
message: 就是传递给 iframe 的内容, 通常是 string,如果你想传 object 对象也可以。不过使用前请参考这一句话:

> [Objects listed in transfer are transferred, not just cloned, meaning that they are no longer usable on the sending side.](https://html.spec.whatwg.org/multipage/comms.html#web-messaging)

意思就是，希望亲爱的不要直接传 Object。 如果有条件，可以使用是 JSON.stringify 进行转化。这样能保证不会出 bug.
targetOrigin: 接受你传递消息的域名，可以设置绝对路径，也可以设置""或者"/"。 表示任意域名都行，"/"表示只能传递给同域域名。

来个栗子:

```xml
<iframe src="http://tuhao.com" name="sendMessage"></iframe>
```

```js
//当前脚本
let ifr = window.frames["sendMessage"];
//使用iframe的window向iframe发送message。
ifr.postmessage("give u a message", "http://tuhao.com");
//tuhao.com的脚本
window.addEventListener("message", receiver, false);
function receiver(e) {
  if (e.origin == "http://tuhao.com") {
    if (e.data == "give u a message") {
      e.source.postMessage("received", e.origin); //向原网页返回信息
    } else {
      alert(e.data);
    }
  }
}
```

当 targetOrigin 接受到 message 消息之后,会触发 message 事件。 message 提供的 event 对象上有 3 个重要的属性，data,origin,source.

> data：postMessage 传递进来的值
> origin：发送消息的文档所在的域
> source：发送消息文档的 window 对象的代理，如果是来自同一个域，则该对象就是 window，可以使用其所有方法，如果是不同的域，则 window 只能调用 postMessage()方法返回信息

属性的使用方法，如上头那个 demo 说的。 很贴切， 很完美~

### finally iframe

iframe 的大概我们差不多了解了。 想说的是，iframe 在后面会越用越少的。但是应该是不会被 w3c 标准废除的。到是前端的发展，真尼玛太快了，每过一段时间，一个新工具，新技术的出现，好到是好，不过学的真心蛋疼。不过，我们要以一个大前端的姿态要求我们，md,现在不学，那什么时候学~
