## 消息设计

直接用Java类保存比较占空间，引用变量4字节，还要padding对齐

Kafka本质上使用Java NIO的ByteBuffer保存消息，是紧凑的二进制字节结构，无需padding对齐，省去很多空间，同时会使用文件系统的页缓存









## 集群管理

Kafka通过zk实现分布式自动化服务发现与成员管理

每台broker启动时会将自己注册到zk下的一个节点与zk临时节点

- broker生命周期

  zk的临时节点与客户端会话绑定，客户端会话失效，节点自动清除

  broker利用zk临时节点来管理生命周期，broker启动时会对应创建zk临时节点，同时创建监听器监听节点状态

  broker启动，监听器同步整个集群信息到该broker

  broker崩溃，与zk的会话失效，节点被删除，监听器处理broker崩溃的后续事宜

书讲的太浅，深入了解需重新找资料









## 副本与ISR

每个分区的数据做多份副本（备份），将副本分摊到所有broker上，follower从leader请求数据并做出响应

#### 位移信息

- 起始位移 -- 该副本当前所含第一条消息的offset  
- 高水印值HW -- 该副本最新一条**已提交**消息的offset，leader副本的HW决定了consumer能获取该分区消息的上限
- 日志末端位移LEO -- 副本日志中下一条待写入消息的offset（消息落盘后LEO+1）



#### follower同步

假设一个分区有三个副本，1个leader + 2个follower，数据同步过程大致如下：

- leader收到producer发来的消息落盘，LEO+1
- 两个follower发送请求给leader
- leader将消息推送给follower
- follower接收到消息落盘，各自LEO+10
- leader接收到follower的数据请求响应后，更新HW，此时该消息对consumer可见

*当producer的acks=-1，则需要做完以上操作，producer才能正常返回，才代表消息发送成功*

具体数据备份详见下方 “水印-数据备份过程” 部分



#### ISR设计

ISR是topic分区维度的概念，每个topic分区都有自己的 ISR 列表，ISR本质是某个分区与leader**同步**的副本集合

leader副本在ISR中，ISR中的副本才可选为leader，消息被ISR所有副本接收到才视为提交

所以最重要就是ISR成员的定义，即判定副本的同步/不同步状态

##### 老版判定规则与缺陷

0.9.0.0之前，用follower的LEO落后于leader的**消息数**（replica.lag.max.messages**全局**参数）来判定follower是否 ”不同步“

当follower落后leader超过这个数则将其踢出ISR。但是这样相对死板，当瞬间接收一波高峰流量，远超这个数时，副本就会被认为不同步，被踢出ISR，尽管他是存活的，只能等下次请求加入ISR时追上leader的LEO，于是会出现同步、不同步、再同步，不停地加入退出ISR的情况

**对于一个参数的设置，用户应该对他们知道的参数进行设置，而不是对他们需要进行猜测的参数进行设置。对于该参数来说，只能猜测应该设置成哪些值，无法根据需要对其进行设置**  



##### 新版判定规则

新版使用最大时间间隔（replica.lag.time.max.ms参数）来控制，默认10s，尽管follower的LEO落后于leader，只要不是持续性落后，就不会被踢出ISR，这样即使遇到流量高峰，也不会出现反复加入退出ISR的情况









## 水印

流式处理领域，水印表示时间，在Kafka中表示位置信息，即位移

副本有三类：leader、follower、ISR副本集合

consumer无法消费到leader上你位移大于HW水印的消息

follower会有两套LEO，一套保存在follower上，一套保存在leader上，称为remote LEO

#### LEO更新机制

- follower -- 向leader FETCH到数据落盘时
- leader -- 将producer的数据落盘时

#### HW更新机制

- follower -- 更新LEO后，取当前LEO与FETCH响应中leader的HW的小者
- leader -- 有4个时机，取所有满足条件的follower在leader的LEO与leader的LEO的小者
  - 副本成为leader时
  - broker崩溃导致副本被踢出ISR时
  - producer向副本写入消息时
  - leader处理follower的FETCH请求时



#### 数据备份过程

