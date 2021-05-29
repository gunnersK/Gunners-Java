- Tomcat是Servlet容器
- 端口是应用监听的，从OS层面看，数据传输与端口无关，只需要IP
- TCP是可靠传输控制协议，由OS实现，只管数据传输，不管数据的含义
- Socket是TCP协议的接口，OS内部把建立TCP连接的系统调用以Socket的形式暴露出来
- Http协议则是规范数据的含义，由应用程序去实现



#### Tomcat服务端视角

在Tomcat中使用本地方法（JVM中的C语言实现）建立Socket，本地方法又会调用OS的相关C语言方法，即系统调用，返回Socket

绑定端口，监听Socket

使用BIO/NIO模型接收请求数据（Tomcat8开始只支持NIO，之前可同时支持BIO和NIO）

从Socket中取数据，先解析Http请求行，再解析Http请求头

封装Request对象，通过Engine、Host、Context、Wrapper的Pipeline传给Servlet



#### 浏览器客户端视角

指定服务器IP和端口发起Http请求

走TCP协议，与服务端进行三次握手，建立连接

将数据包通过TCP连接传给服务端Socket

Tomcat监听Socket读取数据并根据Http协议解析





`tcp和http，http短连接，tcp长连接之间的关系不理解`

`服务端建立的socket和数据到达的socket有何关系，每个连接都新建一个socket，那在nio模型下，岂不是要等现有请求处理完才可以继续接收tcp连接？`