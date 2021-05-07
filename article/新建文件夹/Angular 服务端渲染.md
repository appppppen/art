## 服务端渲染 3 大好处

-   SEO 即搜索引擎优化，方便爬虫抓取网页。
-   提高应用在移动端和低性能设备上的体验
-   更快的展示页面。

## 工具包

angular 的服务端面渲染依赖于 platform-server 这个包，它里面封装了一些和 DOM 交互，发送 XMLHttpRequest 请求，以及一些在低性能设备上不支持的工具方法。我们将使用它在服务端渲染出整个应用。

在服务端代码中我们需要这个包中的 renderModuleFactory()方法。这个方法接收一个 HTML 模板作为输入，通常是 index.html，一个 angular 模块以及一个路由地址。每一个从客户端发送上来的路由信息都会被映射成一个静态页面，当客户端请求时，由 renderModuleFactor()方法负责渲染相应的视图，最终把静态页面发送给客户端。

## 开发流程

安装依赖。
更新 angular 前端代码。
在 angular.json 中修改构建配置。
使用 Nodejs 建立一个服务端。
服务端的 webpack 配置。
第 4，5 步实际需要根据具体的项目来设置，这里仅是为了演示。服务端渲染可以用在任何语言开发的服务器上，只需要服务器能够调用这个包中的 renderModuleFactory()方法即可。

## 开始

## 安装依赖

```
$ npm install --save @angular/platform-server @nguniversal/module-map-ngfactory-loader ts-loader
// 回显
```

第一个包前面已经介绍过了。@nguniversal/module-map-ngfactory-loader 用来处理懒加载的内容，ts-loader 用来协助 web-pack 进行构建。

## 更新 angular 前端代码

对需要服务端渲染的应用，我们需要进行以下改造：

1. Angular Universal 支持
   应用的根模块，通常是 app.module.ts 中添加以下配置

```
@NgModule({
  bootstrap: [AppComponent],
  imports: [
    BrowserModule.withServerTransition({appId: 'may-app'}), // 打包时的id，只要保证和其它配置不重复即可。
    ...
  ],

})
export class AppModule {}
```

2. 创建 server root 模块

这个模块作为在服务端运行时的根模块，这里直接引入原先应用的根模块 AppModule，然后添加一些服务端面渲染所需的模块。这里我们创建一个名为 app.server.module.ts 的文件，命名上就可以看出它是服务端渲染时的根模块。

import { NgModule } from '@angular/core';
import { ServerModule } from '@angular/platform-server';
import { ModuleMapLoaderModule } from '@nguniversal/module-map-ngfactory-loader';

import { AppModule } from './app.module';
import { AppComponent } from './app.component';

@NgModule({
imports: [
AppModule,
ServerModule,
ModuleMapLoaderModule // 如需处理懒加载模块，切记引入它！
],
bootstrap: [AppComponent] // 此模块需要明确知道从哪个组件启动
})
export class AppServerModule {}

3. 添加入口文件，暴露出 server root 模块

在 src/目录下创建 main.server.ts 的入口文件。

export { AppServerModule } from './app/app.server.module';

4. 配置 AppServerModule 文件

创建一个 tsconfig.server.json 文件，直接把 tsconfig.app.json 的内容复制过来，但是还更改一些配置。

{
"extends": "../tsconfig.json",
"compilerOptions": {
"outDir": "../out-tsc/app",
"baseUrl": "./",
"module": "commonjs", // 此处有变化
"types": []
},
"exclude": [
"test.ts",
"**/*.spec.ts"
],
// 新增下面的配置项，path#moduleName 的写法相信你已经见过了
"angularCompilerOptions": {
"entryModule": "app/app.server.module#AppServerModule"
}
}

## 在 angular.json 中修改构建配置

在 angular.json 中的 architect 增加写的新的打包配置，这里我们使用 server 作为打包的名称， 同时为了便于管理，把之前 build 时的输出路径修改成 browser。

```
"architect": {
  "build": {
  ...
  "options": {
        ...
        "outputPath": "dist/browser",
    }
  },
  "server": {
    "builder": "@angular-devkit/build-angular:server",
    "options": {
      "outputPath": "dist/server",
      "main": "src/main.server.ts",
      "tsConfig": "src/tsconfig.server.json"
    }
  }
}
```

