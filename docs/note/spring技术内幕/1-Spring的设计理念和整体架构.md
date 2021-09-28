# Spring的设计理念和整体架构

## 主要子项目

- `Spring Framework（Core）` Spring项目的核心，包含了Ioc容器的设计以及AOP功能，包含了其他Spring的基本模块，比如MVC、JDBC、事务处理模块的实现。
- `Spring Web Flow` 一个建立在Spring MVC基础上的Web工作流引擎。
- `Spring BlazeDS Integration` 提供Spring与Adobe Flex技术集成的模块。
- `Spring Security` 广泛使用的基于Spring的认证和安全工具。
- `Spring Security OAuth` OAuth在Spring上的集成提供支持。
- `Spring Dynamic Modules` 可以让Spring应用运行在OSGi的平台上。
- `Spring Batch` 提供构建批处理应用和自动化操作的框架。
- `Spring Integration` 体现了“企业集成模式”的具体实现，并为企业的数据集成提供解决方案。
- `Spring AMQP` 为Spring应用更好地使用基于AMQP（高级消息队列协议）的消息服务而开发的。
- `Spring.NET` Spring在.NET环境中的移植。
- `Spring Android` 为Android终端开发应用提供Spring的支持。
- `Spring Mobile` Spring Mobile和Spring Android不同，它能使工作在Spring传统的服务器端完成。
- `Spring Social` 是Spring框架的扩展，可以帮助Spring应用更方便SNS(Social Network Service)。
- `Spring Data` 为Spring应用提供使用非关系型数据的能力。

## 设计目标

Spring为开发者提供的是一个一站式的轻量级应用开发框架（平台）。抽象并解决开发应用中常见的共性问题。

## 整体架构

### Spring IoC

包含了最为基本的IoC容器BeanFactory的接口与实现。

Spring IoC容器还提供了一个容器系列。

国际化的消息源和应用支持事件这些特性，也都在这个模块中配合IoC容器来实现，这些功能围绕着IoC基本容器和应用上下文的实现，构成了整个Spring IoC模块设计的主要内容。

### Spring AOP

Spring的核心模块，围绕着AOP的增强功能，Spring集成了AspectJ作为AOP的一个特定实现，同时还在JVM动态代理/CGLIB的基础上，实现了一个AOP框架，作为Spring集成其他模块的工具。

### Spring MVC

这个模块以DispatcherServlet为核心，实现了MVC模式，包括怎样与Web容器环境的集成，Web请求的拦截、分发、处理和ModelAndView数据的返回，以及如何集成各种UI视图展现和数据表现，如PDF、Excel等，通过这个模块，可以完成Web的前端设计。

### Spring JDBC/Spring ORM

Spring对JDBC做了一层封装，使通过JDBC完成的对数据库的操作更加简洁。

Spring JDBC包提供了JdbcTemplate作为模板类，封装了基本的数据库操作方法，如数据的查询、更新等。

SpringJDBC还提供了RDBMS的操作对象，这些操作对象可以使应用以更面向对象的方法来使用JDBC。

Spring还提供了许多对ORM工具的封装，这些封装包括了常用的ORM工具，如Hibernate、iBatis等，这一层封装的作用是让应用更方便地使用这些ORM工具，而不是替代这些ORM工具。

### Spring事务处理

Spring事务处理是一个通过Spring AOP实现自身功能增强的典型模块。

在这个模块中，Spring把在企业应用开发中事务处理的主要过程抽象出来，并且简洁地通过AOP的切面增强实现了声明式事务处理的功能。

### Spring远端调用

通过Spring的封装，为应用屏蔽了各种通信和调用细节的实现。

### Spring应用

这个模块不属于Spring的范围。这部分的应用支持，往往来自一些使用得非常广泛的Spring子项目，或者该子项目本身就可以看成是一个独立的Spring应用。

## 应用场景

- 最上层的Web UI。
- 底层的数据操作。
- 其他企业信息数据的集成。
- 各种J2EE服务的使用。

特点

- Spring是一个非侵入性（non-invasive）框架，其目标是使应用程序代码对框架的依赖最小化，应用代码可以在没有Spring或者其他容器的情况下运行。
- Spring提供了一个一致的编程模型，使应用直接使用POJO开发，从而可以与运行环境（如应用服务器）隔离开来。
- Spring推动应用的设计风格向面向对象及面向接口编程转变，提高了代码的重用性和可测试性。
- Spring改进了体系结构的选择，虽然作为应用平台，Spring可以帮助我们选择不同的技术实现。