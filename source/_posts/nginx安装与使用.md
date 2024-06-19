---
title: nginx
date: 2024-06-18 15:37:52
tags:
    - 后端
    - nginx
    - 运维
categories:
    - [后端]
index_img: https://img.zphl.top/blog/articleImg/nginx.jpg
excerpt: nginx的安装与相关配置
link: /posts/nginx.html
---

### 1. nginx 安装

Linux nginx 安装参照文档: [nginx 安装文档](https://nginx.org/en/linux_packages.html)

Windows 下载：[nginx 下载](https://nginx.org/en/download.html)

nginx 文档：[nginx 文档](https://nginx.org/en/docs/)

nginx 中文文档：[nginx 中文文档](https://github.com/DocsHome/nginx-docs/tree/master)

### 2. nginx 配置

#### nginx 文件配置结构

**配置文件**： `nginx.conf` linux 一般在 `/etc` 目录

**配置属性**

-   events： 用户网络连接相关配置
-   http: 配置代理、缓存、日志
    -   server: 配置虚拟主机
        -   location: 地址定向，数据缓存，应答控制，以及第三方模块的配置

#### nginx 配置属性介绍

##### 1. 用户配置

` user [yourname];`

##### 2. 进程数

`worker_processes [processes number];`

进程数需要与电脑核心数一致

##### 3. nginx PID 存放路径

`pid [path]`

该文件存储的是 nginx 的进程号

##### 4. 错误日志存放路径

`errror_log [path]`

##### 5. 配置文件引入

`include [path]`

##### 6. events 相关

-   accrpt_mutex
    -   on/off
    -   对多个 nginx 进程接收连接进行序列化，防止多个进程对连接的争抢（惊群）
-   multi_accept
    -   on/off
    -   允许一个进程接收多个新连接
-   worker_connections
    -   每个进程允许的最多连接数

##### 7. http 相关

-   sendfile
    -   on/off
    -   开启高效文件传输模式，sendfile 指令指定 nginx 是否调用 sendfile 函数来输出文件
-   sendfile_max_chunk

    -   size
    -   nginx 每个 worker process 每次调用 sendfile() 传输的数据量的最大值

-   keepalive_timeout
    -   time
    -   设置客户端连接保持活动的超时时间
-   MIME-Type
    -   设置响应头中的 Content-Type 属性
-   access_log
    -   设置日志的格式，以及日志的存储位置
-   keepalive_requests
    -   number
    -   设置客户端请求的最大的单次连接数

###### http: server

server 虚拟主机的配置：

**listen**: 配置监听端口

**server_name**: 配置虚拟主机域名

**配置 HTTPS 证书：**

配置 https 需要制作证书，一般在云服务商处可以生成，包含两个文件: 证书(.crt / .pem / .cer 格式文件)、私钥，将两个文件上传到服务器，并配置：

```json
http: {
    server: {
        listen 443 ssl;
        ssl_certificate /opt/https/server.crt;
        ssl_certificate_key /opt/https/server.key;
        ssl_protocols SSLv3 TLSv1;
        ssl_ciphers HIGH:!ADH:!EXPORT57:RC4+RSA:+MEDIUM;
        ssl_prefer_server_ciphers on;
        ssl_session_cache shared:SSL:2m;
        ssl_session_timeout 5m;


    }
}
```

**localtion 配置:**

`localtion [url]` ：配置地址定向，接收具体 url 或者 正则表达式

配置项:

-   root
    -   [path] : 网站根目录地址
-   index
    -   配置首页

```shell
# 错误页配置
location /404.html {
    root /myserver/errorpages/;
}
```

权限配置：

-   deny：拒绝访问
-   allow: 允许访问

```shell
location / {
    deny 192.168.1.1; # 拒绝该 ip 访问
    allow 192.168.1.0/24;
    allow 192.168.1.2/24;
    deny all;
}
```

##### 8.详细配置示例

```shell
#定义 nginx 运行的用户和用户组
user www www;

#nginx 进程数，建议设置为等于 CPU 总核心数。
worker_processes 8;

#nginx 默认没有开启利用多核 CPU, 通过增加 worker_cpu_affinity 配置参数来充分利用多核 CPU 以下是 8 核的配置参数
worker_cpu_affinity 00000001 00000010 00000100 00001000 00010000 00100000 01000000 10000000;

#全局错误日志定义类型，[ debug | info | notice | warn | error | crit ]
error_log /var/log/nginx/error.log info;

#进程文件
pid /var/run/nginx.pid;

#一个 nginx 进程打开的最多文件描述符数目，理论值应该是最多打开文件数（系统的值 ulimit -n）与 nginx 进程数相除，但是 nginx 分配请求并不均匀，所以建议与 ulimit -n 的值保持一致。
worker_rlimit_nofile 65535;

#工作模式与连接数上限
events
{
    #参考事件模型，use [ kqueue | rtsig | epoll | /dev/poll | select | poll ]; epoll 模型是 Linux 2.6 以上版本内核中的高性能网络 I/O 模型，如果跑在 FreeBSD 上面，就用 kqueue 模型。
    #epoll 是多路复用 IO(I/O Multiplexing) 中的一种方式，但是仅用于 linux2.6 以上内核，可以大大提高 nginx 的性能
    use epoll;

    ############################################################################
    #单个后台 worker process 进程的最大并发链接数
    #事件模块指令，定义 nginx 每个进程最大连接数，默认 1024。最大客户连接数由 worker_processes 和 worker_connections 决定
    #即 max_client=worker_processes*worker_connections, 在作为反向代理时：max_client=worker_processes*worker_connections / 4
    worker_connections 65535;
    ############################################################################
}

#设定 http 服务器
http {
    include mime.types; #文件扩展名与文件类型映射表
    default_type application/octet-stream; #默认文件类型
    #charset utf-8; #默认编码

    server_names_hash_bucket_size 128; #服务器名字的 hash 表大小
    client_header_buffer_size 32k; #上传文件大小限制
    large_client_header_buffers 4 64k; #设定请求缓
    client_max_body_size 8m; #设定请求缓
    sendfile on; #开启高效文件传输模式，sendfile 指令指定 nginx 是否调用 sendfile 函数来输出文件，对于普通应用设为 on，如果用来进行下载等应用磁盘 IO 重负载应用，可设置为 off，以平衡磁盘与网络 I/O 处理速度，降低系统的负载。注意：如果图片显示不正常把这个改成 off。
    autoindex on; #开启目录列表访问，合适下载服务器，默认关闭。
    tcp_nopush on; #防止网络阻塞
    tcp_nodelay on; #防止网络阻塞

    ##连接客户端超时时间各种参数设置##
    keepalive_timeout  120;          #单位是秒，客户端连接时时间，超时之后服务器端自动关闭该连接 如果 nginx 守护进程在这个等待的时间里，一直没有收到浏览发过来 http 请求，则关闭这个 http 连接
    client_header_timeout 10;        #客户端请求头的超时时间
    client_body_timeout 10;          #客户端请求主体超时时间
    reset_timedout_connection on;    #告诉 nginx 关闭不响应的客户端连接。这将会释放那个客户端所占有的内存空间
    send_timeout 10;                 #客户端响应超时时间，在两次客户端读取操作之间。如果在这段时间内，客户端没有读取任何数据，nginx 就会关闭连接
    ################################

    #FastCGI 相关参数是为了改善网站的性能：减少资源占用，提高访问速度。下面参数看字面意思都能理解。
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;

    ###作为代理缓存服务器设置#######
    ###先写到 temp 再移动到 cache
    #proxy_cache_path /var/tmp/nginx/proxy_cache levels=1:2 keys_zone=cache_one:512m inactive=10m max_size=64m;
    ###以上 proxy_temp 和 proxy_cache 需要在同一个分区中
    ###levels=1:2 表示缓存级别，表示缓存目录的第一级目录是 1 个字符，第二级目录是 2 个字符 keys_zone=cache_one:128m 缓存空间起名为 cache_one 大小为 512m
    ###max_size=64m 表示单个文件超过 128m 就不缓存了  inactive=10m 表示缓存的数据，10 分钟内没有被访问过就删除
    #########end####################

    #####对传输文件压缩###########
    #gzip 模块设置
    gzip on; #开启 gzip 压缩输出
    gzip_min_length 1k; #最小压缩文件大小
    gzip_buffers 4 16k; #压缩缓冲区
    gzip_http_version 1.0; #压缩版本（默认 1.1，前端如果是 squid2.5 请使用 1.0）
    gzip_comp_level 2; #压缩等级，gzip 压缩比，1 为最小，处理最快；9 为压缩比最大，处理最慢，传输速度最快，也最消耗 CPU；
    gzip_types text/plain application/x-javascript text/css application/xml;
    #压缩类型，默认就已经包含 text/html，所以下面就不用再写了，写上去也不会有问题，但是会有一个 warn。
    gzip_vary on;
    ##############################

    #limit_zone crawler $binary_remote_addr 10m; #开启限制 IP 连接数的时候需要使用

    upstream blog.ha97.com {
        #upstream 的负载均衡，weight 是权重，可以根据机器配置定义权重。weigth 参数表示权值，权值越高被分配到的几率越大。
        server 192.168.80.121:80 weight=3;
        server 192.168.80.122:80 weight=2;
        server 192.168.80.123:80 weight=3;
    }

    #虚拟主机的配置
    server {
        #监听端口
        listen 80;

        #############https##################
        #listen 443 ssl;
        #ssl_certificate /opt/https/xxxxxx.crt;
        #ssl_certificate_key /opt/https/xxxxxx.key;
        #ssl_protocols SSLv3 TLSv1;
        #ssl_ciphers HIGH:!ADH:!EXPORT57:RC4+RSA:+MEDIUM;
        #ssl_prefer_server_ciphers on;
        #ssl_session_cache shared:SSL:2m;
        #ssl_session_timeout 5m;
        ####################################end

        #域名可以有多个，用空格隔开
        server_name www.ha97.com ha97.com;
        index index.html index.htm index.php;
        root /data/www/ha97;
        location ~ .*.(php|php5)?$ {
            fastcgi_pass 127.0.0.1:9000;
            fastcgi_index index.php;
            include fastcgi.conf;
        }

        #图片缓存时间设置
        location ~ .*.(gif|jpg|jpeg|png|bmp|swf)$ {
            expires 10d;
        }

        #JS 和 CSS 缓存时间设置
        location ~ .*.(js|css)?$ {
            expires 1h;
        }

        #日志格式设定
        log_format access '$remote_addr - $remote_user [$time_local] "$request" ' '$status $body_bytes_sent "$http_referer" ' '"$http_user_agent" $http_x_forwarded_for';

        #定义本虚拟主机的访问日志
        access_log /var/log/nginx/ha97access.log access;

        #对 "/" 启用反向代理
        location / {
            proxy_pass http://127.0.0.1:88;
            proxy_redirect off;
            proxy_set_header X-Real-IP $remote_addr;
            #后端的 Web 服务器可以通过 X-Forwarded-For 获取用户真实 IP
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            #以下是一些反向代理的配置，可选。
            proxy_set_header Host $host;
            client_max_body_size 10m; #允许客户端请求的最大单文件字节数
            client_body_buffer_size 128k; #缓冲区代理缓冲用户端请求的最大字节数，

            ##代理设置 以下设置是 nginx 和后端服务器之间通讯的设置##
            proxy_connect_timeout 90; #nginx 跟后端服务器连接超时时间（代理连接超时）
            proxy_send_timeout 90; #后端服务器数据回传时间（代理发送超时）
            proxy_read_timeout 90; #连接成功后，后端服务器响应时间（代理接收超时）
            proxy_buffering on;    #该指令开启从后端被代理服务器的响应内容缓冲 此参数开启后 proxy_buffers 和 proxy_busy_buffers_size 参数才会起作用
            proxy_buffer_size 4k;  #设置代理服务器（nginx）保存用户头信息的缓冲区大小
            proxy_buffers 4 32k;   #proxy_buffers 缓冲区，网页平均在 32k 以下的设置
            proxy_busy_buffers_size 64k; #高负荷下缓冲大小（proxy_buffers*2）
            proxy_max_temp_file_size 2048m; #默认 1024m, 该指令用于设置当网页内容大于 proxy_buffers 时，临时文件大小的最大值。如果文件大于这个值，它将从 upstream 服务器同步地传递请求，而不是缓冲到磁盘
            proxy_temp_file_write_size 512k; 这是当被代理服务器的响应过大时 nginx 一次性写入临时文件的数据量。
            proxy_temp_path  /var/tmp/nginx/proxy_temp;    ##定义缓冲存储目录，之前必须要先手动创建此目录
            proxy_headers_hash_max_size 51200;
            proxy_headers_hash_bucket_size 6400;
            #######################################################
        }

        #设定查看 nginx 状态的地址
        location /nginxStatus {
            stub_status on;
            access_log on;
            auth_basic "nginxStatus";
            auth_basic_user_file conf/htpasswd;
            #htpasswd 文件的内容可以用 apache 提供的 htpasswd 工具来产生。
        }

        #本地动静分离反向代理配置
        #所有 jsp 的页面均交由 tomcat 或 resin 处理
        location ~ .(jsp|jspx|do)?$ {
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://127.0.0.1:8080;
        }

        #所有静态文件由 nginx 直接读取不经过 tomcat 或 resin
        location ~ .*.(htm|html|gif|jpg|jpeg|png|bmp|swf|ioc|rar|zip|txt|flv|mid|doc|ppt|pdf|xls|mp3|wma)$
        { expires 15d; }

        location ~ .*.(js|css)?$
        { expires 1h; }
    }
}
```

### 3.nginx 疑难杂症

**配置文件目录**

nginx 配置文件目录为 `/etc/nginx`

**执行 nginx 命令显示 pid 文件不存在**

手动指定 nginx 配置文件目录即可解决: `nginx -c /etc/nginx/nginx.conf`

**nginx root 自定义目录访问显示 403**

自定义 nginx 资源目录会显式 403，需要赋予 nginx root 权限

或者将静态文件放置 `/usr/share/nginx` 目录下

**nginx 单页面项目部署刷新显示 404**

因为 nginx 是根据 url 匹配目录，而单页面只有 index 文件，所以会导致找不到目录从而显示 404

使用 `try_files $uri /index.html;` 配置，当找不到目录时，重定向到 index.html 文件

```shell
 location / {
        root /usr/share/nginx/blog;
        index  index.html index.htm;
        # 解决刷新后 404的问题，以url访问时例如访问 /login，因为是单页面，nginx下没有相应目录，使用该配置项，当找不到目录时重定向到 index.html
        try_files $uri $uri/ /index.html;
    }
```

### 参考

[^1]: https://juejin.cn/post/6844903701459501070#heading-8
[^2]: https://github.com/DocsHome/nginx-docs/tree/master