输出后的 dist 目录结构

dist/

---browser/

---server/

使用 ng run 命令进行打包，打包名称格式为 项目名称:打包目标，假设项目名称为 my-project，构建刚设置好的 server 这个包，在命令行输入：

```
$ ng build --prod && ng run my-project:server
// 打包过程回显
```

执行完成后，dist/browser 目录下的文件就是正常的打包文件 ， 和不使用 SSR 时是一模一样的，dist/server 下的文件就是服务端渲染所需要的代码，你写的所有代码都会被压缩到它里面。

## 使用 Nodejs 建立一个服务端

把预编译好的 AppServerModule 传入到 PlatformServer 的 renderModuleFactory() 方法中，它会帮助我们初始化应用，将结果返回给客户端。其中，AppServerModuleNgFactory，provideModuleMap 和 LAZY_MODULE_MAP 是经过 webpack 打包后 通过 require 进来的参数或方法，renderModuleFactory 需要从@angular/platform-server 的包中 import 进来

```
app.engine('html', (_, options, callback) => {
  renderModuleFactory(AppServerModuleNgFactory, {
    document: template, // index.html
    url: options.req.url,
    // 懒加载模块配置
    extraProviders: [
      provideModuleMap(LAZY_MODULE_MAP)
    ]
  }).then(html => {
    callback(null, html);
  });
});
```

在使用 express 框架的 nodejs 中，还可以使用更加便捷的方法

## 安装依赖

`$ npm install --save @nguniversal/express-engine`

示例代码

```
import { ngExpressEngine } from '@nguniversal/express-engine';

app.engine('html', ngExpressEngine({
  bootstrap: AppServerModuleNgFactory,
  providers: [
    provideModuleMap(LAZY_MODULE_MAP)
  ]
}));
```

基于 nodejs，express 的完整示例代码

```
// 下面2行必须首先引入
import 'zone.js/dist/zone-node';
import 'reflect-metadata';

import { renderModuleFactory } from '@angular/platform-server';
import { enableProdMode } from '@angular/core';

import * as express from 'express';
import { join } from 'path';
import { readFileSync } from 'fs';

enableProdMode();

// Express server
const app = express();

const PORT = process.env.PORT || 4000;
const DIST_FOLDER = join(process.cwd(), 'dist');

// 生成 index.html
const template = readFileSync(join(DIST_FOLDER, 'browser', 'index.html')).toString();

// *注意， 这里通过require引入，因为这是通过webpack动态生成的。
const { AppServerModuleNgFactory, LAZY_MODULE_MAP } = require('./dist/server/main.bundle');

const { provideModuleMap } = require('@nguniversal/module-map-ngfactory-loader');

app.engine('html', (_, options, callback) => {
  renderModuleFactory(AppServerModuleNgFactory, {
    document: template,
    url: options.req.url,
    extraProviders: [
      provideModuleMap(LAZY_MODULE_MAP)
    ]
  }).then(html => {
    callback(null, html);
  });
});

app.set('view engine', 'html');
app.set('views', join(DIST_FOLDER, 'browser'));

// /browser 目录下的静态文件
app.get('*.*', express.static(join(DIST_FOLDER, 'browser')));

// Universal engine 通用路由
app.get('*', (req, res) => {
  res.render(join(DIST_FOLDER, 'browser', 'index.html'), { req });
});

// 启动node服务器
app.listen(PORT, () => {
  console.log(`Node server listening on http://localhost:${PORT}`);
});
```

## Universal 模板引擎

这个文件中最重要的部分是 ngExpressEngine 函数：

app.engine('html', ngExpressEngine({
bootstrap: AppServerModuleNgFactory,
providers: [
provideModuleMap(LAZY_MODULE_MAP)
]
}));
ngExpressEngine 是对 Universal 的 renderModuleFactory 函数的封装。它会把客户端请求转换成服务端渲染的 HTML 页面。如果你使用不同于 Node 的服务端技术，你需要在该服务端的模板引擎中调用这个函数。

-   第一个参数是你以前写过的 AppServerModule。 它是 Universal 服务端渲染器和你的应用之间的桥梁。
-   第二个参数是 extraProviders。它是在这个服务器上运行时才需要的一些可选的 Angular 依赖注入提供商。当你的应用需要那些只有当运行在服务器实例中才需要的信息时，就要提供 extraProviders 参数。
    ngExpressEngine 函数返回了一个会解析成渲染好的页面的承诺（Promise）。

接下来你的引擎要决定拿这个页面做点什么。 现在这个引擎的回调函数中，把渲染好的页面返回给了 Web 服务器，然后服务器通过 HTTP 响应把它转发给了客户端。

## 创建服务端预渲染的程序：prerender.ts

```
// Load zone.js for the server.
import 'zone.js/dist/zone-node';
import 'reflect-metadata';
import { readFileSync, writeFileSync, existsSync, mkdirSync } from 'fs';
import { join } from 'path';

