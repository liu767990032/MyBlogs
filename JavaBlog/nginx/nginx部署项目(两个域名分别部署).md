1.安装需要的依赖库
```
yum -y install gcc gcc-c++；
yum -y install pcre-devel zlib-devel openssl-devel
```

2.下载nginx,网址:http://nginx.org/en/download.html,然后上传到Linux

3.tar -zxvf解压压缩包

4.进入解压目录的文件夹configure,执行./configure && make && make install,就会有sbin目录,./nginx开启, ./nginx -s reload重新加载配置文件

5.修改nginx/conf里面的nginx.conf文件
```
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;


events {
    worker_connections  1024;
}


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

 # 如果没有显式声明 default server 则第一个 server 会被隐式的设为 default server
#这是为了公网ip直接访问和别人域名访问你的公网ip
    server {
        listen 80;
        server_name _; # _ 并不是重点 __ 也可以 ___也可以
        return 403; # 403 forbidden
    }

server {
    listen    80;
    server_name www.logoxiang.top;
    #charset koi8-r;
    #access_log logs/host.access.log main;
    location / {
      root  html;
      index test1.html;
    }
  }
  server {
    listen    80;
    server_name www.yzh1989.com;
    #charset koi8-r;
    #access_log logs/host.access.log main;
    location / {
      root  yzh1989;
      index index.html;
    }
  }
}
```

6.在nginx根目录创建yzh1989文件夹,里面创建个index.html就可以了,然后执行./nginx 
   -s reload,这是部署静态网页项目

7.打开网页[(点这里)](www.yzh1989.com)即可访问我的静态页面   

