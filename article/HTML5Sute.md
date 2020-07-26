如今屏幕分辨率的范围已经从 320px（iPhone）涵盖到 2560px（大显示器）或者更高了。用户不单单在桌面电脑上浏览网站，用户如今使用移动电话，小的笔记本，平板设备（比如 iPad）来访问互联网。所以传统的固定宽度设计不那么理想了。web 设计需要有自适应能力。 页面布局要可以自动的去适应所有的分辨率和设备。现在是一个移动端的时代，移动先行已经深入骨髓，作为一个 web 前端开发，如果你还在为如何开发移动端页面而迷茫，或者你还在为开发出了一个在你手机上“完美”的移动页面而沾沾自喜却不知移动的世界有多“残酷”的时候，那你应该看看这篇文章了

### 1. 移动设备尺寸及分辨率

下面我们来看看移动设备的分辨率

iPhone 界面尺寸：
| iPhone | iPhone6 Plus | iPhone6 | iPhone5s |
|:-|:-|:-|:-|
| 尺寸 | 5.5 英寸 | 4.7 英寸 | 4 英寸 |
| 像素分辨率 | 1242*2208 px | 750*1334 px | 640\*1136 px |

在设计的时候并不是每个尺寸都要做一套，尺寸按自己的手机尺寸来设计，比较方便预览效果，通常设计图是 750\*1334

iPad 界面尺寸：
| iPad | iPad 3-4-5-6-Air-Air2-mini2| iPad 1-2 | iPad Mini |
|:-|:-|:-|:-|
| 像素分辨率 | 2048×1536 px | 1024×768 px | 1024×768 px |

### 2. 目前移动端做适配几种常用方法：

#### 2.1、CSS3 媒体查询

采用 CSS3 媒体查询可以为不同的媒体设置不同的 css 样式，这里的“媒体”包括页面尺寸，设备屏幕尺寸等，

```typescript
@media 媒体类型 and (视口特性阀值){ /* 媒体类型可以 all screen print 等*/
    // 满足条件的css样式代码
}
```

比如我们要为宽度大于 375px 的页面中的 class="content"的元素设置样式，可以这样写，

```typescript
@media screen and (min-width=375px) {
    .content { styles }
};
```

我们在使用 Media 的时候需要先设置 Meta 标签下面这段代码，来兼容移动设备的展示效果：

```xml
<meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
```

标签中的几个参数解释：

- width = device-width：宽度等于当前设备的宽度

- initial-scale：初始的缩放比例（默认设置为 1.0）

- minimum-scale：允许用户缩放到的最小比例（默认设置为 1.0）

- maximum-scale：允许用户缩放到的最大比例（默认设置为 1.0）

- user-scalable：用户是否可以手动缩放（默认设置为 no，因为我们不希望用户放大缩小页面）

下面我们就来看看 css3 Media 具体的写法：

下面这段 css，估计很多人在响应式的网站经常看到类似下面这段：

```typescript

@media screen and (max-width: 960px){
    body{
        background: #000;
    }
}

```

这个应该算是一个 media 的一个标准写法，上面这段 CSS 代码意思是：当页面小于 960px 的时候执行它下面的 CSS，这个应该没有太大疑问。

下面就是我们最常需要用到的媒体查询器的三个特性，大于，等于，小于的写法：

```typescript

/*页面小于960px的时候*/
@media screen and (max-device-width:960px){
   html{
       font-size: 80px;
   }
}

/*当尺寸大于960px时候 */
@media screen and (min-width:960px){
   html{
       font-size: 90px;
   }
}

/* 当页面宽度大于960px小于1200px的时候执行下面的CSS */
@media screen and (min-width:960px) and (max-width:1200px){
   html{
       font-size: 100px;
   }
}

```

在移动端适配上通常使用的就是上面的方法，在文章后面会介绍国内大厂的具体使用方法，认真往下看

@media 媒体查询器的功能不止这三个功能，下面是 media 的一些参数用法解释：
h