import { enableProdMode } from '@angular/core';
// Faster server renders w/ Prod mode (dev mode never needed)
enableProdMode();

// Import module map for lazy loading
import { provideModuleMap } from '@nguniversal/module-map-ngfactory-loader';
import { renderModuleFactory } from '@angular/platform-server';
import { ROUTES } from './static.paths';

// * NOTE :: leave this as require() since this file is built Dynamically from webpack
const {AppServerModuleNgFactory, LAZY_MODULE_MAP} = require('./dist/server/main.bundle');

const BROWSER_FOLDER = join(process.cwd(), 'browser');

// Load the index.html file containing referances to your application bundle.
const index = readFileSync(join('browser', 'index.html'), 'utf8');

let previousRender = Promise.resolve();

// Iterate each route path
ROUTES.forEach(route => {
    const fullPath = join(BROWSER_FOLDER, route);

    // Make sure the directory structure is there
    if (!existsSync(fullPath)) {
        mkdirSync(fullPath);
    }

    // Writes rendered HTML to index.html, replacing the file if it already exists.
    previousRender = previousRender.then(_ => renderModuleFactory(AppServerModuleNgFactory, {
        document: index,
        url: route,
        extraProviders: [
            provideModuleMap(LAZY_MODULE_MAP)
        ]
    })).then(html => writeFileSync(join(fullPath, 'index.html'), html));
});
```

## 服务端的 webpack 配置

Universal 应用不需要任何额外的 Webpack 配置，Angular CLI 会帮我们处理它们。但是由于本例子的 Node Express 的服务程序是 TypeScript 应用（server.ts 及 prerender.ts），所以要使用 Webpack 来转译它。这里不讨论 Webpack 的配置，需要了解的移步 [Webpack 官网](http://webpack.github.io/)

在后台应用根目录新建一个 webpack 配置以便它可以处理 server.ts（nodejs 应用的启动文件，以项目中的为准），例如 webpack.server.config.js。

```
const path = require('path');
const webpack = require('webpack');

module.exports = {
  entry: {  server: './server.ts' },
  resolve: { extensions: ['.js', '.ts'] },
  target: 'node',
  // this makes sure we include node_modules and other 3rd party libraries
  externals: [/(node_modules|main\..*\.js)/],
  output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name].js'
  },
  module: {
    rules: [
      { test: /\.ts$/, loader: 'ts-loader' }
    ]
  },
  plugins: [
    // Temporary Fix for issue: https://github.com/angular/angular/issues/11580
    // for "WARNING Critical dependency: the request of a dependency is an expression"
    new webpack.ContextReplacementPlugin(
      /(.+)?angular(\\|\/)core(.+)?/,
      path.join(__dirname, 'src'), // location of your src
      {} // a map of your routes
    ),
    new webpack.ContextReplacementPlugin(
      /(.+)?express(\\|\/)(.+)?/,
      path.join(__dirname, 'src'),
      {}
    )
  ]
}
```

## 后台启动应用

### `node dist/server.js`

我们可以在 package.json 中添加增加一些辅助命令，以帮助我们执行以上指令

```
"scripts": {
  // Common scripts
  "build:ssr": "npm run build:client-and-server-bundles && npm run webpack:server",
  "serve:ssr": "node dist/server.js",

  // Helpers for the scripts
  "build:client-and-server-bundles": "ng build --prod && ng run yourProjectName:server",
  "webpack:server": "webpack --config webpack.server.config.js --progress --colors"
}
...
```

### 本地系统中执行以下命令：

`npm run build:ssr && npm run serve:ssr`

浏览器接口处理
在服务端执行应用代码时，是无法引用到浏览器环境中的一些接口的，例如：document，navigator，location 对象等，可以通过逻辑判断来回避这些对象的使用，也可以通过 angular 提供的一些可注入服务，在它们身上找到可以替代浏览器原始对象上的方法，如果 angular 没有提供的话，你可能需要自己抽象出一些方法。

通过逻辑判断

```
import { isPlatformBrowser } from '@angular/common';
import { Inject, PLATFORM_ID, Component, OnInit } from '@angular/core';

