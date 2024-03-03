## 简单介绍 

OpenFeign是一个声明式的http客户端，是SpringCloud在Eureka公司开源的Feign基础上改造而来。

原始代码

```java
// 2.1 根据服务的名称获取服务的实例列表
List<ServiceInstance> instances = discoverClient.getInstances("item-service");
if (CollUtil.isEmpty((instances))) {
return;
}
// 2.2 手写负载均衡，从实例列表中获取实例
ServiceInstance instance = instances.get(RandomUtil.randomInt(instances.size()));
// List<ItemDTO> items = itemService.queryItemByIds(itemIds);
// 利用RestTemplate发起http请求，得到http的响应。

ResponseEntity<List<ItemDTO>> response = restTemplate.exchange(
    instance.getUri() + "/items?ids={ids}",
    HttpMethod.GET,
    null,
    new ParameterizedTypeReference<List<ItemDTO>>() {
    },
    Map.of("ids", CollUtil.join(itemIds, ","))
);

// 2.2 解析响应
if (response.getStatusCode().is2xxSuccessful()) {
    // 查询失败，直接结束。
    return;
}
```

OpenFeign的编写方法大概如下：

```java
@FeignClient(value = "item-service")
public interface ItemClient {
	@GetMapping("/items")
	List<ItemDTO> queryItemByIds(@RequestParam("ids") Collection<Long> ids);
}
```

FeignClient的远程调用大致如下：

```java
List<ItemDTO> items = itemClient.queryItemByIds(List.of(1, 2, 3));
```



## 连接池

OpenFeign的底层是如何做到的？对于创建的client，一开始拿到的对象是一个代理对象，底层都是由InvocationHandler接口实现的。其实现的基本路线如下：

1. 构建请求参数。

2. 构建request对象，能够拿到请求的名称，如`item-service`，而不是请求的ip端口。

3. 通过excute方法对请求的路径进行处理，通过已经拿到的服务id，然后拉取相应的实例的列表。

   ```java
   loadBalancerClient.choose(serviceId, lbRequest);
   ```

4. 拿到真实的ip地址之后通过**delegate**（委托，是一个Client，Client是一个接口，专门用于发送Http请求，底层采用的是HttpURLConnection，效率较低）来发送请求，是一种代理的方式。

Client每一次都需要重新创建连接，效率较低，因此我们可以通过**连接池**为其进行相应的优化。

- HttpURLConnection：默认实现，不支持连接池
- Apache HttpClient：支持连接池
- OKHttp：支持连接池

OpenFeign整合OKHttp的步骤：

1. 引入依赖

   ```yaml
   <!--ok-http-->
   <dependency>
   	<groupId>io.github.openfeign</groupId>
   	<artifactId>feign-okhttp</artifactId>
   </dependency>
   ```

2. 开启连接池功能

   ```
   feign:
     okhttp:
       enabled: true # 开启OKHttp连接支持
   ```

在开启之后，其底层的**delegate**会变成**OKHttpClient**。

