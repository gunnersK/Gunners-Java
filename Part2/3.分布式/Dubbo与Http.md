Dubbo基于TCP传输，直接反序列化TCP接收到的二进制数据，即可得到真实数据

Http接收到TCP二进制数据之后，还要用Http协议进行一层解析，再反序列化body中的内容，比如json，才能得到真实数据

但是Http应用层协议跟序列化协议看起来差不多，有什么区别呢