> type 用来创建新的类型，也可以重命名（别名）已有的类型，建议使用 type 创建简单类型，无嵌套的或者一层嵌套的类型，其它复杂的类型都应该使用 interface, 结合 implements ,extends 实现。

- width: 定义输出设备中的页面可见区域宽度。
- height: 定义输出设备中的页面可见区域高度。
- device-width: 定义输出设备的屏幕可见宽度。
- device-height: 定义输出设备的屏幕可见高度。
- orientation: 检测设备目前处于横向还是纵向状态。
- aspect-ratio: 检测浏览器可视宽度和高度的比例。(例如：aspect-ratio: 16/9)
- device-aspect-ratio: 检测设备的宽度和高度的比例。
- color: 检测颜色的位数。（例如：min-color: 32 就会检测设备是否拥有 32 位颜色）
- color-index: 检查设备颜色索引表中的颜色，他的值不能是负数。
- monochrome: 检测单色楨缓冲区域中的每个像素的位数。（这个太高级，估计咱很少会用的到）
- resolution: 检测屏幕或打印机的分辨率。(例如：min-resolution: 300dpi 或 min-resolution: 118dpcm)。 grid：检测输出的设备是网格的还是位图设备。

详细功能介绍可以[点击查看](https://link.jianshu.com/?t=http%3A%2F%2Fwww.runoob.com%2Fcssref%2Fcss3-pr-mediaquery.html)

#### 2.2、百分比适配

用百分比做适配的方法是子元素相对于父元素的百分之多少，比如父元素的宽度为 100px;设置子元素的宽度可为 60%;这时子元素的宽为 60px;如父元素的宽度改为 200px 时，这时子元素的宽就是 120px; 所以可将 body 默认宽度设置为屏幕宽度（PC 中指的是浏览器宽度），子孙元素按百分比定位（或指定尺寸）就可以了，这只适合布局简单的页面，复杂的页面实现很困难。

#### 2.3、使用 rem 来做适配

rem：相对单位，可理解为”root em”, 相对根节点 html 的字体大小来计算，CSS3 新加属性，chrome/firefox/IE9+支持。// 是截止目前用的最多也是最流行的

在说 rem 之前，我们来说说关于手机视口的概念，通常在前端开发领域，像素有两层含义：设备像素和 css 像素。

- 设备像素： 设备屏幕的物理像素，对于任何设备来讲物理像素的数量是固定的。
- css 像素：这是一个抽象的像素概念，它是为 web 开发者创造的。

一个 CSS 像素的大小是可变的，比如用后缩放页面的时候，实际上就是在缩小或放大 CSS 像素，而设备像素无论大小还是数量都是不变的。

而在移动端中有三个视口，哪三个呢？

1. 布局视口：移动端 css 布局的依据视口，即 css 布局会根据布局视口来计算。
   可以通过以下 JavaScript 代码获取布局视口的宽度和高度：

document.documentElement.clientWidth
document.documentElement.clientHeight

2. 视觉视口：用户看到的区域
3. 理想视口：理想的布局视口

### 什么是 rem ?

> 在前端工程中，import 很多非 js 资源，例如：css, html, 图片，vue, 这种 ts 无法识别的资源时，就需要告诉 ts，怎么识别这些导入的资源的类型。

rem 是相对尺寸单位，rem 是将根节点 html 的 font-size 的值作为整个页面的基准尺寸，计算如下：

**元素的 rem 尺寸 = 元素的 psd 稿测量的像素尺寸 / 动态设置的 html 标签的 font-size 值**

比如默认 html 的 font-size 是 16px，即 1rem=16px，如果某 div 宽度为 32px 你可以设为 2rem。当我们把 html 的 font-size 设为 20px 时，1rem=20px，那么 32px=1.6rem 了。基于此，我们用 rem 来实现不同尺寸屏幕的自适应的方法就是：在页面载入开始时首先判断 window 的宽度 Width（是 window 的宽度(\$(window).width())，不是屏幕分辩率的宽度（screen.width）），若假定屏幕宽度为 750 的，而不同宽度的屏幕处理的处理方法，为了能保证换算容易，需先为 html 设置一个合适的 font-size，计算：100 / 750 = fontSize / Width, fontSize = Width / 750 \* 100 = W / 7.5；

假设我们以 iPhone6 设计图尺寸为标准，在设计图的尺寸下设置一个 font-size 值为 100px。
也就是说：750px 宽的页面，我们设置 100px 的 font-size 值，那么页面的宽度换算为 rem 就等于 750 / 100 = 7.5rem。

### 3. 大厂的做法

下面我们来看看，大厂是怎么做的呢？

1. 京东 的 web 移动端(设计图是 750\*1334)

```typescript

/*  京东 m.jd.com */

@media only screen and (min-width: 320PX) and (max-width:360PX) {
    html {
        font-size:13.65px
    }
}

@media only screen and (min-width: 360PX) and (max-width:375PX) {
    html {
        font-size:15.36px
    }
}

@media only screen and (min-width: 375PX) and (max-width:390PX) {
    html {
        font-size:16px
    }
}

@media only screen and (min-width: 390PX) and (max-width:414PX) {
    html {
        font-size:16.64px
    }
}

@media only screen and (min-width: 414PX) and (max-width:640PX) {
    html {
        font-size:17.664px
    }
}

@media screen and (min-width: 640PX) {
    html {
        font-size:27.31px
    }
}

/*
我们来看看怎么计算的

设计图中 尺寸40px盒子，我们可以看到在媒体查询中，iPhone6 的布局视口是375，即html的font-size为16px，
则计算为rem： 40/16 = 2.5 rem

*/

```

如下图： 京东 m 端在 iPhone 下的状态

iPhone6

在图中可以看到 img 设置的样式 width: 2.5rem; height: 2.5rem; 实际渲染出来的大小是 40\*40
因为我们可以在 css 媒体查询中看到在 iPhone6 下的设置 html 的 font-size 为 16px； 计算就是： 40/16 = 2.5.

![](https://upload-images.jianshu.io/upload_images/4030717-0280d7f53a13132a.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)
iPhone5

我们将其切换到 iPhone5 看看效果，如下图：
同样是刚才的图标在 5 下面大小是多少呢~ ，在 5 下 font-size 的值为 13.65px, 来算算结果就是 2.5\*13.65 = 34.125

![](https://upload-images.jianshu.io/upload_images/4030717-f9f8b4add7d7bccc.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

2. 网易新闻 web 移动端 (设计图大小目前版本本人看不出来，我记得 16 年是 750\*1334)

**注： vw：viewpoint width，视窗宽度，1vw 等于视窗宽度的 1%。**

```typescript


/* 网易新闻 3g.163.com */


@media screen and (max-width: 320px) {
    html {
        font-size:42.667px;
        font-size: 13.33333vw
    }
}

@media screen and (min-width: 321px) and (max-width:360px) {
    html {
        font-size:48px;
        font-size: 13.33333vw
    }
}

@media screen and (min-width: 361px) and (max-width:375px) {
    html {
        font-size:50px;
        font-size: 13.33333vw
    }
}

@media screen and (min-width: 376px) and (max-width:393px) {
    html {
        font-size:52.4px;
        font-size: 13.33333vw
    }
}

@media screen and (min-width: 394px) and (max-width:412px) {
    html {
        font-size:54.93px;
        font-size: 13.33333vw
    }
}

@media screen and (min-width: 413px) and (max-width:414px) {
    html {
        font-size:55.2px;
        font-size: 13.33333vw
    }
}

@media screen and (min-width: 415px) and (max-width:480px) {
    html {
        font-size:64px;
        font-size: 13.33333vw
    }
}

@media screen and (min-width: 481px) and (max-width:540px) {
    html {
        font-size:72px;
        font-size: 13.33333vw
    }
}

@media screen and (min-width: 541px) and (max-width:640px) {
    html {
        font-size:85.33px;
        font-size: 13.33333vw
    }
}

@media screen and (min-width: 641px) and (max-width:720px) {
    html {
        font-size:96px;
        font-size: 13.33333vw
    }
}

@media screen and (min-width: 721px) and (max-width:768px) {
    html {
        font-size:102.4px;
        font-size: 13.33333vw
    }
}

@media screen and (min-width: 769px) {
    html {
        font-size:102.4px;
        font-size: 13.33333vw
    }
}

body {
    font-family: PingFangSC-Regular,Microsoft YaHei,Helvetica;
    background: #f5f7f9
}

body,html a {
    color: #333
}

.hidden,.none {
    display: none!important
}

@media screen and (min-width: 769px) {
    html {
        font-size:102.4px
    }

    html .wap-wrap {
        width: 768px;
        margin: 0 auto
    }
}

/*
我们来看看怎么计算的

比如设计图中 尺寸117px盒子，我们可以看到在媒体查询中，iPhone6 html的font-size为50px，
则计算为rem： 117/50 = 2.34rem

*/

```

如下图： 网易新闻在 iPhone 下的状态

iPhone6

在图中可以看到 img 设置的样式 width: 2.34rem; 实际渲染出来宽的大小是 116.98
因为我们可以在 css 媒体查询中看到在 iPhone6 下的设置 html 的 font-size 为 50px； 计算就是： 116.98/50 = 2.3396.

废话： 我记得 16 年看网易并不是使用媒体查询的

3. 阿里的 手机淘宝 触屏版 (设计图是 750\*1334)

```typescript
/* 阿里手机淘宝  h5.m.taobao.com */

!(function (e, t) {
  var n = t.documentElement,
    d = e.devicePixelRatio || 1; // 设备DPR

  function i() {
    var e = n.clientWidth / 3.75; // iPhone 6 布局视口375
    n.style.fontSize = e + "px";
  }

  if (
    ((function e() {
      t.body
        ? (t.body.style.fontSize = "16px")
        : t.addEventListener("DOMContentLoaded", e);
    })(),
    i(),
    e.addEventListener("resize", i),
    e.addEventListener("pageshow", function (e) {
      e.persisted && i();
    }),
    d >= 2)
  ) {
    var o = t.createElement("body"),
      a = t.createElement("div");
    (a.style.border = ".5px solid transparent"),
      o.appendChild(a),
      n.appendChild(o),
      1 === a.offsetHeight && n.classList.add("hairlines"),
      n.removeChild(o);
  }
})(window, document);
```

如下图是阿里手淘在 iPhone6 下的情况，在图中我们可以看到 html 的 font-size 为 font-size: 100px;; 然后呢他们的样式都是在行内的，而且 window resize 的时候内容区域是不会变化的，需要重新渲染；

![](https://upload-images.jianshu.io/upload_images/4030717-c2c5cfccd5ef7674.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

4. 小米(设计图是 720\*1280)

```typescript
//小米  m.com

!(function (e) {
  var t = e.document,
    n = t.documentElement,
    i = e.devicePixelRatio || 1,
    a = "orientationchange" in e ? "orientationchange" : "resize",
    d = function () {
      var e = n.getBoundingClientRect().width || 360; //  Element.getBoundingClientRect()方法返回元素的大小及其相对于视口的位置。
      (1 == i || e > 720) && (e = 720), (n.style.fontSize = e / 7.2 + "px"); // 小米就是6啊 设计图都是安卓 720*1280界面
    };

  n.setAttribute("data-dpr", i),
    t.addEventListener &&
      (e.addEventListener(a, d, !1),
      "complete" === t.readyState ||
        t.addEventListener(
          "DOMContentLoaded",
          function () {
            setTimeout(d);
          },
          !1
        ));
})(window);
```

下图是雷布斯的小米，小米设计图是根据 5 寸（安卓 7201280）设计的，在 720 手机中如图中 icon 的大小为 7276， html 中 font-size: 50px;，计算为 72/50 = 1.44 rem

![](https://upload-images.jianshu.io/upload_images/4030717-514de4082a8412f1.jpg?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)
