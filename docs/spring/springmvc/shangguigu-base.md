# Spring MVC

## 概述

- Spring 为**展现层**提供的基于 MVC 设计理念的优秀的 web 框架，是目前最主流的 MVC 框架之一。

- `Spring 3.0` 后全面超越 `Struts2`， 成为最优秀的 MVC 框架。
- Spring MVC 通过一套 **MVC 注解**，让 POJO 成为处理请求的控制器，而无须实现任何接口。
- **支持 REST 风格的 URL 请求**。
- 采用了**松散耦合可插拔组件结构**，比其他 MVC 框架更具扩展性和灵活性。

## Spring 的 MVC 实现

![image-20220111220241817](image/image-20220111220241817.png)

## HelloWorld

### web项目版本

- Web 2.5 版本会自动创建web.xml。
- Web 3.0 需要单独配置。
- Web 三大组件
  - Servlet
  - Filter
  - Linstener


### 流程

- 导包
- 写配置
  - SpringMVC需要配置一个前端控制器，`DispatcherServlet`，在`web.xml`中配置。
  - `load-on-startup` 配置启动顺序，否则 Servlet 默认是在第一次访问的时候创建对象，值越小，优先级越高。
  - `contextConfigurationLocation` 指定配置文件位置。
  - `servlet-mapping` 配置 Servlet拦截的请求表达式，`/`。
    - `/` 和 `/*`都拦截所有的请求。
    - `/*`的范围会更大，还会拦截到*.jps请求。 
  - `spring-mvc.xml` 配置文件。
    - 指定扫描的控制器组件的包范围。
    - 配置**视图解析器** `InternalResourceResolver` 进行视图解析。
- 编写代码
  - 编写控制器`Controller`。
  - 编写具有映射请求地址的方法。
- 测试

## 运行流程分析

- 
