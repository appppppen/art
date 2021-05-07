## koa解决跨域的方法

### 1. 自行添加一个中间件
```javascript
app.use(async (ctx, next)=> {
  ctx.set('Access-Control-Allow-Origin', '*');
  ctx.set('Access-Control-Allow-Headers', 'Content-Type, Content-Length, Authorization, Accept, X-Requested-With , yourHeaderFeild');
  ctx.set('Access-Control-Allow-Methods', 'PUT, POST, GET, DELETE, OPTIONS');
  if (ctx.method == 'OPTIONS') {
    ctx.body = 200; 
  } else {
    await next();
  }
});
```
### 2. 使用koa2-cors
#### 2.1先下载
```javascript
npm install koa2-cors --save
```
#### 2.2使用
```javascript
const Koa = require('koa');
const cors = require('koa2-cors');
const app = new Koa();

app.use(cors());
//或者
app.use(
    cors({
        origin: function(ctx) { //设置允许来自指定域名请求
            if (ctx.url === '/test') {
                return '*'; // 允许来自所有域名请求
            }
            return 'http://localhost:8080'; //只允许http://localhost:8080这个域名的请求
        },
        maxAge: 5, //指定本次预检请求的有效期，单位为秒。
        credentials: true, //是否允许发送Cookie
        allowMethods: ['GET', 'POST', 'PUT', 'DELETE', 'OPTIONS'], //设置所允许的HTTP请求方法
        allowHeaders: ['Content-Type', 'Authorization', 'Accept'], //设置服务器支持的所有头信息字段
        exposeHeaders: ['WWW-Authenticate', 'Server-Authorization'] //设置获取其他自定义字段
    })
);
```
### 3. 使用[@koa/cors](https://github.com/koajs/cors)
#### 3.1先下载
```javascript
npm install @koa/cors --save
```
#### 2.2使用
```javascript
const cors = require('@koa/cors')
app.use(cors());
```

只需要两行，接口就会在返回数据的时候带上`Access-Control-Allow-Origin`响应头。默认允许所有请求方式跨域，`Access-Control-Allow-Origin`默认为*。
#### 2.3携带cookies
为了安全考虑，携带cookies的跨域请求只允许Access-Control-Allow-Origin为单一域名，即只支持一个域名在请求的时候携带cookies。且需要带上响应头Access-Control-Allow-Credentials

对 @koa/cors 添加配置origin和credentials：
```javascript
const cors = require('@koa/cors')

app.use(cors({
  origin: 'http://koa.com',    // 前端地址
  credentials: true
}));
```

同时，前端要发起携带cookies的跨域请求，需要设置XMLHttpRequest的withCredentials为true，如果你是使用axios，只需要在请求配置里加上一句withCredentials: true，请看例子：
```javascript
export const upload = (requestParams) => {
  return axios({
    method: 'post',
    url: 'http://localhost:8000/api/admin/upload',
    data: requestParams,
    withCredentials: true
  })
}
```

这样前端（http://koa.com）就可以向后端（http://localhost:8000）发送请求了。
#### 2.4多域携带cookies发送请求
如果你的前端地址只有一个，给`@koa/cors`的origin添加一个域名就能满足需求，如果需要支持跨域的域名有多个呢？

通过观察`@koa/cors`的[单元测试](https://github.com/koajs/cors/blob/71c4d00b170f52fd1324e9fd028816408867f8a6/test/cors.test.js#L85)用例，可以发现origin是支持用函数的方式传入的。这样我们就可以维护一个域名数组，判断请求地址是否在域名数组内，就能知道是否要对请求地址提供携带cookies请求支持了。

要想知道发起请求的前端地址，可以使用`ctx.request.header.origin`。注意`ctx.request.header.origin`和`ctx.request.origin`是不同的。`ctx.request.origin`是接口域名，`ctx.request.header.origin`是发起请求的页面域名。


## koa配置强缓存
```javascript
app.use(async (ctx, next)=> {
  ctx.set('Access-Control-Allow-Origin', '*');
  ctx.response.set('expires', new Date(Date.now() + 2 * 60 * 1000).toString()); // 添加 expires 字段到响应头，过期时间 2 分钟
});

```