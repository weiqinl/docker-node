# docker-node
Docker实战--部署简单nodejs应用  

# 请结合网站看
http://www.cnblogs.com/weiqinl/p/docker_node.html  

如何在Docker的container里运行Node.js程序  
主体思路：一个简单的Node.js web app,来构建一个镜像，然后基于这个镜像，运行一个容器，从而实现快速部署。  
操作环境：  
虚拟机：ubuntu 16.04 LTE 64位  

第一 先拉取基础镜像  
  sudo docker pull node:latest  
node镜像，star数很高，我们使用它作为基础镜像.latest为tag标签，标识是哪个版本。这一步，也可以省略，后面的Dockerfile文件，会自动拉取该镜像。  

第二 创建Node.js程序  
创建 package.json，并写入相关信息和依赖  

  $ mkdir -p node/website && cd node/website
  $ touch package.json
  $ vi package.json

  {
    "name": "website",
    "version": "0.0.1",
    "description": "Node.js on Docker",
    "author": "weiqinl",
    "main": "server.js",
    "scripts": {
      "start": "node server.js"
    },
    "dependencies": {
      "express": "^4.13.3"
    }
  }
创建server.js  

写一个最简单的web，监听8888端口，返回Hello world。  
使用了node官方建议的框架express  

  $ touch server.js  
  $ vi server.js

    'use strict';

    var express = require('express');

    var PORT = 8888;

    var app = express();
    app.get('/', function (req, res) {
      res.send('Hello world\n');
    });

    app.listen(PORT);
    console.log('Running on http://localhost:' + PORT);
    
第三 创建Dockerfile
Docker会依照Dockerfile的内容来构建一个镜像。

  $ cd ..
  $ touch Dockerfile
  $ vi Dockerfile

  #设置基础镜像,如果本地没有该镜像，会从Docker.io服务器pull镜像
  FROM node

  #创建app目录,保存我们的代码
  RUN mkdir -p /usr/src/node
  #设置工作目录
  WORKDIR /usr/src/node

  #复制所有文件到 工作目录。
  COPY . /usr/src/node

  #编译运行node项目，使用npm安装程序的所有依赖,利用taobao的npm安装

  WORKDIR /usr/src/node/website
  RUN npm install --registry=https://registry.npm.taobao.org

  #暴露container的端口
  EXPOSE 8888

  #运行命令
  CMD ["npm", "start"]
  
第四 构建Image  
在Dockerfile文件所在目录下，运行下面命令来构建一个Image  

  sudo docker build -t weiqinl/node .  
构建完后查看一下刚构建的镜像：  

  sudo docker images  
  
第五 运行镜像  
  sudo docker run -d --name nodewebsite -p 8888:8888 weiqinl/node:latest
-d 表示容器在后台运行  
--name 表示给容器别名 nodewebsite  
-p 表示端口映射。把本机的8888端口映射到容器的8888端口，这样外网就能通过本机的8888端口，访问我们的web了。  
后面的 weiqinl/node 是image的REPOSITORY, latest的镜像的TAG  

第六 测试  
我们先通过curl看是否能访问web  

  curl -i localhost:8888   
通过ubuntu自带的浏览器查看  

如果想进入容器，可以执行命令：  

  sudo docker exec -it weiqinl/node:latest /bin/bash     
到此，Docker部署nodejs应用，已经完成。