@Component({...})
export class MyComponent implement {
    constructor(@Inject(PLATFORM_ID) private platformId: Object) { }

    ngOnInit() {
        if (isPlatformBrowser(platformId)) {
            ....
        }
    }
}
```

2. 使用 angular 提供依赖取代全局对象，例如使用 Location 依赖取代浏览器的全局对象 location：

```
import { Location } from '@angular/common';
import { Component } from '@angular/core';

@Componet({})
export class MyComponent {
    constructor(private location: Location) { }
}
```

3. 在服务端 mock 出浏览器环境中的一些全局对象，如 window, location，history，localStorage 等，挂载到 node 的 global 对象上。比较常用的库有 domino，mock-browser 等，使用方法也很简单。

### 执行 npm run prerender

编译应用程序并预渲染应用程序文件，启动一个演示 http 服务器，以便您可以查看它 http://localhost:8080
注意： 要将静态网站部署到静态托管平台，您必须部署 dist/browser 文件夹, 而不是 dist 文件夹

dist 目录：

![](https://segmentfault.com/img/remote/1460000014044667?w=441&h=493)

根据项目实际的路由信息并在根目录的 static.paths.ts 中配置，提供给 prerender.ts 解析使用。

```
export const ROUTES = [
    '/',
    '/lazy'
];
```

因此，从 dist 目录可以看到，服务端预渲染会根据配置好的路由在 browser 生成对应的静态 index.html。如 / 对应 /index.html，/lazy 对应 /lazy/index.html。

## 服务端的模块懒加载

在前面的介绍中，我们在 app.server.module.ts 中导入了 [ModuleMapLoaderModule](https://github.com/angular/universal/tree/master/modules/module-map-ngfactory-loader)，在 app.module.ts。

ModuleMapLoaderModule 模块可以使得懒加载的模块也可以在服务端进行渲染，而你要做也只是在 app.server.module.ts 中导入。

## 服务端到客户端的状态传输

在前面的介绍中，我们在 `app.server.module.ts` 中导入了 `ServerTransferStateModule`，在 `app.module.ts` 中导入了 `BrowserTransferStateModule` 和 [`TransferHttpCacheModule`](https://github.com/angular/universal/tree/master/modules/common)。

这三个模块都与服务端到客户端的状态传输有关：

-   ServerTransferStateModule：在服务端导入，用于实现将状态从服务端传输到客户端
-   BrowserTransferStateModule：在客户端导入，用于实现将状态从服务端传输到客户端
-   TransferHttpCacheModule：用于实现服务端到客户端的请求传输缓存，防止客户端重复请求服务端已完成的请求

使用这几个模块，可以解决 http 请求在服务端和客户端分别请求一次 的问题。

比如在 `home.component.ts` 中有如下代码：

```
import { Component, OnDestroy, OnInit } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs/Observable';

@Component({
    selector: 'app-home',
    templateUrl: './home.component.html',
    styleUrls: ['./home.component.scss']
})
export class HomeComponent implements OnInit, OnDestroy {
    constructor(public http: HttpClient) {
    }

    ngOnInit() {
        this.poiSearch(this.keyword, '北京市').subscribe((data: any) => {
            console.log(data);
        });
    }

    ngOnDestroy() {
    }

    poiSearch(text: string, city?: string): Observable<any> {
        return this.http.get(encodeURI(`http://restapi.amap.com/v3/place/text?keywords=${text}&city=${city}&offset=20&key=55f909211b9950837fba2c71d0488db9&extensions=all`));
    }
}

