## 前言

学习Java微服务时使用RestTemplate，当时对RestTemplate进行注入时@Autowired下方出现波浪线，其原因是目前IDEA和Spring均不再推荐@Autowired进行依赖注入。



**主要原因**

使用@Autowired注解的代码会难以理解和分析。会让开发者对于对象的依赖出处产生疑惑。

对于测试的覆盖性不足，由于对象是自动注入的，测试的时候很难进行模拟和替换，测试的广度不足。增加单元测试的复杂性。

**解决方案**

1. 构造函数依赖注入

   ```java
   @Service
   public class Service{
   	private final RestTemplate restTemplate;
   	public Service(RestTemplate restTemplate){
   		this.restTemplate = restTemplate;
   	}
       // ...
   }
   ```

2. Setter方法注入，在需要使用的方法中执行Setter方法。动态地设置依赖的实例。

3. 对于这些注入方式，都可以使用lombok中的@RequiredArgsConstructor注解来简化代码的开发，但变量都需要加上final关键字。

   

