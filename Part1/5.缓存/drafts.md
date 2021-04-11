- Redis为何能替代掉Memcache？
  Redis有类型，Memcache无类型，但类型并不重要，重要的是：
  - Redis每种类型都有自己的方法
  - 如果要从value中取出某一元素，Memcache需要耗费更多的网络IO，以及客户端需要去解码才能取出元素  