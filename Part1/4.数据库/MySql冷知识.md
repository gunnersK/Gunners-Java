- 使用utf8mb4代替utf8
  
- 这是mysql的bug，真正的utf8每个字符最多占4个字节，而mysql的utf8只支持每个字符最多占3个字节，后来mysql发布了utf8mb4的字符集，这个才是真正的utf8
  
- No operations allowed after statement closed异常

  MySQL5.0以后，自动关闭超过8小时无任何操作的DB连接，连接池自己并不知道该连接已经失效，这时如果有客户端请求连接，仍会把该失效的连接给客户端，造成异常。配置DB时可以配置相应的连接池参数，定时检查连接的有效性，清理无效连接

