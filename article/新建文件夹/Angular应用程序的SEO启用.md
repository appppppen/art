我假设您正在为全球所有用户建立第一个网站。也许您正在构建它，用于教育/商业/市场营销，但是一旦完成，就需要查看用户访问您的应用程序并浏览内容。为了吸引普通用户，您的简单域名还不足以鼓励用户浏览您的应用程序，毕竟，它不是 Google，Facebook，Microsoft，Apple 等。我们需要依靠他们的服务才能将我们的应用程序带到普通用户手中用户的眼睛。

> 我们需要启用少量功能并保持少量原理，才能将我们的应用程序添加到搜索引擎的索引下。现在，取决于参数，令牌，索引等级的关键字将上升和下降。让我们看看我们如何逐步做到这一点。

## 第 1 步：配置 sitemap.xml 文件和 robots.txt 文件

您可以将所有外部公共访问 URL 包括到 sitemap.xml 文件中，以便搜索引擎可以了解该页面的存在并可以对该页面进行爬网。您也可以从这里创建它。它应具有以下结构，如下所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9" xmlns:image="http://www.google.com/schemas/sitemap-image/1.1">
    <url>
        <loc>https://lazypandatech.com/</loc>
        <changefreq>weekly</changefreq>
        <priority>0.8</priority>
        <lastmod>2020-03-24T11:01:25+00:00</lastmod>
        <image:image>
            <image:loc>image-link</image:loc>
            <image:title>Home</image:title>
        </image:image>
    </url>
</urlset>
```

robots.txt 文件的配置也非常重要。它告诉搜索引擎机器人抓取哪些页面，哪些不抓取。使用它，您可以限制对网站的特定目录进行爬网。

robots.txt 的外观如何-

```typescript
Sitemap: https://lazypandatech.com/sitemap.xml
User-agent: *
Disallow: /admin/
```

## 步骤 2：尽量不要使用深层链接

深层链接是指带有＃的 URL，例如`www.your-site.com/#/home`。这些很难被索引。因此，在创建路由时，请尝试为页面使用简单路径。并在路由模块中进行以下更改。

例如：`www.your-site.com/blog/ios/layout`

```typescript
RouterModule.forRoot(routes, {
  scrollPositionRestoration: "enabled",
  anchorScrolling: "enabled",
  useHash: false,
  enableTracing: false,
});
```

## 步骤 3：尽量不要使用\* ngIf 隐藏内容

尽量不要使用 ngIf 来隐藏和显示 HTML 页面上的内容。相反，您可以利用 CSS 功能来隐藏和显示，如下所示

```css
.hide {
  display: none;
}
.show {
  display: block;
}
```

## 步骤 4：尽量不要使用“虚拟锚”

许多应用程序使用以下行从一页导航到另一页。

this.router.navigate([‘/home’]);

请使用简单的定位标记（`<a>`）或 routerLink 导航页面。您还可以基于防护服务添加页面导航限制。

## 步骤 5：标题也很重要

标题对于搜索引擎搜寻器很重要。每个页面应该有一个不同的标题。

请注意–包含标题标签的组件的排列方式不应使<h1>出现在<h2>内部。

## 第 6 步：为您的页面添加元标记

@ ngx-meta 是很好的角度库之一，可用于动态包含/更新标签。

https://www.npmjs.com/package/@ngx-meta/core

请通过链接和说明在您的应用程序中添加元标记。您也可以使用角度提供的元服务来添加/更新/删除元标签。像下面-

