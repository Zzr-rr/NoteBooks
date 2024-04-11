## 1、自我介绍；做过的项目中有什么印象深刻的地方？

在我实现的Rpc框架中，服务端需要去注册服务，我希望通过注解的方式来自动完成对服务的注册，于是我就寻找相应的一些解决方案，最后通过目录扫描的方式完成对目录的扫描，拿到相应的类，通过获取到的类拿到了注解信息，通过注解信息来完成了相应类的注册，方案实现以后，我意识到了框架能够做的事情其实是非常多的，一个好的框架就是要让使用者的操作在安全的基础上变得更加简单便捷。

## 2、springboot相较于spring有哪些好处

简化配置：SpringBoot省去了Spring应用所需要的XML配置，提高了开发的生态系统。

提供内嵌服务器：SpringBoot提供了一些内嵌服务器，比如Tomcat、Jetty和Undertow服务器；避免了独立打成war包的繁琐操作。

测试方便：SpringBoot提供了集成测试和单元测试的方法，能够自动化的进行测试。

微服务支持：SpringBoot和SpringCloud配合的很好，非常便于开发微服务的相关应用。

## 3、springboot常用注解

首先是三层架构，@Controller，@Service，@Repository，它们都和@Component相关，@Component将对象标注为Spring的一个Bean，在后面依赖注入的时候会扫描这些对象。

其次依赖注入中最常用的注解是@Autowired，用于自动注入对象。

服务常用的注解有@RestController，是@Controller和@ResponseBody注解的组合，用于创建Restful Web服务。

对于路径请求@RequestMapping, @GetMapping, @PostMapping等等，用于将Http请求映射到特定的处理器上。

对于请求的参数可以使用@PathVariable,@RequestParam来将请求的参数进行映射。

## 4、springboot如何实现自动配置的？你自己实现过xxxx-starter吗？怎么用？直接引入jar包吗？

SpringBoot的自动配置是通过@EnableAutoConfiguration注解来实现的。@EnableAutoConfiguration会告诉SpringBoot基于项目中的jar依赖来猜测如何配置Spring。

如果项目中包含了spring-boot-starter-web，Spring会自动配置项目支持Web功能，比如说设置一个DispatcherServlet。

1. @EnableAutoConfiguration会通过`@Import({AutoConfigurationImportSelector.class})`来根据classpath以及定义的bean来自动导入配置。
2. `EnableAutoConfigurationImportSelector`会加载`META-INF/spring.factories`文件，这个文件位于Spring Boot的各个自动配置模块内，会列出可用的自动配置类。
3. SpringBoot会读取这些配置类，根据条件注释决定是否启用自动配置。

**实现一个starter的基本步骤：**

1. 编写自动配置类，使用Conditional相关注解实现条件化配置。
2. 配置`spring.factories`，通常是在`resouce/META-INF/spring.factories`文件下自动配置类。
3. 打包成一个jar包，进行发布。

**使用就通过如下的方式进行使用：**

```
<dependency>
    <groupId>your.group.id</groupId>
    <artifactId>my-custom-starter</artifactId>
    <version>your.version</version>
</dependency>
```



## 5、spirngboot什么场景下注解会失效？

Maven仓库冲突、未引入Maven依赖、条件注解不满足、包扫描路径错误、Bean覆盖问题、异步处理问题。

## 6、@transactional注解什么情况下会失效？

1. @transactional修饰的方法为非public方法的时候，事务的注解会失效。
2. 在类内部调用类内部@Transactional标注的方法。
3. 事务内部捕捉了异常，没有抛出新的异常。

## 7、java并发包下都用过什么？

线程安全的数据结构

ConcurrentHashMap，ConcurrentLinkedQueue、CopyOnWriteArrayList

线程池

CachedThreadPool、ScheduledThreadPool

锁

ReentrantLock、ReadWriteLock

原子操作

AtomicInteger

## 8、项目中用过volatile关键字吗？怎么用的？

volatile关键字声明的变量可以只放在主内存中而不是线程的本地缓存中，确保所有的线程都能**看到共享变量的最新值**。

可以用作线程之间的信号，确保线程的可见性等等。

## 9、Mybatis只有接口没有实现类，为什么能调用去操作数据库？

这是通过Mybatis的核心组件——SqlSession和Mapper接口实现的。当配置Mybatis应用程序时后，需要指定映射器接口和对应的XML文件或注解。Mybatis使用这些信息来创建每个映射器的动态代理对象。

## 10、Mysql一条sql语句，select * from t where a in (),b in () and c in ();走(a,b),(b,c),(c,a)哪个索引？为什么？

不确定，主要取决于

- 查询中 `a`, `b`, `c` 列的 `IN` 子句的选择性如何。
- 这些列在相应索引中的顺序。
- 是否存在可以作为覆盖索引的索引。

## 11、java项目上线后cpu占用率100%，可能是因为什么？如何排查？

可能出现了死锁，高并发处理不当，内存泄漏导致频繁的垃圾回收。

可以通过top指令查看哪个进程对CPU的占用最大，找到相应的进程之后对其进行进一步排查。

可以用jProfiler来对Java代码进行进一步的定位。

## 12、有25枚硬币，24枚真的，1枚假的，假的比真的轻，一个天枰，怎么以最快的速度找到假硬币？

4次。二分

## 13、有10瓶药，每瓶有10粒药，其中有一瓶是变质的。好药每颗重1克，变质的药每颗比好药重0.1克。问怎样用天秤称一次找出变质的那瓶药。

第一瓶1颗，第二瓶2颗，第三瓶3颗...



## 14、手撕代码：给一个二维矩阵字符，一个目标字符串，每个字符只能用一次，只能上下左右移动，问能不能构成目标字符串?

dfs。

dfs维护目前路径的字符串，或者从起始点开始维护，如果走到不一样就返回。