## SEO 不完全指南

### 什么是 SEO

搜索引擎优化（Search engine optimization，简称 SEO），指为了提升网页在搜索引擎自然搜索结果中（非商业性推广结果）的收录数量以及排序位置而做的优化行为，是为了从搜索引擎中获得更多的免费流量，以及更好的展现形象。

SEM（Search engine marketing，搜索引擎营销），则既包括了 SEO，也包括了付费的商业推广优化。

本文主要介绍的是前端如何在代码上做 SEO 以及单页项目如何实现 SEO。

### 搜索引擎工作原理

要了解 SEO，首先得了解搜索引擎的工作原理，其原理是比较复杂，流程简化如下：

#### 爬虫抓取网页内容

一般爬虫抓取页面内容是先从一个页面出发，从中提取出其他页面的链接，然后当作下一个请求的对象，一直重复这个过程。所以要有良好的 SEO，需要你在各大网站上拥有外链，这样会提高你的网站被搜索引擎爬虫的几率。

#### 分析网页内容

爬虫拿到 HTML 之后，就会对其内容进行分析。一般需要进行去杂、分词、建立索引数据库。什么是索引数据库呢？简单地说就是记录一个词在哪些文档中出现、出现次数、出现的位置等等。为什么要建立索引数据库呢？是为了快速查找。

#### 搜索和排序

搜索会根据你输入的关键词，分别查询其对应的索引数据库，并对结果进行处理和排序。

### 前端编码的 SEO

#### 网站结构

网站结构要清晰。一般网站的结构是树形的，一般分为三个层次：首页 → 频道页（列表页） → 文章页（详情页）。

网站的结构要扁平。结构层数越少越好，一般不要超过三层，搜索引擎一般到了第三层就不想继续深入地爬取了。多数的网站，例如掘金、雪球等，他们的网站结构是两层，他们的首页和频道页是同一个页面。

#### 导航

页面应该要有简明的导航。导航可以让搜索引擎知道网站的结构，也可以让搜索引擎知道当前页面在网站结构所在的层次。 建议：

每一个页面都包含导航。
对于内容较多的网站可以采用面包屑导航。
链接使用文字链接，如果是图片，则通过 alt 属性告知搜索引擎链接的指向。

#### 规范的 URL

规范、简单、易理解的 URL 能让搜索引擎更好地抓取内容。建议：

同一个页面，只对应一个 url 。多个 url 可以采用 301 进行重定向。
url 可以反应网页内容以及网站结构信息。例如www.a.com/blog、www.a.com/blog/123、www.a.com/article。
url 尽量简短。
尽量减少动态 url 中包含的变量参数。

#### 提交 Sitemap

Sitemap 可通知搜索引擎他们网站上有哪些可供抓取的网页，以便搜索引擎可以更加智能地抓取网站。

这是一个非常好用的手段，百度搜索引擎会把 Sitemap 中所有的 URL 都进行抓取，但是我是提交了很久之后，百度搜索引擎才进行抓取的。

#### robot.txt

搜索引擎爬行网站第一个访问的文件就是 robots.txt。在这个文件中声明该网站中不想被蜘蛛访问的部分，这样，该网站的部分或全部内容就可以不被搜索引擎访问和收录了，或者可以通过 robots.txt 指定使搜索引擎只收录指定的内容。

#### 合理的 HTTP 返回码

不同的返回码，搜索引擎的处理逻辑是不一样的。

如果站点临时关闭，当网页不能打开时，建议使用 503 状态。503 可以告知百度 spider 该页面临时不可访问，请过段时间再重试。
如果百度 spider 对您的站点抓取压力过大，请尽量不要使用 404，同样建议返回 503。这样百度 spider 会过段时间再来尝试抓取这个链接，如果那个时间站点空闲，那它就会被成功抓取了。
有一些网站希望百度只收录部分内容，例如审核后的内容，累积一段时间的新用户页等等。在这种情况，建议新发内容暂时返回 403，等审核或做好处理之后，再返回正常状态的返回码。
站点迁移，或域名更换时，请使用 301 返回。

