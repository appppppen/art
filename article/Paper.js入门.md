> [paperjs](http://paperjs.org/examples/boolean-operations/)


您可能没有听说过Paper.js 。 因此，让我们从问题开始：什么是Paper.js？ 它是允许您创建和使用矢量图形的库。 官方网站将其描述为矢量图形脚本的瑞士军刀。

尽管图书馆提供了很多东西，但是即使您从未听说过它，也很容易学习。 在本教程中，我将从库的基本知识开始，然后再转到复杂的主题。

## 使用PaperScript
有两种使用库的方法。 您可以使用PaperScript，它是JavaScript的扩展，可以帮助您更快地完成工作，也可以只使用普通的旧JavaScript。

PaperScript与您一直使用的旧JavaScript相同。 但是，它增加了对point和size对象的数学运算符的支持。 它还简化了Project ， View和鼠标Tool对象的事件处理程序的安装。

加载PaperScript时，必须使用通常的脚本标签，其类型设置为“文本/ paperscript”。 如果要从外部加载代码，则还需要添加带有适当URL的`<script>`标记以加载代码。 您需要指定的最后一个属性是data-paper-canvas="canvasId" ，或简写版本canvas="canvasId" ，它告诉库需要处理的画布。 下面的代码在PaperScript中创建一个四边形。
``` javascript
<script type="text/paperscript" canvas="quad">
  var path = new Path();
  path.strokeColor = 'black';
  var pointOne     = new Point(100, 20);
  var pointTwo     = new Point(-100, 100);
  var pointThree   = new Point(300, 30);
  path.moveTo(pointOne);
  path.lineTo(pointOne + pointTwo);
  path.lineTo(pointTwo + pointThree);
  path.lineTo(pointOne + pointThree);
  path.closed = true;
</script>
```
## 使用JavaScript
如果您对PaperScript不满意，也可以在项目中使用JavaScript。 如果您决定采用这种方式，则必须再添加几行代码。 您需要做的第一件事是检查DOM是否准备就绪，因为在此之前您将无法使用画布。 之后，您可以使用paper对象设置项目和视图。 现在，所有Paper.js类和对象都只能通过paper对象访问。

正如我之前指出的，在使用Point和Size时，必须使用Math函数而不是运算符。 下面的代码说明了所有这些区别：
``` javascript
window.onload = function() {
  var canvas = document.getElementById('quad');
  paper.setup(canvas);
  var path = new paper.Path();
  path.strokeColor = 'black';
  var pointOne     = new paper.Point(100, 20);
  var pointTwo     = new paper.Point(-100, 100);
  var pointThree   = new paper.Point(300, 30);
  path.moveTo(pointOne);
  path.lineTo(pointOne.add(pointTwo));
  path.lineTo(pointTwo.add(pointThree));
  path.lineTo(pointOne.add(pointThree));
  path.closed = true;
  paper.view.draw();
}
```
从上面的代码片段可以明显看出，在使用Paper.js时，使用PaperScript相对容易。 因此，从现在开始的所有示例都将基于PaperScript。

## 项目层次结构
如果您曾经使用过Adobe Photoshop或Illustrator之类的图形设计应用程序，则必须熟悉图层的概念。 这些程序中的每一层都有自己的内容，当与其他层结合时，它们会产生最终结果。 Paper.js中也存在类似的层，可以使用project.layers进行访问。

最初，每个项目都有一个可通过project.activeLayer访问的单层。 您创建的任何新项目都将作为其子级添加到当前活动层中。 可以使用活动层的layer.children属性访问特定层中的所有子级。

有多种方法可以访问所有这些孩子。 如果仅需要访问任何项目的第一个和最后一个孩子，则可以分别使用item.firstChild和item.lastChild 。 您还可以为任何子代分配一个特定的名称，然后在以后使用该名称访问它。 假设您正在处理的图层大约有30个孩子。 一步一步地遍历所有这些都是不切实际的。 因此，该库具有layer.children.length属性，您可以使用该属性获取子代总数，然后使用for循环遍历列表。

此代码段使用我们刚刚讨论的所有属性访问各个子级：
``` javascript
var circleA = new Path.Circle(new Point(45, 150), 45);
var circleB = new Path.Circle(new Point(110, 150), 20);
var circleC = new Path.Circle(new Point(165, 150), 35);
var circleD = new Path.Circle(new Point(255, 150), 55);
var circleE = new Path.Circle(new Point(375, 150), 65);
var circleF = new Path.Circle(new Point(475, 150), 35);
circleC.name = 'GreenCircle';
project.activeLayer.firstChild.fillColor = 'orange';
project.activeLayer.lastChild.fillColor = 'pink';
project.activeLayer.children[1].fillColor = 'purple';
project.activeLayer.children['GreenCircle'].fillColor = 'lightgreen';
for (var i = 3; i < 5; i++) {
  project.activeLayer.children[i].fillColor = 'tomato';
}
```
下面的嵌入式[演示](http://codepen.io/Shokeen/full/KzJwLp)显示了正在运行的脚本。 您可以验证所有圆圈的颜色是否与我们在上面的代码中分配给它们的颜色匹配。

您还可以使用item.parent方法访问项目的父项，例如用于访问其所有子项的item.children方法。 每当您创建新项目时，其父项将始终是项目的当前活动层。 但是，可以通过将项目添加为另一个layer或group的子项来更改它。

在继续之前，让我解释一下一个group实际含义。 老实说， layer和group都非常相似。 两者之间的主要区别是，您创建的任何新项目都会自动添加到活动层，但是如果是组，则必须自己添加项目。

您可以通过多种方式将项目添加到组中。 您可以将数组项传递给组构造函数，并将它们全部添加到组的item.children数组中。 要在创建组后将其添加到组中，可以使用item.addChild(item)函数。 您还可以使用item.insertChild(index, item)函数在特定索引处插入子item.insertChild(index, item) 。

删除项目也和添加项目一样容易。 要从项目中删除任何项目，可以使用item.remove()函数。 请记住，这不会破坏项目，您可以随时将其添加回项目中。 如果您删除的项目有任何孩子，那么所有孩子也将被删除。 如果要删除所有子项但保持原样，该怎么办？ 这可以通过使用item.removeChildren()函数来实现。

## 了解项目
现在，术语item已在本教程中出现多次。 那是什么 Paper.js项目中出现的所有内容都是一个item 。 这包括layers ， paths ， groups等。尽管不同的项目具有特定于它们的属性，但其他属性也适用于所有这些属性。

如果要向用户隐藏项目，则可以通过将item.visible值item.visible为false 。 您还可以使用item.clone()函数克隆任何项目。 此函数返回克隆的项目，您可以将其存储在变量中并在以后进行操作。 您还可以使用item.opacity属性更改任何项目的不透明度。 0到1之间的任何值都将使该项目透明。

您还可以使用item.blendMode属性为任何项目设置混合模式。 混合模式需要作为string传递。 该库还提供了item.selected属性，如果将其设置为true ，则会在该元素的顶部创建视觉轮廓。 这在调试期间非常有用，因为它允许您查看路径的构造，各个分段点和项目的边界框。

## 项目转换
可以在Paper.js项目中轻松缩放，旋转或移动项目。 在本节中，我将简要介绍所有这些转换。

要更改位置item ，您可以使用它item.position属性和设置位置，以一个新的起点。 如果要移动元素，可以在+=运算符的帮助下进行。

您还可以使用item.scale(scale)函数缩放任何项目。 这将围绕其中心点缩放项目。 您可以通过将其指定为第二个参数来围绕其他点缩放项目，例如item.scale(scale, point) 。 此外，该库还允许您通过传递两个数字作为参数来在垂直和水平方向上缩放项目，例如item.scale(scaleX, scaleY) 。

旋转项目类似于缩放它们。 您可以使用item.rotate(angle)函数围绕元素的中心旋转元素。 角度以度为单位指定，旋转沿顺时针方向进行。 要围绕特定点旋转项目，还可以将一个点作为第二个参数传递，例如item.rotate(angle, point) 。

下面的代码片段应用了我们刚刚在三个不同的矩形上讨论的所有转换和操作。
``` javascript
var rectA = new Path.Rectangle(new Point(250, 70), new Size(120, 120));
rectA.fillColor = 'pink';
 
var rectB = rectA.clone();
rectB.fillColor = 'purple';
rectB.position += new Point(80, 80);
rectB.opacity = 0.6;
rectB.blendMode = 'color-burn';
rectB.scale(1.5);
rectB.rotate(45);
 
var rectC = rectB.clone();
rectC.fillColor = 'lightgreen';
rectC.position += new Point(-180, 0);
rectC.blendMode = 'color-dodge';
rectC.scale(1.5);
```
该代码几乎是不言自明的。 我从矩形A克隆矩形B，矩形B获取矩形A的所有属性。矩形B和C也是一样。

请注意，我使用了上面讨论的+=运算符来移动项目。 此运算符相对于其旧位置移动项目，而不使用绝对值。

下面的演示向您展示了所有这些转换之后的最终结果。 您可以尝试不同的混合模式或在演示中更改其他属性，以查看它们如何影响最终结果。

## 最后的想法
如前所述，Paper.js易于学习，可让您轻松创建矢量图形。 本教程介绍了使用该库所需的非常基本的知识。 很快，我们将发布该系列的下一个教程，该教程将详细讨论路径和几何。

同时，必须指出，JavaScript已成为在网络上工作的事实语言之一。 它并非没有学习曲线，并且有许多框架和库也让您忙碌。 如果您正在寻找其他资源来学习或在工作中使用，请查看[Envato](https://codecanyon.net/category/javascript)市场中可用的资源 。

在此之前，我建议您创建一些我们自己的基本演示并实践到目前为止所学的知识。 如果您对本教程有任何疑问，请在评论中告诉我。
