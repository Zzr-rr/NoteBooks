## Spring启动过程

在Spring使用过程中的最方便的地方就是不需要自己去new一个新的对象。这就是通过IOC（控制反转）来实现的。Spring会把对象放在一个容器内部，可以通过context.getBean(Person.class)来获取我们的bean对象。容器内通过一堆的`map`结构来存储`key-value`的结构。

如何告诉Spring呢？可以通过不同的配置文件，比如XML或者注解。

XML文件：首先通过I/O流来读取，读取完之后放到内存里。XML文件是一个常用的格式化文件，可以通过dom4j/SAX等工具来解析，解析完后得到document，可以放到某个对象中去。

注解：类似于XML文件，通过**反射**的方式获取到Class对象，通过getAnnotation获取相应的注解。

无论是XML文件或是注解，都是为了获取定义好对象的基本信息（bean的定义信息）。容器内有map结构用于存放这些东西。叫做BeanDefinitionMap（一个接口），存放的就是Bean的定义信息。在xml/注解到BeanDefinition中间，需要定义一个接口叫做BeanDefinitionReader，用于约束和规范其它配置文件，这样就可以解析不同的格式文件，**大大提高了Spring的可扩展性。**

在获取了BeanDefinition bean的定义信息之后需要创建bean对象、使用bean对象、销毁bean对象。

创建bean对象的时候通过反射的方式去实现。只有当创建很多的对象时才会出现较为低下的效率。

```java
// 1.获取Class对象
Class clazz = Class.forName("包名+类名");
Class clazz = 对象.getClass();
Class clazz = 类目.class();
// 2.获取构造器
Constructor ctor = clazz.getDeclaredConstructor();
// 3.创建对象
Object obj = ctor.newInstance();
```

当通过bean对象创建一个BeanDefinition时，如果需要替换掉里面的动态定义的值等等进一步的操作，就可以通过`PostProcessor(可以理解为后置处理器|增强器)`，其意义就在于提供某些额外的扩展功能。

常用的有两个接口，一个是BeanFactoryPostProcessor，另一个是BeanPostProcessor，提供某些额外的扩展功能。他们针对的操作对象不同。BeanFactory和Bean，Bean就是我们常说的Bean对象，BeanFactory就是用于管理容器的，BeanFactory约等于容器。而可以通过BeanFactoryPostProcessor对其进行修改。

BeanFactoryPostProcessor是一个接口，其上有一个子类叫做BeanDefinitionRegistryPostProcessor。能够更加方便地进行扩展，能够适应不同的配置文件。

## Bean的生命周期

从对象的创建到使用到销毁的过程。

1. 实例化：在堆空间中申请空间，此时对象的属性值一般是默认值。就是反射创建对象的过程 --> createBeanInstance 反射创建对象。

2. 初始化：

   - 实例化后为自定义**属性赋值** --> populateBean中的set方法来赋值。

   - 容器对象**属性赋值**。invokeAwareMethods。

     当执行到此步骤之后，对象的创建和属性的赋值都已完成，那么此时对象是否可以直接拿来使用？理论上可以，但是Spring要考虑扩展性。

   - 获得普通对象，是否需要进行扩展？

     - 执行前置处理方法：BeanPostProcessor

     - 执行初始化方法：invokeInitMethods，在这里会进行判断，检测bean是否实现了InitializingBean接口，如果实现了会调用一个方法afterPropertiesSet方法。

     - 执行后置处理方法：BeanPostProcessor

       其实就是AOP，AOP的底层实现逻辑就是动态代理，动态代理的几种实现方式：jdk，cglib。AOP是IOC整体流程中的一个扩展点。

3. 使用Bean对象

4. 销毁Bean对象：什么时候会导致Bean对象的销毁，关闭容器。



Spring中的bean对象按照使用者可以分为两类：自定义对象和容器对象。容器对象：BeanFactory、ApplicationFactory等。