#### 合适的 title

title 是告诉搜索引擎网页的主要内容。

每个网页应该有一个独一无二的标题，切忌所有的页面都使用默认标题
标题要主题明确和精练，包含这个网页中最重要的内容，且不罗列与网页内容不相关的信息
用户浏览通常是从左到右的，重要的内容应该放到 title 的靠前的位置
百度建议描述：

首页：网站名称 或者 网站名称*服务介绍/产品介绍
频道页：频道名称*网站名称
文章页：文章标题*频道名称*网站名称

#### 合适的 description

description 是对网页内容的精练概括。这个标签存在与否不影响网页权值，只会用做搜索结果摘要的一个选择目标。 百度推荐做法：

为每个网页创建不同的 description，避免所有网页都使用同样的描述
网站首页、频道页、产品参数页等没有摘要的网页最适合使用 description
准确的描述网页，不要堆砌关键词，长度合理

#### HTML 语义化

HTML 语义化是用标签和属性来描述内容。所以 HTML 语义化是 SEO 的基石。一般建议：

HTML 结构要清晰和简洁
跳转使用 `<a>` 标签，不要使用 js 跳转
图片加 alt 说明
文章用<article>标签承载

关于这部分的内容比较多，本人有一篇笔记《HTML 语义化》

### 提供您SEO初學者指南下載

