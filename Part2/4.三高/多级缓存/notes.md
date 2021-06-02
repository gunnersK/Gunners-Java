- Nginx+Lua：做网关，访问Redis、Kafka

  将网关进来的高并发请求一层层过滤，尽可能少地请求落到Tomcat

  将Tomcat接到的请求尽可能少地落到MySQL

- 复杂业务不适合放在Nginx，会导致Nginx性能下降，应该放在Java后端业务

  Nginx能做的事很少，就是接高并发请求，所以性能高

  Nginx+Lua性能高是因为Nginx性能高，不是Lua本身性能高

- Lua是C语言实现的，是胶水语言，可以把不同的异构项目连接起来



#### 多级缓存

- 先把大量请求分摊给各地CDN
- CDN找不到到Nginx找（Nginx可以基于Lua脚本做本地缓存，可提前把数据放到Nginx中缓存）
- Nginx找不到，通过Nginx的Lua脚本给Redis集群发送请求找
- Redis找不到，由Nginx的Lua脚本直接转发请求到应用服务器走数据库找