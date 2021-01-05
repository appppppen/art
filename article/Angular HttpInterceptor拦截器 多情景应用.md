HttpInterceptor，是 Angular 提供用于在全局应用程序级别处理 HTTP 请求的内置工具，拦截并处理 HttpRequest 或 HttpResponse。拦截器在实战中的作用有很多，比如：统一配置网关地址，设置 Http 请求头，处理 Http 请求返回数据，统一错误处理等都是常见的需求。

这里介绍三种不同的拦截器的实现：

-   处理 Http 请求头 (Http Request Headers)
-   处理 Http 响应 (Http Response)
-   Http 错误处理 (Http Error)

先看一下拦截器的最基本的实现

```typescript
import { Injectable } from "@angular/core";
import {
    HttpInterceptor,
    HttpEvent,
    HttpResponse,
    HttpRequest,
    HttpHandler,
} from "@angular/common/http";
import { Observable } from "rxjs";

@Injectable()
export class CustomInterceptor implements HttpInterceptor {
    public intercept(
        req: HttpRequest<any>,
        next: HttpHandler
    ): Observable<HttpEvent<any>> {
        return next.handle(req);
    }
}
```

要创建一个自定义拦截器，我们需要实现 @angular/common/http 中提供的 HttpInterceptor。每一个通过 HttpClient 调用的请求，都会触发拦截器调用 intercept()方法。当调用 intercept()方法时，Angular 会向 httpRequest 对象传递一个引用对象。然后我们可以根据需要修改它。完成处理之后，调用 next，将更新后的请求返回到应用程序。
我们需要将拦截器其注册为一个多提供者，因为在一个应用程序中可以运行多个拦截器。注：拦截器将只拦截使用 HttpClient 服务发出的请求。

```typescript
import { NgModule } from "@angular/core";
import { BrowserModule } from "@angular/platform-browser";
import { HttpClientModule, HTTP_INTERCEPTORS } from "@angular/common/http";

import { CustomInterceptor } from "./custom.interceptor";
import { AppComponent } from "./app.component";

@NgModule({
    imports: [BrowserModule, HttpClientModule],
    declarations: [AppComponent],
    bootstrap: [AppComponent],
    providers: [
        {
            provide: HTTP_INTERCEPTORS,
            useClass: CustomInterceptor,
            multi: true,
        },
    ],
})
export class AppModule {}
```

下面根据我们的需求实现我们的 Http 拦截器

## 处理 Http 请求头 (Http Request Headers Interceptor)

通常我们处理 Http 请求的时候，需要在请求头中加入一些身份验证信息，比如 token,我们可以通过拦截器为所有的请求加上 token 信息

```typescript
import { Injectable } from "@angular/core";
import {
    HttpInterceptor,
    HttpRequest,
    HttpHandler,
    HttpEvent,
} from "@angular/common/http";
import { Observable } from "rxjs";

@Injectable()
export class HeaderInterceptor implements HttpInterceptor {
    public intercept(
        req: HttpRequest<any>,
        next: HttpHandler
    ): Observable<HttpEvent<any>> {
        return next.handle(this.addToken(req, "bearer my token"));
    }

    private addToken(req: HttpRequest<any>, token: string): HttpRequest<any> {
        return req.clone({ setHeaders: { Authorization: token } });
    }
}
```

创建一个 Http 请求，来验证我们添加的 token

```typescript
 constructor(private httpClient: HttpClient) {
        this.httpClient.get('/assets/json/data.json').subscribe(ret => {
            // TODO:
        });
    }
```

[](https://upload-images.jianshu.io/upload_images/8730589-bb7ccad16725ea6b.png?imageMogr2/auto-orient/strip|imageView2/2/w/510/format/webp)

## 处理 Http 响应 (Http Response Interceptor)

当我们需要对 Http 请求的返回数据进行统一处理时，例如，当返回结果是如下格式，status 为 200 的时候，我们只保留我们需要的信息。

```typescript
{
  "data": [
      { "id": 1, "name": "赵钱" },
      { "id": 2, "name": "孙李" },
      { "id": 3, "name": "周吴" },
      { "id": 4, "name": "郑王" }
  ],
  "requestDatetime": "1949-10-01"
}
```

根据上面的需求，来新增一个 ResponseHandlerInterceptor

```typescript
import {
    HttpEvent,
    HttpHandler,
    HttpInterceptor,
    HttpRequest,
    HttpResponse,
} from "@angular/common/http";
import { Injectable } from "@angular/core";
import { Observable } from "rxjs";
import { filter, map } from "rxjs/operators";

@Injectable()
export class ResponseHandlerInterceptor implements HttpInterceptor {
    public intercept(
        req: HttpRequest<any>,
        next: HttpHandler
    ): Observable<HttpEvent<any>> {
        return next.handle(req).pipe(
            filter(
                (event) => event instanceof HttpResponse && event.status === 200
            ),
            map((event: HttpResponse<any>) =>
                event.clone({ body: event.body.data })
            )
        );
    }
}
```

上面的代码中，当返回的数据状态是 200 的时候，我们只返回 body 中我们需要的 data 数据。

## Http 错误处理 (Http Error Interceptor)

在 Http 请求中，我们需要对错误情况进行统一处理，这时候，我们也可以借助拦截器实现（错误提示或者重试等）。

```typescript
import {
    HttpEvent,
    HttpHandler,
    HttpInterceptor,
    HttpRequest,
} from "@angular/common/http";
import { Injectable } from "@angular/core";
import { Observable, throwError } from "rxjs";
import { catchError, retry } from "rxjs/operators";

@Injectable()
export class ErrorHandlerInterceptor implements HttpInterceptor {
    intercept(
        req: HttpRequest<any>,
        next: HttpHandler
    ): Observable<HttpEvent<any>> {
        return next.handle(req).pipe(
            catchError((error) => {
                switch (error.status) {
                    case 401: // Unauthorized
                        // todo
                        break;
                    case 403: // Forbidden
                        // todo
                        break;
                    default:
                        // todo
                        return throwError(error);
                }
            }),
            retry(3)
        );
    }
}
```

在上面的 ErrorHandlerInterceptor 拦截器会拦截 http 请求的错误情况，我们可以根据具体的业务需求进行处理，也可以通过 retry(3)进行错误重试（3 表示重试三次）
当然，创建的时候要记得配置注入

```typescript
providers: [
    {
        provide: HTTP_INTERCEPTORS,
        useClass: HeaderInterceptor,
        multi: true,
    },
    {
        provide: HTTP_INTERCEPTORS,
        useClass: ResponseHandlerInterceptor,
        multi: true,
    },
    {
        provide: HTTP_INTERCEPTORS,
        useClass: ErrorHandlerInterceptor,
        multi: true,
    },
];
```
