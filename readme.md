# spring-cloud-demo-v1

这是个springcloud的简单demo，由eureka，gateway，config-server, service1,service0组成。

config-server设置为本地存储方式，采用configmap挂载方式

```yaml
server:
  port: 8084
spring:
  application:
    name: config-server
  profiles:
    active: native
  cloud:
    config:
      server:
        native:
          search-locations: file:/config

```

eureka注册中心配置

```yaml
server:
  port: 8761 # 注册中心占用8761端口
eureka:
  instance:
    hostname: eureka-server
  client:
    registerWithEureka: false
    fetchRegistry: false
    serviceUrl:
      defaultZone: http://${eureka.instance.hostname}:${server.port}/eureka/ 
```

各个service从config-server获取配置信息

```yaml
spring:
  application:
    name: service0
  cloud:
    config:
      uri: http://config-server:8084
```

网关配置

```yaml
eureka:
  client:
    serviceUrl:
      defaultZone: http://eureka-server:8761/eureka/
spring:
  application:
    name: gateway
server:
  port: 8083
zuul:
  routes:
    service0: /service/0/**
    service1: /service/1/**
```

通过网关访问service1,调用service0返回对应的结果

```java
@FeignClient(name = "service0", fallbackFactory = Service0FallbackFactory.class)
public interface Service0Client {

    @RequestMapping(method = RequestMethod.GET, path = "user/{userId}/{sleepSec}")
    String test(
            @PathVariable("userId") String userId,
            @PathVariable("sleepSec") int sleepSec
    );

    @RequestMapping(
            method = RequestMethod.GET,
            path = "test"
```

