当我们开始学习 javascript 的时候，我们就知道 js 其实是单线程的，所以当我们在浏览器中运行某些耗时算法或者阻塞线程的代码时，浏览器就会出现卡顿的现象

然而浏览器的单个页面却拥有多个线程，比如渲染界面线程、浏览器事件触发线程、http 请求线程、事件轮询处理线程等

如果我们能够将一部分代码放在一个新的线程中执行，比如 http 请求的方法、需要大量计算耗时较多的方法，既能够保证页面对用户及时响应，又不会阻塞页面

这在以前是不可能的，但是现在，H5 为我们提供了一个方法 —— webWorker

### 什么是 webWorker
webWorker 是浏览器为我们提供的一个可以再浏览器后台开启一个新的线程的 API，使得运行在浏览器中的 js 有了多线程的能力。但是这并不意味这 js 本身就支持多线程

webWorker 有两种类型，一种是只能在当前页面使用的 webworker，另一种是可以再多个页面之间共享线程的 webWorker，前者随着当前页面关闭而关闭，而后者在同域的前提下，可以被多个页面访问

### webWorker 的创建与使用
``` javascript
// webWorker 是在主线程中通过传入一个 js 文件的路径来实现的，它返回一个 webWorker 的实例对象，该对象是主线程与该线程通信的桥梁
let worker = new Woker ('webWorker.js')
```
webWorker 在主线程和子线程之间实现通信的方法有两个：
``` javascript
// 监听一个线程向另一个发送的消息并执行指定方法
// 回调方法接受一个 event 参数，event.data 为接受到的数据
onmessage = (event) => {}

// 一个线程向另一个线程发送消息
postMessage(data)
```

实际使用时可以像下面这样使用：
``` javascript
/**
 * 主线程
 */

let worker = new Worker ('worker.js')
worker.onmessage = (e) => {
  console.log(e.data) // I post a message to main thread
}
worker.postMessage('main thread got a message')

/**
 * 子线程 worker.js
 */
onmessage = (e) => {
    console.log(e.data) // main thread got a message
}
postMessage('I post a message to main thread')
```
### 终止 webWorker
``` javascript
// 在主线程中终止
worker.terminate()

// 在子线程中终止自身
self.close()
```
### 错误监听

当子线程发生错误时，可以在主线程中监听到该错误
``` javascript
worker.addEventListener('error', (e) => {
    console.error(e.filename) // 导致错误的 worker 脚本名称
    console.error(e.message) // 错误信息
    console.error(e.lineno) // 错误行号
})
```
### 引入其他脚本
``` javascript
// 引入一个脚本
importScripts('xxx.js');

// 引入多个脚本
importScripts('aaa.js', 'bbb.js', 'ccc.js');
```
importScripts 会按顺序加载每一个脚本，当有任何失败或者错误时，会抛出 SYNTAX_ERR 异常

### 共享线程的使用
主线程中创建一个共享线程
``` javascript
let shareWorker = new SharedWorker ('shareWorker.js')

shareWorker.port.start()
shareWorker.port.postMessage('...')
shareWorker.port.onmessage = (e) => {...}
```
子线程 shareWorker.js
``` javascript
onconnect = (e) => {
    let port = e.ports[0]
    port.addEventListener('message', (e) => {
    console.log(e.data[0])
    port.postMessage(...);
  });
  port.start();
}
```
根据主线程发来的数据，Worker 线程可以调用不同的方法
``` javascript
self.addEventListener('message', function (e) {
    var data = e.data;
    switch (data.cmd) {
        case 'start':
        self.postMessage('WORKER STARTED: ' + data.msg);
        break;
        case 'stop':
        self.postMessage('WORKER STOPPED: ' + data.msg);
        self.close(); // Terminates the worker.
        break;
        default:
        self.postMessage('Unknown command: ' + data.msg);
    };
}, false);
```

### postMseeage 方法
postMseeage 传递数据的过程其实是一个值拷贝的过程，会现将数据 JSON.stringify 之后再 JSON.parse

postMseeage 也可以传送二进制数据，但是当数据过大时，由于值拷贝，浏览器会再生成一个该文件的拷贝，这样可能会引起浏览器性能的问题

