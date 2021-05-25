架构原理：https://apppukyptrl1086.pc.xiaoe-tech.com/detail/i_5d8b8edb901ab_7jWy2jO8/1?from=p_5d887e7ea3adc_KDm4nxCm&type=6



各种MQ选型对比

- 如何集群部署扛高并发
- 海量信息如何分布式存储
- 如何实现主从多备份HA架构
- 如何实现集群路由让别人找到对应机器发送和接收消息





#### 架构及原理

- 架构：NameServer集群+Broker集群
- 每个Broker（主从节点）都会向每个NameServer注册
- 生产者主动去NameServer拉取Broker信息
- 每个Broker每隔30s给每个NameServer发心跳，超过120s没发NameServer认为Broker挂了
- 生产者消费者对宕机的Broker有容错机制



#### 主从同步

Broker主从间通信：slave不停的到master拉取消息



#### 消费规则

RocketMQ没有实现读写分离，消费者先发请求到master，master返回消息给消费者时，会根据master负载和slave同步情况，向消费者建议下次从master拉还是slave拉，防止master当时正在面对高并发（写）



#### 生产规则

写到master，读到master和slave



#### 主从切换

4.5之前，若slave挂了，不会自动切换为master，手动运维恢复

4.5开始，使用Dledger（基于Raft协议）进行选举

ry24