先记住三个类

DefaultListableBeanFactory

AbstractAutowireCapableBeanFactory

BeanFactory



- Aware接口的作用

  当容器创建的bean在进行具体操作的时候，如果需要容器的其他对象，可以通过将对象实现Aware接口来达到目的

- Spring对象有两种

  普通对象：我们自定义的对象

  容器对象：内置对象、Spring需要的对象