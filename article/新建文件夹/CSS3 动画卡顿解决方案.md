前端时间用animation实现H5页面中首页动画过渡，很简单的一个效果，首页加载一个客服头像，先放大，停留700ms后再缩小至顶部。代码如下

```html
<!DOCTYPE html>
<html>
<head lang="zh-cn">
    <meta charset="utf-8">
    <meta name="viewport" content="width=device-width,initial-scale=1.0,maximum-scale=1.0,user-scalable=1" >
    <script type="text/javascript" src="http://apps.bdimg.com/libs/jquery/2.1.1/jquery.min.js"></script>
    <title>首页加载动画</title>
    <head>
        <style>
            .welcome-main{
                display: none;
                padding-bottom: 40px;
            }
            .top-info{
                width: 100%;
                position: absolute;
                left: 0;
                top: 93px;
            }
            .wec-img{
                width: 175px;
                height: 175px;
                position: relative;
                padding: 23px;
                box-sizing: border-box;
                margin: 0 auto;
            }
            .wec-img:before{
                content: '';
                position: absolute;
                left: 0;
                top: 0;
                width: 100%;
                height: 100%;
                background: url("./images/kf-welcome-loading.png");
                background-size: 100%;
            }
            .wec-img .img-con{
                width: 100%;
                height: 100%;
                border-radius: 50%;
                /*box-sizing: border-box;*/
                background: url("./images/kf_1.jpg");
                background-size: 100%;
                padding: 1px;
            }
            .wec-img .img-con img{
                width: 100%;
                height: 100%;
                border-radius: 50%;
            }
            .loaded .wec-img{
                -webkit-transform-origin: center top;
            }               
            .loading.welcome-main{
                display: block;
            }
            .loading .wec-img{
                -webkit-animation:fadeIn .3s  ease both;
            }
            .loading .wec-img:before{
                -webkit-animation:rotate .6s .2s linear both;
            }
            .loaded .top-info{
                -webkit-animation:mainpadding 1s 0s ease both;
            }
            .loaded .wec-img{
                -webkit-animation:imgSmall 1s 0s ease both;
            }
            @-webkit-keyframes mainpadding{
                0%{
                    -webkit-transform:translateY(0)
                }
                100%{
                    -webkit-transform:translateY(-87px)
                }
            }
            @-webkit-keyframes imgSmall{
                0%{
                    width: 175px;
                    height: 175px;
                    padding: 23px;
            
                }
                100%{
                    width: 60px;
                    height: 60px;
                    padding: 0;
            
                }
            }
            @-webkit-keyframes fadeIn{
                0%{opacity:0;-webkit-transform:scale(.3)}
                100%{opacity:1;-webkit-transform:scale(1)}
            }
            @-webkit-keyframes rotate{
                0%{opacity:0;-webkit-transform:rotate(0deg);}
                50%{opacity:1;-webkit-transform:rotate(180deg);}
                100%{opacity:0;-webkit-transform:rotate(360deg);}
            }
          </style>
        <body>
            <div class="welcome-main">
                <div class="top-info">
                    <div class="wec-img"><p class="img-con"><img src="" alt=""></p></div>
                </div>
            </div>
            <script>
                $('.welcome-main').addClass('loading');
                setTimeout(function(){
                    $('.hi.fst').removeClass('loading');
                    $('.welcome-main').addClass('loaded');
                },700);
            
            </script>
        </body>
    </html>

```

在chrome上测试ok，但在提测给QA的时候发现部分机型，如华为，系统4.2，oppo系统5.1的出现卡顿情况。

百思不得其解，后来参考文章[深入浏览器理解CSS animations 和 transitions的性能问题](http://blog.csdn.net/leer168/article/details/25917093)一文，将图片缩放中动画元素改成transform，如下
```css
@-webkit-keyframes imgSmall{
    0%{
        -webkit-transform:scale(1);
    }
    100%{
        -webkit-transform:scale(.465);
    }
  }
```

果然啊，卡顿问题解决了。

文章深入浏览器理解CSS animations 和 transitions的性能问题是这么解释的，现代的浏览器通常会有两个重要的执行线程，这2个线程协同工作来渲染一个网页：主线程和合成线程。

一般情况下，主线程负责：运行JavaScript；计算HTML 元素的 CSS 样式；页面的布局；将元素绘制到一个或多个位图中；将这些位图交给合成线程。

相应地，合成线程负责：通过 GPU将位图绘制到屏幕上；通知主线程更新页面中可见或即将变成可见的部分的位图；计算出页面中哪部分是可见的；计算出当你在滚动页面时哪部分是即将变成可见的；当你滚动页面时将相应位置的元素移动到可视区域。

假设我们要一个元素的height从 100 px 变成 200 px，就像这样：
```css
div {
    height: 100px;
    transition: height 1s linear;
}

div:hover {
    height: 200px;
}
```

主线程和合成线程将按照下面的流程图执行相应的操作。注意在橘黄色方框的操作可能会比较耗时，在蓝色框中的操作是比较快速的。

![](https://segmentfault.com/img/remote/1460000006760850)

而使用transform:scale实现
```css
div {
    transform: scale(0.5);
    transition: transform 1s linear;
}

div:hover {
    transform: scale(1.0);
}
```
此时流程如下：

![](https://segmentfault.com/img/remote/1460000006708780)

也就是说，使用transform,浏览器只需要一次生成这个元素的位图，并在动画开始的时候将它提交给GPU去处理 。之后，浏览器不需要再做任何布局、 绘制以及提交位图的操作。从而，浏览器可以充分利用 GPU 的特长去快速地将位图绘制在不同的位置、执行旋转或缩放处理。

为了从数量级上去证实这个理论，我打开chrome的Timeline查看页面FPS

![](https://segmentfault.com/img/bVCjnD)

其中，当用height做动画元素时，在切换过程的FPS只有44，我们知道每秒60帧是最适合人眼的交互，小于60，人眼能明显感觉到，这就是为什么卡顿的原因。

![](https://segmentfault.com/img/bVCjoo)

rendering和painting所花的时间如下：

![](https://segmentfault.com/img/bVCjow)

再来看看用transform:scale

![](https://segmentfault.com/img/bVCjoA)

FPS达到66，且rendering和painting时间减少了3倍。

到此为止问题是解决了，隔了几天，看到一篇[解决Chrome动画”卡顿”的办法](http://www.cnblogs.com/xdoudou/p/4524758.html)，发现还能通过开启硬件加速的方式优化动画，于是又试了一遍
```css
webkit-transform: translate3d(0,0,0);
-moz-transform: translate3d(0,0,0);
-ms-transform: translate3d(0,0,0);
-o-transform: translate3d(0,0,0);
transform: translate3d(0,0,0);
```

惊人的事情发生了，FPS达到72：
![](https://segmentfault.com/img/bVCjpw)

![](https://segmentfault.com/img/bVCjpz)

## 总结解决CSS3动画卡顿方案
尽量使用transform当成动画熟悉，避免使用height,width,margin,padding等；

要求较高时，可以开启浏览器开启GPU硬件加速。