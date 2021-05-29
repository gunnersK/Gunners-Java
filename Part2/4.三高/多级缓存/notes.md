- Nginx+Lua：做网关，访问Redis、Kafka

  将网关进来的高并发请求一层层过滤，尽可能少地请求落到Tomcat

  将Tomcat接到的请求尽可能少地落到MySQL

- 复杂业务不适合放在Nginx，会导致Nginx性能下降，应该放在Java后端业务

  Nginx能做的事很少，就是接高并发请求，所以性能高

  Nginx+Lua性能高是因为Nginx性能高，不是Lua本身性能高

- Lua是C语言实现的，是胶水语言，可以把不同的异构项目连接起来