所以当传输较大数据时，可以直接将数据转移给另一个线程，而不进行值拷贝，只是这样会导致原线程无法再使用这些数据，也能够防止多个线程同时修改的情况发生，这叫做零拷贝
``` javascript
// 指定传输的所有数据都是零拷贝
let data = new ArrayBuffer(64)
worker.postMessage(data, [data])

// 指定数据中的某个属性零拷贝
let obj = {a: 1, b: 2, c: 3}
worker.postMessage(obj, [obj.a, obj.c])
```

### 数据通信
主线程与 Worker 之间的通信内容，可以是文本，也可以是对象。需要注意的是，这种通信是拷贝关系，即是传值而不是传址，Worker 对通信内容的修改，不会影响到主线程。事实上，浏览器内部的运行机制是，先将通信内容串行化，然后把串行化后的字符串发给 Worker，后者再将它还原。

主线程与 Worker 之间也可以交换二进制数据，比如 File、Blob、ArrayBuffer 等类型，也可以在线程之间发送。下面是一个例子。
``` javascript
// 主线程
var uInt8Array = new Uint8Array(new ArrayBuffer(10));
for (var i = 0; i < uInt8Array.length; ++i) {
  uInt8Array[i] = i * 2; // [0, 2, 4, 6, 8,...]
}
worker.postMessage(uInt8Array);

// Worker 线程
self.onmessage = function (e) {
  var uInt8Array = e.data;
  postMessage('Inside worker.js: uInt8Array.toString() = ' + uInt8Array.toString());
  postMessage('Inside worker.js: uInt8Array.byteLength = ' + uInt8Array.byteLength);
};
```

但是，拷贝方式发送二进制数据，会造成性能问题。比如，主线程向 Worker 发送一个 500MB 文件，默认情况下浏览器会生成一个原文件的拷贝。为了解决这个问题，JavaScript 允许主线程把二进制数据直接转移给子线程，但是一旦转移，主线程就无法再使用这些二进制数据了，这是为了防止出现多个线程同时修改数据的麻烦局面。这种转移数据的方法，叫做Transferable Objects。这使得主线程可以快速把数据交给 Worker，对于影像处理、声音处理、3D 运算等就非常方便了，不会产生性能负担。

如果要直接转移数据的控制权，就要使用下面的写法。
```
// Transferable Objects 格式
worker.postMessage(arrayBuffer, [arrayBuffer]);

// 例子
var ab = new ArrayBuffer(1);
worker.postMessage(ab, [ab]);
```

### 同页面的 Web Worker

通常情况下，Worker 载入的是一个单独的 JavaScript 脚本文件，但是也可以载入与主线程在同一个网页的代码。

```
<!DOCTYPE html>
  <body>
    <script id="worker" type="app/worker">
      addEventListener('message', function () {
        postMessage('some message');
      }, false);
    </script>
  </body>
</html>
```
上面是一段嵌入网页的脚本，注意必须指定`<script>`标签的type属性是一个浏览器不认识的值，上例是app/worker。

然后，读取这一段嵌入页面的脚本，用 Worker 来处理。

```
var blob = new Blob([document.querySelector('#worker').textContent]);
var url = window.URL.createObjectURL(blob);
var worker = new Worker(url);

worker.onmessage = function (e) {
  // e.data === 'some message'
};
```
上面代码中，先将嵌入网页的脚本代码，转成一个二进制对象，然后为这个二进制对象生成 URL，再让 Worker 加载这个 URL。这样就做到了，主线程和 Worker 的代码都在同一个网页上面。

### 实例：Worker 线程完成轮询
有时，浏览器需要轮询服务器状态，以便第一时间得知状态改变。这个工作可以放在 Worker 里面。

function createWorker(f) {
  var blob = new Blob(['(' + f.toString() +')()']);
  var url = window.URL.createObjectURL(blob);
  var worker = new Worker(url);
  return worker;
}

var pollingWorker = createWorker(function (e) {
  var cache;

  function compare(new, old) { ... };

  setInterval(function () {
    fetch('/my-api-endpoint').then(function (res) {
      var data = res.json();

      if (!compare(data, cache)) {
        cache = data;
        self.postMessage(data);
      }
    })
  }, 1000)
});

pollingWorker.onmessage = function () {
  // render data
}

pollingWorker.postMessage('init');

上面代码中，Worker 每秒钟轮询一次数据，然后跟缓存做比较。如果不一致，就说明服务端有了新的变化，因此就要通知主线程。

### Worker 新建 Worker

Worker 线程内部还能再新建 Worker 线程（目前只有 Firefox 浏览器支持）。下面的例子是将一个计算密集的任务，分配到10个 Worker。

