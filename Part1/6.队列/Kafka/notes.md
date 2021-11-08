- AKF

- broker选举controller时会用到zk

- 消费者组

  组间隔离，一个分区可以给多个组消费，重复利用

- offset

  早期Kafka将offset直接维护在ZK中，但ZK是分布式协调工具，同时用来存储数据性能差

  就出现了一个过渡方案，将offset放第三方，比如Redis/MySQL等

  现在Kafka自带了一个虚拟topic中，将offset维护在其中

- 指定同一个key的消息会发送到同一个partition   `但是这样能顺序消费嘛？不太懂`



#### 消费方式

- Kafka消费方式也有拉取和推送两种

  推送是server主动推送，如果consumer消费不过来，会导致网卡打满

  可以切换拉取方式，让consumer自主按需拉取server数据

#### 消费粒度

即一条条消费，还是一批批消费

2 011500