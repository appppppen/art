![](https://blog.logrocket.com/wp-content/uploads/2021/01/css-feathers.png)

容器查询是 CSS 中最受关注和要求的功能之一。如此之多，以至于它已成为开发人员社区中的俗话。但是，容器查询到底是什么？

容器查询是可以帮助我们根据容器的内容（例如宽度和高度）来设置其内容样式的查询。这与媒体查询采用的方法不同，媒体查询可以帮助我们根据视口的更改来设置网页/网站的样式。

截至 2020 年，容器查询目前尚未在 CSS 中使用，但是我们真的需要正式添加容器才能获得附带的一些优势吗？

好吧，不完全是。我们可以进行一些调整，以在代码中实现类似容器查询的行为。我们将在下面介绍其中的一些。

## 使用 flexbox 进行容器查询

Flexbox 是一种单向布局模型，可让我们创建响应速度更快的网站。

在本文中，我们将使用一种根植于 flexbox 并由 Heldon Pickering 创建的技术来模仿容器查询。我们的目标是使用 flexbox 创建三列，并将它们全部移至具有特定容器宽度的行。

在首选目录中创建一个名为“ container_query”的文件夹，然后继续在代码编辑器中将其打开。接下来，让我们在“ container_query”文件夹中创建两个文件，并将其分别命名为“ index.html”和“ style.css”。

继续将下面的代码粘贴到您的“ index.html”文件中：

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <title>container queries</title>
  </head>
  <body>
    <div class="container">
      <div class="child child1">
        Lorem ipsum dolor sit amet consectetur adipisicing elit. Ex, enim!
      </div>
      <div class="child child2">
        Lorem ipsum Lorem ipsum dolor sit amet consectetur, adipisicing elit.
        Laudantium aliquid esse fugiat!
      </div>
      <div class="child child3">
        consectetur adipisicing elit. Tempora totam, eos rerum ipsum
        consequuntur suscipit?
      </div>
    </div>
  </body>
</html>
```

在上面的 HTML 代码中，我们创建了一个带有两个其他“ div”作为内容的“ div”容器。

接下来，让我们添加一些基本的 CSS 样式以使其具有形状。

将以下代码添加到“ style.css”文件中：

```css
.child {
  background: rgb(231, 227, 227);
  padding: 1em;
  font-size: 18px;
}
```

现在，让我们开始应用 Heldon 方法。我们将添加所有 CSS 代码，然后一个接一个地解释它们。让我们用以下文件更新我们的 CSS 文件：

```css
.container {
  display: flex;
  flex-wrap: wrap;
  gap: 1em;
}
.child {
  flex-basis: calc(calc(500px - 100%) * 999);
  flex-grow: 1;
  background: rgb(231, 227, 227);
  padding: 1em;
  font-size: 18px;
}
```

我们给具有“ .container”类的容器显示“ flex”和“ flex-wrap”的“ wrap”。这是必要的，因为它可以帮助我们的子元素在必要时弯曲和突破。

主容器的所有子代均被赋予“伸缩增长”（flex-grow）值 1。这使元素可以超出其原始宽度并填充剩余空间。

'flex-basis'属性帮助我们设置所有子元素的理想宽度。请注意，flexbox 并不严格遵循此原则，它会在必要时进行调整。

较大的负“ flex-basis”将导致子元素最终从初始列位置开始占据行位置。

上面我们在'flex-basis'属性中所做的事情是，我们利用'calc（）'来设置'flex-basis'属性，这样它将在 500px 以下的某个地方给我们一个负值。这将导致所有子元素从列位置开始占据行位置。
![](https://blog.logrocket.com/wp-content/uploads/2021/01/flex-basis-property-in-action.gif)
我们可以采用与上述相同的方法，并在第一个子元素中创建两个'p'标签，随着父级宽度的减小，该标签将在列与行之间移动。

让我们使用以下代码更新“ index.html”文件中第一个子元素的内容：

```html
<p>
  Lorem ipsum, dolor sit amet consectetur adipisicing elit. Ducimus,
  perferendis!
</p>
<p>Lorem ipsum dolor sit amet consectetur adipisicing elit. Nostrum, soluta.</p>
```

将以下其他代码添加到我们的“ style.css”文件中：

```css
.child1 {
  display: flex;
  flex-wrap: wrap;
}
```

```css
.child1 p {
  flex-basis: calc(calc(400px - 100%) * 999);
  flex-grow: 1;
}
```

就像在第一个示例中一样，我们利用'flex-basis'属性设置理想宽度，然后在值变为负数时允许带有'p'标签的元素占据行位置。

下面是结果代码：
![](https://blog.logrocket.com/wp-content/uploads/2021/01/flex-basis-property-with-p-tags.gif)

## 使用 CSS 网格进行容器查询

就像 flexbox 一样，我们可以借助 CSS 网格及其属性来实现条件容器的样式。

我们将使用与上面示例中相同的 HTML 代码，因此请继续并在新索引文件上方的示例中复制 HTML 代码。

现在，让我们将以下代码复制到我们的新 CSS 文件中。我将相应地解释每个添加项：

```css
.container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  grid-gap: 1em;
}
.child {
  background: rgb(231, 227, 227);
  padding: 1em;
  font-size: 18px;
}
```

我们首先将显示设置为网格，然后为子容器赋予 1em 的“网格间隙”。在这种情况下，'grid-template-columns'属性发挥了所有作用。我们正在使用'repeat（）'创建一个最小宽度为'300px'，最大宽度为'1fr'，且限制为'auto-fit'的网格。这允许网格填充整个视口。

结果是 3 列布局，当父容器低于'300px'时，该布局切换到行：

![](https://blog.logrocket.com/wp-content/uploads/2021/01/three-column-layout.gif)

## 看箱子

创建监视框的唯一目的是解决无容器查询问题。要使用它，我们可以在安装后将其导入到我们的代码中，如下所示：

```javascript
import WatchedBox from "./path/to/watched-box.min.js";
```

然后，我们可以通过将其包装在我们的 HTML 代码周围来使用它：

```html
<watched-box widthBreaks="70ch, 900px" heightBreaks="50vh, 60em">
  <!-- HTML and text stuff here -->
</watched-box>
```

我们分别使用'widthBreaks'和'heightBreaks'设置宽度和高度断点。

最后，您可以将以下代码添加到 CSS 文件中：

```css
watched-box {
  display: block;
}
```

在[此处](https://github.com/Heydon/watched-box)访问官方 Github 存储库，了解有关受监视盒子的更多信息。

## 结论

尽管容器查询尚未正式添加到 CSS 中，但我们始终可以克服上述任何一种方法都无法使用它们的局限性。Bootstrap 提供了与上述某些解决方案类似的解决方案，但有一定的局限性，您可以在此处阅读有关 bootstrap 的更多信息。
