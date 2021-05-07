![](https://pic2.zhimg.com/80/v2-9ab94852cdca3e31cbfd365efe988c35_1440w.jpg)
一、前言
前端缓存主要是分为HTTP缓存和浏览器缓存。其中HTTP缓存是在HTTP请求传输时用到的缓存，主要在服务器代码上设置；而浏览器缓存则主要由前端开发在前端js上进行设置。
缓存可以说是性能优化中简单高效的一种优化方式了。一个优秀的缓存策略可以缩短网页请求资源的距离，减少延迟，并且由于缓存文件可以重复利用，还可以减少带宽，降低网络负荷。
对于一个数据请求来说，可以分为发起网络请求、后端处理、浏览器响应三个步骤。浏览器缓存可以帮助我们在第一和第三步骤中优化性能。比如说直接使用缓存而不发起请求，或者发起了请求但后端存储的数据和前端一致，那么就没有必要再将数据回传回来，这样就减少了响应数据。
————————————————

HTTP请求缓存一般也是建立在get请求上面的，get和post的区别
![](https://img-blog.csdnimg.cn/20201127132953249.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3dlaXhpbl80NDczMDg5Nw==,size_16,color_FFFFFF,t_70)
一、CDN的定义
CDN：Content Delivery Network/Content Ddistribute Network，即内容分发网络

客户端访问网站的过程：

没有CDN：

1、用户在浏览器访问栏中输入要访问的域名；

2、浏览器向DNS服务器请求对该域名的解析；

3、DNS服务器返回该域名的IP地址给浏览器

4、浏览器使用该IP地址向服务器请求内容。

5、服务器将用户请求的内容返回给浏览器。

使用了CDN：

1、用户在浏览器中输入要访问的域名。

2、浏览器向DNS服务器请求对域名进行解析。由于CDN对域名解析进行了调整，DNS服务器会最终将域名的解析权交给CNAME指向的CDN专用DNS服务器。

3、CDN的DNS服务器将CDN的负载均衡设备IP地址返回给用户。

4、用户向CDN的负载均衡设备发起内容URL访问请求。

5、CDN负载均衡设备会为用户选择一台合适的缓存服务器提供服务。

选择的依据包括：根据用户IP地址，判断哪一台服务器距离用户最近；根据用户所请求的URL中携带的内容名称，判断哪一台服务器上有用户所需内容；查询各个服务器的负载情况，判断哪一台服务器的负载较小。

基于以上这些依据的综合分析之后，负载均衡设置会把缓存服务器的IP地址返回给用户。

6、用户向缓存服务器发出请求。

7、缓存服务器响应用户请求，将用户所需内容传送到用户。

如果这台缓存服务器上并没有用户想要的内容，而负载均衡设备依然将它分配给了用户，那么这台服务器就要向它的上一级缓存服务器请求内容，直至追溯到网站的源服务器将内容拉取到本地。

二、关于缓存
没有CDN：浏览器缓存

使用了CDN：浏览器缓存+CDN缓存

在用户第一次访问网站后，网站的一些静态资源如图片等就会被下载到本地，作为缓存，当用户第二次访问该网站的时候，浏览器就会从缓存中加载资源，不用向服务器请求资源，从而提高了网站的访问速度，而若使用了CDN，当浏览器本地缓存的资源过期之后，浏览器不是直接向源站点请求资源，而是向CDN边缘节点请求资源，CDN边缘节点中也存在缓存，若CDN中的缓存也过期，那就由CDN边缘节点向源站点发出回源请求来获取最新资源。

浏览器缓存以及CDN缓存都有一套判断文件是否需要更新的机制：

浏览器在加载资源时，先根据这个资源的一些http header判断它是否命中强缓存，如果命中，浏览器直接从自己的缓存中读取资源，不会发请求到服务器，当强缓存没有命中的时候，浏览器一定会发送一个请求到服务器，服务器端依据资源的另外一些http header验证这个资源是否命中协商缓存，如果命中，服务器会将这个请求返回，但是不会返回这个资源的数据，而是告诉客户端可以直接从缓存中加载这个资源，于是浏览器还是从自己的缓存中加载资源，当协商缓存也没有命中的时候，浏览器直接从服务器加载资源数据。

CDN节点缓存机制在不同服务商中是不同的，但一般都遵循HTTP协议，通过http响应头中的Cache-Control:max-age的字段来设置CDN节点文件缓存时间。当客户端向CDN节点请求数据时，CDN会判断缓存数据是否过期，若没有过期，则直接将缓存数据返回给客户端，否则就向源站点发出请求，从源站点拉取最新数据，更新本地缓存，并将最新数据返回给客户端。CDN服务商一般会提供基于文件后缀、目录多个维度来指定CDN缓存时间，为用户提供更精细化的缓存管理。CDN缓存时间会对“回源率”产生直接的影响，若CDN缓存时间短，则数据经常失效，导致频繁回源，增加了源站的负载，同时也增大了访问延时；若缓存时间长，数据更新时间慢，因此需要针对不同的业务需求来选择特定的数据缓存管理。

浏览器缓存刷新：

1、在地址栏中输入网址后按回车或者点击转到按钮

浏览器以最少的请求来获取网页的数据，浏览器会对所有没有过期的内容直接使用本地缓存即使用强缓存，从而减少了对服务器的请求，Expires、max-age标志只对这种方式有效。

2、按F5或浏览器刷新按钮

浏览器会在请求中附加必要的缓存协商，但不允许浏览器直接使用本地缓存即跳过强缓存的判断，直接进行协商缓存的判断，Last-Modified、ETag在这种方式发挥作用。

3、按Ctrl+F5或按Ctrl并点击刷新按钮

强制刷新，完全不使用缓存

CDN缓存刷新：

CDN节点对开发者时透明的，可以通过CDN服务商提供的“刷新缓存”接口来达到清理CDN节点缓存的效果，强制使数据过期，从而获取到最新的数据。

三、浏览器缓存
这一点主要解析浏览器缓存以及缓存机制的详细过程。

1.1强缓存：

当浏览器对某个资源的请求命中了强缓存时，返回的http状态码为200，在chrome开发者工具中的network中的size会显示from cache

强缓存时利用Expires或者Cache-Control这两个http header实现的，都用来表示资源在客户端缓存的有效期

Expires是http1.0提出的一个header，描述的是一个绝对时间，由服务器返回，用GMT格式的字符串表示，如Exprires：Thu，31 Dec 2037 23：55：55 GMT

缓存过程：

1、浏览器第一次跟服务器请求一个资源，服务器在返回这个资源的同时，在response的header加上Expires的header

2、浏览器在接收到这个资源后，会把这个资源连同所有的response header一起缓存下来，所以缓存命中的请求返回的header并不是来自服务器，而是来自之前缓存的header

3、浏览器再请求这个资源时，先从缓存中寻找，找到这个资源后，拿出Expires跟当前的请求时间比较，如果请求时间在Expires指定的时间之前，就能命中缓存，否则就不行。

4、如果缓存没有命中，浏览器直接从服务器加载资源时，Expires Header在重新加载的时候会被更新

Expires是服务器返回的一个绝对时间，在服务器时间与客户端时间相差较大时，缓存管理容易出现问题，比如随意修改下客户端时间，就能影响缓存命中的结果，所以在http1.1的时候，提出了一个新的header，也就是Cache-Control，这是一个相对时间，在进行缓存命中的时候，都是利用客户端时间进行判断，因此更有效安全一些，在配置缓存的时候，以秒为单位，用数值表示：如：Cache-Control：max-age=315360000，它的缓存过程是：

1、浏览器第一次跟服务器请求一个资源，服务器在返回这个资源的同时，在response的header加上Cache-Control的header

2、浏览器在接收到这个资源的时候，会把这个资源连同所有response header一起缓存下来

3、浏览器再次请求这个资源的时候，先从缓存中寻找，找到这个资源之后，再拿这个过期时间跟当前的请求时间比较，如果请求时间在过期时间之前，就能命中缓存，否则就不行。

4、如果缓存没有命中，浏览器直接从服务器加载资源时，Cache-Control在重新加载的时候会被更新

这两个header可以只用一个，也可以同时用两个，同时存在时，Cache-Control优先级高于Expires

1.2 强缓存的管理

两种方式来设置是否启用强缓存：

1、通过代码的方式，在web服务器返回的响应中添加Expires和Cache-Control Header

2、通过配置web服务器的方式，让web服务器在响应资源的时候统一添加Expires和Cache-Control Header

1.3 强缓存的应用

强缓存是前端性能优化最有力的工具，对于有大量静态资源的网页，一定要利用强缓存，提高响应速度，通常是为这些静态资源全部配置一个超时时间超长的Expires或Cache-Control，这样用户只会在第一次访问网站时加载静态资源，其他时间只要缓存没有失效并且用户没有强制刷新的条件下都会从缓存中加载。

然而这种缓存配置方式会带来一个问题，就是当资源更新时，客户端由于有缓存不会向服务器请求最新的资源，这个问题已有解决方案：

通过更新页面中引用的资源路径，让浏览器主动放弃缓存，加载新资源。

但要实现有更新的文件才需要浏览器重新加载，因此必须让url的修改与文件内容相关联，利用数据摘要算法对文件求摘要信息，摘要信息与文件内容一一对应，这一点许多前端构建工具都做到了，如webpack

1.4 浏览器默认缓存使开发环境下常因为资源没有及时更新而看不到效果

解决方法：

1、ctrl+F5

2、浏览器隐私模式开发

3、chrome开发者工具里将Disable cache选项打勾，阻止缓存

4、在开发阶段，给资源加上一个动态的参数，由于每次资源的修改都要更新引用的位置，同时修改参数的值，所以操作起来不是很方便，除非是在动态页面比如jsp里开发就可以用服务器变量来解决，或者用前端构建工具来处理这个参数修改的问题。

5、如果资源引用的页面被嵌入到了一个iframe里面，可以在iframe的区域右键重新加载该页面

6、如果缓存问题出现在ajax请求中，最有效的解决办法就是ajax的请求地址追加随机数

7、动态设置iframe的src时，有可能因为缓存问题导致看不到最新效果，在src后面添加随机数即可

8、通过前端开发工具grunt gulp等的插件来启动一个静态服务器，则在这个服务器下所有资源返回的response header中，Cache-Control始终被设置为不缓存

1.5 发布问题

发布问题：若页面和它引用的资源路径同时更新了，不管是先部署页面还是先部署资源都会带来各种问题，这是由于资源是覆盖式发布的，即用待发布资源覆盖已发布资源。

解决办法就是实现非覆盖式发布：把有修改的资源文件作为一个新的文件发布，不对已有的资源文件进行覆盖，这样用户还可以请求旧的资源文件，不至于发生页面错乱的问题，这样先部署静态资源，再覆盖式部署页面，等到用户访问新页面的时候，新的资源文件也已发布，就可以正确请求，即可解决问题。

2.1 协商缓存

如果命中协商缓存，请求响应返回的http状态为304以及一个Not Modified字符串，协商缓存利用的是【Last-Modified、If-Modified-Since】、【ETag、If-None-Match】这两对header来管理的。

【****Last-Modified、If-Modified-Since】：

1、浏览器第一次跟服务器请求一个资源，服务器在返回这个资源时，在response的header加上Last-Modified的header，表示这个资源在服务器上的最后修改时间

2、浏览器再次向服务器请求这个资源时，在request的header加上If-Modified-Since的header，这个header的值就是上一次请求时返回的Last-Modified的值

3、服务器再次收到资源请求时，根据浏览器传过来If-Modified-Since和资源在服务器上的最后修改时间判断资源是否有变化，如果没有变化则返回304 Not Modified，但是不会返回资源内容，如果有变化就返回资源内容，当服务器返回304 Not Modified的响应时，response header中不会再添加Last-Modified的header，因为资源没有变化，Last-Modified的值也不变

4、浏览器收到304的响应后，就会从缓存中加载资源

5、如果协商缓存没有命中，浏览器直接从服务器加载资源时，Last-Modofied header在重新加载的时候会被更新，下次请求时，If-Modified-Since会采用上一次返回的Last-Modified的值

这一对header都是根据服务器时间返回的，有时候会有服务器资源有变化，但最后修改时间却没有变化的情况，因此有了

【Etag、If-None-Match】：

1、浏览器第一次向服务器请求一个资源，服务器在返回这个资源的同时，在response的header加上ETag的header，这个header是服务器根据当前请求的资源生成的一个唯一标识，是一个字符串，只要资源内容发生改变，这个字符串也会改变，跟时间没有关系

2、浏览器再次请求这个资源的时候，在request的header上加上If-None-Match的header。这个header的值是上一次请求返回的ETag的值

3、服务器再次收到资源请求时，根据客户端传过来的If-None-Match和重新生成的该资源的新的ETag做比较，相同则返回304 Not Modified，不会返回资源内容，如果不同则返回资源内容，但这里即使资源没有发生变化，也会返回ETag，因为这个ETag重新生成过，即使没有ETag没有变化

4、浏览器收到304响应后，就从缓存中加载资源

2.2 协商缓存的管理

一般服务器上的【Last-Modified、If-Modified-Since】和【Etag、If-None-Match】会同时启用，协商缓存需要配合强缓存使用

![](https://upload-images.jianshu.io/upload_images/13168254-20f172ed4bb05684.png?imageMogr2/auto-orient/strip|imageView2/2/w/842/format/webp)

![](https://upload-images.jianshu.io/upload_images/13168254-9e0ff8040439a4ad.png?imageMogr2/auto-orient/strip|imageView2/2/w/411/format/webp)

![](https://upload-images.jianshu.io/upload_images/13168254-eb220515b94bb946.png?imageMogr2/auto-orient/strip|imageView2/2/w/554/format/webp)

## HTTP缓存

http请求做为影响前端性能极为重要的一环，因为请求受网络影响很大，如果网络很慢的情况下, 页面很可能会空白很久。对于首次进入网站的用户可能要通过优化接口性能和接口数量来解决。但是，对于重复进入页面的用户，除了浏览器缓存，http缓存可以很大程度对已经加载过的页面进行优化。

1. 缓存位置

![](https://res-static.hc-cdn.cn/fms/img/4564441173140c33293139666fac62d81603447265429)

从缓存位置上来看，分为4种，从上往下依次检查是否命中，如果但都没有命中则重新发起请求。

* Service Worker 是运行在浏览器背后的独立线程，一般可以用来实现缓存功能。使用 Service Worker的话，传输协议必须为 HTTPS。
* Memory Cache 也就是内存中的缓存，主要包含的是当前中页面中已经抓取到的资源, 例如页面上已经下载的样式、脚本、图片等。读取内存中的数据肯定比磁盘快, 内存缓存虽然读取高效，可是缓存持续性很短，会随着进程的释放而释放。一旦我们关闭 Tab 页面，内存中的缓存也就被释放了。

内存缓存中有一块重要的缓存资源是preloader相关指令（例如）下载的资源。它可以一边解析js/css文件，一边网络请求下一个资源。

* Disk Cache 也就是存储在硬盘中的缓存，读取速度慢点，但是什么都能存储到磁盘中，比之 Memory Cache 胜在容量和存储时效性上。

绝大部分的缓存都来自Disk Cache，在HTTP 的协议头中设置。

* Push Cache（推送缓存）是 HTTP/2 中的内容，当以上三种缓存都没有命中时，它才会被使用。它只在会话（Session）中存在，一旦会话结束就被释放，并且缓存时间也很短暂，在Chrome浏览器中只有5分钟左右，同时它也并非严格执行HTTP头中的缓存指令。

2. 用户操作对缓存的影响

![](https://res-static.hc-cdn.cn/fms/img/03f7e8647bcc53808095f375ec182cf91603447265429)
![](https://res-static.hc-cdn.cn/fms/img/fffa97732f2a16e13edb69c4a7497d911603447265432)

二、浏览器本地存储

浏览器本地缓存最常用的是cookie、localStroage、sessionStroage、webSql、indexDB。

1. cookie使用

cookie的用法很简单, 可以通过服务端设置，js也可以通过documnet.cookie="名称=值; "（不要忘记以; 分割）来设置。
cookie的值字符串可以用encodeURIComponent()来保证它不包含任何逗号、分号或空格(cookie值中禁止使用这些值).
cookie一般用做为登陆态保存、密码、个人信息等关键信息保存使用，所以为了安全也是遵守同源策略原则的。
可以通过下面参数具体设置：
; path=path (例如 '/', '/mydir') 如果没有定义，默认为当前文档位置的路径。
; domain=domain (例如 'example.com'， 'subdomain.example.com') 如果没有定义，默认为当前文档位置的路径的域名部分。与早期规范相反的是，在域名前面加 . 符将会被忽视，因为浏览器也许会拒绝设置这样的cookie。如果指定了一个域，那么子域也包含在内。
; max-age=max-age-in-seconds (例如一年为606024*365)
; expires=date-in-GMTString-format 如果没有定义，cookie会在对话结束时过期这个值的格式参见Date.toUTCString() 
; secure (cookie只通过https协议传输)
; HttpOnly 限制web页面程序的browser端script程序读取cookie

缺点
容量有限制，不能超过4kb
在请求头上带着数据安全性差

2. localStorage和sessionStorage使用

html5新增本地存储，localStorage生命周期是永久，除非主动清除localStorage信息，否则这些信息将永远存在。存放数据大小为一般为5MB, sessionStorage仅在当前会话下有效，关闭页面或浏览器后被清除。而且它仅在客户端（即浏览器）中保存，不参与和服务器的通信。也是遵守同源策略原则的

``` 

// 1、保存数据到本地
// 第一个参数是保存的变量名，第二个是赋给变量的值
localStorage.setItem('key', 'value');
//复杂类型储存需要**利用JSON.stringify**将对象转换成字符串；
//利用**JSON.parse**将字符串转换成对象
// 2、从本地存储获取数据
localStorage.getItem('key');
// 3、从本地存储删除某个已保存的数据
localStorage.removeItem('key');
// 4、清除所有保存的数据
localStorage.clear();
```

**比较一下Cookie、LocalStorage、sessionStorage的异同：**

cookie会在在同源的http请求中携带，不能超过4K，参与和服务器交互；
sessionStorge和locaStorage保存数据在本地，限制最多5M，不参与和服务器交互；
sessionStorage仅在当前窗口关闭前有效；
localstorage会一直保存在浏览器中，除非手动删除；
cookie在设置过期时间之前一直有效，不设置过期时间窗口关闭则清除；
sessionStorage不在不同的浏览器窗口中共享，即使是同一个页面，
localstorage和cookie在所有同源窗口中共享

3. Web SQL

WebSQL是前端的一个独立模块，是web存储方式的一种，我们调试的时候会经常看到，只是一般很少使用。并且，当前只有谷歌支持，ie和火狐均不支持。
主要方法：

1.openDatabase：这个方法使用现有的数据库或者新建的数据库创建一个数据库对象。
2.transaction：这个方法让我们能够控制一个事务，以及基于这种情况执行提交或者回滚。
3.executeSql：这个方法用于执行实际的 SQL 查询。

创建表：

create table if not exists product(id integer primary key autoincrement, name text not null, price double); 

添加一条数据：

insert into product(name, price) values('斗战圣皇', '1.2'); 

insert into product(name, price) values('布衣神探：最后的证据', '0.9'); 

删除一条数据：

delete from product where id=2; 

更新一条数据：

update product set name='斗战圣皇第二部', price='0.99' where id=1; 

查询表格：

select * from product; 

删除表格：

drop table IF EXISTS product; 
![](https://pic3.zhimg.com/80/v2-2e5df0ee52ec09898eac54ad6b9b6382_1440w.jpg)

4. indexDB

IndexedDB 就是浏览器提供的本地数据库，它可以被网页脚本创建和操作。IndexedDB 允许储存大量数据，提供查找接口，还能建立索引。这些都是 LocalStorage 所不具备的。就数据库类型而言，IndexedDB 不属于关系型数据库（不支持 SQL 查询语句），更接近 NoSQL 数据库。

open()打开或创建数据库
deleteDatabase()删除数据库；
transaction()打开事物
add()添加数据
get()查找数据
delete()根据ID删除数据
clear()清除全部数据

![](https://pic2.zhimg.com/80/v2-59c9e1b45bbaa7db978b296d6d3d5bcd_1440w.jpg)