```

代码运行之后，

服务端请求并打印：

![](https://segmentfault.com/img/remote/1460000014044668?w=555&h=363)

客户端再一次请求并打印：

![](https://segmentfault.com/img/remote/1460000014044669?w=909&h=310)

### 方法 1：使用 TransferHttpCacheModule

使用 TransferHttpCacheModule 很简单，代码不需要改动。在 app.module.ts 中导入之后，Angular 自动会将服务端请求缓存到客户端，换句话说就是服务端请求到数据会自动传输到客户端，客户端接收到数据之后就不会再发送请求了。

### 方法 2：使用 BrowserTransferStateModule

该方法稍微复杂一些，需要改动一些代码。

调整 home.component.ts 代码如下：

```
import { Component, OnDestroy, OnInit } from '@angular/core';
import { makeStateKey, TransferState } from '@angular/platform-browser';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs/Observable';

const KFCLIST_KEY = makeStateKey('kfcList');

@Component({
    selector: 'app-home',
    templateUrl: './home.component.html',
    styleUrls: ['./home.component.scss']
})
export class HomeComponent implements OnInit, OnDestroy {
    constructor(public http: HttpClient,
                private state: TransferState) {
    }

    ngOnInit() {

        // 采用一个标记来区分服务端是否已经拿到了数据，如果没拿到数据就在客户端请求，如果已经拿到数据就不发请求
        const kfcList：any[] = this.state.get(KFCLIST_KEY, null as any);

        if (!this.kfcList) {
            this.poiSearch(this.keyword, '北京市').subscribe((data: any) => {
                console.log(data);
                this.state.set(KFCLIST_KEY, data as any); // 存储数据
            });
        }
    }

    ngOnDestroy() {
        if (typeof window === 'object') {
            this.state.set(KFCLIST_KEY, null as any); // 删除数据
        }
    }

    poiSearch(text: string, city?: string): Observable<any> {
        return this.http.get(encodeURI(`http://restapi.amap.com/v3/place/text?keywords=${text}&city=${city}&offset=20&key=55f909211b9950837fba2c71d0488db9&extensions=all`));
    }
}
```

-   使用 const KFCLIST_KEY = makeStateKey('kfcList') 创建储存传输数据的 StateKey
-   在 HomeComponent 的构造函数中注入 TransferState
-   在 ngOnInit 中根据 this.state.get(KFCLIST_KEY, null as any) 判断数据是否存在（不管是服务端还是客户端），存在就不再请求，不存在则请求数据并通过 this.state.set(KFCLIST_KEY, data as any) 存储传输数据
-   在 ngOnDestroy 中根据当前是否客户端来决定是否将存储的数据进行删除

## 总结

总体来说对于 angular 项目，要实现 seo 还是非常简单的，关键在于理解和把握以下几点：

打包后的服务端代码中包含了整个项目的所有代码和功能，因此在服务端运行时，除了环境导致的 API 使用差异外，其它过程都是相同的，因此一些依赖包服务端也必不可少。
服务端执行渲染的核心方法是：renderModuleFactory，它根据客户端请求上来的 url 和打包好的前端代码生成静态页面。
对于服务端面渲染的项目，编码过程中要尽量避免一些全局对象及环境依赖。

## Angular 服务器渲染常遇的坑

1. 使用浏览器 API 报错问题

在运行服务的时候，通常会遇到一下的一些报错

ReferenceError: window is not defined

或者

ReferenceError: document is not defined

由于 Universal 应用并没有运行在浏览器中，因此该服务器上可能会缺少浏览器的某些 API 和其它能力。比如，服务端应用不能引用浏览器独有的全局对象，比如 window、document、navigator 或 location。如果直接使用会导致运行的时候出现报错。因此，我们需要对使用浏览器的 API 方法做好兼容。

方案 1：在 server.ts，引入 domino 做兼容

```
const domino = require('domino');
const win = domino.createWindow(template);
global['window'] = win;
global['document'] = win.document;
global['CSS'] = null;
global['Prism'] = null;
global['DOMTokenList'] = win.DOMTokenList;
global['Node'] = win.Node;
global['Text'] = win.Text;
global['HTMLElement'] = win.HTMLElement;
global['object'] = win.object;
global['navigator'] = win.navigator;
global['localStorage'] = null;
global['sessionStorage'] = null;
```

但是，domino 并非兼容了所有浏览器的 api，只是兼容了大部分方法但是如果是用到的 api 不多，可以考虑用这个方案。如果是一些复杂项目建议还是用下面官方推荐的方法比较好。

方案 2：使用 Angular 官方推荐的方法

通过 PLATFORM_ID 令牌注入的对象来检查当前平台是浏览器还是服务器，从而解决该问题。判断是浏览器环境，才执行使用到浏览器方法的代码片段。不过个人觉得有些麻烦，因为在用到浏览器独有 API 方法的地方都得做引入判断兼容。

```
import { PLATFORM_ID } from '@angular/core';
import { isPlatformBrowser, isPlatformServer } from '@angular/common';

