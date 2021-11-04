- AKF

- broker选举controller时会用到zk

- 消费者组

  组间隔离，一个分区可以给多个组消费，重复利用

- offset

  早期Kafka将offset直接维护在ZK中，但ZK是分布式协调工具，同时用来存储数据用性能差

  就出现了一个过渡方案，将offset放第三方，比如Redis/MySQL等

  现在Kafka自带了一个虚拟topic中，将offset维护在其中

