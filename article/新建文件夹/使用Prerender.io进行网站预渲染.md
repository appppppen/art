## 前言

使用 Angular，Vue，React 进行单页网站开发，用户浏览时浏览器动态解析 JS，呈现出最终的页面，用户体验比较好，网站性能也提高不少。
但网络爬虫并不会动态解析 Js，访问所有 URL 得到的只会是项目入口文件中的代码，不能得到具体的内容，也就无法做网站 SEO。
使用 Prerender.io 做网站预渲染，可以将网站页面渲染之后再返回给网络爬虫，间接完成网页的解析。
Prerender 相较于其他的解决方案，配置相对要简单一些，不用修改项目源码，代码零侵入，是一个不错的解决方案。

## 目标

搭建基于 Centos 7 和 Nginx 环境的 Prerender 渲染服务，完成 Angular 项目中网页的预渲染
预渲染流程图

![](https://user-gold-cdn.xitu.io/2019/6/17/16b635d6f09ea8f0?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

运行流程
安装中间件

1. 首先注册登录 Prerender.io，并且获得个人 token

![](https://user-gold-cdn.xitu.io/2019/6/17/16b635d6f3cc4b89?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

token
根据开发文档，配置对应的中间件，如 Nginx，Apache 等。
配置 Nginx 中间件，参考配置如下：

``` 
server {
    listen 80;
    server_name example.com;

    root   /path/to/your/root;
    index  index.html;

    location / {
        try_files $uri @prerender;
    }

    location @prerender {
        # 将 YOUR_TOKEN替换为你的个人token
        proxy_set_header X-Prerender-Token YOUR_TOKEN;

        set $prerender 0;
        if ($http_user_agent ~* "googlebot|bingbot|yandex|baiduspider|twitterbot|facebookexternalhit|rogerbot|linkedinbot|embedly|quora link preview|showyoubot|outbrain|pinterest|slackbot|vkShare|W3C_Validator") {
            set $prerender 1;
        }
        if ($args ~ "_escaped_fragment_") {
            set $prerender 1;
        }
        if ($http_user_agent ~ "Prerender") {
            set $prerender 0;
        }
        if ($uri ~* "\.(js|css|xml|less|png|jpg|jpeg|gif|pdf|doc|txt|ico|rss|zip|mp3|rar|exe|wmv|doc|avi|ppt|mpg|mpeg|tif|wav|mov|psd|ai|xls|mp4|m4a|swf|dat|dmg|iso|flv|m4v|torrent|ttf|woff|svg|eot)") {
            set $prerender 0;
        }

        #resolve using Google's DNS server to force DNS resolution and prevent caching of IPs
        resolver 8.8.8.8;

        if ($prerender = 1) {

            # 后续将service.prerender.io替换为自己的prerender服务，如127.0.0.1:3000
            set $prerender "service.prerender.io";
            rewrite .* /$scheme://$host$request_uri? break;
            proxy_pass http://$prerender;
        }
        if ($prerender = 0) {
            rewrite .* /index.html break;
        }
    }
}
```

复制代码参考配置：[gist.github.com/thoop/81658…](https://gist.github.com/thoop/8165802)

检测 nginx 配置，并重启 nginx

nginx -t
service nginx restart
复制代码
中间件安装完成

## 安装 Prerender 服务

1. 在服务器上安装 Node 环境

2. 下载 Prerender 服务

git clone https://github.com/prerender/prerender.git
复制代码若没有安装 git 服务，可手动从 Github 下载再上传到/usr 文件夹下，再解压到当前目录下 3. 安装 npm 依赖
cd /usr/prerender

# Phantomjs 官方的下载地址会超时，此处重新指定其下载地址为淘宝镜像

export PHANTOMJS_CDNURL=https://npm.taobao.org/mirrors/phantomjs
npm install
复制代码文件结构如下：
文件结构

4. 运行 server.js

# 启动 Server.js， 默认监听 3000 端口

node server.js
复制代码此时，如果预先没有安装过 Chrome，则会启动失败
提示启动 Chrome 失败，未检测到 Chrome，此时安装 Chrome 就好了
为什么要安装 Chrome 呢，因为 Prerender 并不负责真正的网页解析，Prerender 只负责解析前后的处理，实际是由 Chrome 负责网页的解析。

## 安装 Chrome

1. 配置 yum 源

   因为国内无法访问 Google，所以需要自己配置 yum 源，在目录 /etc/yum.repos.d/ 下新建 google-chrome.repo 文件

``` 
cd /ect/yum.repos.d/
touch google-chrome.repo
```

2. 写入内容

   vi google-chrome.repo

``` 
[google-chrome]
name=google-chrome
baseurl=http://dl.google.com/linux/chrome/rpm/stable/$basearch
enabled=1
gpgcheck=1
gpgkey=https://dl-ssl.google.com/linux/linux_signing_key.pub
```

3. 安装运行

``` 
# 国内推荐
yum -y install google-chrome-stable --nogpgcheck
复制代码
安装路径
安装成功后，Chrome的安装路径应该是
/opt/google/chrome
默认情况下，root用户不能直接运行chrome，所以可以新建另一个用户如other来运行

cd /opt/google/chrome
su other
./chrome
```

Chrome 安装完成

## 启动 Prerender.io 服务

已 other 用户再次运行 server.js

``` 
su other
cd /usr/prerender
node ./server.js
```

此时应该是可以成功启动的，并且可以看到该服务监听 3000 端口
启动结果：
启动结果 2. 修改 nginx 配置

``` 
if ($prerender = 1) {

           # 修改如下：
            # set $prerender "service.prerender.io";
            set $prerender "127.0.0.1:3000";
            rewrite .* /$scheme://$host$request_uri? break;
            proxy_pass http://$prerender;
        }
```

3. 保存重启 Nginx
4. 再次启动 Prerender 服务

``` 
nohup node ./server.js &
```

其中 nohup 命令是将该服务加入守护进程，避免 ssh 对话窗口关闭导致服务关闭，参考 Linux 设置 Jar 后台运行

5. 如果开启了防火墙，需要将 3000 端口加入防火墙

``` 
firewall-cmd —zone=public —add-port=3000/tcp —permanent
# 重启防火墙
firewall-cmd —reload
```

至此，Prerender 服务已经安装并启动成功

查看端口

![](https://user-gold-cdn.xitu.io/2019/6/17/16b635d6f2109fc7?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

Node，Google-Chrome，Nginx 服务都应在后台运行

测试
If you use html5 push state (recommended):

``` 
Just add this meta tag to the <head> of your pages

<meta name="fragment" content="!">
```

``` 
If your URLs look like this:
http://www.example.com/user/1

Then access your URLs like this:
http://www.example.com/user/1?_escaped_fragment_=
```

If you use the hashbang (#!):

``` 
If your URLs look like this:
http://www.example.com/#!/user/1

Then access your URLs like this:
http://www.example.com/?_escaped_fragment_=/user/1
```

通过 curl 命令测试

``` 
curl http://www.example.com/user/1?_escaped_fragment_=
```

在配置 prerender 服务前，以上返回的只是 index.html 的内容, 如果配置成功则会返回解析后的内容。
