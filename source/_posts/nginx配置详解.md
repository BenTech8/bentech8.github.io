---
title: nginx配置详解
date: 2025-03-13 17:54:28
tags:
categories: Nginx
---

## 配置文件结构

nginx配置文件通常位于/etc/nginx/nginx.conf和/etc/nginx/conf.d/目录。其中/etc/nginx/nginx.conf用于存放全局配置信息。/etc/nginx/conf.d/目录用于存放业务配置信息。

### nginx.conf

nginx.conf配置文件主要存放全局配置信息，主要有以下几个部分组成：

- 全局块：配置影响Nginx全局的指令。
- events块：配置影响Nginx服务器与客户端网络连接的指令。
- http块：配置HTTP服务器相关指令。

```nginx
user nginx;
# 定义运行nginx的用户和用户组
worker_processes	auto;
# 定义工作进程数，通常设置为CPU核心数
error_log	/home/wwwlogs/nginx_error.log	crit;
# 定义错误日志文件路径
pid		/usr/local/nginx/logs/nginx.pid;
# 定义Nginx主进程的PID文件路径
events{
    use epoll;
    # 使用epoll模型
    worker_connections 51200;
    # 每个工作进程的最大连接数
    multi_accept on;
    # 允许一个工作进程同时接受多个连接
}
http{
    include /etc/nginx/mime.types;
    # 包含MIME类型配置文件
    default_type application/octet-stream;
    # 默认MIME类型
    log_format main
        '$remote_addr - '
        '$remote_user [$time_local] "$request" '
        '$status $body_bytes_sent "$http_referer" "$http_user_agent" '
        '$request_length $request_time $upstream_addr '
        '$upstream_response_length $upstream_response_time $upstream_status';
    # 定义日志格式
    access_log /var/log/nginx/access.log main;
    # 定义访问日志文件路径和格式
    sendfile on;
    # 开启高效文件传输模式
    tcp_nopush on;
    # 防止网络阻塞
    tcp_nodelay on;
    # 防止网络延迟
    keepalive_timeout 65;
    # 保持连接的超时时间
    gzip on;
    # 开启Gzip压缩
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;
    # 定义需要压缩的文件类型
    include /etc/nginx/conf.d/*.conf;
    # 包含基他配置文件
}
```



### /etc/nginx/conf.d/default.con

default.conf用于存放业务配置信息，主要有以下几部分组成：

- server块：配置虚拟主机的相关指令。
- location块：配置请求的路由和处理规则。

```nginx
server {
    listen 80;
    # 监听端口
    server_name example.com;
    # 定义服务器名称(域名)
    location / {
        root /usr/share/nginx/html;
        # 定义根目录
        index index.html index.htm;
        # 定义默认首页文件
    }
    location /api {
        proxy_pass http://backend;
        # 反向代理到后端服务器
        proxy_set_header Host $host;
        # 将客户端请求的原始Host字段(即域名)传递给后端服务器。
        proxy_set_header X-Real-IP $remote_addr;
        # 将客户端的真实IP传递给后端服务器。$remote_addr是nginx内置变量，表示客户端的真实IP地址
        proxy_set header X-Forwarded-For $proxy_add_x_forwarded_for;
        # 传递客户端IP列表。包含客户端请求经过的所有代理服务器的IP地址列表，格式为client,proxy1,proxy2。
    }
    location /static/ {
        alias /var/www/static/;
        # 定义静态文件目录
        expires 30d;
        # 设置缓存时间
    }
    error_page 500 503 503 504 /50x.html;
    # 定义错误页面
    location = /50x.html {
        root /usr/share/nginx/html;
    }
}
```



## 注意事项

- 注意配置文件中的结尾有“;”作为结束。

- 每次实验修改完配置文件后需要重新加载nginx配置才会生效。

  ```shell
  # 栓检查配置文件语法
  nginx -t
  # 重新加载配置文件
  nginx -s reload
  ```



## 应用示例

### nginx状态统计

- 安装nginx时将--with-http_stub_status_module模块开启。

- 修改nginx配置文件(写入要访问的server标签中)

  ```nginx
  location /nginx_status {
      stub_status	on;
      access_log	off;
  }
  ```

- 客户端访问网址：http://IP/nginx_status
  - Active connections：表示当前的活动连接数；
  - server accetps handled request：表示已经处理的连接信息。三个数字依次表示已处理的连接数、成功的TCP握手次数、已处理的请求数。



### 目录保护

- 原理和apache的目录保护原理一样。

- 在状态统计的location中添加：

  ```nginx
  auth_basic "Welcome to nginx_status!";
  auth_basic_user_file	/usr/local/nginx/html/htpasswd.nginx;
  ```

- 使用http的命令htpasswd进行用户密码文件的创建

  ```
  htpasswd -c /usr/local/nginx/html/htpasswd.nginx user
  ```

- 重启nginx并再次访问统计页面



### 基于IP的身份验证(访问控制)

- 在状态统计的location中添加：

  ```nginx
  allow	192.168.88.1;
  deny	192.168.88.0/24;
  # 仅允许192.168.88.1访问服务器
  ```

  

