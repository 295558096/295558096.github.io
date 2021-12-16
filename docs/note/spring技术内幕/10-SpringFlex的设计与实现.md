# Spring Flex的设计与实现

## Spring Flex模块的应用场景

- Spring Flex模块，也被称作`Spring BlazeDS`。
- Spring Flex 的作用是把Flex客户端的开发和Spring后端这两种异构技术集成起来（Adobe的Flex和Java）。

### Flex

- Flex是Adobe公司的产品。
- 通过Flex，可以设计出表现力很强的软件产品。
- Flex是一个高效、免费的开放源框架，利用它可构建出具有表现力的移动、网络和桌面应用程序。
- Flex允许构建共享一个公共代码库的网络和移动应用程序，从而减少应用程序创建的成本。

### BlazeDS

- BlazeDS是Adobe公司的开源产品，为客户端和Java服务器端提供数据通信和消息服务。

## Spring Flex的应用过程

### 配置Spring MVC的拦截器

```xml
<servlet>
  <servlet-name>Spring MVC Dispatcher Servlet</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet
  </servlet-class>
  <init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/config/web-application-config.xml</param-value>
  </init-param>
  <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
  <servlet-name>Spring MVC Dispatcher Servlet</servlet-name>
  <url-pattern>/messagebroker/*</url-pattern>
</servlet-mapping>
```

### SpringFlex的Bean配置

```java
<bean id="_messageBroker" class="org.springframework.flex.core.MessageBrokerFactoryBean">
  <property name="servicesConfigPath" value="classpath*:services-config.xml"/>
</bean>
```



## Spring Flex的设计与实现

略