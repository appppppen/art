最近碰到一个 CSS3 的属性，继而深感 CSS 的强大，又接着深入探索了一下 CSS 的动画与 JS 的动画。因此，本篇文章就从一个神奇的属性开始谈起。

老规矩，推荐看的文章放在开头：[A](https://www.smashingmagazine.com/2016/12/gpu-animation-doing-it-right/)，[B](http://taobaofed.org/blog/2016/04/25/performance-composite/)

文章的开头先插一个与本文主题无关的东西，但真的是很有用的一个 CSS3 特性。就从 mouseover 入手吧。

## mouseover mouseenter
这两个属性的区别我想不用多说了，但显然一般情况下我们的需求都是 mouseenter。但是！一个 css 属性能让 mouseover 展现出 mouseenter 的特性。

css 中很多属性，也包括很多属性中的值都是参考 svg（饿，我还没去了解）。pointer-events 就是其中之一，但是 css 中只支持两种，一种是 auto，一种是 none，默认情况下都是 auto。当给一个元素设置了 pointer-events: none 之后，当前元素及其所有子元素都不会再响应任何鼠标事件。

也就是默认情况下所有子元素会继承这个 none，但是子元素取消时，还有点区别。举个例子 `<div><p></p></div>`，我们给 div 注册一个点击事件，又给 p 注册一个点击事件，给 div 加上 pointer-events: none ，给 p 加上 pointer-events: auto ，此时当我们点击 p 的时候， p 和 div 都会响应，也就是事件的 path 中依然在冒泡中会被 div 捕获，这点要注意。当然，若是点击的是 div 不是 p，那么是不会有反应的。

你可能会觉得这有一些奇怪的感觉，那么先不管，一会再谈，我们先看给子元素设置成 none 会怎样，此时就一点不奇怪了。此时我们给父元素绑定 mouseover，但是由于子元素不会响应鼠标事件，因此表现就和 mouseenter 一模一样，如果此时你想切换成 mouseenter 效果，那么只需要变换 css 属性即可。

需要注意的是，当子元素 none，父元素 auto，二者都注册了点击事件时，当你点击子元素的时候，父元素一样会响应的哦，至于为什么，也包括上面的那种奇怪的感觉，其实这本质上还是因为事件流，好好理解一下事件流，浏览器下 console.log(event.path) 你就明白了。

## 从一个 canvas 时钟入手
代码见 [codeOpen](https://codepen.io/cyboning/pen/daajeL) ，这是为了学习 canvas 而写的一个时钟。当然这里不会去谈 canvas 具体的基础学习，而是默认为都有了一些关于 canvas 的基础。这里谈的是我代码中用到的一些关于 canvas 性能的优化。

在一系列考虑中，其中最关键的莫过于是 offscreen canvas，也就是将不变的内容绘制在内存中的 canvas（非显示在页面上的 DOM），每次我们通过调用页面上的真实存在 DOM 的 canvas 的 drawImage 方法，把内存中的 canvas 当做 img 一样绘制在 DOM 上，性能会大幅提升，最新实验标准已经有直接提供离屏 canvas 的 api。

除此之外，offscreen canvas 也并非只能是不变的，它也可以拥有图形变化，我们可以将其交给 web woker 来做，使用 OffscreenCanvas api，注意此时是不涉及 DOM 的，之后得出了图片以后将其交给主线程就好。除了最关键的离屏之外，还有一些有效的性能优化点。

- 多层画布 canvas，通过 z-index 调控。图片分层，也就是下面层级的相当于背景图，不会变，或者相对变得少。
- 像素级别操作尽量用整数，还有注意一点，例如从 (0,0) 到 (5,0)， lineWidth 为 1，实际上并不会画出真实的 1 的效果。因为从 1 开始两边分别扩展 0.5 为 1，之后的 0.5 为模糊段。
- requestAnimationFrame
- 清除画布的时候，尽可能少的清除，也就是重新渲染的范围尽量小。
- 尽量少改变 canvas 状态
关于 canvas 基本上到此结束，但由此就引申出了更多的内容。事实上，上面提出的有几点性能优化我比较在意，我觉得这可能是一块通用的概念，毕竟浏览器说简单点，给我的感觉就像是一个大的 canvas 在渲染。

多层画布？清除范围减小？离屏？

现在有了这些初识，就可以真的进入正题了。

## 如何看性能
再谈动画性能衡量之前，先说一下我是如何衡量的。打开 chrome 调试面板，点击右上角的竖三点，鼠标移动到 More tools 上就可以看到工具了。本篇中主要用到了 Layers，Rendering 以及 Performance monitor。

## 渲染，重绘，回流
请注意以下内容的两大主角

1. GPU
2. composition
大家都知道 CSS 动画性能要比 JS 动画性能好。原因在哪里呢？在于上面提的两大主角，先来了解一下 GPU。

GPU 即图形处理器，专门处理和绘制图形相关的硬件。基本上就是为了让 CPU 尽可能少地参与图形处理，更好地进行其他任务。而将图形处理交给 GPU 来做，即称之为硬件加速。

通俗一点理解 GPU 的话，就是显卡中包含着显存和 GPU，而 GPU 与显存的关系就像是 CPU 与内存。每一张显示屏上的图像最后都是由显卡呈现给我们的，而显卡的心脏就是 GPU。

好了，言归正传。现在感觉是个前端都会知道尽量避免重绘和回流。但事实上有一点必须要注意，浏览器几乎无时无刻都在进行重绘，浏览器总是需要将最终绘制好的一份位图交给显存。因此，这里将浏览器把一个 DOM 所表现的各种外形特征渲染到一个图层称之为渲染，而最终将多个图层（事实上应该称之为合成层（composition），这里为了方便理解暂且称之为图层）共同绘制成一幅图片称之为绘制（这个过程也称之为层合成）。而常提的重绘和回流，事实上都是在浏览器渲染 DOM 到图层上的这一过程中所发生的事情。下面，就具体的来了解下合成层。

先从上面提到的多层开始说起，浏览器构建 DOM 树，构建 CSSOM 树，两者对应起来后继而渲染页面，这是最笼统的概念，但是这可有讲究了。[打开代码](https://codepen.io/cyboning/pen/WPPzmj)，将其代码复制后用本地浏览器打开（因为我在测试过程中，发现上述的调试在 codeopen 中经常导致网页崩溃）。

代码一开始单纯使用 left 动画，打开浏览器下的性能监控，可以看到
![](https://cyboning.github.io/2019/02/20/%E7%94%B1canvas%E5%BC%95%E5%8F%91%E7%9A%84%E5%AF%B9%E6%B5%8F%E8%A7%88%E5%99%A8%E6%B8%B2%E6%9F%93%E7%9A%84%E6%8E%A2%E7%A9%B6/leftWithoutWill.jpg)

此时只有一个合成层，即 document 根元素，动画在不停地进行，浏览器自然也在不停地渲染动画。还可以看到网页上多了几条线（Rendering 下的 Layer borders），这是因为浏览器会将一整个页面划分成几个部分，仅仅对需要重新渲染的地方的进行重新渲染，而没改动的地方是不需要重新渲染的，在缓存中直接提取性能会大幅上升，这一点原理与 canvas 尽可能少的擦除原则是类似的。

现在，取消掉 will-change 注释
![](https://cyboning.github.io/2019/02/20/%E7%94%B1canvas%E5%BC%95%E5%8F%91%E7%9A%84%E5%AF%B9%E6%B5%8F%E8%A7%88%E5%99%A8%E6%B8%B2%E6%9F%93%E7%9A%84%E6%8E%A2%E7%A9%B6/leftWithWill.jpg)

可以看到 document 下多了一层 div，注意看一下 Compositing Reasons ，也就是合成层存在的原因是由于 wilChange。除此之外，我们还可以看到此时相比于原来多用了 17+ KB 的内存，可以看到相较于原来会有些不一样的地方，这是因为在 will-change 出现了以后，浏览器会专门给一个合成层，它会企图将这个层交给 GPU 渲染，而只需要 CPU 传递一定的数据过去即可。遗憾的是，这里使用的 left 是无法实现的，它只能多出一个合成层，具体为什么不能交给 GPU 来渲染，之后会解释。但就算如此，单独提升一个合成层也是对性能有帮助的。

以上面情况为例，假设 document 所在的底部合成层是个图形很复杂的画面，但是是静态的，或者变化频率很低，范围很小。然而我们频繁变动的只有小红块，也就是 CPU 每次只需要将小红块渲染到一个单独的合成层，之后将两个合成层统一绘制成一幅图呈现出来即可（现代高级浏览器绝对会将底层静态的图像缓存）。这种做法跟 canvas 中在 offscreen canvas 渲染好一幅图像后，页面上的 canvas 最后利用 drawImage 将离屏 canvas 当做一幅普通的背景图片并与自身的内容综合绘制成一幅图像的做法很相似。

现在有必要提一下之前说的重绘和回流了，上面之所以不会送给 GPU 来渲染，就是因为能送给 GPU 渲染是有个前提的，即不发生回流和重绘。你可以在 performance 下看看详细的过程，你或许会感到奇怪，这里的 left 是单位是 px 啊？可是你再想想就会发现 position-absolute 的定位是取决于父容器的。别急，之后我们就会看到一个采用 GPU 渲染的例子。

但不管怎么说，上面的例子中虽然会不断回流，但是这里的回流仅仅针对于另一个合成层，一个专门为小红块而生的合成层，不会影响到其他合成层。那么事实上这时候性能比起之前绝对有提升，而当你没有分离层，让小红块就在底层上不停地变动 left，此时的回流性能消耗相对来说会更大。当然也并不是只有好处，你要注意到此时需要将两个合成层一起传给 GPU 进行整合后绘制成一幅图，这对于性能是有影响的，并且额外的空间开销也必须注意。

现在再将 will-change 注释掉，打开 transform 注释。此时再看浏览器，会发现情景大体还是与之前一样，唯一不同的是你需要注意一下 Compositing Reasons ，此时变成了 composition due to association with an element with a css 3D transform。

## composition
上面提到了两种情形都会让浏览器新建一个合成层，具体还有哪些情况会让浏览器新建一个合成层呢？这里有必要提一下 z-index。关于 z-index，要明白一点它的比较都是同级间的比较，注意这个同级不是 DOM 上的同级，而是 z-index 上的同级。看代码

```html
<div id="div1">
  <p id="p1"></p>
  <p id="p2"></p>
</div>
<div id="div2"></div>
```
如果我给 div1 设置了 z-index 为10，div2 设置了 z-index 为 5。那么不管 p1， p2 的 z-index 是多少，它们的层级都是大于 div2 的（已经默认都处于定位状态）。在 div1 的基础上，p1，p2 的 z-index 才会进一步进行比较。

我个人是这样理解的，浏览器在一开始进行渲染的时候，是依据 z-index 来进行比较确定谁能覆盖住谁的，当同级间的 z-index 确定了，也就是此时已经哪个渲染在哪个之上了，那么显然下一级的 z-index 只能基于上一级排好的结果再进行比较。需要额外注意的是，单纯使用 z-index 并不会触发合成层，z-index 只是让浏览器在初始化渲染的时候能够排列好每一个 DOM 在页面上的层级（即垂直于屏幕离人的距离）。

但是有一种情况下 z-index 是可以触发合成层的（ 下面这段英文实际上并不是完全准确的，但这里可以先这么理解 ）。

Element has a sibling with a lower z-index which has a compositing layer (in other words the it’s rendered on top of a composited layer)

现在，请把 p 标签的注释取消掉，然后你就可以看到每当小红块移动到 p 标签所在位置时，浏览器就会为 p 标签动态专门渲染一个合成层，此时查看 Compostion reason，为 lazyForSquashingContents，而当小红块离开了 p 标签所在位置时，浏览器又会放弃专门为 P 标签渲染一个合成层，原因不言而喻。

这 实在 是有意思，但稍微一想也很容易理解，如果此时不将 p 标签单独放置在一个合成层，那么最终这几个合成层需要结合绘制成一幅图的时候，document 所处的合成层在上还是小红块所处的合成层在上都不合适。

好了，具体还有其他可以触发合成层的条件请自行搜索就好，条件很多，具体明白浏览器存在这个特点，需要性能优化时自己边调试边查文档也就知道了。

当然，离文章结束还早着呢（下面还会再提到 z-index），只是关于 left 的动画到这里就可以结束了，并且现在你知道了 定位 的时候浏览器默认是不会创建一个额外合成层的哦，此时发生的渲染过程很可能有一些是没必要的。此外，还要注意到可能你无意间会创造了更多你本不想要的合成层（上面的 p 标签），每个合成层都需要占用额外的存储空间，且合成层间的最终组合绘制成一幅图需要交给 GPU 来完成，这对网页性能都会有影响。

## CSS 动画
现在请你将 CSS 中的 animateByleft 改为 animateByTranslate ，将 will-change，translateZ(0) ，p 标签都注释掉。之后再到浏览器看一看，你会发现这回居然直接就有了一个合成层！这可太神奇了，老规矩看一看 compostion reason，这回是 activeTransformAnimation，此时你再定睛一看，你发现调试面板下的 paints 图像是固定不变的！

这是因为 CSS 动画默认是启用 GPU 来完成的，浏览器将 CSS 动画所需的各项数据计算以后（不会变化）就将数据交给显存，此后都是由 GPU 固定来渲染这个 CSS 动画到一个专属于它的合成层上，而不是由之前的 CPU 来完成，这一点正是 CSS 动画性能优异的关键所在，这也是为什么你调试面板下看到的是一个不变的图（ CPU 初始计算后第一次渲染一幅图交给 GPU 呈现给用户，第二次之后才能开始分离层级将能交给 GPU 搞定的交给 GPU，此时也会引起回流，重绘要注意）。而最后 GPU 会将自身渲染的和 CPU 提交过来的统一绘制成一幅图（这个例子中即 CPU 渲染一个合成层，GPU 渲染一个合成层，最后两个合成层由 GPU 绘制成一幅图片，当然，因为例子中的底层是不变的，因此实际上浏览器肯定做了对应的缓存优化）。

那么是不是可以认为 CSS 动画性能一定好呢？来，看看上一段我加粗的默认两个字，现在让我们来看个坑。请你将 animateByTranslate 动画中的 300px 都改为 300%。此时你就会发现调试面板中的图又动起来了，并且从 performance monitor 面板下可以明显看到 cpu 使用率大幅上升。

让我们来直观对比下，去 performance 下详细看一看用百分比做单位和用 px 做单位的不同。你会发现用 px 做单位的时候重绘和回流居然为零！而用百分比作单位的时候实际上回流和重绘所占百分比就跟你用 left 做动画的时候一样。还记得之前说的吗，若想使用 GPU 硬件加速，那么至少对于这个合成层而言，它不应当发生回流和重绘，此时渲染画面的工作应该是由 GPU 来完成的！

当你使用了百分比做单位，那么每一次都需要 CPU 来进行计算后才能知道如何移动，而不像 px 为单位是固定的，固定的则会在第一次计算好后就缓存在显存中，CPU 不再参与，注意此时的损耗已经增加了，每次都要发生回流和重绘，进而将数据传递给显存后，GPU 才能进行渲染画面。（注意对于 left 动画而言，不管是百分比还是 px 都会发生回流）。

现在你应该明白并不要固定认为 CSS 动画性能一定好，要明白 CSS 动画好的原因，才能知道怎么利用好它。当然，有时候我们为了维护方便，使用百分比做单位，但是你现在可以看到了，这对于性能可能是一个隐患，这里要做好取舍。

要注意：不管是 left 还是 transform ，只要合成层出现，那么 GPU 总是会参与，因为此时不管你是否成功利用 GPU 来绘制合成层，此时都需要 GPU 来统一将这些合成层绘制成一幅图片，GPU 是处理图像的专家，因此牺牲一些数据传输的时间还是可以接受的。

## 层爆炸与层压缩
现在请你取消掉 p 标签的注释，并且在 p 标签的 CSS 中添加 top: 300px，（目的就是为了让小红块不与文字发生重叠）。

然后你就可以发现， p 标签居然有属于自己的合成层？老规矩，看看 composition reason，这里比较长，我就不贴了。但这里有个英文单词 may 着实有意思。这个单词着实说明了问题。事实上我们从 Layers 下可以知道因为 translate 此时采用的是百分比，因此应该是由 CPU 计算的，进而导致发生了大量的回流和重绘。但遗憾的是应该是它的 composition reason 为 activeTransformAnimation，进而导致浏览器默认还是认为这个层属于 GPU 渲染，因此 CPU （Chrome）认为我可没办法实时知道小红块的位置，并且 p 标签的默认层级是在这个小红块之上的（也就是如果发生重叠，那么 p 标签将是覆盖小红块来显示的）。因此我只能为 p 标签再来一个合成层了。

这可能是 chrome 的一个不足之处，这种情况下我认为可以类似于内存泄漏。首先，它不像 left 的情况下，所有合成层都由 CPU 渲染完毕再交给 GPU 进行最后的合成，因此可以知道什么时候该为 p 标签创建合成层，什么时候不必要。其次，此时明明全是由 CPU 干的活（此时发生了大量的回流和重绘），也就是它明明知道位置信息，可是却装作不知道（可能 chrome 源码中是基于 Composition Reason 来判断的），进而导致 chrome 认为这种 分工合作 的情况下，我始终需要为 p 标签渲染这样一个合成层以免网页发生 BUG。

我们来数一数这种情况下性能有多么恶劣。

- 因为使用百分比作为 CSS 动画单位，导致必须要 CPU 每次计算后进而发生回流和重绘，这并不是想象中的 CSS 动画
- 因为 chrome 默认认为 transform 是会采用 GPU 来渲染画面的，且 p 标签层级默认比 GPU 渲染的画面的层级高，因此必须为 p 标签建立一个多余的合成层，结果造成了多余的画面渲染，多余的内存空间占用。
想象下如果再多一些这种莫名其妙的不知所谓的合成层，那网页基本可以说直接炸了，这种情况称之为层爆炸，或者基于合成层多余内存的特点，也可以认为是内存大量泄漏。RUA，这里就是一个非常非常大的性能提升点了。首先，我建议这种情况下不要使用百分比作为 CSS 动画单位。当然，这种情况下是否真的交给 GPU 渲染事实上无关紧要，因为这对于性能提升很有限，关键在于接下来的第二点！

z-index！之前的 z-index 又闪亮登场了，请取消掉 CSS 中 z-index 注释。此时在浏览器下观察，你就会发现那个莫名其妙的合成层消失啦！稍微思考一下你就明白了，因为此时 CPU 在第一次初始计算的时候就已经知道，这个该 GPU 渲染的合成层的层级将要大于这个 p 标签所渲染的图形的层级，用户又没有显示为 p 标签创建一个合成层，那么直接为 p 标签和根元素创建一个合成层即可，反正都在 GPU 渲染出的合成层的下方。这就是层压缩。

现在，你应该明白，当你固执认为 CSS 动画一定优于 JS 动画时，可能无意中走入了一个大坑–层爆炸，而导火线是两个因素： GPU 硬件加速与浏览器的多合成层渲染机制。

总之，现在我们起码知道，浏览器渲染应当是经过这样的流程

- 确定 DOM 树与 CSSOM 树。
- 为 DOM 树上的每个节点找到匹配的 CSSOM 上的相应信息，即完成 styleObj 的匹配，构建出 render tree（渲染树并不等于 DOM 树，例如 display:none 的是不存在于渲染树中的）。
- 基于渲染树进行 layout，确定任何一个渲染层上的所有 DOM 的排列方式。因为所处渲染层上的任何一个元素的布局发生变化，都有可能联动地引发其他元素的布局发生变化。
- 将每个渲染层根据上一步 layout 的结果渲染绘制出来。
- 将所有渲染层进行 composite，最终由 GPU 显示出一幅图片呈现给用户。
上面的3，4，5步都是性能的真正消耗点，尤其是第三步与第四步，就是前端所熟知的回流与重绘（当不交给 GPU 来做的时候）。因此尽可能希望交给 GPU 来做，注意交给 GPU 的时候，JS 主线程与其互不干扰，这意味着当官 JS 主线程在一次 event loop 中遇到大量计算，进而阻止了 UI 渲染线程的时候，浏览器依然能正常呈现给用户一幅在动的动画。

## 关于 translateZ(0) 与 will-change
上面的例子中一直使用这两个属性，但并没有详细介绍，现在来具体说一说。除了利用 translateZ(0)（注意如果在一开始调用 translatex(100px)，这和你浮动后 left:100px 可以认为是一个尿性，因为它们都会转化成位置信息存储在对应 DOM 节点的 styleObj 中，静态的根本不需要额外合成层） 这种隐式欺骗浏览器进而启用额外合成层的情况，还有一种是显示告诉浏览器需要启用额外合成层的情况，即 will-change，而是否采用 GPU 来渲染，这个取决于是否值需要 CPU 来计算。此外，需要注意这部分内容非 W3C 规范，属于每个浏览器自己的实现 ，因此这里有必要提一下 mdn 上面的红字。


`Important: will-change is intended to be used as a last resort, in order to try to deal with existing performance problems. It should not be used to anticipate performance problems.`

will-change 有两个类型的值，第一类就只有一个 auto，它就是让浏览器去猜，启发式。基本这个值我认为是不要用（从下面的 mdn 上给的例子你就明白了为什么不要用了）。第二类则是 animateable-feature，具体分为以下几种：

1. scroll-postion，它表示接下来将要改变元素的滚动位置，希望能提前做些优化，这个不用说也知道很有用的。

2. contents，它表示元素接下来的内容可能会发生变化。

3. custom-ident，它就表示显示指定要变化的 css 的属性名或者值，例如 will-change: transform。也就是所有跟 transform 相关的，例如 translate，rotate，浏览器都需要注意。

提到这里，有必要提一下 mdn 上的一个典型的反例
```css
.sidebar {
  will-change: transform;
}
```
The above example adds the will-change property directly to the stylesheet, which will cause the browser to keep the optimization in memory for much longer than it is needed and we’ve already seen why that should be avoided.

那么正确的做法是什么呢？
```javascript
var el = document.getElementById('element')
// Set will-change when the element is hovered
el.addEventListener('mouseenter', hintBrowser)
el.addEventListener('animationEnd', removeHint)

function hintBrowser() {
  // The optimizable properties that are going to change
  // in the animation's keyframes block
  this.style.willChange = 'transform, opacity'
}

function removeHint() {
  this.style.willChange = 'auto'
}
```
即动态添加撤销，在真正需要的时候添加它，不要让浏览器时时为了你的优化预留更多不必要的内存空间。总之，结合我之前讲的辣么多也就明白了，其实本质上就是尽可能让浏览器去智能化地实现合成层，因为浏览器会知道什么时候该释放。人工显示设置的合成层很可能会忘记释放（就跟传统的 C 语言什么的一样，申请了空间要释放，要不内存泄漏）。注意这三点

- will-change 应该是所有性能优化的最后一步
- 当有元素需要这个属性的时候为其单独添加，不要滥用
- 用完立即删除，不要一直保留
## 最后
注意上面的测试用例均在 chrome 68 下。并且从上面的讨论已经可以知道事实上对于 transform 的处理是有 BUG 的（当然，如果不使用百分比，使用 px 为单位，那么一样会发生层爆炸，此时可不属于 chrome 的处理 BUG，而是我们人为处理不当）。还要强调下 mdn 对于 will-change 的说法就是 optimization，因为每个浏览器都有自己的优化手段不尽相同，这也不属于 w3c 的规范。因此本篇文章很多都是来源于个人实际测试后猜测得出的结论，并不一定准确。主在了解浏览器的多层渲染机制，在具体实践中探索性能优化才合适。

此外，还有一些优化手段例如 transform: scale(xx) 这种减小合成层所占用内存大小的选项，具体可以查看开头的第一篇参考文章。还可以在 [CSS triggers](https://csstriggers.com/) 上查看每个属性在各个浏览器下的不同实现，可以看到 transform 在 webkit 引擎上原生就会触发重绘和回流，那么显然并不会默认开启 GPU 加速，而上面说 blink 不会触发回流与重绘，但是你已经可以从我上面的例子中发现，事实上当你使用百分比做单位时，是会发生回流和重绘的。