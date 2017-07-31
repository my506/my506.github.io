# docker-node-pm2-sails.js启动传递环境参数

[TOC]

流程：docker run->DockerFile ->docker_start.sh

![mark](http://7vzsqp.com1.z0.glb.clouddn.com/blog/170707/lDHgd5KDG8.png?imageslim)





## docker run

```shell
 docker run -e NODE_ENV=test -d -p 1337:1337 noncar:0.1
```



## DockerFile

```shell
#FROM 设置镜像
FROM node:8-alpine
# 设置镜像作者
MAINTAINER xumy 
# 创建目录
RUN mkdir -p  /app/csp
# 安装npm模块
ADD package.json /app/csp/package.json
# 安装npm模块
WORKDIR /app/csp
# 使用淘宝的npm镜像
RUN npm install --production -d --registry=http://registry.npm.taobao.org
# 再装pm2
RUN npm install -g pm2 --registry=https://registry.npm.taobao.org
# 添加源代码
ADD . /app/csp/
# 设置环境变量
ENV NODE_ENV test
# 用环境变量启动
CMD /bin/sh ./docker_start.sh $NODE_ENV
```



## docker_start.sh

```shell
#!/bin/sh
echo $NODE_ENV
pm2-docker start pm2.json --env $NODE_ENV
```



## sails.js 代码

### config.connections.js

```javascript
  mongo: {
    adapter: 'sails-mongo',
    host: '10.11.1.161', // defaults to `localhost` if omitted
    port: 27017, // defaults to 27017 if omitted
    database: 'cs' // or omit if not relevant
  },
  devmongo: {
    adapter: 'sails-mongo',
    host: '10.2.108.59', // defaults to `localhost` if omitted
    port: 27017, // defaults to 27017 if omitted
    database: 'cs' // or omit if not relevant
  },
  testmongo: {
    adapter: 'sails-mongo',
    host: '10.1.110.74', // defaults to `localhost` if omitted
    port: 27017, // defaults to 27017 if omitted
    database: 'cs' // or omit if not relevant
  }
```



### config.models.js

```javascript
  //默认本机
  connection: 'mongo
```



### config.env

包括dev.js/prod.js/test.js

### config.env.test.js

```javascript
  models: {
    connection: 'testmongo'
  },
```



## 启动验证

```shell
# 启动
docker run -e NODE_ENV=test -d -p 1337:1337 noncar:0.1
# 日志
docker logs -f cf
```





