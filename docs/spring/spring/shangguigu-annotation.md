# Spring注解驱动开发

## 容器相关

### AnnotationConfigApplicationContext

- 基于注解配置的应用上下文。
- 对注解配置进行解析和加载，过程等同于 `ClassPathXmlApplicationContext`。

### @Configuration

- 标记的类会被 `Spring` 认定为配置类，容器启动的时候会加载配置类中配置信息。
- 配置类等同于配置文件，对 Bean 的信息进行配置和注册。
- 配置类也会被当做是组件被注入到 IOC 容器中。

## 组件添加

### @Bean

- 配置类中的方法上标记 `@Bean` 注解，该方法会被认定为是向容器中注册 Bean 的方法。
- Bean 的类型是方法返回值的类型。
- 默认注册到 IOC 容器中的 bean 的 name 是**当前方法的名称**。
- 可以用过 `@Bean` 的 `name` 或者 `value` 属性，指定注入到容器中 Bean 的名称。

### @CompomentScan

- 指定组件的扫描规则，等同于 xml 配置文件中的 `<context:component-scan>` 标签。
- @CompomentScan 注解可以标记在容器**启动类**或者**配置类**上。
- `basePackages` 设定扫描的包。
- `includeFilters` 设定引用过滤器规则。
- `excludeFilters` 设定排除过滤器规则。
- `useDefaultFilters` 设置是否启用默认的过滤器，默认是 **true**。
- 使用 `includeFilters` 情况下需要指定 `useDefaultFilters` 属性为 **false**，禁用默认的过滤器规则。

### @CompomentScans

- 可以用来指定多个 `@CompomentScan`。
- 解决 JDK7 及以下注解不可重复使用的场景下的问题。

### @Filter

- `@Filter` 指定不同类型的过滤器。
- `@Filter` 支持有多种的过滤类型。
  - `FilterType.ANNOTATION` 按照注解过滤。
  - `FilterType.ASSIGNABLE_TYPE` 按照给定的类型过滤。
  - `FilterType.ASPECTJ` 按照 ASPECTJ 表达式过滤。
  - `FilterType.REGEX` 按照正则表达式过滤。
  - `FilterType.CUSTOM` 按照自定义规则进行过滤。

#### TypeFilter

- 用于在配置组件包扫描规则的时候，指定特定的扫描规则的接口。
- 可以通过自定义 `TypeFilter` 指定过滤规则。
- 通过实现 `TypeFilter` 接口和实现 `match` 方法来自定义规则。
  - `MetadataReader` 读取到的当前正在扫描的类的信息。
  - `MetadataReaderFactory` 可以获取到其他任何类信息。

### @Scope

- 通过 @Scope 注解用来设置组件的**作用域**。
- `@Scope` 的效果等同于在 xml 配置文件中的 `<Bean>` 标签中配置的 `scope` 属性。
- 默认情况下组件是单例的，在IOC启动的时候就会创建对象。
- 如果是多例的，会在每次获取对象的时候创建对象。
- `singleton` 单例。
- `prototype` 多例。
- `request` 同一个请求下是一个对象实例。
- `session` 同一个 session下是一个对象实例。

### @Lazy

- 单实例 bean 默认在容器启动的时候创建对象。
- `@Lazy` 是控制单实例Bean的创建时间的，延迟到使用的时候在进行对象创建。

### @Conditional

- 按照一定的条件进行判断，满足条件给容器中注册  bean。
- `Condition` 接口用来实现具体的匹配规则。
- 标记在方法是对当前方法要注入的 bean 进行判断。
- 标记在类上是对当前类的全部方法要注入的 bean 进行判断。

### @Import

- `@Import` 给容器中快速导入一个组件，可以直接指定组件的类名、`ImportSelector`或者 `ImportBeanDefinitionRegistrar`。
- 导入后的 Bean 的 id 是 Bean 的全类名。
- `ImportSelector`接口用于定义要导入类的全类名信息，重写 `selectImports` 方法。
  - `AnnotationMetadata` 当前标注 `@Import` 注解的类的所有注解信息。
  - 方法不要返回 null 值。
