li 是很多人做网站都会用到的，但在显示默认效果时前面总是会有一个小黑点，这个效果很多人不想要，但又不知到如何去除，现在我们可以用以下方法来清除默认黑点。

1. 在 CSS 中写入代码。找到相关性的 CSS，在.li 和.ul 下写入 list-sytle:none;当然有的会这样来写 list-style-type:none。

2. 在相关的页面看下实现的代码

```html
</head>
    <style>
        .u li{ list-style:none; }
    </style>
</head>
<body>
    <ul class="u">
        <li>安徽</li>
        <li>山东</li>
    </ul>
</body>
```

这种方式也可以把前面的黑点去掉，达到效果。

3. 在 li,ul 内加入 list-style。如`<ul style="list-style-type:none><li><a herf="#">中国</a></li></ul>` 当然这种是很麻烦的了。
   最简单的就是第一种，通过 CSS 来控制，这个当然会有不错的效果了。

这几种方法都是通过设置 list-style:none 来设置的，有的是会用 list-style-type，下面有一些它相关的属性。

| -- | -- |

-   none 不使用项目符号
-   disc 实心圆，默认值
-   circle 空心圆
-   square 实心方块
-   decimal 阿拉伯数字
-   lower-roman 小写罗马数字
-   upper-roman 大写罗马数字
-   lower-alpha 小写英文字母
-   upper-alpha 大写英文字母 等

这些都可以来代替上文中的 none,想要什么样的都会有一个相应的对应，然后就能得到效果。

-   a.运用 CSS 格式化列表符：

```css
ul li {
    list-style-type: none;
}
```

-   b.如果你想将列表符换成图像，则：

```css
ul li {
    list-style-type: none;
    list-style-image: url(images/1.jpg);
}
```

-   c.为了左对齐，可以用如下代码：

```css
ul {
    list-style-type: none;
    margin: 0px;
}
```

-   d.如果想给列表加背景色，可以用如下代码：

```css
ul {
    list-style-type: none;
    margin: 0px;
}
ul li {
    background: #ccc;
}
```

-   e.如果想给列表加 MOUSEOVER 背景变色效果，可以用如下代码：

```css
ul {
    list-style-type: none;
    margin: 0px;
}
ul li a {
    display: block;
    width: 100%;
    background: #ccc;
}
ul li a:hover {
    background: #999;
}
```

说明：display:block;这一行必须要加的，这样才能块状显示！

f.LI 中的元素水平排列,关键 FLOAT:LEFT：

```css
ul {
    list-style-type: none;
    width: 100%;
}
```
