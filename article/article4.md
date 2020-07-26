现在 GitHub 上流行的开源库极大地节省了开发者从 0 开发的时间，很多公司和个人都在 GitHub 上开源自己的项目，今天我们就来整理一下 Android 开发中一些非常流行的库，也是我们必须掌握的，这样可以使我们在使用到时快速的查找到，这里的总结基本也都是自己在开发中用到的，也就是一些个人的见解，只做参考，不具有权威性

一、网络库

1. Retrofit

Retrofit 是 Square 公司研发的网络请求库，也是目前 Android 最流行的 HttpClient 库之一，越来越多的公司开始使用这个请求库，并且可以完美结合 RxJava，就像官网介绍的一样，Retrofit 是一款类型安全的网络框架，基于 HTTP 协议，服务于 Android 和 Java 语言

GitHub 地址：Retrofit GitHub 地址

2. okhttp

同样 okhttp 也是 Square 公司研发的网络请求库，是一款基于 HTTP 和 HTTP2.0 协议的网络框架，服务于 Java 和 Android 客户端，okhttp 以 21K 的 stars 排在 GitHub 中 android 子标题的第二名，很多公司都在使用，从 Retrofit 2.0 开始内置 okhttp 框架，Retrofit 专注封装接口完成业务需求，okhttp 专注网络请求的高效安全

3. volley

Google 的 Andorid 开发团队也意识到有必要将 HTTP 的通信操作再进行简化，于是在 2013 年度的 Google I/O 大会上推出了一个新的网络通信框架—Volley，Volley 在性能方面进行了大幅度的调整，它是设计目标是适合进行数据量不大，但通信频繁的网络操作，对于数据量大的网络操作就会表现糟糕

GitHub 地址：volley GitHub 地址

4. Fast Android Networking

基于OkHttp的Fast Android Networking能让网络通信变得简洁（不用样板代码），使得开发者能轻易写出通信代码。它是一个轻量级的快速网络通信库。试试看，你会爱上它。

GitHub 地址：FastAndroidNetworking GitHub 地址



二、图片加载库

在 Android 设备上面，快速高效的显示图片是极为重要的，在过去的很长时间里，我们在如何高效的存储图像这方面遇到了很多问题，例如图片太大，但是内存却比较小，但是越来越多优秀开源框架的使用解决了我们这方面的问题，接下来我们来看看这些优秀的开源框架

1. glide

在泰国举行的谷歌开发者论坛上，谷歌为我们介绍了一个名叫 glide 的图片加载框架，作者是 bumptech 这个库被广泛的应用在 Google 开源的项目中，包括 2014 年 Google  I/O 大会上发布的官方 App

GitHub 地址：Glide GitHub 地址

2. fresco

一款管理图片内存的方案，是目前最强大的图片加载框架之一，facebook 的出身证明了它不是重复的制造轮子，在管理图片的内存上以及渐进式加载、加载 gif 都具有独有特性

GitHub 地址：Fresco GitHub 地址

3. picasso

非常强大的图片下载、缓存框架，picasso 更强调的是图片的下载，更重要的是这也是 square 团队的作品，想必提到 square 团队，它出片的东西我们还是非常的放心使用

GitHub 地址：picasso GitHub 地址

4. Android-Universal-Image-Loader

 看到这个，想必有一定经验的 Android 开发者都会非常的熟悉，曾经的图片加载之王当之无愧，15.4k 的 stars 足以证明它的热门，与 glide 不同的是 UIL 提供了大量的配置方式，图片加载状态的回调，加载动画等，以及提供了移动端图片加载框架的缓存思路，三级缓存策略等

GitHub 地址：UIL GitHub 地址

5. PhotoView

 一款 ImageView 展示框架，支持缩放，响应手势，位于图片排行榜的第五位，PhotoView 与上面不同的是图片的展示功能，可以实现类似微信头像的放大功能，还有就是很多 App 的图片显示响应手势按压式如何是现实的，这里 PhotoView 将都可以轻松实现

GitHub 地址：PhotoView GitHub 地址

6. CircleImageView

圆角 ImageView，在我们的 App 中这个想必是太常见了，也许我们可以有无数种展示圆角图片的方法，但是 CircleImageView 绝对是我们在开发时需要优先考虑的，如果你还不知道 CircleImageView，那么你需要赶快去体验它在处理圆角图片时的强大了，相信你肯定会觉得和 CircleImageView 相见恨晚，需要注意的是这个并不是图片加载库，暂且归类放在这里

GitHub 地址：CircleImageView GitHub 地址

关于图片加载库我们就介绍这 6 个，大家可以根据自己的特定情况来选择使用

三、UI

 1. material-dialogs

是一款自定义View框架，如多你还是一个自定义 View 的新人，对 Dialog 使用还有点生疏，那么通过使用 material-dialogs 可以提升你的 Dilaog 使用能力

GitHub 地址：material-dialogs GitHub 地址

