## 消息设计

直接用Java类保存比较占空间，引用变量4字节，还要padding对齐

Kafka本质上使用Java NIO的ByteBuffer保存消息，是紧凑的二进制字节结构，无需padding对齐，省去很多空间，同时会使用文件系统的页缓存

进度：6.1.2开始