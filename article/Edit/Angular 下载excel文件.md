假设有一个接口返回 excel 文件，我们前段需要请求这个接口获得这个文件，然后保存到本地，达到下载的效果。

首先我们是需要去调用这个接口的，接口返回的 body 里面是一个 excel 文件。

在 Angular 中使用 HttpClient 处理的 GET/POST 请求都是 json 数据格式的处理，那文件响应如何处理？

```typescript
get(
    url: string;
    options: {
        headers?: HttpHeaders | {[header: string]: string | string[]};
        observe?: HttpObserve;
        params?: HttpParams | {[param: string] : string | string[]};
        reportProgress?: boolean;
        responseType?: "arraybuffer" | "blob" | "json" | "text";
        withCredentials?: boolean;
    } = {},
): Observable<any>;
```

在 Angular 中有 15 重重载：

-   observe: 'body';，responseType: 'arraybuffer'; return Observable<ArrayBuffer>。
-   observe: 'body';，responseType: 'blob'; return Observable<Blob>。
-   observe: 'body';，responseType: 'text'; return Observable<string>。
-   observe: 'body';，responseType: 'json'; return Observable<Object>。
-   observe: 'body';，responseType: 'json'; return Observable<T>。
-   observe: 'events';，responseType: 'arraybuffer'; return Observable<ArrayBuffer>。
-   observe: 'events';，responseType: 'blob'; return Observable<Blob>。
-   observe: 'events';，responseType: 'text'; return Observable<string>。
-   observe: 'events';，responseType: 'json'; return Observable<HttpEvent<Object>>。
-   observe: 'events';，responseType: 'json'; return Observable<HttpEvent<T>>。
-   observe: 'response';，responseType: 'arraybuffer'; return Observable<HttpResponse<ArrayBuffer>>。
-   observe: 'response';，responseType: 'blob'; return Observable<HttpResponse<Blob>>。
-   observe: 'response';，responseType: 'text'; return Observable<HttpResponse<string>>。
-   observe: 'response';，responseType: 'json'; return Observable<HttpResponse<Object>>。
-   observe: 'response';，responseType: 'json'; return Observable<HttpResponse<T>>。

发现正好有我们需要的重载，我们需要监听 response，并且返回的 body 里面是 Blob，那么我们可以这样发起一个请求：

```typescript
export class TestComponent implements OnInit {
    construct(private _http: HttpClient) {}

    ngOnInit() {
        this._http
            .post(url, params, {
                responseType: "blob",
                observe: "events",
            })
            .subscribe((res: Response<Blob>) => {
                // 其他需求
            });
    }
}
```

这里需要看下 HttpResponse， 它有个属性 type：

-   Sent = 0 请求通过网络发送
-   UploadProgress = 1 接收到上传事件
-   ResponseHeader = 2 接收到响应状态码和标头
-   DownloadProgress = 3 收到下载进度事件
-   Response = 4 收到 body 在内的所有响应
-   User = 5 来自拦截器或后端的自定义事件（哦哟，这个好像很有意思 🤔）

也就是说我们监测到 HttpResponse.type === 4 的时候才可以去处理响应里面的 Blob：

```typescript
this._http
    .post(url, params, {
        responseType: "blob",
        observe: "events",
    })
    .subscribe((res: Response<Blob>) => {
        if (res.type !== 4) {
            return;
        }
        // 处理响应里面的blob文件
    });
```

## 处理下载

HTML 中的<a>标签我们一般来用于导航，但它有一个很有用的属性 download。这个属性指示浏览器下载 URL 而不是导航到它，因此将提示用户将其保存为本地文件。如果 download 属性有一个值，那么这个值将在下载保存过程中作为预填充的文件名（如果用户需要，仍然可以修改保存的文件名）。此属性对允许的值没有限制，但是/和\会被转换为下划线。大多数文件系统限制了文件中的标点符号，故此，浏览器将响应的调整建议的文件名。参考 MDN 标签。

需要注意的是，这个熟悉仅适用于同源 URL。当然我们可以使用 blob: URL 和 data: URL 以方便用户下载使用 JavaScript 生成的内容（比如使用 canvas 生成的图片等）。

如果 HTTP 响应头信息中有属性 Content-Diposition 设置了文件名，那么浏览器会优先使用此属性。

那我们可以来完成下载逻辑：

```typescript
const objUrl = window.URL.createObjectURL(res.body);
const a = document.createElement("a");
a.href = objUrl;
let fileName = "excel.xls";
a.download = decodeURIComponent(fileName);
a.click();
window.URL.revokeObjectURL(objURL);
```

最终代码：

```typescript
export class TestComponent implements OnInit {
    construct(private _http: HttpClient) {}

    ngOnInit() {
        this._http
            .post(url, params, {
                responseType: "blob",
                observe: "events",
            })
            .subscribe((res: Response<Blob>) => {
                if (res.type !== 4) {
                    // 还没准备好，无需处理
                    return;
                }
                // 处理下载
                const objUrl = window.URL.createObjectURL(res.body);
                const a = document.createElement("a");
                a.href = objUrl;
                let fileName = "excel.xls";
                a.download = decodeURIComponent(fileName);
                a.click();
                window.URL.revokeObjectURL(objURL);
            });
    }
}
```

```typescript

```