- `ImportBeanDefinitionRegistrar` 接口用于定义，`registerBeanDefinitions` 方法。
  - `AnnotationMetadata` 当前类的注解信息。
  - `BeanDefinitionRegistry` BeanDefinition的注册类。
  - 把所有需要添加到容器中的 bean 通过调用 `BeanDefinitionRegistry#registerBeanDefinition`方法手动注册到容器中。

### FactoryBean

- 使用 FactoryBean 注入组件，也叫工厂 Bean，泛型参数代表着当前工厂的所属类型。
- `getObject` 方法用于获取对象。
- `getObjectType` 方法用于获取当前工厂创建的对象的类型。
- `isSingleton` 方法用于返回当前对象是否的单例。
- FactoryBean 通过 `Context#getBean` 方法获取的是调用 `getObject` 创建的对象。
- 通过  `Context#getBean` 方法在参数前缀 `&`，可以获取 `FactoryBean` 本身。

## Bean生命周期管理

### 简介

- Bean 的生命周期是指 从 Bean 创建、初始化、使用中、销毁的整个过程。
- IOC 容器管理 Bean 的生命周期。

### 整体流程

1. 对象创建。
2. set方法赋值。
3. `BeanPostProcessor#postProcessBeforeInitialization`方法。
4. 初始化方法前。
5. 初始化方法。
6. 初始化方法后。
7. `BeanPostProcessor#postProcessAfterInitialization`方法。
8. 销毁。

### 阶段

- 对象构造阶段。
  - 单实例，在容器启动的时候创建对象。
  - 多实例，在每次获取的时候创建对象。
- 初始化阶段。
  - 对象创建完成，并完成属性赋值后，调用初始化方法。
- 销毁阶段。
  - 单例对象在容器关闭的时候被销毁，会调用 Bean 的销毁方法。
  - 多实例对象的销毁不被容器管理。

### 基于 xml 配置指定初始化和销毁方法

- 基于 xml 配置的 Bean 标签下通过 `init-method` 属性指定 Bean 的初始化方法。
- 基于 xml 配置的 Bean 标签下通过 `destory-method` 属性指定 Bean 的销毁方法。

### 基于 @Bean 注解指定初始化和销毁方法

- 通过 `@Bean` 注解中的 `initMethod` 属性指定 Bean 的初始化方法。
- 通过 `@Bean` 注解中的 `destoryMethod` 属性指定 Bean 的销毁方法。

### 通过 InitializingBean 接口指定初始化

- `InitializingBean` 是 Spring 提供的一个接口，其实现类可以在属性赋值结束后被调用 `afterPropertiesSet` 方法进行初始化操作。

### 通过 DisposableBean 接口指定初始化

- `DisposableBean` 是 Spring 提供的一个接口，其实现类可以在单例对象销毁时被调用 `destroy` 方法进行销毁操作。

### 通过 @PostConstruct 进行初始化

- `@PostConstruct` 是 JSR250 中定义的规范。
- 注解标注在方法上，会在 Bean 对象创建完成并且属性赋值完成后被调用，进行一些初始化操作。

### 通过 @PreDestroy 进行销毁

- `@PreDestroy` 是 JSR250 中定义的规范。
- 注解标注在方法上，会在 Bean 对象被销毁之前被调用，进行一些销毁操作。

### 通过 BeanPostProcessor 进行处理操作

#### 简介

- `BeanPostProcessor` 被称为**对象后置处理器**。
- `BeanPostProcessor` 是 Spring 定义的接口，用来在 Bean 的**初始化前后**进行处理操作。
- 
- `postProcessBeforeInitialization` 方法用于对象完成创建和赋值后，**在任何初始化方法被调用之前**进行后置处理工作。
  - `bean` 是容器中创建的对象。
  - `beanName` 是对象在容器中的名称。
  - **返回值是后续要用的Bean实例，可以返回当前对象或者是包装后的对象**。
- `postProcessAfterInitialization` 方法用于在**全部的初始化之执行之后**进行后置处理工作。

#### 原理