2. flexbox-layout

 是一款弹性伸缩布局，FlexboxLayout 作为 LinearLayout 和 RelativeLayout 的替代者，值得大家在项目开发中去尝试使用，毕竟是 Google 出品

GitHub 地址：flexbox-layout GitHub 地址

3. AndroidSwipeLayout

 非常强大的滑动式布局，滑动删除是我们 app 中的常见需求，商品详情的上下滑动需求在实际开发中我们也是经常遇到，AndroidSwipeLayout 在 GitHub 上有 8300 个 stars，证明还是值得使用

GitHub 地址：AndroidSwipeLayout GitHub 地址

4. BaseRecyclerViewAdapterHelper

强大的通用 RecyclerView 适配器，在 GitHub Android 适配器排行榜第一

GitHub 地址：BaseRecyclerViewAdapterHelper GitHub 地址

5. MaterialDrawer

强大的材料风格的抽屉框架，非常灵活，易于使用

GitHub 地址：MaterialDrawer GitHub 地址

6. Android-ObservableScrollView

一款让视图滑动更具有视觉效果的滑动式框架，在 GitHub 上提供了 12 种滑动效果，可以用来提升 App 的滑动体验

GitHub 地址：Android-ObservableScrollView GitHub 地址

7. AppIntro

一款提供快速制作欢迎页的框架，在国内的 App 开发中，ViewPager 开发 App 的欢迎页已经是标配的需求，但是 AppIntro 也是绝对值得你一看

GitHub 地址：AppIntro GitHub 地址

8. ViewPagerIndicator

一款基于 ViewPager 的页面指示器开源框架，作者是 Android 大神 JakeWharton，只是已经很长时间没有更新了，大家可以参考使用

GitHub 地址：ViewPagerIndicator GitHub 地址

好了 UI 相关的库就介绍这么多，以后发现好用的会添加进来，方便查阅

四、动画

1. lottie-android

动画类框架排行榜第一名，一款可以在 Android 端快速展示 Adobe Afeter Effect(AE) 工具所做动画的框架，利用 json 文 件快速实现动画效果是它最大的便利，而这个 json 文件也是由 Adobe 提供的 AE 工具制作的，在 AE 中装一个 Bodymovin 的插件，使用这个插件最终将动画效果生成 json 文件，这个 json 文件即可由 LottieAnimationView 解析并生成绚丽的动画效果，而且它还支持跨平台

GitHub 地址：lottie-android GitHub 地址

2. Material-Animations

 一款提供场景转换过渡能力的动画框架，与 lottie-android 不同的是，Material-Animations 提供的是场景切换的动画效果

GitHub 地址：Material-Animations GitHub 地址

3. AndroidViewAnimations

一款提供可爱动画集的动画框架，在 lottie-android 和 Material-Animations 两个动画框架霸主之后排名第三，可见也是非常厉害

GitHub 地址：AndoridViewAnimations GitHub 地址

4. recyclerview-animators

为 recyclerview 提供扩展动画的框架，recyclerview 已经推出了很长时间，如果你还在使用 ListView，那就说明你老了

GitHub 地址：recyclerview-animators GitHub 地址

五、json 解析框架

1. fastjson

 一款基于 json 解析、生成的框架，是阿里出品，这就保证了代码的质量，在网络请求时使用较多，值得尝试

GitHub 地址：fastjson GitHub 地址

2. GSON

 一个提供Java对象序列化/反序列化至JSON格式的库。

GitHub 地址：gson GitHub 地址

六、内存泄露检测

1. leakcanary

 一款内存检测框架，服务于 Java 和 Andorid 客户端，方便简洁是 leakcanary 最大的特点，只需要在应用的 apllication 中集成，就可以直接使用它，15.9k 的 stars 足够说明它的厉害，最关键是是，它也是 square 团队的作品，就这一条，不用说相信大家也都明白

GitHub 地址：leakcanary GitHub 地址

七、页面路由

1. ARouter

 一款提供服务、页面跳转的路由框架，由阿里出品，该框架提供：从外部 URL 映射到内部页面、跨模块的页面跳转（模块化必备，页面解耦），拦截跳转过程等能力，绝对是一个企业级的开发框架

GitHub 地址：ARouter GitHub 地址

八、数据库框架

1. realm-java

Realm 是一款专门为移动端打造的数据库框架，比普通的数据库更快，力压 greenDAO

GitHub 地址：Realm GitHub 地址

2. greenDAO

greenDAO 是一款高效、快速的 SQLite 型数据库，star 数量和 Realm 不相上下，由 greenrobot 团队开发维护，此团队还有一个很牛的框架便是 EventBus

GitHub 地址：greenDAO GitHub 地址

九、异步 

1. RxJava

