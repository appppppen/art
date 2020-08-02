## JavaScript 键盘鼠标事件处理

### 监听键盘鼠标事件

#### 监听某个按键事件

当键盘上的某个键被按下时，会依次触发一次下面的事件：

- onkeydown: 键盘按下这个动作（按下键盘）
- onkeypress: 键盘被按住（一直按着键盘不动）
- onkeyup: 键盘被弹起（松开键盘）

通过监听 keydown 事件既可以知道键盘被按下：

```javascript
document.onkeydown = function (event) {
  // 键盘按下时触发
  console.log("key down");
};

document.onkeypress = function (event) {
  // 键盘按住时触发
  console.log("key press");
};

document.onkeyup = function (event) {
  // 键盘弹起时触发
  console.log("key up");
};
// 控制台数据的顺序为：key down -> key press -> key up
```

注意到键盘按下的 event 参数，该参数为 KeyboardEvent 事件对象，其中包含按键相关的一些属性。其中：

- type: 事件类型，如'keydown'或者'keyup'
- key: 表示按下的键盘内容是什么即键值，按下字母'p'时，值为'p'
- code: 表示键盘代码，按下字母'p'时，值为'KeyP'
- keyCode(过时): 整数，表示键码，每个键都有唯一的键码，字母'p'的键码为 80
- altKey: 布尔值，表示此时的 alt 键是否也按下
- ctrKey: 布尔值，表示此时的 alt 键是否也按下
- shiftKey: 布尔值，表示此时的 shift 键是否也按下
- metaKey: 布尔值，windows 平台表示 Window 键是否同时按下，mac 表示 Command 键是否同时按下
- repeat: 布尔值，如果一个键一直被按着，则其值为 true，表示重复

可以通过检查这些属性来判断用户按下的是什么键，以及是否 ctrl 和 alt 等键是否同时按下。

```javascript
document.onkeydown = function (event) {
  // 键盘按下是触发
  console.log("key down: " + event.key);
  if (event.altKey) {
    console.log("alt is active");
  }
  if (event.shiftKey) {
    console.log("shift is active");
  }
};
```

#### 监听鼠标事件

相应的也可以监听鼠标相关的事件，触发时的参数 event 为 MouseEvent 对象类型：

- onclick: 鼠标点击事件
- ondblclick: 鼠标双击事件
- onmousedown: 鼠标上的按钮被按下了
- onmouseup: 鼠标按下后松开时触发的事件
- onmousemove: 鼠标移动时触发的事件
- onmouseout: 鼠标离开监听该事件的元素或子元素时触发的事件
- onmouseover: 鼠标移动到监听该事件的元素或子元素时触发的事件
- onmousewheel: 鼠标滚轮事件，一般使用 onscroll 事件

MouseEvent 对象中包含下面比较有用的属性：

- type: 事件类型，如'mosemove'或者'mousedown'
- button: 整型，触发鼠标事件时按下的按钮编号
- buttons: 整型，触发鼠标事件时弹起来的按钮编号
- clientX: 鼠标指针在 DOM 内容区的 X 坐标
- clientY：鼠标指针在 DOM 内容区的 Y 坐标
- offsetX: 鼠标指针相对父节点填充边缘的 X 坐标
- offsetY: 鼠标指针相对父节点填充边缘的 Y 坐标
- screenX: 鼠标指针在全局屏幕的 X 坐标
- screenY: 鼠标指针在全局屏幕的 Y 坐标
- pageX: 鼠标指针在整个 DOM 内容（包括分页）的 X 坐标
- pageY: 鼠标指针在整个 DOM 内容（包括分页）的 Y 坐标
- altKey: 布尔值，表示此时的 alt 键是否也按下
- ctrKey: 布尔值，表示此时的 alt 键是否也按下
- shiftKey: 布尔值，表示此时的 shift 键是否也按下
- metaKey: 布尔值，windows 平台表示 Window 键是否同时按下，mac 表示 Command 键是否同时按下

通过鼠标事件的 event 属性，可以获取鼠标点击的位置，这里有对各种坐标的相关介绍，鼠标点击时是否按住 ctrl 等。

### 模拟键盘和鼠标事件

上面我们说明了如何监听页面的按键和键盘事件，但是有的时候我们需要使用代码模拟按钮操作。

比如看到很多图片的时候，我们需要批量下载，这个时候可以打开控制台，写一段 JS 代码批量模拟下载步骤即可，而不用一个一个的手动点击，非常方便。

#### 模拟鼠标点击

最简单的就是模拟点击了，我们只需要选中一个元素，然后执行 click 函数即可。

下面的代码实现在一个表格中，点击每一个图片。

