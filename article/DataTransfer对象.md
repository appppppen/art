## [HTML 拖放 API](https://developer.mozilla.org/zh-CN/docs/Web/API/HTML_Drag_and_Drop_API)

[拖拽献祭中的黑山羊-DataTransfer 对象](https://www.zhangxinxu.com/wordpress/2018/09/drag-drop-datatransfer-js/)

## DataTransfer 对象的出现

DataTransfer 对象出现在拖拽事件中，具体包括开始拖拽 dragstart 事件，拖拽进入 dragenter 事件，拖拽离开 dragleave 事件，拖拽经过 dragover 事件，拖拽释放 drop 事件以及拖拽结束 dragend 事件。

```javascript
document.addEventListener("dragstart", function (event) {
    console.log("dragstart: " + event.dataTransfer);
});
document.addEventListener("dragenter", function (event) {
    console.log("dragenter: " + event.dataTransfer);
});
document.addEventListener("dragleave", function (event) {
    console.log("dragleave: " + event.dataTransfer);
});
document.addEventListener("dragover", function (event) {
    event.preventDefault();
    console.log("dragover: " + event.dataTransfer);
});
document.addEventListener("drop", function (event) {
    console.log("drop: " + event.dataTransfer);
});
document.addEventListener("dragend", function (event) {
    console.log("dragend: " + event.dataTransfer);
});
```

## DataTransfer 对象属性和方法详解

DataTransfer 对象包含下面 5 个标准属性和 4 个标准方法。

### 标准属性

> DataTransfer.dropEffect

获取当前所选拖放操作的类型，或将拖拽操作设置为新类型。值必须为 none，copy，link 或 move 中的一个。

> DataTransfer.effectAllowed

提供可能的所有类型的操作。必须是 none，copy，copyLink，copyMove，link，linkMove，move，all 或 uninitialized 中的一个。

> DataTransfer.files

拖拽的本地文件列表。如果拖动操作不涉及拖动文件，则此属性为空列表。

> DataTransfer.items （只读）

提供 DataTransferItemList 对象，该对象是所有拖动数据的列表。

> DataTransfer.types （只读）

在 dragstart 事件中设置数据格式，返回的是一个字符串数组。

### 标准方法

> DataTransfer.clearData([format])

删除与给定类型关联的数据。format 参数是可选的。如果类型为空或未指定，则删除所有关联的数据。如果指定类型的数据不存在，或者数据传输不包含任何数据，则此方法无效。

> DataTransfer.getData(format)

返回给定类型的数据，如果该类型的数据不存在或数据传输不包含数据，则返回空字符串。

> DataTransfer.setData(format, data)

设置给定类型的数据。如果该类型的数据不存在，则在末尾添加，以使列表中的最后一项成为新格式类型。如果该类型的数据已存在，则在相同位置把现有数据替换掉。

> DataTransfer.setDragImage(img, xOffset, yOffset)

设置用于拖动的自定义图像。

-   DataTransfer.dropEffect

dropEffect 属性顾名思意拖拽效果，在 PC web 端主要表现在鼠标手形上。不同的 dropEffect 值，鼠标的手形效果是不一样的。

举个例子，假设有个 box 元素，当 dragover 的时候，我们设置其 dropEffect 值分别为 move，copy，link 和 none
![](https://image.zhangxinxu.com/image/blog/201809/2018-09-28_234201.png)
![](https://image.zhangxinxu.com/image/blog/201809/2018-09-28_234233.png)
![](https://image.zhangxinxu.com/image/blog/201809/2018-09-28_234312.png)
![](https://image.zhangxinxu.com/image/blog/201809/2018-09-28_234339.png)

dropEffect 属性的设置主要用在 dragenter 和 dragover 事件中，同时受 effectAllowed 属性影响。

-   DataTransfer.effectAllowed
    表示拖拽允许的效果。支持的属性值较多，如下：

> uninitialized

默认值，表示未初始化。从效果上来讲，和 all 是一样滴。

> none

不允许拖拽。鼠标保持禁止状态。

> copy

可以在新位置复制元素。

> copyLink

允许复制和链接操作。

> copyMove

允许复制和移动操作。

> link

可以在新位置建立链接。

> linkMove

允许链接和移动操作。

> move

可以把元素移动到新位置。

> all

什么操作都允许。

如果 effectAllowed 属性值设置为上面列表以外的其它值则没有任何效果。另外在 IE 浏览器下所有的字都会小写，因此类似 linkMove 会变成 linkmove。

-   effectAllowed 和 dropEffect 的关系 \*

effectAllowed 和 dropEffect 通常应用的事件方法名不一样，effectAllowed 多用在 dragstart 事件中，而 dropEffect 属性的设置主要用在 dragenter 和 dragover 事件中。

effectAllowed 和 dropEffect 的彼此间是有制约关系，当我们给 effectAllowed 设置了对应的属性值，则 dropEffect 只能设置为 effectAllowed 允许的值，否则是无效的。

举个例子，如果我们设置 effectAllowed 值为'copyMove'，则 dropEffect 只有'copy'和'move'这两个属性值才有效。

effectAllowed 看上去很屌，但实际应用的时候相当鸡肋。我们通常的拖拽应用是用不到这个的。只要下面这个场景，那就是当我们有很多个元素需要拖拽，但是需要像垃圾一样分门别类的时候，effectAllowed 就有用了。

原生拖拽事件有这样一种行为，那就是如果 effectAllowed 值和 dropEffect 值不一致，则无法响应 drop 事件。我们可以想象一下，假设我们在网页中放三个垃圾箱，分别回收 move 元素，copy 元素和 link 元素，由于 DOM 元素的转移或者复制我们都是在 drop 事件中完成的，则 effectAllowed 包含 copy 元素的才能拖拽进入 copy 垃圾箱（可以触发 drop 事件）。

平常开发都是简单的拖拽，哪里需要用到分门别类啊，因此 effectAllowed 也就无用武之地了。

//zxx: 这里有个 [CodePen](https://codepen.io/SitePoint/pen/epQPNP)，可以感受下。

link 作为属性值案例一则
这里插播一个案例，实现的是 Chrome 浏览器下拖拽链接也能新窗口打开的实现。在 Firefox 和 IE 浏览器下，链接元素你按住一拖拽，就可以在新的浏览器标签页中打开，Chrome 浏览器则需要拖动到地址栏才可以。如果我们想要实现 Chrome 浏览器下拖拽也能标签页中打开，就可以看看下面这个[案例](https://www.zhangxinxu.com/study/201809/datatransfer-effectallowed-link.php)。

拖拽链接到 demo 页面一个灰色盒子中

![](https://image.zhangxinxu.com/image/blog/201809/2018-09-29_232502.png)

释放鼠标，此时这个链接就会在新标签页中打开。

```html
<a href="#" id="link">拖拽我到下面框框试试</a>
<p id="box" class="box"></p>
```

```javascript
var isOpenLink = null;
link.addEventListener("dragstart", function (event) {
    event.dataTransfer.effectAllowed = "link";
});
box.addEventListener("dragover", function (event) {
    event.preventDefault();
    // 检测是否需要新窗口打开链接
    if (event.dataTransfer.dropEffect != "link") {
        isOpenLink = true;
    }
    event.dataTransfer.dropEffect = "link";
});
box.addEventListener("drop", function (event) {
    event.preventDefault();
    if (isOpenLink) {
        window.open(link.href);
    }
});
```

通过检测 dataTransfer.dropEffect，有效避免和原本支持拖拽新标签页打开链接的浏览器冲突。

### DataTransfer.files

当我们从桌面往网页浏览器中拖文件的时候，DataTransfer.files 就派上用场了，其对应的列表只就是我们拖进去的文件列表。目前在实际开发中应用的比较多的是拖拽上传

[DataTransfer.files 获取桌面文件信息 demo](https://www.zhangxinxu.com/study/201809/datatransfer-files.php)

![](https://image.zhangxinxu.com/image/blog/201809/2018-09-30_000501.png)

![](https://image.zhangxinxu.com/image/blog/201809/2018-09-30_000527.png)

可以看到，两个 txt 文件识别为了 text/plain，而快捷方式文件没有 mime-type，因此，输出内容是空。

```javascript
box.addEventListener("drop", function (event) {
    event.preventDefault();
    // 遍历文件信息
    var files = event.dataTransfer.files || [];
    var length = files.length;
    if (length == 0) {
        this.innerHTML = "<p>没有文件</p>";
        return;
    }
    var html = "";
    for (var index = 0; index < length; index++) {
        html += "<p>类型：" + files[index].type + "</p>";
    }
    this.innerHTML = html;
});
```

什么时候会出现“没有文件”的提示呢？比方说 demo 页面你框选几个文字，拖拽到方框框里面，就会提示“没有文件”了，因为你拖拽进去的本来就不是文件内容。不过，这个拖拽内容我们是可以使用 DataTransfer.items 获取的。

### DataTransfer.items

DataTransfer.items 可以用来获取拖拽的数据信息，只读。

DataTransfer.items 为 DataTransferItem 类型的数据列表集合，而 DataTransferItem 又包含多个属性和方法，属性包括 kind 和 type，方法包括 getAsString()和 getAsFile()，这个和剪切板 item 对象方法是一致的。

我们可以通过一个小案例快速了解一下这些属性含义和作用。

您可以狠狠地点击这里：DataTransferItem 属性和方法信息输出 demo

log 信息打印测试代码如下：

```javascript
console.log(
    "kind: " + DataTransferItem.kind + ", type: " + DataTransferItem.type + "\n"
);
DataTransferItem.getAsString(function (str) {
    console.log("\ngetAsString: " + str);
});
```

主要测试了 kind 和 type 属性和 getAsString()方法。

拖动图片，此时就可以得到 DataTransfer.items 具体都是些什么信息，如下截图：
![](https://image.zhangxinxu.com/image/blog/201809/2018-09-30_181621.png)

其中，包含 3 个 item，kind 全部都是 string，type 指的是 mimeType 类型，包含 3 中不同类型，为：text/plain，text/html 和 text/uri-list。

如果我们拖动页面上文字信息到输入框上显示的就只有 2 中类型项：

![](https://image.zhangxinxu.com/image/blog/201809/2018-09-30_182321.png)

如果从浏览器地址栏拖文字到输入框，就会只有 text/plain 这 1 种类型。

```
//zxx: 浏览器不同的输入容器会自动甄别显示内容，例如：，对于文本框，显示内容为text/plain纯文本（如本demo），如果是富文本输入框（如HTML元素设置contenteditable属性），则显示内容为text/html富文本。
```

![](https://image.zhangxinxu.com/image/blog/201809/2018-09-30_185053.png)

此时，可以调用 getAsFile()方法将其转换成二进制文件对象，然后可以 ajax 上传等处理。

实际开发的处理模板

有个处理模板，无论何种数据类型，大家可以找到对应位置，进行相应的处理，省掉不少自己写逻辑判断的麻烦。

```javascript
handleDataTransferItems = function (items) {
    for (var i = 0; i < items.length; i += 1) {
        var kind = items[i].kind;
        var type = items[i].type;
        // 逻辑开始
        if (kind == "string") {
            if (type.indexOf("text/plain") != -1) {
                items[i].getAsString(function (str) {
                    // str是纯文本，该怎么处理，就在这里处理
                });
            } else if (type.indexOf("text/html") != -1) {
                items[i].getAsString(function (str) {
                    // str是富文本，就在这里处理
                });
            } else if (type.indexOf("text/uri-list") != -1) {
                items[i].getAsString(function (str) {
                    // str是uri地址，在这里进行处理
                });
            }
        } else if (kind == "file") {
            // 如果是图片
            if (type.indexOf("image/") != -1) {
                var file = items[i].getAsFile();
                // file就是图片文件对象，可以上传，或者其他处理
            }
        }
    }
};
```

使用示例：

```javascript
document.addEventListener("drop", function (event) {
    var items = event.dataTransfer.items || [];
    handleDataTransferItems(items);
});
```

### 近亲 clipboardData.items

剪切板对象也有 items 属性，数据类型以及 item 子项的属性和方法和 DataTransfer 对象是一样的，学会了一个自然也就学会了另外一个，不展开。

## DataTransfer.types

DataTransfer.types 用在 dragstart 事件中，包含拖拽内容的包含的 mimeType 类型们，可以遍历出具体的 type 类型。假设页面有如下全局 JS 代码：

```javascript
document.addEventListener("dragstart", function (event) {
    // 遍历并输出拖拽内容的类型
    var types = event.dataTransfer.types || [];
    [].slice.call(types).forEach(function (type) {
        console.log(type);
    });
});
```

则拖动纯图片元素输出内容如下：

![](https://image.zhangxinxu.com/image/blog/201809/2018-09-30_194057.png)

拖动文字如下：

![](https://image.zhangxinxu.com/image/blog/201809/2018-09-30_194328.png)

DataTransfer.types 有什么用呢？如果我们希望页面某些元素可以拖拽，有些不允许，则可以使用 types 属性进行检测，例如，如果是包含 Files，则执行 event.preventDefault()，拖拽行为就会被禁止。

### DataTransfer.getData()

DataTransfer.getData(format)可以快捷获取拖拽的内容。

format 可以理解为就是 DataTransfer.items 中的 type 值，例如，我们想要获取拖拽内容的富文本格式，可以：

```javascript
document.addEventListener("drop", function (event) {
    // 获取拖拽富文本内容
    var html = event.dataTransfer.getData("text/html");
});
```

如果是获取纯文本，可以 text/plain，或者有时候 text/uri-list，实际开发时候，我们会直接使用'text'，例如：

```javascript
document.addEventListener("drop", function (event) {
    // 获取拖拽纯文本内容
    var html = event.dataTransfer.getData("text");
});
```

优化输入框拖拽输入体验
本案例类似于“利用剪切板 JS API 优化输入框的粘贴体验”这篇文章，当我们拖拽本文进入输入框的时候，文本信息可能包含不需要的信息，例如手机号中的 334 分隔符，借助 getData()方法我们可以进行过滤和优化。

[DataTransfer.getData()优化手机号拖拽输入体验 demo](https://www.zhangxinxu.com/study/201809/datatransfer-getdata.php)

可以看到（如下 GIF），手机号明明是 132-0803-3621，但拖到输入框后自动变成了 13208033621，省掉了再编辑的烦恼：

![](https://image.zhangxinxu.com/image/blog/201809/tel-drag.gif)

input.addEventListener('drop', function (event) {
// 获取拖拽文本内容
var text = event.dataTransfer.getData('text');
if (this.value == '' && text.match(/\d/g) && text.match(/\d/g).length == 11) {
event.preventDefault();
input.value = text.replace(/\D/g, '');
input.select();
}
});

### DataTransfer.setData()

DataTransfer.setData(format, data)可以自定义拖拽的内容信息。可以重置原生的拖拽内容，或者用来参数传递。

例如如果我们页面运行了如下 JS 代码：

```
document.addEventListener('dragstart', function (event) {
    // 重置拖拽文本内容
    event.dataTransfer.setData('text', '张鑫旭-鑫空间-鑫生活');
});
```

则所有内容拖拽进入输入框内容都是“张鑫旭-鑫空间-鑫生活”，如下 GIF 示意：

![](https://image.zhangxinxu.com/image/blog/201809/tel-drag2.gif)

### DataTransfer.clearData()

DataTransfer.clearData()只能用在 dragstart 事件中，会清除所有的数据。也就是执行了此方法后，DataTransfer.getData()方法只能获得空字符串。

使用场景不是很多。

当我们需要使用 setData()方法自定义拖拽数据的时候，为了避免原生拖拽数据干扰，可以先执行一次 clearData()方法。这样可以避免 text/html 类型数据干扰（如果我们自定义的是纯文本数据）。

### DataTransfer.setDragImage()

DataTransfer.setDragImage(img, offsetX, offsetY)用在 dragstart 事件中，可以设置拖拽时候有个图片跟在后面。

其中，img 表示图片 DOM 元素对象，offsetX 表示距离鼠标的水平偏移距离，offsetY 表示距离鼠标的垂直偏移距离。

直接看一个例子：

```
var image = new Image();
image.src = './1f603.svg';
document.addEventListener('dragstart', function (event) {
    // 设置拖拽图片
    event.dataTransfer.setDragImage(image, 10, 10);
});
```

此时，页面上拖拽任意内容，就会看到一个笑脸跟在后面了，例如：

![](https://image.zhangxinxu.com/image/blog/201809/2018-09-30_211447.png)
