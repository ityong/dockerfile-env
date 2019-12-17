### redis哨兵机制

#### 一、主从复制的问题
>我们讲了Redis复制的主要内容，但Redis复制有一个缺点，当主机Master宕机以后，我们需要人工解决切换，比如使用`slaveof no one`。实际上主从复制并没有实现高可用。
高可用侧重备份机器， 利用集群中系统的冗余，当系统中某台机器发生损坏的时候，其他后备的机器可以迅速的接替它来启动服务
一旦主节点宕机，写服务无法使用，就需要手动去切换，重新选取主节点，手动设置主从关系。
如果我们有一个监控程序能够监控各个机器的状态及时作出调整，将手动的操作变成自动的。Sentinel的出现就是为了解决这个问题

#### 二、哨兵机制的原理及实现
> `Redis Sentinel` 一个分布式架构，其中包含若干个 Sentinel 节点和 Redis 数据节点，每个 Sentinel 节点会对数据节点和其余 Sentinel 节点进行监控，当它发现节点不可达时，会对节点做下线标识。
如果被标识的是主节点，它还会和其他 Sentinel 节点进行“协商”，当大多数 Sentinel 节点都认为主节点不可达时，它们会选举出一个 Sentinel 节点来完成自动故障转移的工作，同时会将这个变化实时通知给 Redis 应用方。整个过程完全是自动的，不需要人工来介入，所以这套方案很有效地解决了 Redis 的高可用问题。


#### 三、Redis Sentinel 具有以下几个功能：  

- **监控**：Sentinel 节点会定期检测 Redis 数据节点、其余 Sentinel 节点是否可达  
- **通知**：Sentinel 节点会将故障转移的结果通知给应用方   
- **主节点故障转移**：  实现从节点晋升为主节点并维护后续正确的主从关系  
- **配置提供者**：      在 Redis Sentinel 结构中，客户端在初始化的时候连接的是 Sentinel 节点集合 ，从中获取主节点信息。  


同时**Redis Sentinel** 包含了若个 **Sentinel** 节点，这样做也带来了两个好处：  

- 对于节点的故障判断是由多个 **Sentinel** 节点共同完成，这样可以有效地防止误判。
- Sentinel 节点集合是由若干个 **Sentinel** 节点组成的，这样即使个别 Sentinel 节点不可用，整个 Sentinel 节点集合依然是健壮的。  

但是 Sentinel 节点本身就是独立的 Redis 节点，只不过它们有一些特殊，它们不存储数据，<font face="微软雅黑" size = 3  color = #42A5F5> 只支持部分命令  </font>    


#### 四、部署规划  
	  
容器名称	| 容器IP地址	| 映射端口号	| 服务运行模式
------|------|------|------|
redis-sentinel1	 | 192.168.1.121	|26383  -> 26379	|sentinel
redis-sentinel2	 | 192.168.1.122	|26384  -> 26379	|sentinel
redis-sentinel3	 | 192.168.1.123	|26385  -> 26379	|sentinel
redis-master-1	 | 192.168.1.131	|6380  -> 6379	|Master
redis-slave2	 | 192.168.1.132	|6381  -> 6379	|Slave
redis-slave3	 | 192.168.1.13	    |6382  -> 6379	|Slave


#### 五、自定义网段
```shell
docker network create --subnet=192.168.1.0/24 redis-network
```

#### 六、docker-compose 编排容器
```shell
docker run -itd --name redis-master -v /docker/redis/redis-master:/usr/src/redis -p 6380:6379 --network redis-network --ip 192.168.1.100 redis:5.0.7

docker run -itd --name redis-slave -v /docker/redis/redis-slave:/usr/src/redis -p 6381:6379 --network redis-network --ip 192.168.1.101 redis:5.0.7

```