```typescript
import { Meta } from "@angular/platform-browser";

// update tags based on particular blog
// general
this.metaTagService.updateTag({ name: "author", content: "Lazy Panda" });
this.metaTagService.updateTag({
  name: "sitemap",
  type: "application/xml",
  content: "https://lazypandatech.com/sitemap.xml",
});
this.metaTagService.updateTag({ name: "googlebot", content: "index/follow" });
this.metaTagService.updateTag({ name: "theme-color", content: "#59ab64" });
this.metaTagService.updateTag({
  name: "msapplication-navbutton-color",
  content: "#59ab64",
});
this.metaTagService.updateTag({
  name: "apple-mobile-web-app-status-bar-style",
  content: "#59ab64",
});

// Schema.org markup for Google+
this.metaTagService.updateTag({ name: "name", content: title });
this.metaTagService.updateTag({ name: "description", content: description });
this.metaTagService.updateTag({
  name: "article:modified_time",
  content: new Date().toISOString(),
});
this.metaTagService.updateTag({
  name: "article:author",
  content: "https://www.facebook.com/sudiptapossible/",
});
this.metaTagService.updateTag({
  name: "article:publisher",
  content: "https://www.facebook.com/Lazy-Panda-Tech-108217420821637",
});

// markup for facebook
this.metaTagService.updateTag({ name: "og:title", content: title });
this.metaTagService.updateTag({ name: "og:description", content: description });
this.metaTagService.updateTag({ name: "description", content: description });
this.metaTagService.updateTag({ name: "og:type", content: "blog" });
this.metaTagService.updateTag({ name: "og:url", content: url });
this.metaTagService.updateTag({ name: "og:keywords", content: keyWords });
this.metaTagService.updateTag({
  name: "og:site_name",
  content: "LazyPandaTech",
});
this.metaTagService.updateTag({ name: "og:locale", content: "en_US" });

// markup for twitter
this.metaTagService.updateTag({
  name: "twitter:card",
  content: "LazyPandaTech",
});
this.metaTagService.updateTag({ name: "twitter:title", content: title });
this.metaTagService.updateTag({
  name: "twitter:description",
  content: description,
});
this.metaTagService.updateTag({
  name: "twitter:creator",
  content: "Lazy Panda",
});
```

通用标签也可以添加到 index.html 页面中。

`<meta property="og:type" content="article">`

`<meta name="robots" content="index, follow">`

另外，您可以将站点验证 ID 添加到 HTML 中。就我而言，Google 是我的网站验证提供商，因此我也添加了以下几行。

`<meta name="google-site-verification" content="id-goes-here" />`

注意：在网页中设置适当的元标记将有助于 Google 在其搜索结果中正确显示有关网页的标题，URL 和描述。

## 步骤 7：使用 Angular Universal 进行服务器端渲染

由于 angular 是客户端应用程序，因此需要下载应用程序捆绑包以加载 DOM，搜索引擎搜寻器将无法找到内容，因为这些内容尚未加载。为了克服此问题，请以服务器呈现的形式为应用程序提供服务。最好使用 Angular Universal。

