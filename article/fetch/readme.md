### 前言

> 在 javascript 中，所有的代码都是以单线程的方式执行的，因而网络请求，浏览器事件等操作都需要使用异步的方法。

#### XMLRequest(XHR)

```typescript
var xhr = new XMLHttpRequest();
xhr.open("get", "/some/url", true);
xhr.responseType = "json";
xhr.send();
xhr.onreadystatechange = function () {
  if ((xhr.status = 200 && xhr.readyState == 4)) {
    console.log(xhr.response);
  }
};
```

#### 告别 XMLHttpRequest

fetch()方法与 XMLHttpRequest 类似，fetch 也可以发起 ajax 请求，但是与 XMLHttpRequest 不同的是，fetch 方式使用 Promise，相比较 XMLHttpRequest 更加的简洁。

如果你还不了解 Promise，需要先补充，关于 Promise 的一些知识。

#### Promise

Promise 是进行异步操作的一种解决方案，比传统的处理方法(回调函数/处理事件)更加合理，ES6 将其写入了语言标准,统一了语法，原生提供了 Promise。 Promise 可以想象成一个装有各种结果的容器，里面装有某个时间返回来的结果，你可以在需要的时候拿取它并进行一些事件处理。

从上边的介绍，大概可以看出 Promise 相比较于传统的异步操作的一个优势在于不用在异步操作的同时进行事件的处理，更加的合理性。

#### 使用方法

在 ES6 中规定,Promise 对象是一个构造函数,用来生成 Promise 实例。

```typescript
const promist = new Promise(function(resolve,reject){
    if(/*异步操作成功*/){
        resolve(value);
    }else{
        reject(error);
    }
})
```

- resolve 在异步操作成功时调用，并将异步操作的结果，作为参数传递出去
- reject 在异步操作失败时调用，并将异步操作错误结果，作为参数传递出去
- Promise 实例生成后可以用 then()方法操作成功/失败的回调函数

```typescript
promise.then(function(value){
<!-- 成功的回调处理 -->
},function(error){
<!-- 失败的回调处理 -->
})
```

#### 基本的 fetch 请求

简单的了解了 Promise 后我们就可以对 fetch()方法有一个很好的认识了，fetch 是全局量 window 的一个方法，第一个参数为 URL。

```typescript
// url (必须), options (可选)
fetch("/some/url", {
  method: "get",
})
  .then(function (response) {})
  .catch(function (err) {
    // 出错了;等价于 then 的第二个参数,但这样更好用更直观 :(
  });
```

url 参数是必须要填写的，option 可选，设置 fetch 调用时的 Request 对象，如 method、headers 等
比较常用的 Request 对象有：

- method - 支持 GET, POST, PUT, DELETE, HEAD
- url - 请求的 URL
- headers - 对应的 Headers 对象
- body - 请求参数（JSON.stringify 过的字符串或’name=jim\u0026age=22’ 格式)

如提交 JSON 示例如下：

```typescript
 fetch('/users.json', {
    method: 'POST',
    body: JSON.stringify({
        email: 'huang@163.com'
        name: 'jim'
    })

}).then(function() { /* 处理响应 */ });
```

#### Response 响应

fetch 方法的 then 会接收一个 Response 实例，值得注意的是 fetch 方法的第二个 then 接收的才是后台传过来的真正的数据，一般第一个 then 对数据进行处理等。

例如 fetch 处理 JSON 响应时 回调函数有一个 json()方法，可以将原始数据转换为 json 对象

```typescript
 fetch('/some/url', { method: 'get', })
    // 第一个then  设置请求的格式
        .then(e => e.json())
        // 第二个then 处理回调
        .then((data) => {
         <!-- data为真正数据 -->
    }).catch(e => console.log("Oops, error", e))
```

#### 使用 fetch 请求发送 cookie

```typescript
fetch(url, {
  credentials: "include",
});
```

### 总结

使用 fetch 方法请求数据更加的简单，语法简洁，数据处理过程更加的清晰，基于 Promise 实现。
