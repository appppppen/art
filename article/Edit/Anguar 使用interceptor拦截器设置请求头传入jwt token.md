1.创建 http-interceptors.ts 文件

```
import { Injectable } from "@angular/core";
import {
  HttpInterceptor,
  HttpRequest,
  HttpHandler,
  HttpErrorResponse,
  HttpHeaderResponse,
  HttpResponse,
  HttpEvent
} from "@angular/common/http";
import { Observable } from "rxjs";
import { finalize, tap } from "rxjs/operators";
import { HttpService } from "../http-service/http.service";
import { LoadingService } from "../loading/loading.service";

/** Pass untouched request through to the next request handler. */
@Injectable()
export class AuthInterceptor implements HttpInterceptor {
  constructor(
    private httpService: HttpService,
    private loadingService: LoadingService
  ) {}

  intercept(
    req: HttpRequest<any>,
    next: HttpHandler
  ): Observable<HttpEvent<any>> {
    const authToken = this.httpService.getAuthorizationToken();

    const authReq = req.clone({
      headers: req.headers.set("Authorization", authToken),
      url: req.url.replace(req.url, this.httpService.reqUrl + req.url)
    });

    return next.handle(authReq).pipe(
      tap(
        event => {
          this.loadingService.loading(true);
          if (event instanceof HttpResponse) {
            console.log("success");
          }
        },
        error => {
          console.log("failed");
        }
      ),
      finalize(() => {//请求完成（成功或失败都执行）
        this.loadingService.loading(false);
      })
    );
  }
}

```

2.在 app.module.ts 中添加以下代码：

```
import { HttpClientModule, HTTP_INTERCEPTORS } from "@angular/common/http";

@NgModule({
  providers: [
    HttpService,
    {
      provide: HTTP_INTERCEPTORS,
      useClass: AuthInterceptor,
      multi: true
    }
  ]
}）

```

3.httpService.ts 的代码

```
import { Injectable } from "@angular/core";
import { HttpClient, HttpParams, HttpHeaders } from "@angular/common/http";
import { Router } from "@angular/router";
import { Observable } from "rxjs";

@Injectable()
export class HttpService {
  public isLoding: boolean = false;
  public reqUrl = "http://www.test.com";//服务器地址

  constructor(private router: Router, private http: HttpClient) {}

  getAuthorizationToken() {
    const token = localStorage.getItem("token")
      ? localStorage.getItem("token")
      : "";
    if (token) {
      const jwt = `Bearer ${token}`;
      return jwt;
    } else {
      this.router.navigate(["/passport/login"]);
    }
  }
}

```

4.在组件中使用：

```
import { Component, OnInit, Input } from "@angular/core";
import { HttpClient } from "@angular/common/http";

@Component({
  selector: "app-table",
  templateUrl: "./table.component.html",
  styleUrls: ["./table.component.css"]
})
export class TableComponent implements OnInit {

  constructor(private http: HttpClient) {}

  ngOnInit() {
    this.getTableData();
  }

  getTableData() {
    this.http.get("/api/table").subscribe(res => {
      console.log(res);
    });
  }
}
```