```javascript
const images = document.getElementById("content").getElementsByTagName("img");
for (let image of images) {
  images.click();
}
```

如果要模拟鼠标双击，或者鼠标移动，则没有简单的函数可以调用。这个时候我们需要新建一个 MouseEvent 对象，并手动触发即可。

创建 MouseEvent 对象的语法为：const event = new MouseEvent(typeArg, mouseEventInit);

typeArg 为鼠标事件类型，即上面的监听鼠标事件去掉 on 后的字符串，

- click: 鼠标点击事件
- dblclick: 鼠标双击事件
- mousedown: 鼠标上的按钮被按下了
- mouseup: 鼠标按下后松开时触发的事件
- mousemove: 鼠标移动时触发的事件
- mouseout: 鼠标离开监听该事件的元素或子元素时触发的事件
- mouseover: 鼠标移动到监听该事件的元素或子元素时触发的事件

mouseEventInit 为 MouseEvent 初始化的选项，指定鼠标事件的各种属性，可选值如下：

- button: 整型，触发鼠标事件时按下的按钮编号
- buttons: 整型，触发鼠标事件时弹起来的按钮编号
- clientX: 鼠标指针在 DOM 内容区的 X 坐标
- clientY：鼠标指针在 DOM 内容区的 Y 坐标
- offsetX: 鼠标指针相对父节点填充边缘的 X 坐标
- offsetY: 鼠标指针相对父节点填充边缘的 Y 坐标
- screenX: 鼠标指针在全局屏幕的 X 坐标
- screenY: 鼠标指针在全局屏幕的 Y 坐标
- pageX: 鼠标指针在整个 DOM 内容（包括分页）的 X 坐标
- pageY: 鼠标指针在整个 DOM 内容（包括分页）的 Y 坐标
- altKey: 布尔值，表示此时的 alt 键是否也按下
- ctrKey: 布尔值，表示此时的 alt 键是否也按下
- shiftKey: 布尔值，表示此时的 shift 键是否也按下
- metaKey: 布尔值，windows 平台表示 Window 键是否同时按下，mac 表示 Command 键是否同时按下
  比如下面的示例在坐标 200,200 处触发一个鼠标双击事件：

```javascript
// 创建一个event对象
const createEvent = new MouseEvent("dblclick", { clientX: 300, clientY: 300 });
// 触发该事件
document.dispatchEvent(createEvent);
```

可以使用任意的 Document 对象的 dispatchEvent 方法触发一个事件，这些触发的事件和实际发生的事件一模一样。

#### 模拟键盘输入事件

和模拟鼠标事件一样，不过这儿我们要创建一个 KeyboardEvent 事件对象。

创建 KeyboardEvent 对象的语法类似为：const event = new KeyboardEvent(typeArg, KeyboardEventInit);

typeArg 为键盘输入事件类型，即上面的监听键盘输入事件去掉 on 后的字符串，

- keydown: 键盘按下这个动作
- keypress: 键盘被按住
- keyup: 键盘被弹起
- KeyboardEventInit 为 KeyboardEvent 初始化的选项，指定键盘输入事件的各种属性，可选值如下：
- type: 事件类型，如'keydown'或者'keyup'
- key: 表示按下的键盘内容是什么即键值，按下字母'p'时，值为'p'
- code: 表示键盘代码，按下字母'p'时，值为'KeyP'
- keyCode(过时): 整数，表示键码，每个键都有唯一的键码，字母'p'的键码为 80
- altKey: 布尔值，表示此时的 alt 键是否也按下
- ctrKey: 布尔值，表示此时的 alt 键是否也按下
- shiftKey: 布尔值，表示此时的 shift 键是否也按下
- metaKey: 布尔值，windows 平台表示 Window 键是否同时按下，mac 表示 Command 键是否同时按下
- repeat: 布尔值，如果一个键一直被按着，则其值为 true，表示重复

比如实现在按下字母'a'键时，自动按下'alt+ctrl+a'可以像下面实现。

```javascript
// 监听按键事件
document.onkeydown = function (event) {
  console.log(
    `keyCode: ${event.keyCode} code: ${event.code} alt:${event.altKey}`
  );
  if (event.keyCode === 65 || event.code === "KeyA") {
    // 如果按下的是a键，则新建一个a键按下的事件并触发
    const createEvent = new KeyboardEvent("keydown", {
      altKey: true,
      shiftKey: true,
      keyCode: 65,
      code: "KeyA",
    });
    document.dispatchEvent(createEvent);
  }
};
```

然后你就会发现上面的函数疯狂的输出 A 键被按下，哈哈哈！知道内存达到限制！
