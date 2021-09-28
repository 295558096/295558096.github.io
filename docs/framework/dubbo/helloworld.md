# Hello，Dubbo

## 环境信息

- jdk1.8
- springboot 2.1.13
- dubbo 2.6.2
- curator 2.10

## 注册中心

使用Dubbo默认推荐的注册中心Zookeeper。

## 推荐的工程方案

将接口和相关的DTO或者Entity都封装在公共api的jar包内，服务消费者和服务提供者同时引用此jar包。

提供者实现相关接口，并暴露接口。

消费者订阅接口。

## 基于Xml的配置

### 标签简介

| 标签                  | 用途         | 解释                                                         |
| --------------------- | ------------ | ------------------------------------------------------------ |
| \<dubbo:service/>     | 服务配置     | 用于暴露一个服务，定义服务的元信息，一个服务可以用多个协议暴露，一个服务也可以注册到多个注册中心。 |
| \<dubbo:reference/>   | 引用配置     | 用于创建一个远程服务代理，一个引用可以指向多个注册中心。     |
| \<dubbo:protocol/>    | 协议配置     | 用于配置提供服务的协议信息，协议由提供方指定，消费方被动接受。 |
| \<dubbo:application/> | 应用配置     | 用于配置当前应用信息，不管该应用是提供者还是消费者。         |
| \<dubbo:module/>      | 模块配置     | 用于配置当前模块信息，可选。                                 |
| \<dubbo:registry/>    | 注册中心配置 | 用于配置连接注册中心相关信息。                               |
| \<dubbo:monitor/>     | 监控中心配置 | 用于配置连接监控中心相关信息，可选                           |
| \<dubbo:provider/>    | 提供方配置   | 当 ProtocolConfig 和 ServiceConfig 某属性没有配置时，采用此缺省值，可选。 |
| \<dubbo:consumer/>    | 消费方配置   | 当 ReferenceConfig 某属性没有配置时，采用此缺省值，可选。    |
| \<dubbo:method/>      | 方法配置     | 用于 ServiceConfig 和 ReferenceConfig 指定方法级的配置信息。 |
| \<dubbo:argument/>    | 参数配置     | 用于指定方法参数配置。                                       |

### 配置项详解

- `<dubbo:application/>`

  ```xml
  <dubbo:application name="hello-world-app" />
  ```

  用于指定应用名，这里需要保证应用名唯一，这个应用名在后续的**console admin**中可以在列表中显示，方便管理。

- `<dubbo:registry/>`

  ```xml
  <dubbo:registry address="multicast://172.18.18.18:2333"/>
  ```

  注册中心配置，和服务发现的具体机制有关系。可以是`zookeeper`地址，也可以`eureka`地址，也可以是广播地址。

- `<dubbo:protocol/>`

  ```xml
  <dubbo:protocol name="dubbo" port="20880"/>
  ```

  定义传输协议和默认端口。

## 服务提供者

### 实现接口

```java
public class DemoServiceImpl implements DemoService {
    private static final Logger logger = LoggerFactory.getLogger(DemoServiceImpl.class);

    @Override
    public HelloReply sayHello(HelloRequest request) {
        logger.info("Hello " + request.getName() + ", request from consumer: " + RpcContext.getContext().getRemoteAddress());
        return HelloReply.newBuilder()
                .setMessage("Hello " + request.getName() + ", response from provider: "
                        + RpcContext.getContext().getLocalAddress())
                .build();
    }

    @Override
    public CompletableFuture<HelloReply> sayHelloAsync(HelloRequest request) {
        return CompletableFuture.completedFuture(sayHello(request));
    }
}
```



### 暴露服务(XML)

```xml
<bean id="demoServiceImpl" class="org.apache.dubbo.demo.provider.DemoServiceImpl"/>
<dubbo:service serialization="protobuf" interface="org.apache.dubbo.demo.DemoService" ref="demoServiceImpl"/>
```

### 

## 服务消费者

xxx

