# zookeeper-demo
zookeeper操作文档以及实验
<br><br>

## 本地单机运行
---
### 1. 下载安装，设置代理
```shell
wget http://archive.apache.org/dist/zookeeper/zookeeper-3.4.12/zookeeper-3.4.12.tar.gz -e "http_proxy=http://192.168.28.99:1081"
tar -zxvf zookeeper-3.4.12.tar.gz -C /home/hc
```

### 2. 修改默认配置文件
```shell
cd zookeeper-3.4.12/conf/
cp zoo_sample.cfg zoo.cfg 
vi zoo.cfg
...
dataDir=/home/hc/tmp/zookeeper    #修改数据目录
...
```

### 3. 启动zookeeper
```shell
cd zookeeper-3.4.12
bin/zkServer.sh start    #停止stop，状态status
```

### 4. 客户端连接zookeeper
```shell
cd zookeeper-3.4.12
bin/zkCli.sh
```
<br>

## 本地集群运行
---
ps: 请先启动3台服务器并解压好zookeeper-3.4.12
### 1. 修改zoo.cfg并拷贝到三台服务器的 zookeeper-3.4.12/conf 下
```shell
vi zoo.cfg
...
dataDir=/home/hc/tmp/zookeeper
...
server.1=ip1:2888:3888    #ip1:服务器有效ip地址，2888:服务器之间通信端口，3888:leader选举端口
server.2=ip2:2888:3888
server.3=ip3:2888:3888
```
### 2. 在dataDir目录下新建myid文件，内容为本机对应的server.x，以server.1为例
```shell
echo 1 >> /home/hc/tmp/zookeeper/myid
```
### 3. 启动三台服务器zookeeper
```shell
bin/zkServer.sh start    #三台
```

### 4. 客户端连接其中一台zookeeper创建znode，其他两台会同步数据
```shell
bin/zkCli -server ip1:2181,ip2:2181,ip3:2181
```
<br>

## Docker单机运行
---
### 1. 拉取zookeeper镜像
```shell
sudo docker pull zookeeper:3.4.14
```

### 2. 启动服务端
```shell
sudo docker run --name some-zookeeper -d zookeeper:3.4.14
```

### 3. 启动客户端连接服务端
```shell
sudo docker run -it --rm --link some-zookeeper:zookeeper zookeeper:3.4.14 zkCli.sh -server zookeeper
# -it 交互式运行容器
# --rm 退出删除容器
# --link 连接容器，继承some-zookeeper容器里的环境变量，暴露端口等，ZOOKEEPER_*开头
#        修改/etc/hosts文件，用主机名zookeeper能找到some-zookeeper容器内网ip
# 参考连接：https://www.jianshu.com/p/21d66ca6115e
```
<br>

## Docker集群运行
---
### 1. linux环境下先安装docker-compose
```shell
sudo curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.4/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

### 2. 编写docker-compose.yml
```shell
vi docker-compose.yml
version: '2'
services:
    zoo1:
        image: zookeeper:3.4.14
        restart: always
        container_name: zoo1
        ports:
            - "2181:2181"
        environment:
            ZOO_MY_ID: 1
            ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888
 
    zoo2:
        image: zookeeper:3.4.14
        restart: always
        container_name: zoo2
        ports:
            - "2182:2181"
        environment:
            ZOO_MY_ID: 2
            ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888
 
    zoo3:
        image: zookeeper:3.4.14
        restart: always
        container_name: zoo3
        ports:
            - "2183:2181"
        environment:
            ZOO_MY_ID: 3
            ZOO_SERVERS: server.1=zoo1:2888:3888 server.2=zoo2:2888:3888 server.3=zoo3:2888:3888
```

### 3. 运行docker-compose
```shell
sudo COMPOSE_PROJECT_NAME=zk_test docker-compose up
```

### 4. 
## 基本操作
ps: 请先用zkCli.sh 进入客户端界面
操作|指令
:-|:-
添加节点|create /my_test testData    #在/下添加一个名为my_test的znode，数据为testData
删除节点|delete /my_test    #只能删除最后一级目录
获取节点数据|get /my_test
修改节点数据|set /my_test testDataV2
查看节点结构|ls /    #查看/下挂在的节点


