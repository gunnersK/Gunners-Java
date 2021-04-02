- Java是跨平台语言，JVM跟Java无关，只跟class文件有关，JVM不看语言，任何语言只要能编译为class文件就可以放到JVM执行，所以JVM是跨语言平台
- JVM是一种规范，本身就是一款操作系统，有各种实现，hotspot(oracal官方实现)、TaobaoVM(hotspot深度定制版)
- JVM < JRE < JDK  
- 每种文件开头都会有特定标识，class文件开头是CA FE BA BE

### class文件结构

- magic：CA FE BA BE
- minor version：class文件的版本号：00 00
- major version：class文件的版本号：00 34
- constant_pool_count：常量池长度
- constant_pool：长度为constant_pool_count - 1的表
- 观察字节码的方法 
  - javap类文件路径(Java自带命令)
  - idea插件：jclasslib
- **对class文件做过深入研究，了解class文件详细信息**
- 常量池重要性在于后面的东西都是去引用他的

### 类加载

- 加载

- 连接
  - 验证：验证类文件（是不是cafebabe开头）
  - 准备：静态变量赋**默认值**
  - 解析 
  
- 初始化：静态变量赋**初始值**，调用静态代码块

- 类加载(调用ClassLoader的loadClass方法)时会生成两块内容：1.把二进制内容丢到内存；2.生成并返回class类对象，指向这块内存（我们不能直接访问二进制内容，只能通过class类对象去访问）

- 类加载器：bootstrap、extension、application、custom classloader

- 双亲委派一层层往父加载器判断加载，但跟父加载器不是继承的关系

- 打印bootstrap加载器会返回空，因为他是cpp实现的模块，java里面并没有一个类于他对应

- 类加载器的也需要类加载器加载进来，溯源到最后都是bootstrap加载器，但他们不是继承关系

- 双亲委派：类加载过程一层层往上委托，再往下，绕了一圈，找不到类就抛异常，**写详细**

- 每个类加载器都会先到自己的缓存拿，拿到就直接返回了，不会重复加载，没有再委托给父加载器加载  

- 双亲委派解释：从父到子，从子到父的过程

- 为什么要有双亲委派：主要为了安全（有人写了类库同名的类，比如java.lang.String，也只能加载类库里的那个类），次要为了效率（不重复加载）

- 反射就是用class类对象去访问二进制代码

- 自定义类加载器：继承ClassLoader类，重写findClass方法即可

  - 找到要load的二进制内容到内存
  - 再在defineClass方法(在ClassLoader类中)转化为class类对象就ok
  - 重写findClass方法就是模板方法设计模式的体现，留一部分功能给子类去实现（01:21:10）
  - 01:40:40

- 写框架、类库都会用自定义ClassLoader，加载指定目录下的类

- 多数JVM的实现都是懒加载，需要才加载类，一般不会把整个jar文件加载进来

- 对某个值异或一次加密，再异或一次就解密了(str ^ seed)

- 看到1:57

  