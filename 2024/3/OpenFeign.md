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

