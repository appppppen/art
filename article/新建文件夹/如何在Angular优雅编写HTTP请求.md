## 引言

基本上当下的应用都会分为前端与后端，当然这种前端定义不在限于桌面浏览器、手机、APP等设备。一个良好的后端会通过一套所有前端都通用的 RESTful API 序列接口作为前后端之间的通信。
这其中对于身份认证都不可能再依赖传统的Session或Cookie；转而使用诸如OAuth2、JWT等这种更适合API接口的认证方式。当然本文并不讨论如何去构建它们。

## 一、API 设计

首先虽然并不会讨论身份认证的技术，但不管是OAuth2还是JWT本质上身份认证都全靠一个 Token 来维持；因此，下面统一以 token 来表示身份认证所需要的值。
一套合理的API规则，会让前端编码更优雅。因此，希望在编写Angular之前，能与后端相互达成一种“协议”也很有必要。可以尝试从以下几点进行考虑。

1. 版本号

可以在URL（例：https://demo.com/v1/）或Header（例：headers: { version: 'v1' }）中体现，相比较我更喜欢前者的直接。

2. 业务节点

以一个节点来表示某个业务，比如：

* 商品 https://demo.com/v1/product/ 
* 商品SKU https://demo.com/v1/product/sku/ 

3. 动作

由HTTP动词来表示：

* GET 请求一个商品 /product/${ID} 
* POST 新建一个商品 /product 
* PUT 修改一个商品 /product/${ID} 
* DELETE 删除一个商品 /product/${ID} 

4. 统一响应

这一点非常重要，特别是当我们新建一个商品时，商品的属性非常多，但如果我们缺少某个属性时。可以使用这样的一种统一的响应格式：

``` javascript
{
    "code": 100, // 0 表示成功
    "errors": { // 错误明细
        "title": "商品名称必填"
    }
}
```

其中 code 不管成功与否都会有该属性。

5. 状态码

后端响应一个请求是包括状态码和响应内容，而每一种状态码又包含着不同的含义。

* 200 成功返回请求数据
* 401 无权限
* 404 无效资源

## 二、如何访问Http？

首先，需要导入 HttpClientModule 模块。

``` javascript
import {
    HttpClientModule
} from '@angular/common/http';
@NgModule({
    imports: [
        HttpClientModule
    ]
})
```

然后，在组件类注入 HttpClient。

``` javascript
export class IndexComponent {
    constructor(private http: HttpClient) {}

}
```

最后，请求点击某个按钮发送一次GET请求。

``` javascript
user: Observable < User > ;
getUser() {
    this.user = this.http.get < User > ('/assets/data/user.json');
}
```

打印结果：

``` javascript
{
    {
        user | async |json
    }
}
```

三个简单的步骤，就是一个完整的HTTP请求步骤。
然后，现实与实际是有一些距离，比如说身份认证、错误处理、状态码处理等问题，在上面并无任何体现。
可，上面已经足够优雅，要让我破坏这种优雅那么此文就变得无意义了！
因此……

## 三、拦截器

1. `HttpInterceptor` 接口

正如其名，我们在不改变上面应用层面的代码下，允许我们把身份认证、错误处理、状态码处理问题给解决了！
写一个拦截器也是非常的优雅，只需要实现 `HttpInterceptor` 接口即可，而且只有一个 intercept 方法。 

``` javascript
@Injectable()
export class JWTInterceptor implements HttpInterceptor {
    intercept(req: HttpRequest < any > , next: HttpHandler): Observable < HttpSentEvent | HttpHeaderResponse | HttpProgressEvent | HttpResponse < any > | HttpUserEvent < any >> {
        // doing
    }
}
```

intercept 方法有两个参数，它几乎所当下流行的中间件概念一般，req 表示当前请求数据（包括：url、参数、header等），next 表示调用下一个“中间件”。

2. 身份认证

req 有一个 clone 方法，允许对当前的请求参数进行克隆并且这一过程会自行根据一些参数推导，不管如何用它来产生一个新的请求数据，并在这个新数据中加入我们期望的数据，比如：token。

``` javascript
const jwtReq = req.clone({
    headers: req.headers.set('token', 'xxxxxxxxxxxxxxxxxxxxx')
});
```

当然，你可以再折腾更多请求前的一些配置。
最后，把新请求参数传递给下一个“中间件”。

``` javascript
return next.handle(jwtReq);
```

等等，都 return 了，说好的状态码、异常处理呢？

3. 异常处理

仔细再瞧 next.handle 返回的是一个 Observable 类型。看到 Observable 我们会想到什么？flatMap、catch 等一大堆东西。
因此，我们可以利用这些操作符来改变响应的值。

**flatMap**

