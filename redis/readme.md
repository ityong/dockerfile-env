### docker主从部署规划

|容器名|ip|映射端口|宿主机ip|服务模式|
|:---|:---|:---|:---|:---|
|redis-master|192.168.2.100|6380->6379|x.x.x.x|master|
|redis-slave|192.168.2.101|6381->6379|x.x.x.x|slave|


### 自定义网段
```shell
docker network create --subnet=192.168.2.0/24 redis-network
```

#### 启动容器
```shell
docker run -itd --name redis-master -v /docker/redis/redis-master:/usr/src/redis -p 6380:6379 --network redis-network --ip 192.168.2.100 redis:5.0.7

docker run -itd --name redis-slave -v /docker/redis/redis-slave:/usr/src/redis -p 6381:6379 --network redis-network --ip 192.168.2.101 redis:5.0.7

```


#### docker-compose 启动
```shell
docker-compose up -d
```


#### 主从复制启用
1. 配置文件 在从服务器的配置文件中加入：`slaveof <masterip> <masterport>`
2. 启动命令 redis-server启动命令后加入 `--slaveof <masterip> <masterport>`
3. 客户端命令 Redis服务器启动后，直接通过客户端执行命令：`slaveof <masterip> <masterport>`，则该Redis实例成为从节点。 通过  `info  replication` 命令可以看到复制的一些参数信息


### 重要配置信息参考
```$xslt
###########从库############## #设置该数据库为其他数据库的从数据库
slaveof <masterip> <masterport>
#主从复制中，设置连接master服务器的密码(前提master启用了认证) masterauth <master-password>
slave-serve-stale-data yes
# 当从库同主库失去连接或者复制正在进行，从库有两种运行方式:
# 1) 如果slave-serve-stale-data设置为yes(默认设置)，从库会继续相应客户端的请求
# 2) 如果slave-serve-stale-data设置为no，除了INFO和SLAVOF命令之外的任何请求都会返回一个错误"SYNC with master in progress"
#当主库发生宕机时候，哨兵会选择优先级最高的一个称为主库，从库优先级配置默认100，数值越小优先级越高 slave-priority 100
#从节点是否只读;默认yes只读，为了保持数据一致性，应保持默认 slave-read-only yes

########主库配置############## 
#在slave和master同步后(发送psync/sync)，后续的同步是否设置成TCP_NODELAY假如设置成yes，则redis会合并小的TCP包从而节省带宽，但会增加
同步延迟(40ms)，造成master与slave数据不一致假如设置成no，则redis master会立即发送同步数据，没有延迟 #前者关注性能，后者关注一致性
repl-disable-tcp-nodelay no
#从库会按照一个时间间隔向主库发送PING命令来判断主服务器是否在线，默认是10秒 repl-ping-slave-period 10
#复制积压缓冲区大小设置 
repl-backlog-size 1mb
#master没有slave一段时间会释放复制缓冲区的内存，repl-backlog-ttl用来设置该时间长度。单位为秒。 repl-backlog-ttl 3600
#redis提供了可以让master停止写入的方式，如果配置了min-slaves-to-write，健康的slave的个数小于N，mater就禁止写入。master最少得有多少个
健康的slave存活才能执行写命令。这个配置虽然不能保证N个slave都一定能接收到master的写操作，但是能避免没有足够健康的slave的时候，master不 能写入来避免数据丢失。设置为0是关闭该功能。
min-slaves-to-write 3 min-slaves-max-lag 10
```

