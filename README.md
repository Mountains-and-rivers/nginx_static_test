# 资源对应列表

| 打包前缀 | 对应目录      | 进程访问（pc本机）         | 容器访问（服务器192.168.31.240） |
| -------- | ------------- | -------------------------- | -------------------------------- |
| ./       | vue-static-01 | http://localhost:3000      | http://192.168.31.240:8001/      |
| /test    | vue-static-02 | http://localhost:3000/test | http://192.168.31.240:8001/test/ |



# nginx静态目录请求配置

### 打包前缀为 ./

修改打包前缀为 ./

![image](https://github.com/Mountains-and-rivers/nginx_static_test/blob/main/images/vue_1.png)

```
cd vue-demo
npm install # 安装依赖

输出静态目录为 dist
```

### 打包前缀为 /test

修改打包前缀为 /test
![image](https://github.com/Mountains-and-rivers/nginx_static_test/blob/main/images/vue_2.png)
```
cd vue-demo
输出静态目录为 dist
```

# 读取静态文件验证

## 服务进程验证

初始化express功能

```
express 是一个后端服务框架
mkdir express
cd express-demo
npm install -g express
npm install -g express-generator
# 初始化后端工程
express -e vue-demo
cd express-demo/vue-demo
# 安装依赖
npm install 
```

配置 ./ 对应的后端静态路径
![image](https://github.com/Mountains-and-rivers/nginx_static_test/blob/main/images/express_1.png)

启动验证  
```
npm start
```
结果  
![image](https://github.com/Mountains-and-rivers/nginx_static_test/blob/main/images/process_1.png)

配置 /test 对应的后端静态路径
![image](https://github.com/Mountains-and-rivers/nginx_static_test/blob/main/images/express_2.png)
启动验证  
```
npm start
```
结果  
![image](https://github.com/Mountains-and-rivers/nginx_static_test/blob/main/images/process_2.png)
## nginx静态路径配置验证

制作镜像

```
FROM centos
MAINTAINER by wangguilin

RUN yum install nginx -y \
    && rm -rf /usr/share/nginx/html/index.html \
    && useradd -s /sbin/nologin -M www \
    && rm -rf /etc/nginx/nginx.conf

COPY dist/ /usr/share/nginx/html

ADD nginx.conf /etc/nginx/
RUN echo "daemon off;" >> /etc/nginx/nginx.conf
EXPOSE 9876
CMD ["nginx"]
```

其中：

```
dist 为打包输出目录
nginx.conf 为对应nginx配置
```

nginx 配置如下

```
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/

user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

# Load dynamic modules. See /usr/share/doc/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;

    server {
        listen       9876 default_server;
        listen       [::]:9876 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location /test {
	  alias   html;
	  #index  index.html index.htm;
	  try_files $uri $uri/ /index.html;
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

# Settings for a TLS enabled server.
#
#    server {
#        listen       443 ssl http2 default_server;
#        listen       [::]:443 ssl http2 default_server;
#        server_name  _;
#        root         /usr/share/nginx/html;
#
#        ssl_certificate "/etc/pki/nginx/server.crt";
#        ssl_certificate_key "/etc/pki/nginx/private/server.key";
#        ssl_session_cache shared:SSL:1m;
#        ssl_session_timeout  10m;
#        ssl_ciphers PROFILE=SYSTEM;
#        ssl_prefer_server_ciphers on;
#
#        # Load configuration files for the default server block.
#        include /etc/nginx/default.d/*.conf;
#
#        location / {
#        }
#
#        error_page 404 /404.html;
#            location = /40x.html {
#        }
#
#        error_page 500 502 503 504 /50x.html;
#            location = /50x.html {
#        }
#    }

}
```

镜像分片

```
cd docker-image
cat xa* > nginx.tar
docker load -i nginx.tar
```



启动容器

```
docker run -d -p 8001:9876 nginx:test
```

访问验证

