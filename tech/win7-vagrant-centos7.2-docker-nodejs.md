# win7-vagrant-centos7.2-docker-nodejs环境安装



## virtual box安装

1. 百度下载virtural box
2. 点击下一步，一直安装；
3. 改一下【管理】->【全局设定】->【常规】->【默认虚拟电脑位置】，太吃系统盘了。

## vagrant 安装

1. [http://www.vagrantup.com/](http://www.vagrantup.com/)下载软件
2. 点击下一步，一直安装；
3. 设置环境变量:VAGRANT_HOME，指向非系统盘，用于存放add的box；太吃系统盘了。
4. 注意 win7中vagrant 1.9.5（1.9.6就不行）才能配合 virtual box 5.1.X,http://blog.csdn.net/cow66/article/details/77993908

##  centos7 box下载

1. http://www.vagrantbox.es/下载，
2. 如果被强，可以用http://120.52.52.72/your-url 缓存代理 （已失效）
3. 如果找不到具体下载地址，可以执行vagrant add box <name> ，在日志中可以看到下载地址；

## vagrant box安装

```powershell
# 执行处理化，如果没有进行会自动下，因为被墙，我们只用这个命令得到box下载地址；
vagrant add box centos
### 得到地址，就用其他方式下载；
# 加载box
vagrant box add box-name  box-file.box
# 看看已经加载的box
vagrant box list
# 使用box初始化一个虚拟机,生成VagrantFile，我们在：C:\project\js\csp
vagrant box init box-name
# 启动虚机(在VagrantFile所在目录)
vagrant up
### 使用ssh客户端登录默认2222->22

```

## docker安装

```powershell
###不执行这两条命令，每次都要sudo
# Add the docker group if it doesn't already exist.
$ sudo groupadd docker
#改完后需要重新登陆用户
$ sudo gpasswd -a ${USER} docker
###
# 安装docker
sudo yum install docker #下很多东西，冲杯咖啡吧，3、5分钟总要的。
# 检查安装情况
docker version
# 启动docker服务
sudo systemctl  start docker.service
# 设为开机服务
sudo systemctl  enable docker.service
# 下载centos镜像
sudo docker pull centos #下很多东西，喝杯茶吧
# 下载nodejs瘦身镜像
sudo docker pull node:8-alpine
# 查看已下载的镜像
sudo docker images
# 启动镜像进入终端一看,alpine是：/bin/sh,其他是/bin/bash，注意区别
docker run -it image_name /bin/sh
# 打包镜像,.为当前目录，要有DockerFile->FROM node:8-alpine
sudo docker build -t csp-docker .
# 看看刚刚打的镜像
docker images
# 删除所有镜像
#docker rmi $(docker images | grep none | awk '{print $3}' | sort -r)
sudo docker rmi $(docker images | grep none | awk '{print $3}' | sort -r)
# 启动服务
sudo docker run -d -p 1337:1337 csp-docker
# 查看所有的容器
sudo docker ps -a
# 查勘日志 docker logs -f <容器名orID>
sudo docker logs -f 39
# 删除所有容器
docker rm $(docker ps -a -q)
# 删除单个容器
docker rm <容器名orID>
# 停止、启动、杀死一个容器
docker stop <容器名orID>
docker start <容器名orID>
docker kill <容器名orID>
# 查看docker容器地址
docker inspect --format='{{.NetworkSettings.IPAddress}}' $(docker ps -a -q)  

#################################
# 使用docker镜像
#################################
# 增加私服配置，重启（私服已存在，只是使用）
vi /etc/docker/daemon.json
# 添加以下内容
{"insecure-registries":["registry.XXX.com.cn:5000"]}
# image打标签
docker tag 68edf809afe7 registry.xxx.com.cn:5000/csp/nonauto:1.0
# 最后将新的docker images推送到私服上
docker push registry.xxx.com.cn:5000/csp/nonauto:1.0
# 换一台机器，测试pull情况
docker pull registry.xxx.com.cn:5000/csp/nonauto:1.0
# 查看镜像下载
dokcer images
# 用镜像启动容器
sudo docker run -d -p 1337:1337 5eddaf6ed196 --name=nonauto
```

## nodejs安装

本来没必要在centos安装nodejs，因npm下载遇到问题，装了一个调试，修改https为http解决

```shell
# 下载
sudo wget http://nodejs.org/dist/v8.1.3/node-v8.1.3-linux-x64.tar.gz
# 解压
sudo tar -zxvf node-v8.1.3-linux-x64.tar.gz
# 重命名
mv node-v8.1.3-linux-x64 node
# 进入 node 目录下的bin目录，执行 ls命令：
cd node/bin && ls
./node -v
# 首先在 root 目录下找到 .bash_profile 文件，编辑
vim ~/.bash_profile
# 找到 PATH=$PATH:$HOME/bin，在后面添加路径为：
PATH=$PATH:$HOME/bin:/usr/local/src/node/bin
# 生效
source ~/.bash_profile
```

## DockerFile

```dockerfile
FROM node:8-alpine

# 设置镜像作者
MAINTAINER xumy <xumingyong@xxx.com.cn>


WORKDIR /app/csp

# 安装npm模块
ADD package.json /app/csp/package.json

# 使用淘宝的npm镜像，用http，不要用https
RUN npm install --production -d --registry=http://registry.npm.taobao.org

# 添加源代码
ADD ./api /app/csp/api
ADD ./assets /app/csp/assets
ADD ./config /app/csp/config
#ADD ./linux_node_modules /app/csp/node_modulesassets
ADD ./tasks /app/csp/tasks
ADD ./views /app/csp/views
ADD ./app.js /app/csp/app.js
ADD ./.sailsrc /app/csp/.sailsrc
ADD ./Gruntfile.js /app/csp/Gruntfile.js

# 运行app.js
CMD ["node", "/app/csp/app.js"]
```



## 镜像加速

试过阿里、daocloud等加速，要注册，效果也不好，docker-cn.com反而快。

```
sudo cp -n /lib/systemd/system/docker.service /etc/systemd/system/docker.service
sudo sed -i "s|ExecStart=/usr/bin/docker daemon|ExecStart=/usr/bin/docker daemon --registry-mirror=https://registry.docker-cn.com|g" /etc/systemd/system/docker.service
sudo systemctl daemon-reload
sudo service docker restart

```



## 参数传递

docker run > DockerFile >docker_start.sh

```shell
#NODE_ENV=$1 /app/csp/app.js
#echo $1
#export NODE_ENV=production
#node ./app.js
```





![1499262411026](F:\pic/1499262411026.png)