https://juejin.cn/post/6844904009577267207

自动装配原理

当我们的SpringBoot项目启动的时候，会先导入AutoConfigurationImportSelector，这个类会帮我们选择所有候选的配置，我们需要导入的配置都是SpringBoot帮我们写好的一个一个的配置类，那么这些配置类的位置，存在与META-INF/spring.factories文件中，通过这个文件，Spring可以找到这些配置类的位置，于是去加载其中的配置。



#### Spring和SpringBoot整体解析理解

- 在启动类上有@SpringBootApplication注解，里面有@EnableAutoConfiguration注解
- @EnableAutoConfiguration中有@Import注解，导入了AutoConfigurationImportSelector类
- AutoConfigurationImportSelector类中有getCandidateConfigurations()方法，可以识别到EnableAutoConfiguration类，这时候就可以把它从配置文件加载回来了
- 但是getCandidateConfigurations()方法又是什么时候被调用的呢

---

- 当执行BeanFactoryPostProcessor时，涉及到一个核心类ConfigurationClassPostProcessor
- 他主要用来处理相关注解的解析工作，包含@Component，@PropertySource，@ComponentScan，@Bean，@Import
- 解析@Import时，会从启动类开始一步步深入，挨个进行查找
- 最后在里面能识别到AutoConfigurationImportSelector类
- 在对这个类进行加载时，有一个延迟加载的属性
- 在延迟加载时，会通过getImports()方法，在里面获取AutoConfigurationEntry对象
- 用entry对象调用getCandidateConfigurations()方法，此时就能把配置文件中对应的属性值加载回来，以完成自动装配





####  开箱即用

- 项目需要的配置都先写好放在一个个的配置类中，类路径定义在META-INF/spring.factories文件中

- @Import注解会导入AutoConfigurationImportSelector类，会到META-INF/spring.factories文件中把所有配置类全部加载进来



#### 约定大于配置

约定好配置文件的名称、路径



#### 配置文件属性自动注入

- 配置类中打@ConfigurationProperties注解，里面定义配置前缀，比如a.b，以及定义属性field
- 在yml等配置文件定义前缀a.b+属性名field并赋值，完成属性field注入
- 利用这个功能可以注入springboot自带配置类的属性，比如修改服务器默认端口属性啥的