主线程代码如下。

```
var worker = new Worker('worker.js');
worker.onmessage = function (event) {
  document.getElementById('result').textContent = event.data;
};
```
Worker 线程代码如下。

```
// worker.js

// settings
var num_workers = 10;
var items_per_worker = 1000000;

// start the workers
var result = 0;
var pending_workers = num_workers;
for (var i = 0; i < num_workers; i += 1) {
  var worker = new Worker('core.js');
  worker.postMessage(i * items_per_worker);
  worker.postMessage((i + 1) * items_per_worker);
  worker.onmessage = storeResult;
}

// handle the results
function storeResult(event) {
  result += event.data;
  pending_workers -= 1;
  if (pending_workers <= 0)
    postMessage(result); // finished!
}
```
上面代码中，Worker 线程内部新建了10个 Worker 线程，并且依次向这10个 Worker 发送消息，告知了计算的起点和终点。计算任务脚本的代码如下。

```
// core.js
var start;
onmessage = getStart;
function getStart(event) {
  start = event.data;
  onmessage = getEnd;
}

var end;
function getEnd(event) {
  end = event.data;
  onmessage = null;
  work();
}

function work() {
  var result = 0;
  for (var i = start; i < end; i += 1) {
    // perform some complex calculation here
    result += 1;
  }
  postMessage(result);
  close();
}
```
### API

1. 主线程

浏览器原生提供Worker()构造函数，用来供主线程生成 Worker 线程。


var myWorker = new Worker(jsUrl, options);
Worker()构造函数，可以接受两个参数。第一个参数是脚本的网址（必须遵守同源政策），该参数是必需的，且只能加载 JS 脚本，否则会报错。第二个参数是配置对象，该对象可选。它的一个作用就是指定 Worker 的名称，用来区分多个 Worker 线程。


// 主线程
var myWorker = new Worker('worker.js', { name : 'myWorker' });

// Worker 线程
self.name // myWorker
Worker()构造函数返回一个 Worker 线程对象，用来供主线程操作 Worker。Worker 线程对象的属性和方法如下。

- Worker.onerror：指定 error 事件的监听函数。
- Worker.onmessage：指定 message 事件的监听函数，发送过来的数据在Event.data属性中。
- Worker.onmessageerror：指定 messageerror 事件的监听函数。发送的数据无法序列化成字符串时，会触发这个事件。
- Worker.postMessage()：向 Worker 线程发送消息。
- Worker.terminate()：立即终止 Worker 线程。
2. Worker 线程
Web Worker 有自己的全局对象，不是主线程的window，而是一个专门为 Worker 定制的全局对象。因此定义在window上面的对象和方法不是全部都可以使用。

Worker 线程有一些自己的全局属性和方法。

- self.name： Worker 的名字。该属性只读，由构造函数指定。
- self.onmessage：指定message事件的监听函数。
- self.onmessageerror：指定 messageerror 事件的监听函数。发送的数据无法序列化成字符串时，会触发这个事件。
- self.close()：关闭 Worker 线程。
- self.postMessage()：向产生这个 Worker 线程发送消息。
- self.importScripts()：加载 JS 脚本。



需要注意的是，通过 webWorker 创建的线程的运行环境中没有全局对象 window，也无法访问 DOM / BOM 对象，所以他只能用来执行纯粹的 javascript 计算
当然，他也可以获取到部分浏览器提供的 API，如：
XMLHttpRequest
navigator
location (read only)
setTimeout()， clearTimeout()， setInterval()， clearInterval()
Promise 等等
还有哪些，小伙伴们可以自己去尝试发掘（可以将 self 打印出来看看）

Web Worker 有以下几个使用注意点。

（1）同源限制

分配给 Worker 线程运行的脚本文件，必须与主线程的脚本文件同源。

（2）DOM 限制

Worker 线程所在的全局对象，与主线程不一样，无法读取主线程所在网页的 DOM 对象，也无法使用document、window、parent这些对象。但是，Worker 线程可以navigator对象和location对象。

（3）通信联系

Worker 线程和主线程不在同一个上下文环境，它们不能直接通信，必须通过消息完成。

（4）脚本限制

Worker 线程不能执行alert()方法和confirm()方法，但可以使用 XMLHttpRequest 对象发出 AJAX 请求。

（5）文件限制

Worker 线程无法读取本地文件，即不能打开本机的文件系统（file://），它所加载的脚本，必须来自网络。