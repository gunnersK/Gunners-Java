## 消息设计

直接用Java类保存比较占空间，引用变量4字节，还要padding对齐

Kafka本质上使用Java NIO的ByteBuffer保存消息，是紧凑的二进制字节结构，无需padding对齐，省去很多空间，同时会使用文件系统的页缓存

进度：6.1.2开始









## 集群管理

Kafka通过zk实现分布式自动化服务发现与成员管理

每台broker启动时会将自己注册到zk下的一个节点与zk临时节点

- broker生命周期

  zk的临时节点与客户端会话绑定，客户端会话失效，节点自动清除

  broker利用zk临时节点来管理生命周期，broker启动时会对应创建zk临时节点，同时创建监听器监听节点状态

  broker启动，监听器同步整个集群信息到该broker

  broker崩溃，与zk的会话失效，节点被删除，监听器处理broker崩溃的后续事宜









## 副本与ISR

每个分区的数据做多份副本（备份），将副本分摊到所有broker上，follower从leader请求数据并做出响应

ISR是topic分区维度的概念，每个topic分区都有自己的 ISR 列表，ISR本质是?某个分区与leader**同步**的副本集合

leader副本在ISR中，ISR中的副本才可选为leader，消息被ISR所有副本接收到才视为提交

- 位移信息

  起始位移、高水印值、日志末端位移

**对于一个参数的设置，有一点是很重要的：用户应该对他们知道的参数进行设置，而不是对他们需要进行猜测的参数进行设置。对于该参数来说，我们只能猜测应该设置成哪些值，而不是根据需要对其进行设置**  



#### ISR设计

同步/不同步的定义









## 水印

流式处理领域，水印表示时间，在Kafka中表示位置信息，即位移

副本有三类：leader、follower、ISR副本集合

消费者无法消费到leader上你位移大于HW水印的消息

follower会有两套LEO，一套保存在follower上，一套保存在leader上，称为remote LEO

LEO更新机制

- follower -- 向leader FETCH到数据落盘时
- leader -- 将producer的数据落盘时

HW更新机制

- follower -- 更新LEO后，取当前LEO与FETCH响应中leader的HW的小者
- leader -- 有4个时机（归纳），取所有满足条件的follower在leader的LEO与leader的LEO的小者

follower的FETCH请求因为无数据而暂时被寄存到 leader端的 purgatroy中 ，超时强制完成。期间有数据会自动唤醒请求

leader的HW值是在第二轮FETCH中确定的，因为要比对follower的remote LEO

水印备份机制有数据丢失、数据（顺序）不一致的缺陷，因为HW是异步延迟更新，**而且崩溃恢复之后会做日志截断**

- 数据丢失 -- leader已更新HW=2，follower更新HW=2之前就宕机，故HW=1，重启后日志截断，发送FETCH之前leader宕机，则follower成为leader，原leader重启后成为follower，也做日志截断，则数据永远丢失
- 数据不一致 -- leader已更新HW=2，follower未更新，HW=1，此时同时挂掉，follower先重启回来成为leader，收到新消息，此时HW=2，只有一节点，故分区HW=2，原leader重启回来，发现自身HW=2，分区HW=2，无需日志截断调整，导致两节点数据（顺序）不一致

只要做日志截断丢数据后的follower成为leader，数据就永远丢失 

用leader epoch，即一对值（epoch，offset），来解决以上缺陷。







P199 新版本解决缺陷

之后完整归纳副本、ISR、水印更新流程