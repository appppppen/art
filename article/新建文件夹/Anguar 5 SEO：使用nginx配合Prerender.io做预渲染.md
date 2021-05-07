服务器需预装 nodejs 环境

使用步骤：

1、启动 prereder server

```
git clone https://gitee.com/qiqj/prerender.git

cd prerender

npm install

# 生产环境建议使用forever指令启动，具体见度娘
node server.js # 启动服务
```

2、nignx.conf 配置示例

```

#user  nobody;
worker_processes  1;#工作进程数量，一般和cpu核心数相同

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  102400; #单个工作进程最大连接数

    #use epoll;  #epoll事件模型，Windows不支持所以先注释掉
}


http {
    include       mime.types;
    default_type  application/octet-stream;
    client_max_body_size    2000m;
    charset utf-8;
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                     '$status $body_bytes_sent "$http_referer" '
                     '"$http_user_agent" "$http_x_forwarded_for"';

    # access_log  logs/access.log;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  60;

    gzip on;
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_http_version 1.1;
    gzip_disable "MSIE [1-6].";
    gzip_comp_level 4;
    gzip_types text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png;
    gzip_vary off;


    # prerender 负载均衡
    server {
        # 监听80端口
        listen       80;
         #域名
        server_name  xxx.com;

        # 设置 x-forwarded-host Header信息
        set $nbhost "xxx.com";

        # 支持非nginx标准请求头
        underscores_in_headers on;

        charset utf-8;
        access_log  logs/host1.access.log;
        error_log logs/host1.error.log;
        error_page  404              /404.html;
        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;

        # 拦截所有请求，交由 @prerender 处理
        location / {
            try_files $uri @prerender;
        }
        location @prerender {
            # 若使用Prerender 云服务时启用；token在Prerender.io注册账号后获取
            # proxy_set_header X-Prerender-Token xxxxxxxxx;

            proxy_set_header x-forwarded-host $nbhost;
     	    proxy_set_header X-Real-IP $remote_addr;
     	    proxy_set_header REMOTE-HOST $remote_addr;
     	    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		    proxy_redirect  off;
		    client_max_body_size     150m;
            client_body_buffer_size  128k;
            #proxy_connect_timeout    600;
            proxy_read_timeout       600;
            proxy_send_timeout       6000;
            proxy_buffer_size        32k;
            proxy_buffers            4 64k;
            proxy_busy_buffers_size 128k;
            proxy_temp_file_write_size 512k;
		    proxy_connect_timeout 10s;
		    proxy_next_upstream http_502 http_504 error invalid_header;

            # 自定义请求头，需添加 "$http_"
            proxy_set_header project $http_project;

            set $prerender 0;
            if ($http_user_agent ~* "baiduspider|twitterbot|facebookexternalhit|rogerbot|linkedinbot|embedly|quora link preview|showyoubot|outbrain|pinterest|slackbot|vkShare|W3C_Validator") {
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
                #setting prerender as a variable forces DNS resolution since nginx caches IPs and doesnt play well with load balancing
                set $prerender "127.0.0.1:3000";
                rewrite .* /$scheme://$nbhost$request_uri? break;
                proxy_pass http://$prerender;
            }
            if ($prerender = 0) {
                proxy_pass http://127.0.0.1;
            }
        }

    }
    # web Server
    server {
        listen 80;
        server_name  127.0.0.1;
        access_log  logs/host2.access.log;
        error_log logs/host2.error.log;
        # 设置 x-forwarded-host Header信息
        set $nbhost "xxx.com";

        location / {
            # ng5-base 为我所写demo的项目名
            root  /Users/qiqj/WebstormProjects/ng5-base/dist;
            index  index.html;
            try_files $uri $uri/ /index.html =404;
        }
        location /api/ {
            proxy_pass   http://myserver/;
            proxy_set_header x-forwarded-host $nbhost;
		    proxy_set_header X-Real-IP $remote_addr;
		    proxy_set_header REMOTE-HOST $remote_addr;
		    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
		    proxy_redirect  off;
		    client_max_body_size    2000m;
        }
        location ~ ^\.(js)$ {
            root /Users/qiqj/WebstormProjects/ng5-base/dist;
        }
        # 静态文件
        location /assets {
            alias /Users/qiqj/WebstormProjects/ng5-base/dist/assets/;
        }
    }
    # 上游服务器（后端接口）
    upstream myserver {
        server 127.0.0.1:8080;
        keepalive 2000;
    }

}
```
