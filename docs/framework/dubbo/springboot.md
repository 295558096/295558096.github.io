# Dubbo与SpringBoot整合

## 导入依赖

```xml
<dependency>
  <groupId>com.alibaba.boot</groupId>
  <artifactId>dubbo-spring-boot-starter</artifactId>
  <version>0.2.0</version>
</dependency>
```

### 依赖关系

| versions | java | Spring Boot | Dubbo  |
| -------- | ---- | ----------- | ------ |
| 0.2.0    | 1.8+ | 2.0.x       | 2.6.2+ |
| 0.1.1    | 1.7+ | 1.5.x       | 2.6.2+ |

## 编写配置

在`application.properties`或者`application.yml`中编写dubbo相关的配置。

```properties
# dubbo提供者的别名，只是个标识
spring.dubbo.application.name=provider
# zk地址
spring.dubbo.registry.address=zookeeper://192.168.1.160:2181
# dubbo协议
spring.dubbo.protocol.name=dubbo
# duboo端口号
spring.dubbo.protocol.port=20880
# 这是你要发布到dubbo的接口所在包位置
spring.dubbo.scan=test.spring.dubboService
```

## 开启Dubbo

在主程序上标记`@EnableDubbo`注解开启基于注解的Dubbo功能。

## 暴露服务

在服务提供者的类上标记`@Service`注解，注解引用` com.alibaba.dubbo.config.annotation.Service`。

## 远程引用

在消费者的引用类处标记注解`@Reference`注解，注解引用`com.alibaba.dubbo.config.annotation.Reference`。