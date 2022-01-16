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

### 总结

- `@RequestMapping` 指定映射的请求路径。
- `web.xml`不指定配置文件位置，默认去`/WEB-INF/xxx-servlet.xml`找配置，`xxx`表示配置的Servlet的名称。
- 前端控制的配置问题
  - 服务器的全局`web.xml`中有一个 `DefaultServlet` 是 `url-pattern=/`。
  - SpringMVC配置的前端控制器也是`url-pattern=/`，会禁用掉Tomcat中的DefaultServlet。
  - DefaultServlet是服务于各种静态资源，除了JSP和Servlet之外，都属于静态资源。
  - `/*`会拦截所有的请求，DispatcherServlet配置的过程中，使用`/`，也是为了可能兼容Rest风格的请求。

## RequestMapping 注解

### 简介

- Spring MVC 使用 `@RequestMapping` 注解为控制器指定可以处理哪些URL请求。
- 可以标记在控制器的类或者方法上。
  - `类` 提供初步的请求映射信息。相对于 WEB 应用的根目录。
  - `方法` 提供进一步的细分映射信息。相对于类定义处的URL。
- `DispatcherServlet` 截获请求后，就通过控制器上`RequestMapping`提供的映射信息确定请求所对应的处理方法。
- 

### 注解参数

#### value

- 指定请求路径。
- 默认可以不写。

#### method

- 指定限制具体的请求类型。
- 如果不指定，默认是POST、GET均可。

#### params

- 规定请求参数。
- 支持简单的表达式，`=`、`!=`...。

#### headers

- 规定请求头参数。
- 支持简单的表达式。

#### consumes

- 指定可以接受请求的**内容类型**。
- 规定请求头中的`Content-Type`。

#### produces

- 告诉浏览器返回的内容类型是什么。
- 相当于给响应头加上`Content-Type`。

### 模糊匹配

>URL地址可以使用模糊的通配符进行定义。

#### 通配符

- `?` 替代任意一个字符，0个多个都不行。
- `*` 能替代任意多少个字符或者一层路径。
- `**` 能替代任意多少个字符或者多层路径。

#### 规则

- 精确和模糊都匹配的情况下，精确优先。 



## PathVariable注解

>路径占位符

### 简介

- 带**占位符**的 URL 是 Spring3.0 新增的功能。
- 支持占位符请求是 SpringMVC 向 `REST` 目标挺进发展过程中具有里程碑的意义。
- 通过 `@PathVariable` 可以**将 URL 中占位符参数绑定到控制器处理方法的入参中**。
  - URL中指定`{xxx}`。
  - 参数中使用`@PathVariable("xxx")`获取参数值。
- 路径占位符只能占一层路径。

### HiddenHttpMethodFilter

>对PUT、DELETE请求的支持。

### 简介

- `SpringMVC` 中有一个 Filter，`HiddenHttpMethodFilter`，可以**把普通的请求转化为规定形式的请求**。

- 在 `web.xml` 中配置并开启 HiddenHttpMethodFilter。
- 配置拦截的路径是 `/*`。
- post请求中携带一个参数`_method`，值为目标请求类型。
- 高版本的Tomcat（大于8.0）处理JSP页面的PUT、DELETE请求遇到405报错的情况下，在JSP页面上配置`isErrorPage=true`。

### 实现原理

- 获取表单中`_method`的值。
- 如果不是POST请求，直接放行。
- 如果是是POST，并且`_method`的参数值合法，将参数值转大写。
- 包装请求为指定类型的请求重写当前请求的`getMethod()`方法，`HttpMethodRequestWrapper`，并放行。