- 情况一：leader写入消息后，follower发送FETCH请求

  > leader接收到消息后，将消息写入日志，更新leader的LEO
  >
  > leader尝试更新HW（符合更新HW的第3个时机）：取remote LEO（此时是0）与自己LEO的最小值，即是0，故不更新HW值
  >
  > 此时follower发来FETCH请求
  >
  > leader通过FETCH请求的offset确认是否需更新remote LEO，此时不需要
  >
  > leader尝试更新HW（符合更新HW的第4个时机）：比较自己LEO和remote LEO，还是0，不更新HW
  >
  > 将数据和当前分区HW（即leader HW）发送给follower
  >
  > follower接收到FETCH resp之后，将消息写入日志，更新follower LEO
  >
  > follower尝试更新HW：取自己LEO与resp中leader HW的最小值，即是0，故follower HW=0
  >
  > 此时follower发第二轮FETCH，leader收到后读取log数据，更新remote LEO=1（因为第二轮FETCH请求的offset是1，因为上一轮结束后 follower LEO 被更新为1）
  >
  > leader尝试更新HW：取remote LEO（此时是1）与自己LEO的最小值，即是1，故更新leader HW=1，即分区HW更新为1，自此该消息对消费者可见
  >
  > 接着leader把数据（此时无数据）和分区HW发送给follower
  >
  > follower接收到FETCH resp之后，尝试更新HW：取自己LEO与resp中leader HW的最小值，即是1，故更新follower HW=1
  >
  > 此时完成数据备份的完整过程

follower的FETCH请求因为无数据而暂时被寄存到 leader端的 purgatroy中 ，超时强制完成。期间有数据会自动唤醒请求

leader的HW值是在第二轮FETCH中确定的，因为要比对follower的remote LEO

- 情况二：FETCH请求保存在purgatory中时生产者发来消息

  > leader将消息写入本地log，更新LEO
  >
  > 唤醒purgatory中的FETCH请求，尝试更新HW
  >
  > 唤醒FETCH后的处理与第一种情况一致

##### 缺陷

水印备份机制有数据丢失、数据（顺序）不一致的缺陷，因为HW是异步延迟更新，**而且崩溃恢复之后会做日志截断**

- 数据丢失 -- leader已更新HW=2，follower更新HW=2之前就宕机，故HW=1，重启后日志截断，发送FETCH之前leader宕机，则follower成为leader，原leader重启后成为follower，也做日志截断，则数据永远丢失
- 数据不一致 -- leader已更新HW=2，follower未更新，HW=1，此时同时挂掉，follower先重启回来成为leader，收到新消息，此时HW=2，只有一节点，故分区HW=2，原leader重启回来，发现自身HW=2，分区HW=2，无需日志截断调整，导致两节点数据（顺序）不一致

只要做日志截断丢数据后的follower成为leader，数据就永远丢失 

用leader epoch，即一对值（epoch，offset），来解决以上缺陷。

**没搞明白leader epoch工作原理**









## 日志存储设计

将原生消息和元数据信息组装成record，按时间顺序追加写入日志尾部，每条记录都会被分配位移递增的id

每个分区都有自己的日志，由日志段文件和日志段索引文件组成

每个分区一个子目录，例如test-0、test-1，目录中包含日志相关文件

日志文件命名 -- 使用文件中第一条消息的offset来命名

日志文件大小 -- 日志段文件可设置上限大小，默认1G（log.segment.bytes参数），日志段文件填满记录后，会自动创建一组新的日志段文件和索引文件，称为日志切分

当前激活日志段 -- Kafka正在写入的分区日志段文件称为当前激活日志段



#### 索引文件

位移索引、时间戳索引

- 位移索引文件

  保存与索引文件起始位移（文件名）的相对位移，升序排列

- 时间戳索引文件

索引文件读写模式、文件大小

>  非当前日志段，索引文件只读打开，当前日志段的索引文件可读打开

> 索引文件预先分配好空间，默认10M（log.index.size.max.bytes参数），索引文件切分时，会将文件裁剪到真实大小，所以当前日志段索引文件一般比已切分索引文件大



#### 日志留存

基于时间留存

基于大小留存

异步清除日志，不清除当前日志段



#### 日志campaction

没看懂



看完日志存储设计，接下来通信协议、controller、broker请求处理待理解