constructor(@Inject(PLATFORM_ID) private platformId: Object) { ... }

ngOnInit() {
  if (isPlatformBrowser(this.platformId)) {
    // 浏览器代码
    // eg:let url=window.location.href;
 ...
  }
  if (isPlatformServer(this.platformId)) {
    // 服务器代码
...
  }
}
```

2. 使用第三方库，例如 jq，echart，layer 等等报错

ReferenceError: $ is not defined

ReferenceError: layer is not defined

和上面一样，检查当前平台是浏览器还是服务器，执行相应的代码。

```
import { PLATFORM_ID } from '@angular/core';
import { isPlatformBrowser, isPlatformServer } from '@angular/common';

constructor(@Inject(PLATFORM_ID) private platformId: Object) { ... }

ngOnInit() {
  if (isPlatformBrowser(this.platformId)) {
    // 浏览器代码
    // eg:let userID =$.cookie('userID');
    // eg: layer.msg('测试');
 ...
  }
}
```

\*\*

懒加载路由无法加载问题**
由于我的项目原来是 angular6 版本的，后来升级到了 angular9 后路由写法有所改变，在浏览器模式下是没有问题的，安装官方例子改造\***之后懒加载路由无法正常使用，这里需要把路由改成 v9 版本推荐的写法，修改如下：
修改前

```
{
      path: 'doc',
      loadChildren: 'app/lazy-module/doc.module#DocModule'
}
```

修改后：

```
{
      path: 'doc',
      loadChildren: loadChildren: () => import('./lazy-module/doc.module').then(m => m.DocModule)
}
```

4. 打包 node 版本报错

报错信息如下：

由于我升级了 node 到 12 版本，这个报错是 node12 本身的 bug 官方暂时还没有修复这个问题，因此需要把 node 回滚到 10.15.3 版本，版本回滚之后就没有报错了。

5. 响应时间 TTFB 过长问题

Angular Universal 渲染过程中，如果代码中有一些延迟异步任务，可能会阻塞渲染。

主要包括例如 setTimeout、setInterval、全局调用 Observables 等方法的使用。在不取消它们的情况下调用它们，或者让它们运行超过服务器所需的时间可能会导致渲染效果欠佳，从而使得首次加载响应时间过长。

因此需要检查项目中是否用到以上方法，如有在用完之后需求做清除。

此外，如果需要执行的异步任务没有必要在服务器端使用的话，可以参考问题 2 的方式，使用 isPlatformBrowser 跳过服务器执行。

## 牢记几件事情

-   尽量**限制**或**避免**使用`setTimeout`。它会减慢服务器端的渲染过程。确保在组件的`ngOnDestroy`中删除它们

-   对于 RxJs 超时，请确保在成功时 _取消_ 它们的流，因为它们也会降低渲染速度。
-   不要直接操作 nativeElement，使用 Renderer2，从而可以跨平台改变应用视图。

```
constructor(element: ElementRef, renderer: Renderer2) {
 this.renderer.setStyle(element.nativeElement, 'font-size', 'x-large');
}
```

-   解决应用程序在服务器上运行 XHR 请求，并在客户端再次运行的问题

使用从服务器传输到客户端的缓存（TransferState）

-   清楚了解与 DOM 相关的属性和属性之间的差异
-   尽量让指令无状态。对于有状态指令，您可能需要提供一个属性，以反映相应属性的初始字符串值，例如 img 标签中的 url。对于我们的 native 元素，src 属性被反映为元素类型 HTMLImageElement 的 src 属性
