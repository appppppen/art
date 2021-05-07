我已经写了一篇文章，但无法在文章中添加任何 AdSense 脚本。我当时在考虑如何将 Google AdSense 代码段添加到博客的 HTML 中。因为我博客的内容是简单的 HTML，它也是由我创建的定制 CMS（内容管理系统）生成的。但是在创建内容时，我始终专注于内容文本或图像，以此可以证明本文的目的。但是，当我发布它时，我确实也需要将其货币化。让我们看看创建内容后的样子。

![](https://lazypanda-blog-images.s3.ap-south-1.amazonaws.com/3c9dc49056b0a9989b15b09ced13ed782509b9f0.png)

在上面可以看到，内容是用简单的 HTML 编写的，实际上是使用我创建的编辑器生成的。编辑器有一些限制，创建内容时我不能包含任何脚本标签。结果，我在创建内容时不能包含 Google AdSense 代码。我需要在运行时在两段之间添加代码段。因此，使用 Google 广告可以正确地看到内容。

## 我想在这里实现...

我想在两段之间添加 Google AdSense 代码段。

![](https://lazypanda-blog-images.s3.ap-south-1.amazonaws.com/211932d86be0a3efddfc7adb3b0e08f485fbbe40.png)

为了实现我的目标，我添加了一个带有 div 标签的简单 ID。在将代码添加到主页模板中时，代码将搜索各自的 ID，并将所需的脚本标签放在两个 div 之间。
![](https://lazypanda-blog-images.s3.ap-south-1.amazonaws.com/dcbd780a35a20801c5f7ac61301c7bbdecf52885.png)
将相应的 ID 添加到 HTML 中后，我需要搜索该 ID 并放置 AdSense JavaScript 代码。

```typescript
injectAdSense = (htmlString: string, idCount: number) => {
  const domParser = new DOMParser();
  const htmlElement = domParser.parseFromString(htmlString, "text/html");
  for (let index = 1; index <= idCount; index++) {
    const adSnippets = htmlElement.getElementById(`lazy-loaded-ad-${index}`);
    if (adSnippets) {
      // Keep this line, if the url not included in header section
      // const adsenseNodeCreation = window.document.createElement('script');
      // adsenseNodeCreation.async = true;
      // adsenseNodeCreation.src = 'https://pagead2.googlesyndication.com/pagead/js/adsbygoogle.js';
      // end

      const insHtml = window.document.createElement("ins");
      insHtml.className = "adsbygoogle";
      insHtml.style.display = "block";
      insHtml.style.textAlign = "center";
      insHtml.setAttribute("data-ad-format", "fluid");
      insHtml.setAttribute("data-ad-layout", "in-article");
      insHtml.setAttribute("data-ad-client", "ca-pub-27113xxxxxxxxx");
      insHtml.setAttribute("data-ad-slot", "347xxxxxxx");
      // adSnippets.appendChild(adsenseNodeCreation);
      adSnippets.appendChild(insHtml);
    }
  }
  return htmlElement.body;
};
```

为了调用该方法，我使用了以下方式-

`const blogDetails = this.injectAdSense(description, 2);` 其中第一个参数描述是 HTML 代码，数据类型是 String。第二个参数是要添加到 HTML 中的 AdSense ID 的数量。

```typescript
document.getElementById("my-blog").appendChild(blogDetails);
```

完成更改后，它可能如下所示-

![](https://lazypanda-blog-images.s3.ap-south-1.amazonaws.com/af4f60ef7d8a099643dd287c36b8cb119871616b.png)

现在，本文最重要的部分是，您需要获得**In-view**和**out-view** 点，您可以在其中以编程方式注入脚本标签。每当 div 基于用户**滚动操作**显示在视口中时，我们都会在视口上**延迟**加载**AdSense 脚本**。

![](https://lazypanda-blog-images.s3.ap-south-1.amazonaws.com/5ba9e0d81a5515295e7c800c13759943e1659e42.png)

为了获得用户滚动动作，我们需要观看**window.scroll 事件**。使用角度我正在使用 **@HostListener** 来获取滚动位置。像下面

```typescript
@HostListener('window:scroll') onScroll(e: Event): void {
    for (let i = 1; i <= this.totalNumberOfAds; i++) {
        this.isItInView(`lazy-loaded-ad-${i}`);
    }
}
```

当用户滚动页面时，上述方法将执行，我们将验证对应的 div 的 ID 是在视图内还是在视图外。

```typescript
isItInView(id: string) {
    const adsElement: HTMLElement = document.getElementById(id);
    if (adsElement) {
        const bounding = adsElement.getBoundingClientRect();
        if (
        bounding.top >= 0 && bounding.left >= 0 &&
        bounding.right <= (window.innerWidth || document.documentElement.clientWidth) &&
        bounding.bottom <= (window.innerHeight || document.documentElement.clientHeight)
        ) {
            if (!this.isAdsLoaded.includes(id)) {
                this.isAdsLoaded.push(id);
                this.loadAdsScriptDynamically(id);
            } else {
                const adscriptElement: HTMLElement = document.getElementById('ads');
                if (adscriptElement) {
                    adscriptElement.remove();
                }
            }
        }
    }
}
```

上面的函数将准确告诉您特定的目标 div 是视图内还是视图外，如果 ID 在视图外部，我们将删除该 ID。如果它在视图内部，我们将执行脚本值。让我们来调用脚本。

我创建了一个脚本加载器服务，该服务有助于加载外部脚本文件。从角度来看，我正在发送目标 div ID，并根据该 ID 执行 AdSense 代码段。

```typescript
import { Injectable } from "@angular/core";
import { ScriptStore } from "./script-loader";

declare var document: any;

@Injectable({
  providedIn: "root",
})
export class ScriptService {
  private scripts: any = {};

  constructor() {
    ScriptStore.forEach((script: any) => {
      this.scripts[script.name] = {
        loaded: false,
        src: script.src,
      };
    });
  }

  load(idValue: string, ...scripts: string[]) {
    const promises: any[] = [];
    scripts.forEach((script) =>
      promises.push(this.loadScript(script, idValue))
    );
    return Promise.all(promises);
  }

  loadScript(name: string, idValue: string) {
    return new Promise((resolve, reject) => {
      const script = document.createElement("script");
      script.type = "text/javascript";
      script.id = "ads";
      script.src = this.scripts[name].src;
      script.setAttribute("data-name", idValue);

      if (script.readyState) {
        // IE
        script.onreadystatechange = () => {
          if (
            script.readyState === "loaded" ||
            script.readyState === "complete"
          ) {
            script.onreadystatechange = null;
            this.scripts[name].loaded = true;
            resolve({ script: name, loaded: true, status: "Loaded" });
          }
        };
      } else {
        // Others
        script.onload = () => {
          this.scripts[name].loaded = true;
          resolve({ script: name, loaded: true, status: "Loaded" });
        };
      }
      script.onerror = (error: any) =>
        resolve({ script: name, loaded: false, status: "Loaded" });
      document.getElementsByTagName("head")[0].appendChild(script);
    });
  }
}
```

同样是脚本加载器，您需要传递路径外部脚本（js）文件。像下面-

```typescript
import { environment } from "src/environments/environment";

interface Scripts {
  name: string;
  src: string;
}

export const ScriptStore: Scripts[] = [
  {
    name: "google-ads",
    src: `${environment.webAppURL}assets/ad-script/ga-lazy.js`,
  },
];
```

设置几乎完成了，只有一部分保留了 ga-lazy.js 文件中的内容以及我如何调用该函数。

**ga-lazy.js**

```typescript
$(document).ready(function () {
  var scriptTag = document.getElementById("ads");
  var key = scriptTag.getAttribute("data-name");
  adsbygoogle = window.adsbygoogle || [];
  $(
    "<script>(adsbygoogle = window.adsbygoogle || []).push({})</script>"
  ).appendTo(key);
  eval((adsbygoogle = window.adsbygoogle || []).push({}));
});
```

我如何执行脚本？

```typescript
loadAdsScriptDynamically(id: string) {
    this.scriptLoader.load(`#${id}`, 'google-ads').then(data => {
        console.log('data: ', data);
    }).catch(err => {
        console.log(err);
    });
}
```

您已经准备好进行设置，现在就可以部署该网站，并且可以看到广告在用户滚动时显示在文章的中间。如果您发现此帖子对您有用，请在下面分享和评论。

祝您编码愉快！
