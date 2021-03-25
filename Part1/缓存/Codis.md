- 是一款Redis的Proxy，主要负责把Redis的请求分发到不同的Redis实例当中

- Codis代理了Redis服务，所以我们发起请求的时候，并不是请求到Redis-server所在的机器上，而是到Codis机器上，Codis机器再根据一定的路由规则进行分发，
  最终请求到Redis-Server的机器上，也就是说，如果我们使用一个get请求，可能会到多台机器上

