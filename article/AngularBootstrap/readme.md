# angular6 学习(四)：引入 bootstrap 样式框架

两种方法：

第一种：引用链接

1、 在 index.html 中添加 bootstrap 引用链接

bootstrap 链接地址可在官网复制：官网

引用的链接参考如下（新版本无效果，建议用下面这个测试学习）

```xml
<link rel="stylesheet" href="http://cdn.bootcss.com/bootstrap/3.3.0/css/bootstrap.min.css">
```

![](https://img-blog.csdn.net/20180619175934158?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEzMjE1NDY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

2、打开 app.component.html 添加 css 样式 [点此复制按钮样式](http://v3.bootcss.com/components/#btn-dropdowns-split) \*在 AppModule 里面的 imports 里面导入 RouterModule。

![](https://img-blog.csdn.net/20180619173452227?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEzMjE1NDY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

显示如下：

![](https://img-blog.csdn.net/20180619180025622?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEzMjE1NDY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
第二种方法：安装 bootstrap 框架

1、在 webstorm 中用快捷方式：Alt+F12 调出控制台

输入： `npm i bootstrap -s` 安装 bootstrap 框架

![](https://img-blog.csdn.net/20180619174403789?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEzMjE1NDY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

安装成功后找到安装目录：
![](https://img-blog.csdn.net/20180619174717901?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEzMjE1NDY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

将目录下的 ../node_modules/bootstrap/dist/css/bootstrap.min.css 添加到项目的 angular.json 文件的 styles 中

![](https://img-blog.csdn.net/20180619185045399?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3UwMTEzMjE1NDY=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

2.同第一种的步骤 2

第二种方法在添加到项目的 angular.json 文件的 styles 中注意事项：

1. 安装位置是在项目中安装，所以引用时加一个 . 即可

2. 要重新启动服务才能生效