请求过程中会会有一些过程状态，比如请求前、上传进度条、请求结束等，Angular在每一次这类动作中都会触次 next。因此，我们只需要在返回 Observable 对象加上 flatMap 来观察这些值的变更，这样有非常大的自由空间想象。

``` javascript
return next.handle(jwtReq).flatMap((event: any) => {
    if (event instanceof HttpResponse && event.body.code !== 0) {
        return Observable.create(observer => observer.error(event));
    }
    return Observable.create(observer => observer.next(event));
})
```

只会在请求成功才会返回一个 HttpResponse 类型，因此，我们可以大胆判断是否来源于 HttpResponse 来表示HTTP请求已经成功。
这里，统一对业务层级的错误 code !== 0 产生一个错误信号的 Observable。反之，产生一个成功的信息。

**catch**

catch 来捕获非200以外的其他状态码的错误，比如：401。同时，前面的 flatMap 所产生的错误信号，也会在这里被捕获到。

``` javascript
.catch((res: HttpResponse < any > ) => {
    switch (res.status) {
        case 401:
            // 权限处理
            location.href = ''; // 重新登录
            break;
        case 200:
            // 业务层级错误处理
            alert('业务错误：' + res.body.code);
            break;
        case 404:
            alert('API不存在');
            break;
    }
    return Observable.throw(res);

})
```

4. 完整代码

至此，拦截器所要包括的身份认证token、统一响应处理、异常处理都解决了。

``` javascript
@Injectable()
export class JWTInterceptor implements HttpInterceptor {
    constructor(private notifySrv: NotifyService) {}
    intercept(req: HttpRequest < any > , next: HttpHandler): Observable < HttpSentEvent | HttpHeaderResponse | HttpProgressEvent | HttpResponse < any > | HttpUserEvent < any >> {
        console.log('interceptor')
        const jwtReq = req.clone({
            headers: req.headers.set('token', 'asdf')
        });
        return next
            .handle(jwtReq)
            .flatMap((event: any) => {
                if (event instanceof HttpResponse && event.body.code !== 0) {
                    return Observable.create(observer => observer.error(event));
                }
                return Observable.create(observer => observer.next(event));
            })
            .catch((res: HttpResponse < any > ) => {
                switch (res.status) {
                    case 401:
                        // 权限处理
                        location.href = ''; // 重新登录
                        break;
                    case 200:
                        // 业务层级错误处理
                        this.notifySrv.error('业务错误', `错误代码为：${res.body.code}`);
                        break;
                    case 404:
                        this.notifySrv.error('404', `API不存在`);
                        break;
                }
                // 以错误的形式结束本次请求
                return Observable.throw(res);
            })
    }

}
```

发现没有，我们并没有加一大堆并不认识的事物，单纯都只是对数据流的各种操作而已。

NotifyService 是一个无须依赖HTML模板、极简Angular通知组件。

5. 注册拦截器

拦截器构建后，还需要将其注册至 HTTP_INTERCEPTORS 标识符中。

``` javascript
import {
    HttpClientModule
} from '@angular/common/http';

@NgModule({
    imports: [
        HttpClientModule
    ],
    providers: [{
        provide: HTTP_INTERCEPTORS,
        useClass: JWTInterceptor,
        multi: true
    }]
})
```

以上是拦截器的所有内容，在不改变原有的代码的情况下，我们只是利用短短几行的代码实现了身份认证所需要的TOKEN、业务级统一响应处理、错误处理动作。

## 四、async 管道

一个 `Observable` 必须被订阅以后才会真正的开始动作，前面在HTML模板中我们利用了 `async` 管道简化了这种订阅过程。

``` javascript
{
    {
        user | async |json
    }
}
```

它相当于：

``` javascript
let user: User;
get() {
    this.http.get < User > ('/assets/data/user.json').subscribe(res => {
        this.user = res;
    });
}
```

``` javascript
{
    {
        user | json
    }
}
```

然而， `async` 这种简化，并不代表失去某些自由度，比如说当在获取数据过程中显示【加载中……】，怎么办？

```javascript 
<div *ngIf="user | async as user; else loading">

    {
    {
        user | json
    }

}

</div>
<ng-template #loading>加载中……</ng-template>
```

## 五、结论

Angular在HTTP请求过程中使用 `Observable` 异步数据流控制数据，而利用 rxjs 提供的大量操作符，来改变最终值；从而获得在应用层面最优雅的编码风格。
当我们说到优雅使用HTTP这件事时，易测试是一个非常重要，因此，我建议将HTTP从组件类中剥离并将所有请求放到 Service 当中。当对某个组件编写测试代码时，如果受到HTTP请求结果的限制会让测试更困难。