- 在容器初始化过程中，Spring Ioc 容器的 refresh 方法被调用，完成的对象的初始化、实例化和依赖注入。
- 首先会加载 Bean 的属性及配置并生成对应的 BeanDefinition 对象。
- 根据 BeanDefinition 和 Bean 的类型，决定是否在 IOC 容器启动的过程中进行对象创建。
- 容器对非懒加载的单例对象进行立即对象的创建和赋值，调用的是 `DefaultListableBeanFactory#populateBean`。
- 完成对象创建和赋值后，调用 `postProcessBeforeInitialization` 进行自定义处理。
  - 获取全部的 `BeanPostProcessor` ，遍历全部的 `postProcessBeforeInitialization` 方法。
  - 如果返回 **null**，不会继续调用后续的 `BeanPostProcessor`。
- 调用 init 方法进行初始化方法。
- 调用 `postProcessAfterInitialization` 进行自定义处理。

#### Spring 的底层应用

- Spring 提供了各种 `XXXAware` 接口，继承自 `Aware` 接口，用于感知和处理各种增强事件。
- Spring 提供了各种 `XxxAwareProcessor` 接口，用于对在容器启动阶段通过 `BeanPostProcessor` 的自定义扩展处理方法对 Bean 的功能增强或者功能扩展。
- Spring 框架底层对 **Bean 赋值**、**组件注入**、**@Autowired**、**生命周期注解**、**@Async** 等功能，都是通过特定的 `BeanPostProcessor` 完成的。

##### ApplicationContextAwareProcessor

- Spring 提供的接口，是应用程序上下文感知处理器。
- 实现此接口的类可以通过实现 `setApplicationContext` 方法获得到 Ioc 容器。

##### BeanValidationPostProcessor

- 对数据属性进行校验的后置处理器。

##### InitDestroyAnnotationBeanPostProcessor

- Spring 容器框架内部的后置处理器。
- 用于处理 `@PostConstruct`、`@PreDestroy` 注解标记的方法，分别在对象初始化后和对象销毁前调用对应的方法。

##### AutowiredAnnotationBeanPostProcessor

- Spring 容器框架内部的后置处理器。
- 用于处理在容器启动过程中通过 `@Autowired` 进行依赖注入的对象关系。

## 组件赋值

### @Value

- 通过 `@Value` 标记在 Bean 对象的属性上，可以完成对属性的赋值操作。
- 可以通过 @Value 进行赋值的途径。
  - 基本数值。
  - SpEL表达式，`#{}`。
  - 配置文件中的值、运行环境变量中的值，`${}`。

### @PropertySource

- 通过 `@PropertySource` 加载外部配置文件的 `key-value`，并保存到运行的环境变量中 。
- `value` 属性用于指定配置文件的路径。
  - `classpath:/com/xxx/app.properties`
  - `file:/path/to/file`
- 通过 `${}` 可以获取到配置文件中的属性。
- 通过 `ConfigurableEnvironment#getProperty` 方法可以获取到配置文件中的属性。

### @PropertySources

- 可以一次性指定多个配置文件地址的注解。
- `value` 属性是 `PropertySource` 数组。

## 自动装配

- Spring 利用依赖注入 `DI`，完成对 IOC 容器中各个组件的依赖关系赋值。

### @Autowired

- 自动注入。
- **默认优先按照类型去容器中找对应的组件**。
  - `ApplicationContext#getBean(xxx.class);`
- **如果找到多个相同类型的组件，再将属性的名称作为组件的 id 去容器中查找。**
  - `ApplicationContext#getBean("xxxName");`
- 默认情况下，如果找不到对应的Bean，不能完成依赖注入，会抛出 `NoSuchBeanException` 的异常。
- 属性 `require` 可以指定 `@Autowired` 注入行为是否必须完成，默认为 `true`。

### @Qualifier

- 使用 `@Qualifier` 指定需要装配的组件的 `id`，不使用属性名称匹配。

### @Primary

- 在进行属性自动装配的时候，**默认使用首选的 Bean。**
- 在 `@Qualifier` 标记的属性下不生效。
- 如果不想注入首选的 Bean，可以使用`@Qualifier` 指定注入 Bean 的 id。

### AOP

xx

### 声明式事务

xx

## 扩展原理

### BeanFactoryPostProcessor

xx

### BeanDefinitionRegistryPostProcessor

xxx

### ApplicationListener

xx

### Spring容器创建过程

xx

## Web

### Servlet3.0

xx

### 异步请求指定初始化和销毁方法