RxJava 是 ReactiveExtensions 的 Java VM 实现：用于通过使用 observable 序列来组合异步和基于事件程序的库，它扩展观察者模式以支持数据/事件序列，并添加允许你以声明组合序列的操作符，同时提取对低级线程、同步、线程安全性和并发数据结构等问题的隐藏

GitHub 地址：RxJava GitHub 地址

2. RxAndroid

一款 Android 客户端组件间异步通信框架，位于通信框架排行榜的第二名，仅在 EventBus 之后，两者的区别是 EventBus 是用来取代组件之间繁琐的 Interface，而 RxAndroid 是用来取代 AnsyTask 的，两者并不冲突

GitHub 地址：RxAndroid GitHub 地址

3. agera

Agera 是一组类和接口，可以帮组编写 Android 的功能，异步和无效应用程序，需要 Android SDK 版本 9 或更高，是 Google 官方出品

GitHub 地址：Agera GitHub 地址

4. RxBinding

一款提供 UI 组件事件响应能力的框架，通过 RxBinding 可以理解响应式编程的快乐，让项目的事件流程更加的清晰

GitHub 地址：RxBinding GitHub 地址

十、事件消息

1. EventBus

事件间通信框架 stars 第一，在大型项目的 Activities、Fragments、Threads、Services 都有使用场景，尽管 EventBus 在向未创建的组件传递事件时有些局限，仅适合在活着的组件之间传递消息，但任然不妨碍在各个大型项目的场景中使用

GitHub 地址：EventBus GitHub 地址

十一、图表

1. MPAndroidChart

MPAndroidChart 是一款图表框架，以快速、简洁，强大著称的图表框架，支持线条、饼型、气泡和烛台图，以及缩放、拖动和动画

GitHub 地址：MPAndroidChart GitHub 地址

十二、生成模板代码

1. butterknife

使用注解生成模板代码，将 view 与方法和参数绑定，配合 Android Studio 提供的 ButterKnife 插件，帮组开发者省却了频繁的 findViewById 的烦恼，最新的 ButterKnife 还提供了 onclick 绑定以及字符串的初始化，初学者可以查阅 ButterKnife 以及 ButterKnife 进一步学习，作者是 JakeWharton，是大名鼎鼎的 square 的团队成员之一

GitHub 地址：butterknife GitHub 地址

今天的总结就先到这里，后续会不断更新

十三、其它

1.Device Year Class 

Device Year Class会告知当前设备的内存，CPU核和时钟频率在哪一年的产品线里属于高配。它可以让开发者根据手机的硬件性能来让app做出不同的行为。

GitHub 地址：DeviceYearClass GitHub 地址

2.Network Connection Class

Network Connection Class能够查询当前用户的网络连接质量。它会根据网络质量的不同分成好几种”Connection Classes”（连接分类）让开发更容易。这个库通过监听app已有的网络流量情况并在通信速度改变的时候通知用户。开发者能够通过网络连接情况调节app的行为（比如使用更低质量的影音，停止使用输入提示等等）。

GitHub 地址：NetworkConnection GitHub 地址

3.Android Debug Database

Android Debug Database是一个功能强大的用于调试安卓数据库和共享首选项(shared preference)的库。它是一个在浏览器里浏览数据库和共享首选项的简单易用的工具。

GitHub 地址：DebugDatabase GitHub 地址

4.LeakCanary

LeakCanary是一个安卓和Java上用于检测内存泄漏的一个库。

GitHub 地址：Leakcanary GitHub 地址

5.Dagger

安卓和java的快速的依赖注入库。它简化了对于共用实例的读写，使复杂的依赖设置变的简单，让单元测试和集成测试更加容易。

GitHub 地址：Dagger GitHub 地址

6.Realm

简单存储，高速查询，节省大量开发时间。Realm Mobile Database是SQLite的一个替代品，一个ORM解决框架。

GitHub 地址：Realm GitHub 地址

7.Timber

在安卓原有的Log class之上提供有小型，可扩展API的一个Logger.

GitHub 地址：Timber GitHub 地址

8.Hugo

通过标注触发为你的debug build自动记录方法调用的日志。作为一个程序员，你经常需要加入logging函数打印程序里面的函数调用，以及参数和返回值，并花时间执行。这不是什么问题，我们每个人都这样做。只是能不能够让它变得更简单一些呢？只要在函数头加上@DebugLog你就能得到我们刚才所讲的所有信息。

GitHub 地址：Hugo GitHub 地址

9.Android GPU Image

提供安卓上高效的基于OpenGL的滤镜的库。

GitHub 地址：GPU Image GitHub 地址

10.ExoPlayer

ExoPlayer是一个应用级的安卓媒播放器。它提供的API支持播放本地或者网络上的音频。ExoPlayer支持当前安卓媒体播放器API不支持的功能，比如DASH和Smooth Streaming adaptive playbacks(根据带宽自动实时调节播放分辨率).

GitHub 地址：ExoPlayer GitHub 地址

原文链接：http://blog.csdn.net/qdjdeveloper