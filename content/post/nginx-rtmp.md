---
title: nginx-rtmp
cover: https://pic2.zhimg.com/80/v2-d68668239cfffeaec79fcd1a8262a581_720w.jpg
---

编译一个最小的nginx
并且添加rtmp模块用来直播 
```shell
./configure \
--prefix=/etc/nginx \
--sbin-path=/usr/sbin/nginx \
--conf-path=/etc/nginx/nginx.conf \
--error-log-path=/var/log/nginx/error.log \
--http-log-path=/var/log/nginx/access.log \
--pid-path=/var/run/nginx.pid \
--lock-path=/var/run/nginx.lock \
--http-client-body-temp-path=/var/cache/nginx/client_temp \
--http-proxy-temp-path=/var/cache/nginx/proxy_temp \
--http-fastcgi-temp-path=/var/cache/nginx/fastcgi_temp \
--http-uwsgi-temp-path=/var/cache/nginx/uwsgi_temp \
--http-scgi-temp-path=/var/cache/nginx/scgi_temp \
--with-file-aio \
--with-threads \
--with-http_addition_module \
--with-http_auth_request_module \
--with-http_dav_module \
--with-http_flv_module \
--with-http_gunzip_module \
--with-http_gzip_static_module \
--with-http_mp4_module \
--with-http_random_index_module \
--with-http_realip_module \
--with-http_secure_link_module \
--with-http_slice_module \
--with-http_ssl_module \
--with-http_stub_status_module \
--with-http_sub_module \
--with-http_v2_module \
--with-mail \
--with-mail_ssl_module \
--with-stream \
--with-stream_realip_module \
--with-stream_ssl_module \
--with-stream_ssl_preread_module \
--add-module=./nginx-rtmp-module
# 这行很关键
```


```dockerfile

FROM 438577872/ubuntu

COPY ./nginx /nginx

WORKDIR /nginx

COPY nginx/nginx.conf /nginx.conf

RUN apt update && apt -y install gcc make libpcre3-dev  git libssl-dev  zlib1g-dev \
     && mv /nginx/ffmpeg /usr/sbin \
     && git clone https://gitee.com/jeremycurry/nginx-rtmp-module \
     && ./build.sh  \
     && make  \
     && make install\
     && mkdir /var/cache/nginx  \
     && mkdir /etc/nginx/logs \
     && apt -y remove gcc make libpcre3-dev  git libssl-dev  zlib1g-dev \
     && cp -f /nginx.conf /etc/nginx/nginx.conf \
     && rm -rf /nginx \
     && apt -y remove gcc make libpcre3-dev  git libssl-dev  zlib1g-dev  \
     && apt clean \
     && rm  -rf /var/lib/apt/lists/*


VOLUME ["/etc/nginx/conf/rtmp","/etc/nginx/conf/proxy","/mnt"]
EXPOSE 80
EXPOSE 1935
EXPOSE 8080

CMD ["nginx" ,"-g", "daemon off;"]


```


```nginx
rtmp {
    server {
        listen 1935; # Listen on standard RTMP port
        chunk_size 4000;
        # ping 30s;
        # notify_method get;
        # This application is to accept incoming stream
        # This is the HLS application
        application live {
            live on; # Allows live input from above application
            deny play all; # disable consuming the stream from nginx as rtmp

            hls on; # Enable HTTP Live Streaming
            hls_fragment 3;
            hls_playlist_length 20;
            hls_path /share/hls/;  # hls fragments path

            # MPEG-DASH
            dash on;
            dash_path /share/dash/;  # dash fragments path
            dash_fragment 3;
            dash_playlist_length 20;
        }
    }
}
```

```nginx

#user  nobody;
worker_processes  2;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  4096;
}


include "./conf/rtmp/*.nginx";
include "./conf/rtmp/*.conf";


http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

    server {
        listen       80;
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }
    include "./conf/proxy/*.nginx";
    include "./conf/proxy/*.conf";

    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}
```



```html
<!DOCTYPE html>
<html>
<head>
    <meta charset=utf-8 />
    <title>videojs-contrib-hls embed</title>
    <link href="https://unpkg.com/video.js/dist/video-js.css" rel="stylesheet">
    <script src="https://unpkg.com/video.js/dist/video.js"></script>
    <script src="https://unpkg.com/videojs-contrib-hls/dist/videojs-contrib-hls.js"></script>
</head>
<body>
<h1>Video.js Example Embed</h1>
<video id="my_video_1" class="video-js vjs-default-skin" controls preload="auto" width="640" height="268"
       data-setup='{}'>
    <source src="/hls/home2.m3u8" type="application/x-mpegURL">
</video>
</body>
</html>
```
