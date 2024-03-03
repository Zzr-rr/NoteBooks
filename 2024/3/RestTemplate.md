## 简单介绍

微服务架构中，各个服务之间可能会存在相互调用的关系，为了解决这一类问题，可以通过网络请求来调用其它服务的接口，为此可以其它业务提供的服务。

RestTemplate是Spring为我们提供的一个工具，可以方便的实现Http请求的发送。

使用步骤如下：

1. 注入RestTemplate到Spring容器

   ```java
   @Bean
   public RestTemplate restTemplate(){
   	return new RestTemplate();
   }
   ```

2. 发起远程调用

   ```java
   public <T> ResponseEntity<T> exchange(
   		String url, // 请求路径
   		HttpMethod method, // 请求方式，HttpMethod 是一个枚举
   		@Nullable HttpEntity<?> requestEntity, // 请求实体，可以为空
   		Class<T> responseType, // 返回值类型 User.class 帮我们做解析
   		Map<String, ?> uriVariables // 请求参数 指定请求url中的参数
   )
   ```

   

## 实际使用

在启动类中通过SpringBoot的创建一个Bean对象，方便后续通过依赖注入进行使用。

```Java
@MapperScan("com.hmall.cart.mapper")
@SpringBootApplication
public class CartApplication {
    public static void main(String[] args) {
        SpringApplication.run(CartApplication.class, args);
    }

    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}
```

Spring不再推荐使用常用的@Autowired来进行依赖注入，而是通过写一个构造函数来注入，而构造函数可以通过lombok来简化。

```java
@RequiredArgsConstructor
public class CartServiceImpl extends ServiceImpl<CartMapper, Cart> implements ICartService {
    private final RestTemplate restTemplate;
}
```



请求构造方式：

```java
ResponseEntity<List<ItemDTO>> response = restTemplate.exchange(
    "http://localhost:8081/items?ids={ids}",
    HttpMethod.GET,
    null,
    new ParameterizedTypeReference<List<ItemDTO>>() {
    },
    Map.of("ids", CollUtil.join(itemIds, ","))
);
```

字节码里是没有泛型的，泛型会被擦除。因此如果再指定需要返回的类型时想要指定`List<ItemDTO>.class`是不可行的。

解决方案是使用参数类型的引用，虽然字节码的泛型会被擦除，但是new出来的对象其泛型仍然是存在的。

```java
new ParameterizedTypeReference<List<ItemDTO>>() {
}
```



## 远程服务调用时存在的问题

一个服务可能部署多份，多个实例同时启动（如同一个`item-service`占用端口`8081`，`8082`，`8083`等），实现负载均衡；在实现代码时请求地址如果是写死的，那么就不能实现负载均衡的效果。

这种问题的解决统一称之为**服务治理**。

需要使用到**注册中心**



## 注册中心原理

服务调用者(`cart-service`)和服务提供者(`item-service`)。

注册中心：1.注册服务信息，ip是什么，端口是什么，你能提供什么服务。2.注册表：记录服务信息。3.订阅item-service的信息。

所有服务不仅仅是需要注册到注册中心，还需要一个心跳续约，隔一段时间向注册中心报告服务仍可使用，超时后会将该服务从服务列表中剔除掉，同时会给服务调用者推送变更。

![download_image](imgs/download_image.jpeg)



Nacos是目前国内企业中占比最多的注册组件之一。