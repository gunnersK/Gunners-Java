先记住三个类

DefaultListableBeanFactory

AbstractAutowireCapableBeanFactory

BeanFactory



- Aware接口的作用

  当容器创建的bean在进行具体操作的时候，如果需要容器的其他对象，可以通过将对象实现Aware接口来达到目的

- Spring对象有两种

  普通对象：我们自定义的对象

  容器对象：内置对象、Spring需要的对象



#### prepareRefresh

做容器刷新前的准备工作

- 设置开启时间、关闭、活跃标志位

- 获取环境对象，设置环境对象属性值

- 设置监听器和需要发布的事件的集合



#### 获取刷新bean工厂

如何在不用阶段处理不同的工作？

运用观察者模式：监听器、监听事件、多播器



#### 启动流程

setConfigLocations()

创建环境对象、处理ClassPathXmlApplicationContext传入字符串的占位符，保存在configLocations数组成员变量中

- 设置配置文件路径到成员变量



**先重点看配置文件解析为beanDefinition和对象初始化两部分**

**然后看aop和springmvc和springboot**

05-015317：解析完bean标签基本属性，对bean标签其他属性进行详细解析 -> 然后给beanDefinition的className和parent属性设置完成，接下来是设置beanDefinition的其他属性