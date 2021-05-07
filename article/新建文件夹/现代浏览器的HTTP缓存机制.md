## 前言

> 写这篇文章的原因是最近在研究PWA如何在项目中落地实施，这就无可避免的需要接触到浏览器缓存的相关知识，因此特意写下这篇文章巩固下自己对浏览器缓存的认识。

## 什么是HTTP缓存

缓存是一种保存资源副本并在下次请求时直接使用该副本的技术。当web缓存发现请求的资源已经被存储，它会拦截请求，返回该资源的拷贝，而不会去源服务器重新下载。这样带来的好处有：缓解服务器端压力，提升性能(获取资源的耗时更短了)。

## HTTP缓存的作用

我们都知道浏览器是基于HTTP协议和服务端进行通信的，一个网站一旦同时请求过多或者请求过大就容易造成页面渲染时长过长等性能问题，而且并非所有资源都需要实时更新的，将长久或一段时间内的资源进行缓存，能很大的缓解服务器压力和提升网站性能。
毫不夸张的说，HTTP缓存是达到高性能的重要组成部分。

> 注意：缓存需要合理配置，因为并不是所有资源都是永久不变的：重要的是对一个资源的缓存应截止到其下一次发生改变（即不能缓存过期的资源）。

## HTTP头缓存相关字段及优先级

强缓存：Expires: Date/Cache-Control：max-age=N
协商缓存：Last-Modified：Date和Etag：String
通过查询标准我们知道Cache-Control和Etag属于HTTP1.1版本，Expires和Last-Modified属于HTTP1.0版本，所以得出以下优先级：
强缓存：Cache-Control > Expires
协商缓存：Etag > Last-Modified

> 注意：Expires存在的缺陷是返回的到期时间是服务器端的时间，可能与客户端的时间有较大的时间差，所以在HTTP1.1版开始使用Cache-Control: max-age=秒替代

Last-Modified的缺陷：由于只能精确到秒，如果一个文件在1秒内多次修改，这时客户端无法识别，因此HTTP1.1版本使用Etag标识资源内容是否有变更来确认资源是否需要更新，相对来说更加精确

## 强缓存与协商缓存

**强缓存**：资源一旦被强缓存，在缓存时间内，浏览器发起二次请求时会直接读取本地缓存，不与服务器进行通讯。
强缓存时间过期的，浏览器会判断资源的响应头是否有Last-Modified和Etag字段，有的话执行协商缓存策略
**协商缓存**：如果响应头中的包括有Etag和Last-Modified字段，则客户端将If-None-Match：Etag的值和If-Modified-Since：Last-Modified的值添加到请求头发送给服务器，由源服务器校验，如果资源未过期则返回304状态码，浏览器直接使用缓存，否则返回200OK状态码和新资源。
当两种情况都存在时，**强缓存优先级要高于协商缓存**。

## Chrome浏览器的三种缓存策略

选择Chrome是因为它是现在最流行的网页调试工具也是最多人用的浏览器。
Chrome浏览器返回缓存http状态码总共有以下三个
1、200 from memory cache
客户端不与服务器通讯，直接从内存中读取缓存。此时的数据时缓存到内存中的，当关闭浏览器后，数据自然就被当垃圾回收清空。
2、200 from disk cache
客户端不与服务器通讯，直接从磁盘中读取缓存，因为数据存在磁盘中，就算关闭浏览器数据还是存在，下次打开只要数据不过期就可以直接读取。
3、304 Not Modified
客户端与服务器通讯，服务器验证资源是否需要更新，如果不需要更新服务器返回304状态码，然后客户端直接从缓存中读取数据

> 注意：经过测试，我发现Safari和Firefox都有三种缓存策略，IE和其他浏览器大家可以各自测试一下

浏览器三种缓存示例图
Chrome和Safari似乎没有办法在浏览器中直接查看缓存情况，因此只能实践中查看。
Chrome示例图:
状态码：200 OK

![](https://user-gold-cdn.xitu.io/2019/4/2/169dcb55bbcbaeda?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

状态码：200 from memeory cache
![](https://user-gold-cdn.xitu.io/2019/4/2/169dcb4991d763c8?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

状态码：200 from disk cache
![](https://user-gold-cdn.xitu.io/2019/4/2/169dcb42902d8303?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

状态码：304 Not Modified
![](https://user-gold-cdn.xitu.io/2019/5/14/16ab56159ddd120f?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Safari示例图： 响应头：

![](https://user-gold-cdn.xitu.io/2019/4/2/169dcb326c639c06?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

状态码：200 OK
![](https://user-gold-cdn.xitu.io/2019/4/2/169dcb14ec14959e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

状态码：200 内存

![](https://user-gold-cdn.xitu.io/2019/4/2/169dcb1672918d81?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

状态码：200 磁盘

![](https://user-gold-cdn.xitu.io/2019/4/2/169dcb25f548ac1e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Firefox：在url上输入about:cache 可以看到对应的缓存情况，大家可以试一下

![](https://user-gold-cdn.xitu.io/2019/4/2/169dcb0da5236214?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
从截图中可以看到Firefox也分为内存缓存和磁盘缓存，304Not Modified自然也是有的。
![](https://user-gold-cdn.xitu.io/2019/4/2/169dcaff2d9e09fa?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
资源被强缓存后状态码依然是200 OK，不过会在传输列下显示已缓存，但是无法看出是内存缓存还是磁盘缓存。
![](https://user-gold-cdn.xitu.io/2019/4/2/169dcad1b91a7025?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
Firefox的304与Chrome和Safari差别不大。

## 三种缓存策略实际执行的条件

我在网上看到有人写文章说 **js、图片和字体保存在内存中而css则保存在磁盘**，很明显，只要自己稍微测试一下就知道这种说法是站不住脚的，那么这三种情况究竟是怎样的呢？
经过简单的测试以后我发现这三种策略并不复杂，默认配置情况下，Chrome第一次请求资源后，如果资源的响应头有Cache-Control或者Expires且有效期大于现在，则加载数据后将强缓存资源到内存和磁盘。
刷新页面，Chrome发起整个页面的二次请求后，通过开发者工具可以看到强缓存资源都会从内存进行读取，这就是200 from memory cache的情况。
示例：
这时关闭浏览器后，重新打开浏览器并打开关闭前的页面，通过开发者工具可以看到之前强缓存资源都会从磁盘中读取，这是因为关闭了浏览器后系统回收了内存资源，因此内存没有了之前的强缓存资源，需要从磁盘中读取，这就是200 from disk cache的情况。
示例：
如果这时使用ctrl + f5强刷页面则会发现全部资源都是200 OK状态要从服务器中获取新数据。
304 Not Modified的情况则完全不同，如果资源的响应头是Last-Modified或Etag，第一次请求资源后缓存到本地磁盘，但第二次也必须发起请求到服务器进行查询该资源是否过期或被修改过，当服务器验证资源没有过期后才会返回304 Not Modified状态码，同时响应体为空，这样可以节省流量并提高响应速度，客户端接收到304状态码后从本地读取数据，因此304比200 from cache响应速度要慢，但比200 OK快得多。

## Chrome浏览器缓存机制流程图

 
![](https://user-gold-cdn.xitu.io/2019/4/2/169dc4ce582cbd65?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)