![](https://lazypanda-blog-images.s3.ap-south-1.amazonaws.com/angular_SEO.png)

为什么要使用 Angular Universal？

如果您的应用程序以 Google 搜索引擎为目标，则可能不需要，因为 Google 搜索会抓取 JavaScript。但是，其他搜索引擎和社交网站将需要在服务器端进行渲染，以对您的页面进行爬网以建立索引。
将现有的角度应用程序迁移到通用角度应用程序：

1. 安装以下节点模块

ng add @nguniversal/express-engine （它将自动更新您的所有代码。）

2. 安装完成后，请运行以下命令在 http：// localhost：4000 中为您的应用程序提供服务

npm run build:ssr && npm run serve:ssr

在此命令期间，您可能会收到以下错误-

Node Express 服务器在 http：// localhost：4000 上侦听

错误 ReferenceError：窗口未定义

![](https://lazypanda-blog-images.s3.ap-south-1.amazonaws.com/error.png)

要解决此错误，您需要修改 server.ts 文件中的少量代码-（一个完整的 server.ts 文件，您可以复制整个代码）

```typescript
import "zone.js/dist/zone-node";
import { ngExpressEngine } from "@nguniversal/express-engine";
import * as express from "express";
import { join } from "path";
import { AppServerModule } from "./src/main.server";
import { APP_BASE_HREF } from "@angular/common";
import { existsSync } from "fs";

const domino = require("domino");
const fs = require("fs");
const path = require("path");
const template = fs
  .readFileSync(path.join(".", "dist/blog-fe/browser", "index.html"))
  .toString();
const win = domino.createWindow(template);

// tslint:disable-next-line:no-string-literal
global["window"] = win;
// tslint:disable-next-line:no-string-literal
global["document"] = win.document;
// tslint:disable-next-line:no-string-literal
global["DOMTokenList"] = win.DOMTokenList;
// tslint:disable-next-line:no-string-literal
global["Node"] = win.Node;
// tslint:disable-next-line:no-string-literal
global["Text"] = win.Text;
// tslint:disable-next-line:no-string-literal
global["HTMLElement"] = win.HTMLElement;
// tslint:disable-next-line:no-string-literal
global["navigator"] = win.navigator;
// tslint:disable-next-line:no-string-literal
global["MutationObserver"] = getMockMutationObserver();

function getMockMutationObserver() {
  return class {
    observe(node, options) {}
    disconnect() {}
    takeRecords() {
      return [];
    }
  };
}

// The Express app is exported so that it can be used by serverless Functions.
export function app() {
  const server = express();
  const distFolder = join(process.cwd(), "dist/blog-fe/browser");
  const indexHtml = existsSync(join(distFolder, "index.original.html"))
    ? "index.original.html"
    : "index";

  server.engine(
    "html",
    ngExpressEngine({
      bootstrap: AppServerModule,
    })
  );

  server.set("view engine", "html");

  server.set("views", distFolder);

  server.get(
    "*.*",
    express.static(distFolder, {
      maxAge: "1y",
    })
  );

  // All regular routes use the Universal engine
  server.get("*", (req, res) => {
    res.render(indexHtml, {
      req,
      providers: [{ provide: APP_BASE_HREF, useValue: req.baseUrl }],
    });
  });
  return server;
}

function run() {
  const port = process.env.PORT || 4000;
  // Start up the Node server
  const server = app();
  server.listen(port, () => {
    console.log(`Node Express server listening on http://localhost:${port}`);
  });
}

// Webpack will replace 'require' with '__webpack_require__'
// '__non_webpack_require__' is a proxy to Node 'require'
// The below code is to ensure that the server is run only when not requiring the bundle.

declare const __non_webpack_require__: NodeRequire;
const mainModule = __non_webpack_require__.main;
const moduleFilename = (mainModule && mainModule.filename) || "";

if (moduleFilename === __filename || moduleFilename.includes("iisnode")) {
  run();
}

export * from "./src/main.server";
```

然后再次运行，并查看浏览器选项卡– `http://localhost:4000`，该应用程序将平稳运行。

### 注意：如果您的应用程序使用的是 localHost，则可能会遇到另一个问题

使用以下步骤可以解决此问题：

1. 在构造函数中注入以下服务

```typescript
@Inject(PLATFORM_ID) private platformId: object
```

2. 导入缺少的依赖项

```typescript
import { Injectable, Inject, PLATFORM_ID } from "@angular/core";
```

3. 使用代码段

```typescript
let token = "";
if (isPlatformBrowser(this.platformId)) {
  token = localStorage.getItem("access_token");
}
```

4. 构建并运行

## 步骤 8：将您的 sitemap.xml 文件 ping 到 google 或通过网络浏览器 bing。

您可以使用它从以下端点 ping 您的站点地图。

- http://www.google.com/webmasters/sitemaps/ping?sitemap=URLOFSITEMAP
- http://www.bing.com/webmaster/ping.aspx?siteMap=URLOFSITEMAP

第 9 步：最后但并非最不重要的一点是，您可以利用 Google 搜索 API 来抓取您的页面。

Google 搜索 API 是确保您的网页在接下来的 48 小时内（可能）得到抓取的另一种方法。要使用 Google 搜索 API，您需要创建一个服务帐户，然后再启用搜索 API。完整的文档在[这里](https://developers.google.com/search/apis/indexing-api/v3/reference/indexing/rest/v3/urlNotifications/publish)。您可以使用自动 API 控制台提交端点，但是我建议创建一个服务帐户，将您自己添加为所有者，并编写一个简单的 node js 应用程序以提交端点。

我用来提交端点的 nodejs 脚本如下所示-
![](https://lazypanda-blog-images.s3.ap-south-1.amazonaws.com/seo-script.png)