[《Google搜尋引擎最佳化初學者指南》](http://static.googleusercontent.com/media/www.google.com/en/us/intl/zh-tw/webmasters/docs/search-engine-optimization-starter-guide-zh-tw.pdf)
[《Google 搜尋引擎最佳化 (SEO) 速記指南》](https://storage.googleapis.com/support-kms-prod/SNP_DE441AD549FEE9AE5B638F82579D99472297_3027140_zh-TW_v1)

### 如何檢測我的網站 SEO?

*   [關鍵字機會分析工具 Keyword Explorer](https://moz.com/explorer/lists/keywords)
*   [網站速度檢測 Google Pagespeed](https://developers.google.com/speed/pagespeed/insights/?hl=zh-TW)
*   [網站速度分析工具 WebPagetest](https://www.webpagetest.org/)
*   [網站應用工具審查 Lighthouse Web App](https://chrome.google.com/webstore/detail/lighthouse/blipmdconlkpinefehnmjammfjpmpbjk?hl=zh-tw)
*   [行動裝置相容性測試 Google](https://search.google.com/test/mobile-friendly?utm_source=mft&utm_medium=redirect&utm_campaign=mft-redirect)
*   [網站優化工具 Google Search Console](https://www.google.com/webmasters/tools/home?hl=zh-TW)
*   [網站分析統計工具(1) Google Analytics](https://analytics.google.com/analytics/web/)
*   [網站分析統計工具(2) Clicky](https://clicky.com/)
*   [反向連結抓取工具(1) Open Site Explorer](https://moz.com/researchtools/ose/)
*   [反向連結抓取工具(2) Majestic](https://zh.majestic.com/)
*   [品牌知名度檢測工具(需註冊登入) Fresh Web Explorer](https://moz.com/researchtools/fwe/)
*   [圖片優化工具 TinyPNG](https://tinypng.com/)

### Google 三種動物演算法你知道嗎?

*   [Google 熊貓演算法(網站品質)](https://www.newscan.com.tw/all-seo/google-panda.htm)：「熊貓演算法」更新的目的是獎勵高質量的網站，並減少 Google 有機搜尋引擎結果中低質量網站的存在，它最初也被稱為“農夫”。 據 Google 聲稱，「熊貓演算法」在首次推出幾個月內的英語搜尋結果高達 12％，2011 年至 2015 年，我們已經追蹤了「熊貓演算法」28 次更新紀錄。

*   [Google 企鵝演算法(網站品質)](https://www.newscan.com.tw/all-seo/google-penguin.htm)：緊隨「熊貓演算法」之後，Google 的「企鵝演算法」是一項新的努力，主要是在獎勵高質量的網站，並減少搜尋引擎結果頁面（SERP）涉及操縱連結和關鍵字填充存在的網站。

*   [Google 蜂鳥演算法](https://www.newscan.com.tw/all-seo/google-hummingbird.htm)：與之前發布的「熊貓演算法」和「企鵝演算法」更新不同，後者最初是作為 Google 現有算法的附件發佈的，目前「蜂鳥演算法」已徹底轉變成為核心算法引用， 雖然核心算法中許多先前組件還是保持著，但「蜂鳥演算法」表明 Google 開始深入了解搜尋者「查詢資料的意圖為何?」，並將其與相關結果進行匹配。

### SEO 延伸探討

雖然我們常常在網路上搜尋資料、尋找推薦餐廳、專業建議、線上購買商品，但是我們往往會忽略搜尋引擎的商業的運作模式，很多專業 SEO 人員會專注於短期的操作成效，往往會思考，如果我對網站做了這樣的改變，會發生什麼呢? 當然這些部分也是很重要的，但是我們應該更關注以下幾個關鍵性的問提，可以更全面的探討 SEO。

許多 SEO 專家的經驗是：「優化你的標題標籤，確保你的網站快速讀取，然後寫了很多部落格文章，並要求一些連結回網站，你將會看網站的搜尋排名越來越好」，這些專家將會告訴你，他們比你認為的做得更多更深入，不僅僅上述是那些，那些只是他們提供的服務內容的概略，如果你只有基礎的 SEO 知識，這些聽起來是非常合理的，沒有理由期待更多的內容，直到你網站的排名一直沒有提升，才會去詢問。

*   [搜尋引擎本身如何賺錢?](https://www.newscan.com.tw/all-seo/how-search-engines-make-money.htm)

*   [搜尋引擎除了賺錢以外的目的是什麼?](https://www.newscan.com.tw/all-seo/search-engine-another-purpose.htm)

*   [搜尋引擎的未來性?](https://www.newscan.com.tw/all-seo/future-search-engines-context.htm)

### 相關文章連結

[第一章：搜尋引擎的運作方式](https://www.newscan.com.tw/all-seo/how-search-engines-work.htm)

搜尋引擎有兩個主要的功能：抓取與建立索引，以及提供用戶最具關聯的搜尋結果列表。全球的資訊網路如同是大城市的地鐵系統。 每個車站都是一份文件（通常是網頁，但有時是 PDF，JPEG，或是其它檔案）。而搜尋引擎需要一種方法去“檢索”整個城市以及尋找各個站點的方法，最好的方法就是使用網頁設計上的”連結”方式（Link）。

[第二章：與搜尋引擎的互動](https://www.newscan.com.tw/all-seo/interaction-with-search-engines.htm)

SEO 搜尋引擎最佳化”的市場策略，最重要是讓你的客戶有認同感。一旦掌握目標市場的需求，就更容易行銷並且保留你的客戶。(在網頁設計時必須了解商品性質進行市場策略)

[第三章：為何 SEO 搜尋引擎行銷是必要的？](https://www.newscan.com.tw/all-seo/why-seo-is-necessary.htm)

SEO 搜尋引擎最佳化”最重要的一個任務是讓你的網站容易被用戶和網路蜘蛛了解，雖然搜尋引擎已經變得越來越精密，但仍然無法用人類的角度來看網頁，製作”SEO 搜尋引擎最佳化”用意是幫助搜尋引擎更容易解讀每個網頁，與分析網頁是否能夠帶給用戶所需要的內容。

[第四章：SEO 搜尋引擎最佳化的基礎開發與設計 (上)](https://www.newscan.com.tw/all-seo/the-basics-of-seo-first.htm)

因為目前搜尋引擎無法完全解讀網頁內容，所以網頁設計時需要以搜尋蜘蛛容易索引方式製作。對搜尋引擎來說，它們看到的網頁跟我們並不相同。在這章節中，我們將把重點集中在網站的技術層面上，這樣的網站架構，在搜尋引擎與用戶的眼中才會相似。可以把這個章節分享給程式設計師，前端工程師，和網頁設計師，讓所有參與網站製作的人都有相同知識。

[第四章：SEO 搜尋引擎最佳化的基礎開發與設計 (下)](https://www.newscan.com.tw/all-seo/the-basics-of-seo-second.htm)

不管是對用戶體驗還是搜尋引擎，網頁的 title 標籤一定要盡可能簡單明確的描述整個網頁的內容。基於這個因素，它也是網頁設計時，網站管理者操作 SEO 最重要的部分。只要注意下面幾個重點，你就能在得到不錯的效益。

[網站內 SEO - On-Site SEO](https://www.newscan.com.tw/all-seo/on-site-seo.htm)

網頁 SEO (也可以稱為網站內 SEO)是指單在網站上執行的優化網站的元素(連結到其它網際網路與其它外部信息，統稱為非網頁 SEO，)，從自然搜尋上，提高網站搜尋排序並賺取更多有意義的流量，藉由優化網頁內容與 HTML 程式碼的網頁來達成。

[網頁因素 - On-Page Factors](https://www.newscan.com.tw/all-seo/on-page.htm)

以一個網頁的內容來說，為什麼值得目前搜尋的排名結果，是應該依照搜尋者的觀點來看的，當然對於搜尋引擎來說這也是非常重要的，因此，創建優質內容是非常重要的， 什麼是優質內容呢？ 從 SEO 的角度來看，所有好的內容都需要有兩個要點，好的內容必須提供需求，並且是可以被連結的。

[標題標籤 - Title Tag](https://www.newscan.com.tw/all-seo/title-tag.htm)

標題標籤是指定網頁標題的 HTML 元素。 標題標籤顯示在搜尋引擎結果頁面（SERP）上，做為特定結果的可點擊標題，對可用性，搜尋引擎優化和社交分享非常重要。 網頁的標題標籤是對網頁內容的準確和簡潔的描述。

[網頁描述 - Meta Description](https://www.newscan.com.tw/all-seo/meta-description.htm)

meta 描述標籤雖然不受搜尋引擎排名的限制，但對於從 SERP 獲得用戶點擊是非常重要的。 這些簡短的描述是網站管理員向搜尋者“廣告”內容的機會，搜尋者有機會確認內容是否相關，是否有包含他們查詢中搜尋的訊息。

[alt 描述](https://www.newscan.com.tw/all-seo/alt-text.htm)

在 HTML 代碼中使用 Alt 替代文字，也稱為“alt 屬性”，“alt 描述”，以及俗稱但技術上不正確的“alt 標籤”來描述頁面上圖片 img 的外觀和功能。

[重複內容 - Duplicate Content](https://www.newscan.com.tw/all-seo/duplicate-content.htm)

重複的內容是指，相同的內容出現在網際網路上多個地方。 「一個地方」被定義為唯一網址（URL），因此如果相同的內容出現在多個網址上，則表示你的網站內容重複。

[SSL 憑證是什麼?&為什麼 SSL 憑證很重要呢?](https://www.newscan.com.tw/all-seo/what-ssl-certificate.htm)

SSL 憑證是在網頁伺服器(主機)與網頁瀏覽器(客戶端)之間建立一個密碼連結的標準技術，使二者之間可以有可靠聯繫，讓資訊傳送時可以保留私人資訊與內部資訊，SSL 憑證是一種企業標準規格，被數百萬個網站使用來保護他們的線上交易與顧客。

[你的網站，是不是用 SEO 方式製作?](https://www.newscan.com.tw/all-seo/do-websites-use-SEO.htm)

SEO 網站架構需為 HTML5 + CSS 製作，並且適當的使用＜ H1 ＞＜ H2>＜ H3 ＞標籤，才是好的建置方式，也是一切 SEO 的基礎，劣質的架構會造成，0 分乘與 100 分還是 0 分的窘境。

[Robots Meta 指令 - Robots Meta Directives](https://www.newscan.com.tw/all-seo/robots-meta-directives.htm)

Robots Meta 指令（有時也稱為“Meta 標籤”）是一些程式語法，它們提供網頁爬蟲如何抓取或索引網頁內容的抓取指令，雖然 robots.txt 文件指令也可以幫網路爬蟲提供了如何抓取網站的建議，但 Robots Meta 指令提供了更為嚴格的，指導網頁爬蟲如何抓取和索引頁面的內容。

[Schema.org 結構化資料 - Schema.org Structured Data](https://www.newscan.com.tw/all-seo/schema-structured-data.htm)

Schema.org（通常稱為 Schema 或是 Schema.org 結構化資料）是一種特定的標籤（或 Microdata）詞彙表，你可以將其添加到 HTML 中以改進頁面在 SERP 中的表示方式。

[HTTP 狀態碼 - HTTP Status Codes](https://www.newscan.com.tw/all-seo/http-status-codes.htm)

HTTP 狀態碼是服務器對瀏覽器請求的回應，當你訪問一個網站時，你的瀏覽器發送一個請求到站點的服務器，然後服務器用一個三位數的代碼來回應瀏覽器的請求，這個就是 HTTP 狀態碼。

[網頁速度 - Page Speed](https://www.newscan.com.tw/all-seo/page-speed.htm)

「網頁速度」通常與「網站速度」相混淆，網站速度實際上是網站頁面瀏覽頁面速度，網頁速度可以用「網頁讀取時間」（在特定網頁上完全顯示內容所需的時間）或「第一個文字的時間來描述（從打開網頁時瀏覽器從網路伺服器讀取第一個文字訊息需要多長時間）。

[轉化率優化 - Conversion Rate Optimization](https://www.newscan.com.tw/all-seo/conversion-rate-optimization.htm)

轉化率優化（CRO）是增加採取預期行動的網站訪問者的百分比的系統過程，即填寫表格，成為客戶或以其他方式，CRO 流程涉及了解使用者如何在你的網站上移動，他們採取了什麼行動，以及阻止他們完成目標的原因。

[網址、網域、網名 - Domains](https://www.newscan.com.tw/all-seo/domains.htm)

網址是網站唯一的、人類可讀的網際網路地址，它們由三部分組成：頂級網址（有時稱為擴展網址或網址後綴）、一個網址（或 IP 地址）和一個可選的子網址。

[網址連結 - URLs](https://www.newscan.com.tw/all-seo/url.htm)

URL（全球資源定址器）（更一般地稱為“網址”）指定網際網路上資源（例如網頁）的位置。 該 URL 還指定如何檢索該資源（也稱為“協議”，如 HTTP，HTTPS，FTP 等）。

[規範化 - Canonicalization (“rel canonical”)](https://www.newscan.com.tw/all-seo/canonicalization.htm)

canonical 標記（又名“rel canonical”）是一種告訴搜尋引擎特定的 URL 代表主網頁與副本的區分方法，使用規範標記可防止由於多個 URL 中出現相同或“重複”內容而導致的問題，實際上 canonical 標記告訴搜尋引擎，你想在搜尋結果中出現哪個版本的 URL。

[轉址 - Redirection](https://www.newscan.com.tw/all-seo/redirection.htm)

轉址是將一個 URL 跳轉到另一個 URL 的過程。主要有三種轉址：301，302 和 meta refresh。

### 总结

以上就是我想讲的关于前端编码 SEO 的全部内容，总而言之，就是

合适的 HTML 标签和属性
合理的 HTTP 状态码
Sitemap & robot.txt
合适的渲染方案
