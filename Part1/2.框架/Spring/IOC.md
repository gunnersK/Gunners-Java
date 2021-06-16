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









#### prepareBeanFactory()的添加属性编辑器注册对象（可自定义扩展）

直接跳到finishBeanFactoryInitialization()深层注册属性编辑器

org.springframework.beans.support.ResourceEditorRegistrar#registerCustomEditors

自定义一个继承PropertyEditorSupport类的编辑器

让Spring能够识别到此编辑器，需要自定义一个实现PropertyEditorRegistrar接口的属性编辑器注册器

让Spring能够识别到对应的注册器

看到手动实现以	上流程，selfEditor.xml定义CustomEditorConfigurer，new容器使用这个xml文件



#### BeanFactoryPostProcessor

如果想在Spring中做任意扩展，唯一需要获取的对象就是BeanFactory，可以对里面的任意东西做更改

如果需要修改BeanFactory中的属性，可以通过实现BeanFactoryPostProcessor接口来修改

看到004400  自定义MyClassPathXmlApplicationContext和MyBeanFactoryPostProcessor，重写customizeBeanFactory方法将MyBeanFactoryPostProcessor加进去

- 自定义BeanFactoryPostProcessor：新建MyBeanFactoryPostProcessor实现BeanFactoryPostProcessor

- 自定义ApplicationContext：创建MyClassPathXmlApplicationContext继承ClassPathXmlApplicationContext
- 重写customizeBeanFactory，调用父类addBeanFactoryPostProcessor添加之前自定义的MyBeanFactoryPostProcessor



#### ConfigurationClassPostProcessor实现BeanFactoryPostProcessor

注解作用原理

在xml配置文件中用component:scan开启注解扫描时，Spring会自动注入IntenalConfiguration，他会指向ConfigurationClassPostProcessor，用来识别和解析注解

以递归的方式解析注解



#### Condition接口

自定义实现Condition接口，可放在@Bean定义的Bean上面，当Bean符合自定义的Condition条件才加载



循环依赖：Spring创建对象分为实例化和初始化两个过程，通过把已实例化但未初始化的对象提前暴露出去来实现



尽管能听懂看懂但是很难表述出来，就准备好面试常问的几个面试题，面试回答这些题的时候带上源码里的关键字，比如类名方法名，带上细节的东西，体现真实性



#### 实例化bean

getBean() -> doGetBean() -> createBean() -> doCreateBean()

实例化之前，有些对象已经实例化好了，无需重复实例化，在beanFactory对象的singletonObjects中

##### BeanFactory和FactoryBean

BeanFactory和FactoryBean都是用来创建对象

FactoryBean可以跳过BeanFactory的复杂的对象流程（实例化、初始化、invokeAwareMethod、invokeInitMethod等），以更加简单便捷的方式创建

FactoryBean提供了三个方法：getObject、getObjectType、isSingleton

只需自己实现FactoryBean接口，重写三个方法即可

13-0200：自己实现FactoryBean接口



#### FactoryBean

自定义类实现FactoryBean接口，在重写的getObject方法中返回对象

那么，实现了FactoryBean接口的子类对象，和getObject方法中返回的对象都归Spring管理

但是存放对象的缓存集合不同

实现了FactoryBean接口的子类对象在一级缓存，getObject方法中返回的对象在factoryBeanObjectCache

Spring会调用FactoryBean接口的子类对象的getObject方法来返回对象



#### 注解IOC

AnnotationConfigApplicationContext中的scan方法将注解包装为BeanDefinition放进BeanFactory

在构造方法中调用refresh方法

15拉完

