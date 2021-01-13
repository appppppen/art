> 一行文字时垂直居中，两行时也垂直居中？css 该怎默写？

单行文本，只需要将文本行高(line-height)和所在区域高度(height)设为一致即可：

```html
<!--html代码-->
<div id="div1">这是单行文本垂直居中</div>
```

```css
/*css代码*/
#div1 {
    width: 300px;
    height: 100px;
    margin: 50px auto;
    border: 1px solid red;
    line-height: 100px; /*设置line-height与父级元素的height相等*/
    text-align: center; /*设置文本水平居中*/
    overflow: hidden; /*防止内容超出容器或者产生自动换行*/
}
```

多行文本垂直居中分为两种情况，一个是父级元素高度不固定，随着内容变化；另一个是父级元素高度固定。
父级高度不固定的时，高度只能通过内部文本来撑开。这样，我们可以通过设置内填充（padding）的值来使文本看起来垂直居中，只需设置 padding-top 和 padding-bottom 的值相等：

```html
<!--html代码-->
<div id="div1">
    这是多行文本垂直居中， 这是多行文本垂直居中， 这是多行文本垂直居中，
    这是多行文本垂直居中。
</div>
```

```css
/*css代码*/
#div1 {
    width: 300px;
    margin: 50px auto;
    border: 1px solid red;
    text-align: center; /*设置文本水平居中*/
    padding: 50px 20px;
}
```

父级高度固定的时，设置父级 div 的 display 属性：display: table;然后再添加一个 div 包含文本内容，设置其 display:table-cell;和 vertical-align:middle

```html
<!--html代码-->
<div id="outer">
    <div id="middle">
        这是固定高度多行文本垂直居中， 这是固定高度多行文本垂直居中，
        这是固定高度多行文本垂直居中， 这是固定高度多行文本垂直居中。
    </div>
</div>
```

```css
/*css代码*/
#outer {
    width: 400px;
    height: 200px;
    margin: 50px auto;
    border: 1px solid red;
    display: table;
}
#middle {
    display: table-cell;
    vertical-align: middle;
    text-align: center; /*设置文本水平居中*/
    width: 100%;
}
```

flex 的垂直居中，这种做法可以不定义内部元素的高度

```html
<div class="mod">
    <div class="inner">居中</div>
</div>
```

```css
.mod {
    display: flex;
    align-items: center; /*垂直居中*/
    height: 300px;
}
.inner {
}
```

![](https://segmentfault.com/img/bV5Rsf?w=551&h=394)

-webkit-box

```html
//html
<div>
    <p>n行文字n行文字n行</p>
</div>
```

```css
//css
div {
    height: 100px;
    display: -webkit-box;
    -webkit-box-align: center; /* 垂直居中 */
    -webkit-box-pack: center; /* 水平居中 */
    background: #eee;
}
```