### nginx的虚拟主机(基于域名)

- 提前准备好两个网站的域名，并且规划好两个网站网页存放目录

- 在Nginx主配置文件中并列编写两个server标签，并分别写好各自信息

  ```nginx
  server {
      listen 			80;
      server_name		blog.atguigu.com;
      index 			index.html index htm index.php;
      root 			html/blog;
      access_log		logs/blog-access.log main;
  }
  server {
      listen			80;
      server_name		bbs.atguigu.com;
      index 			index.html index.htm index.php;
      root			html/bbs;
      access_log		logs/bbs-access.log main;
  }
  ```

- 分别访问两个不同的域名验证结果



### nginx的反向代理

代理：找别人代替你去完成一件你完成不成的事(代购)，代理的对象是客户端

反向代理：替成厂家卖东西的人就叫反向代理(烟酒代理)，代理的对象是服务器端



- 在另外一台机器上安装apache，启动并填写测试页面

- 在nginx服务器的配置文件中添加（写在某一个网站的server标签内）

  ```nginx
  location / {
      proxy_pass http://192.168.88.100:80;
  }
  ```

- 重启nginx，并使用客户端访问测试



### 负载调度(负载均衡)

负载均衡（Load Balance）其意思就是将任务分摊到多个操作单元上进行执行，例如Web服务器、FTP服务器、企业关键应用服务器和其他关键任务服务器等，从而共同完成工作任务。

- 使用默认的rr轮询算法，修改nginx配置文件

  ```nginx
  upstream bbs {
      server 192.168.88.100:80;
      server 192.168.88.200:80;
  }
  server {
      ......;
      location / {
          proxy_pass		http://bbs;			# 添加反向代理，代理地址填写upstream声明的名字
          proxy_set_header Host $host;		# 重写请求头部，保证网站所有页面都可访问成功
      }
  }
  ```

- 开启并设置两台88.100和88.200的主机

  安装apache并设置不同的index.html页面内容(设置不同页面是为了看实验效果)

- 重启nginx，并使用客户端访问测试

rr算法实现加权轮询

```nginx
upstream bbs {
    server	192.168.88.100:80 weight=1;
    server	192.168.88.200:80 weight=2;
}
```



### nginx实现https（证书+rewrite）

- 安装nginx时，需要将--with-http_ssl_module模块开启

- 在对应要进行加密的server标签中添加以下内容开启SSL

  ```nginx
  server {
      ......;
      ssl							on;
      ssl_certificate				/usr/local/nginx/conf/ssl/atguigu.crt;
      ssl_certificate_key			/usr/local/nginx/conf/ssl/atguigu.key;
      ssl_session_timeout			5m;
      ssl_protocols 				TLSv1 TLSv1.1 TLSv1.2;
      ssl_prefer_server_ciphers	on;
      ssl_ciphers 				"......";
  }
  ```

- 生成证书和密钥文件

- 设置http自动跳转https功能

  原有的server标签修改监听端口

  ```nginx
  server {
      ......;
      listen	443;
  }
  ```

  新增以下server标签(利用虚拟主机+rewrite的功能)

  ```nginx
  server {
      listen			80;
      server_name		bbs.atguigu.com;
      rewrite ^(.*)$ https://bbs.atguigu.com permancent;
      root			html;
      index			index.html index.htm;
  }
  ```

- 重启nginx，并测试



### Nginx防盗链

Nginx防盗链(Referer-based Access Control)是一种防止其他网站盗用你的资源(如图片、视频、文件等)的技术。通过检查请求头中的Referer字段，Nginx可以判断请求是否来自合法的来源，从而阻止非法盗链。

Nginx防盗链配置

```nginx
server {
    listen                80;
    server_name           example.com;
    location ~* \.(jpg|jpeg|png|gif|mp4|flv|pdf)$ {
        valid_referers    none blocked example.com *.example.com;
        if ($invalid_referer) {
            return        403;
            # return      302 https://example.com/anti-leech.html;
            # 或者可以重定向到一个提示页面
        }
        root              /var/www/html;
    }
}
```

- location ~* \.(jpg|jpeg|png|gif|mp4|flv|pdf)$

  匹配需要防盗链的文件类型(图片、视频、PDF等)，~*表示不区分大小写的正则匹配。

- valid_referers

  定义合法的Rerferer来源：

  - none：允许没有Referer字段的请求（如直接访问）。

  - blocked：允许Referer字段被防火墙或代理修改的请求。

    blocked实际效果：

    - 如果请求头中没有Referer字段，Nginx会将其视为blocked，并允许请求通过。
    - 如果Referer字段被修改为非标准值（如unknown或空值），Nginx也会将其视为blocked，并允许请求通过。

  - example.com和*.example.com：允许来自example.com及其子域名的请求。

- $invalid_referer

  如果Referer不在valid_referers列表中，该变量为1，否则为0。

- if ($invalid_referer)

  如果Referer不合法，执行以下操作：

  - return 403：返回403禁止访问。
  - return 302 https://example.com/anti-leech.html：重定向到一个提示页面。

- root /var/www/html：定义资源的根目录。

