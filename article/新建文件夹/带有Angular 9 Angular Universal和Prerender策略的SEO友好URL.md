对于 SPA（单页应用程序），要使搜索引擎爬网到您的网站，查找关键字并将其列在 google / bing 搜索结果的辛勤工作中确实很困难。我一直在尝试将搜索引擎抓取到我的网站，以便各个博客都可以显示在搜索结果中。但是尽管取得了成功，但每次我在 sitemap.xml 文件中进行修复并要求 google 抓取我的页面时，它都被 404（找不到页面）拒绝，尽管我的页面可以在浏览器中正确打开并且可以正常工作好吧。然后我意识到，URL 结构中非常需要某些规则，并且当您在应用程序中构建路由规则时，您必须考虑所有规则，然后如此配置您的应用程序，以便 Google 可以抓取并获取所有关键字。

``` 
"
SEO友好的URL创建以及将Universal与Prerender策略结合使用，可以使网页表现良好，并提供流畅的用户体验。我们开始做吧...
"
```

我相信您已经使用 Angular Universal 构建了您的应用程序，如果没有，并且有兴趣使用 Universal，请查看我以前的博客“[ Angular 应用程序的 SEO 启用](https://lazypandatech.com/blog/Angular/2/SEO-Enablement-to-an-Angular-application)”。Angular Universal 有两个部分-

* 服务器端渲染（SSR）

  在这种情况下，Angular 应用程序将运行到节点服务器中。例如，如果您使用 AWS 存储桶进行代码部署，则无法使用静态网页托管来执行此操作，因为节点服务器将不会运行。您必须使用 EC2 实例或无服务器 Lambda 函数来部署代码，为此，您需要支付 EC2 实例运行或执行无服务器函数所需的费用。最终可能会产生大量账单。
  当请求从客户端浏览器发送到节点服务器时，服务器端渲染不像客户端渲染那样首先，节点服务器将处理您的请求，然后将其发送回客户端，客户端浏览器将渲染页面。它比客户端渲染慢得多。

* 客户端渲染

  客户端渲染正在利用 Prerender 策略。在构建期间，它将根据 URL 结构在 dist 文件夹中生成不同的 index.html 文件。由于 dist 文件夹中已经存在所有相应的 HTML，因此它非常快，并且当浏览器发出请求时，节点服务器将完全不运行。这就是为什么此代码只能用作静态网页托管的 S3 存储桶的原因。无需 EC2 实例或无服务器 Lambda 函数即可运行您的网站。您可以使用 S3 存储桶以非常低的成本（约<1 美元）托管网站

在此博客中，我将使用具有角度通用性的 Prerender 策略，以便在性能和启用 SEO 方面，各自的网站将是第一个。要使用预渲染，您需要正确配置路由，以便搜索引擎可以抓取您的网页。

不使用基于查询字符串的 URL-（深层链接）

示例：（错误） Google 永远不会抓取您返回 404 的页面，尽管该页面将在实际环境中运行。

https://example.com/blog/details?id=10&category=5&title=My%20first%20blog

示例：（正确） Google 会抓取您的页面

https://example.com/details/iOS/3/my-first-blog

创建网址结构时，需要注意以下几点：

1. 您的 URL 不应在任何登录名后面。搜索引擎只能爬网可公开访问的页面（未经身份验证的页面）。
2. 使用您的关键字来构建 URL，请勿从任何其他网站复制该 URL。
3. 构建您的 URL 以供将来增强。

   像https://example.com/blog/mobile/ andoid / fragments
   https://example.com/blog/mobile/ iOS / story-board-layout

4. 如果要在 URL 中添加标题，请确保使用“-”（连字符）而不是“ \_”（下划线）。URL 编码对于爬网没有太大帮助。
5. 请勿在您的 URL 中多次使用相同的单词。
6. URL 长度必须小于 512 像素，Google 的其余部分将在其搜索结果页面中截断。尽可能多地使用短网址是很好的。
7. 正确使用 URL 的规范标签。
8. 同时创建您的应用程序 sitemap.xml 和 robots.txt 文件。

应该创建以下网址结构-清晰明了-

![](https://lazypanda-blog-images.s3.ap-south-1.amazonaws.com/1586867823546.jpg)

创建另一个名为- `app.browser.module.ts ` 的模块-

``` typescript
@NgModule({
    imports：[
        BrowserModule.withServerTransition({appId：'serverApp'}),
        AppModule,
        BrowserTransferStateModule
    ],
    bootstrap:[AppComponent],
})

export class AppBrowserModule {}
```

创建一个 HTTP 拦截器，或者如果您已经拥有 HTTP 拦截器，则使用以下代码进行增强-

``` typescript
intercept(request: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {

    const timeoutValue = request.headers.get('timeout') || this.defaultTimeout;
    const timeoutValueNumeric = Number(timeoutValue);
    const key: StateKey<string> = makeStateKey<string>(request.url);

    if (isPlatformServer(this.platformId)) {
        // serverSide
        return next.handle(request).pipe(
            timeout(timeoutValueNumeric),
            tap((event) => {
                this.transferState.set(key, (event as HttpResponse<any>).body);
            }),
            catchError((error: HttpErrorResponse) => {
                return this.networkErrorScenario(error, request, next);
            })
        );
    } else {
        // browserSide
        const storedResponse = this.transferState.get<any>(key, null);
        if (storedResponse) {
            const response = new HttpResponse({body: storedResponse, status: 200});
            return of(response);
        } else {
            return next.handle(request).pipe(
                timeout(timeoutValueNumeric),
                tap((response: HttpResponse<any>) => {
                return response;
                }),
                catchError((error: HttpErrorResponse) => {
                return this.networkErrorScenario(error, request, next);
                })
            );
        }
    }
}
```

也如下更改 main.ts 文件-

``` typescript
document.addEventListener("DOMContentLoaded", () => {
  platformBrowserDynamic()
    .bootstrapModule(AppBrowserModule)
    .catch((err) => console.error(err));
});
```

在父文件夹中创建一个 route.txt 文件，并在其中提到不同的路由路径。您也可以通过编程方式添加路径，以在构建期间运行一个简单的 js 文件。

进口 `ServerTransferStateModule` 到 `app.server.module.ts` 文件中。就是这样，现在建立- `npm run prerender`

您将看到以下区别-

在提交之前- `npm build --prod`

![](https://lazypanda-blog-images.s3.ap-south-1.amazonaws.com/1586870746405.jpg)

在 Prerender 之后-dist 文件夹结构-

![](https://lazypanda-blog-images.s3.ap-south-1.amazonaws.com/1586870935710.jpg)

由于它是基于路由创建的多个 index.html 文件，因此它通常比服务器端渲染和 SEO 控制的速度快 5 倍。有关更多信息，请在下面的视频中通过某种形式的路由自动化可视化整个应用程序。此外，Google Search Console 网站上还展示了分析优化节目，介绍了应用程序的性能以及如何在该网站上发布您的站点地图。
![](https://youtu.be/eY7nzjYLZTM)
随时添加评论和建议。

快乐编码与探索！
