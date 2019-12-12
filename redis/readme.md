### docker主从部署规划

|容器名|ip|映射端口|宿主机ip|服务模式|
|:---|:---|:---|:---|:---|
|redis-master|192.168.1.100|6380->6379|x.x.x.x|master|
|redis-slave|192.168.1.101|6381->6379|x.x.x.x|slave|


### 自定义网段
```shell
docker network create --subnet=192.168.1.0/24 redis-network
```

#### 启动容器
```shell
docker run -itd --name redis-master -v /docker/redis/redis-master:/usr/src/redis -p 6380:6379 --network redis-network --ip 192.168.1.100 redis:5.0.7

docker run -itd --name redis-slave -v /docker/redis/redis-slave:/usr/src/redis -p 6381:6379 --network redis-network --ip 192.168.1.101 redis:5.0.7

```

#### 主从复制启用
1. 配置文件 在从服务器的配置文件中加入：`slaveof <masterip> <masterport>`
2. 启动命令 redis-server启动命令后加入 `--slaveof <masterip> <masterport>`
3. 客户端命令 Redis服务器启动后，直接通过客户端执行命令：`slaveof <masterip> <masterport>`，则该Redis实例成为从节点。 通过  `info  replication` 命令可以看到复制的一些参数信息
  

