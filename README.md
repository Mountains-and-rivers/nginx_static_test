# 资源对应列表

| 打包前缀 | 对应目录      | 进程访问                   | 容器访问                         |
| -------- | ------------- | -------------------------- | -------------------------------- |
| ./       | vue-static-01 | http://localhost:3000      | http://192.168.31.240:8001/      |
| /test    | vue-static-02 | http://localhost:3000/test | http://192.168.31.240:8001/test/ |



# nginx静态目录请求配置

### 打包前缀为 ./

修改打包前缀为 ./



```
cd vue-demo
npm install # 安装依赖

输出静态目录为 dist
```

### 打包前缀为 test

修改打包前缀为 /test

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

配置 /test 对应的后端静态路径

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

