---
title: Nginx常用配置

date: 2018-07-26 08:31

updated: 2018-07-26 08:31

tags:
- Nginx
- 总结

categories: Java

permalink: nginx-conf
---



## 常用命令

### 安装

~~~shell
yum install nginx
~~~

### 启动

~~~shell
systemctl start nginx
~~~

### 修改配置

~~~shell
vi /etc/nginx/nginx.conf
~~~

### 重启

~~~shell
systemctl restart nginx
~~~



## 监听80端口并将请求转发到其他端口的Web服务器

~~~nginx
location /api {									  # /api 是访问的上下文
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-Host $host;
    proxy_set_header X-Forwarded-Server $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass http://127.0.0.1:8082/;               # 8082 是Web服务器的端口
}
~~~

效果是原来通过`http://xxx.xxx.xxx.x:8082/something`访问的请求可以通过`http://xxx.xxx.xxx.x/api/something`来访问



## 暴露本地文件路径

~~~nginx
location / {                                  # / 访问的上下文
    root   /opt/some-dir;                     # 希望暴露的路径
    index  index.html index.htm;
}
~~~

效果是`/opt/some-dir`目录及其子目录下的静态文件可以通过类似`http://xxx.xxx.xxx.x/file.txt`的形式来访问



## HTTPS

~~~nginx
server {
    listen       443 ssl;
    server_name  localhost;
    ssl_certificate      certs/demo-cert.pem;            # 需要把证书放在/etc/nginx/certs/目录下
    ssl_certificate_key  certs/demo-cert.key;            # 需要把证书放在/etc/nginx/certs/目录下
    ssl_session_cache    shared:SSL:1m;
    ssl_session_timeout  5m;
    ssl_ciphers  ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_prefer_server_ciphers  on;

    # api的location

    # 静态文件location

}
~~~