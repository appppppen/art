## 知识前导
> Iconfont 与 font 一样都可以通过 CSS 来进行单色改变，多色或者颜色渐变时建议考虑使用 SVG

## 制作 SVG
以现有图片制作（前提是 实时上色 步骤之后仍能大致保持原来的轮廓）
- 使用 PS 将 Icon 原型图转换为黑白两色的位图（自行 Google），注意选取一个锯齿较小的阈值

- Adobe Illustrator 打开位图并选中

- 对象——图像临摹

- 对象——实时上色

- 对象——拼合透明度

- 对象——取消编组

- 导出 SVG 文件

## 自行动手来制作
- 在 Adobe Illustrator 进行绘制 SVG，只制作能通过 CSS 改变颜色的单一颜色部分，这样效果最佳

- 多种颜色则需要对同一种颜色进行 路径选择器——合并 操作，虽然得到的 SVG 正常显示，但是得到的 Icon 是无法使用 CSS 改变形成多色的，多色时建议直接下载对应的 JPG 或者 PNG 来使用，因为有些低端的浏览器对渲染 SVG 的能力还是比较弱的

- 白纸上手绘再拍照或者扫描上传到 PC 端，再用 PS 或者 Adobe Illustrator 打开照片，重新绘制并导出 SVG 即可

## 效果不佳的措施
- 对文字：Adobe Illustrator 选中文字，进行 文字——创建轮廓 操作，将文字转换为图形

- 对画布：遗憾的是Iconfont的画布是强制正方形的，因此上传的 SVG 要设计成几乎占满正方形画布的形状，Icomoon没有这样的限制，因此当需要显示成长方形时，优选Icomoon，可以根据如何操纵 CSS 的属性来选择是正方形还是长方形，画布主要影响 CSS 的位置属性与对齐方式

- 对其他：选中对象，利用 Adobe Illustrator 的路径选择器将同色部分进行合并操作，方能让 Icon 轮廓分明

## Iconfont 不满足预期的措施（Adobe Illustrator）
- 对象——取消编组

- 对象——扩展

- 对象——扩展外观

- 对象——路径——轮廓化描边

## 项目操作
### 需要借助的网站
- ①[Icomoon](https://links.jianshu.com/go?to=https%3A%2F%2Ficomoon.io%2F)
> 创建一个 Iconfont 项目，将制作好的 SVG 上传到项目当中，最好以合适的英文来命名图标，下载 Iconfont 项目即可，如果在上传之后发现图标显示不正确，此网站一般会提出相应的解决方案，根据提供的方案来修改 SVG 直至显示正确，再进行下载 Iconfont 项目

- ②[Iconfont](https://links.jianshu.com/go?to=http%3A%2F%2Fwww.iconfont.cn%2F)
> 创建一个 Iconfont 项目，将制作好的 SVG 上传到项目当中，最好以合适的英文来命名图标，下载 Iconfont 项目即可，遗憾的是 Iconfont 并不像 Icomoon 一样会给出相应的修改建议，因此 SVG 的验证优选 Icomoon

## 项目最后的验证
> 下载项目后进行解压，通过浏览器来打开其中的 HTML 文件，检查是否显示完全，是否轮廓清晰，是否 Hover 效果正确，满足则再使用 VS Code 等 IDE 来打开 HTML 文件，HTML 文件中会告诉你怎么引用 Iconfont 才能在 HTML 中显示相应的 Iconfont

