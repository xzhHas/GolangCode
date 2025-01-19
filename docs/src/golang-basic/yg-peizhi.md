---
title: Ubuntu系统安装docker以及安装yg系统所能使用到的插件
shortTitle: 27.电商系统的插件配置教程
description: Ubuntu系统安装docker以及安装yg系统所能使用到的插件,配置文件信息
author: GolangCode
category:
  - Go
tags:
 - Go
date: 2024-05-02
---


> 前言：建议大家使用ubuntu系统的时候，直接永久关闭防火墙目前我们处于学习状态，这样有利于提高开发效率。

- 项目地址：[https://github.com/xzhHas/yg](https://github.com/xzhHas/yg)
- 原文链接：[https://blog.csdn.net/m0_73337964/article/details/139523540](https://blog.csdn.net/m0_73337964/article/details/139523540)

## 一、安装docker

1. **更新现有的软件包**：

   ```bash
   sudo apt update
   ```

2. **安装必要的软件包**：

   ```bash
   sudo apt install apt-transport-https ca-certificates curl software-properties-common
   ```

3. **添加Docker的官方GPG密钥**：

   ```bash
   curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
   ```

4. **设置Docker稳定版的APT源**：

   ```bash
   echo "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
   ```

5. **更新APT包索引**：

   ```bash
   sudo apt update
   ```

6. **确保安装的是从Docker官方仓库而不是Ubuntu默认仓库**：

   ```bash
   apt-cache policy docker-ce
   ```

   你应该能看到输出中显示了 `https://download.docker.com/linux/ubuntu` 字样，表示APT源已正确配置。

7. **安装Docker**：

   ```bash
   sudo apt install docker-ce
   ```

8. **验证Docker是否安装成功**：

   ```bash
   sudo systemctl status docker
   ```
  9. **给docker配置阿里云镜像加速**：[阿里云跳转网址](https://account.aliyun.com/login/login.htm?oauth_callback=http://dc.console.aliyun.com/next/index?spm=5176.21213303.J_qCOwPWspKEuWcmp8qiZNQ.24.7bf72f3da6Y327##/overview)


   如果Docker正在运行，你会看到类似以下的输出：

<img src="https://golang-code.oss-cn-beijing.aliyuncs.com/images/202501071534879.png" alt="在这里插入图片描述" style="zoom:50%;" />


9. **将当前用户添加到docker用户组**（可选，方便以后无需使用sudo运行docker命令）：

   ```bash
   sudo usermod -aG docker ${USER}
   ```

   然后退出当前终端并重新登录，或者使用以下命令：

   ```bash
   su - ${USER}
   ```

10. **验证用户组更改**：

    ```bash
    id -nG
    ```

    确保你在docker组中。

11. **测试Docker**：

    ```bash
    docker run hello-world
    ```

出现以下界面表示docker安装成功：
<img src="https://golang-code.oss-cn-beijing.aliyuncs.com/images/202501071534623.png" alt="在这里插入图片描述" style="zoom:50%;" />

## 二、安装Redis，并使用resp验证连接

1. **拉取Redis镜像**：

   ```bash
   docker pull redis
   ```

2. **运行Redis容器**：

   ```bash
   docker run -d --name redis -p 6379:6379 redis
   ```


   你应该能看到Redis容器在运行。

3. **验证MySQL是否运行**：

   ```bash
   docker ps -f name=mysql
   ```
	![在这里插入图片描述](https://cdn.golangcode.cn/images/202501181839034.png)


4. **连接到Redis容器**：使用resp连接
<img src="https://golang-code.oss-cn-beijing.aliyuncs.com/images/202501071534672.png" alt="在这里插入图片描述" style="zoom: 50%;" />

## 三、安装Mysql8.0，搭建逻辑卷，链接navicate以及导入sql的数据（sql文件在docs里面）

1. **拉取MySQL 8.0镜像**：

   ```bash
   docker pull mysql:8.0
   ```

2. **创建Docker卷以存储MySQL数据**：

   ```bash
   docker volume create mysql-data
   ```

3. **运行MySQL容器**：

   ```bash
   docker run -d --name mysql \
     -e MYSQL_ROOT_PASSWORD=your_password \
     -e MYSQL_DATABASE=your_database \
     -e MYSQL_USER=your_user \
     -e MYSQL_PASSWORD=your_password \
     -p 3306:3306 \
     -v mysql-data:/var/lib/mysql \
     mysql:8.0
   ```

   - 使用 `your_password` 作为MySQL root用户的密码（请替换为你的实际密码）。
   - 使用 `your_database` 创建一个初始数据库（请替换为你需要的数据库名称）。
   - 使用 `your_user` 和 `your_password` 创建一个普通用户（请替换为你的实际用户名和密码）。
   - 将MySQL数据存储在Docker卷 `mysql-data` 中。

使用docker成功安装之后使用navicate连接并且导入sql文件：
<img src="https://golang-code.oss-cn-beijing.aliyuncs.com/images/202501071534531.png" alt="在这里插入图片描述" style="zoom:50%;" />
导入数据：（这些sql文件都会上传到GitHub去，直接获取就可以了），如果想使用apifox测试接口的话，可以直接使用mallshop.json直接导入到apifox中即可测试接口了，也可以查看我已经导出来的mallshop文档。
<img src="https://cdn.golangcode.cn/images/202501181839926.png" alt="在这里插入图片描述" style="zoom:50%;" />

## 四、Nacos安装及yg系统的配置信息 

1、安装Nacos：
```go
docker run --name nacos-standalone -e MODE=standalone -e JVM_XMS=512m -e JVM_XMX=512m -e JVM_XMN=256m -p 8848:8848 -d nacos/nacos-server:latest
```
2、设置开机自动启动：

```
docker container update --restart=always 770ecbd6b209（替换为你的容器id）
```

检查是否安装成功：（修改为你的ip地址）

http://192.168.124.51:8848/nacos/index.html

**此处，nacos的具体配置信息因为篇幅有限，我就放到我的vx公众号上了，如果有需要的话直接搜索 ‘席万里要学习’，然后发送信息‘yg-nacos’，就可得到需要配置的信息了。**

## 五、安装Consul服务中心

1、docker安装Consul：

**注：这里的ip地址，需要修改为你自己的ip**
```
docker run -d -p 8500:8500 -p 8300:8300 -p 8301:8301 -p 8302:8302 -p 8600:8600/udp consul consul agent -dev -client=0.0.0.0
```
默认访问端口为8500：http://192.168.91.129:8500/，ip地址根据自己的地址修改

8600端口是dns的端口，8500是http的端口：注册与服务发现都是通过8500端口

<img src="https://golang-code.oss-cn-beijing.aliyuncs.com/images/202501071534740.png" alt="在这里插入图片描述" style="zoom:50%;" />

我们可以看到所有的服务都已经启动了，这个需要启动后端，一开始配置好Consul就行了，后期在看这个（如果搞注册中心的时候遇到问题都可以向我反馈，我也遇到了许多的问题）。

## 六、安装ES、Kibana、Ik
#### 1、安装ES

1. **创建Elasticsearch的配置文件夹和数据目录**：

   ```bash
   sudo mkdir -p /data/elasticsearch/config
   sudo mkdir -p /data/elasticsearch/data
   sudo mkdir -p /data/elasticsearch/plugins
   ```

2. **设置目录权限**：

   ```bash
   sudo chmod 777 -R /data/elasticsearch
   ```

3. **写入配置到 `elasticsearch.yml` 文件**：

   ```bash
   echo "http.host: 0.0.0.0" | sudo tee /data/elasticsearch/config/elasticsearch.yml
   ```


重启Docker服务，以确保Docker能正确访问新的目录配置：

```bash
sudo systemctl restart docker
```


1. **拉取Elasticsearch镜像**（如果还没有）：

   ```bash
   docker pull elasticsearch:7.10.1
   ```

2. **运行Elasticsearch容器**：

   ```bash
   docker run --name elasticsearch \
     -p 9200:9200 -p 9300:9300 \
     -e "discovery.type=single-node" \
     -e ES_JAVA_OPTS="-Xms128m -Xmx256m" \
     -v /data/elasticsearch/config/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml \
     -v /data/elasticsearch/data:/usr/share/elasticsearch/data \
     -v /data/elasticsearch/plugins:/usr/share/elasticsearch/plugins \
     -d elasticsearch:7.10.1
   ```



   **访问Elasticsearch**：

   打开浏览器，访问 `http://localhost:9200`，应该能看到Elasticsearch的欢迎信息。（这里输入你自己的IP地址）
   <img src="https://golang-code.oss-cn-beijing.aliyuncs.com/images/202501071534603.png" alt="在这里插入图片描述" style="zoom: 33%;" />

#### 二、安装Kibana

1. **拉取Kibana镜像**（如果还没有）：

   ```bash
   docker pull kibana:7.10.1
   ```

2. **运行Kibana容器**：

   ```bash
   docker run -d --name kibana \
     -e ELASTICSEARCH_HOSTS="http://192.168.124.51:9200" \
     -p 5601:5601 \
     kibana:7.10.1
   ```

   请将 `192.168.124.51` 替换为你的虚拟机的实际IP地址。

**访问Kibana**：

   打开浏览器，访问 `http://192.168.124.51:5601`（用你的实际IP地址替换），应该能看到Kibana的界面。
<img src="https://golang-code.oss-cn-beijing.aliyuncs.com/images/202501071534826.png" alt="在这里插入图片描述" style="zoom: 33%;" />

#### 三、安装IK分词器

IK官方地址：https://github.com/medcl/elasticsearch-analysis-ik/releases

下载：elasticsearch-analysis-ik-7.10.1.zip

将这个下载的zip文件修改为ik.zip上传到Linux上的 `/usr/share/elasticsearch/plugins`下即可。

![在这里插入图片描述](https://cdn.golangcode.cn/images/202501181840734.png)
大功告成！

## 最后我们可以看到docker运行的容器
<img src="https://golang-code.oss-cn-beijing.aliyuncs.com/images/202501071534693.png" alt="在这里插入图片描述" style="zoom:33%;" />


----------------

感谢大家的观看，有什么需要的资料或者信息都可以私聊我，目前开发文档正在完善中。nacos的配置信息，可以查看这个公众号里面发布的信息。

![vximg](https://cdn.golangcode.cn/images/202501181841253.png)

