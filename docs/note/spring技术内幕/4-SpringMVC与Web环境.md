# Spring MVC与Web环境

## Spring MVC概述

- Spring 为展现层提供的基于 MVC 设计理念的优秀的Web 框架，是目前最主流的 MVC 框架之一。
- Spring 3.0 后全面超越 Struts2，成为最优秀的 MVC 框架。
- Spring MVC 通过一套 MVC 注解，让 POJO 成为处理请求的控制器，而无须实现任何接口。
- 支持 REST 风格的 URL 请求。
- 采用了松散耦合可插拔组件结构，比其他 MVC 框架更具扩展性和灵活性。

### MVC模式

**模型**

- 封装应用数据状态。
- 响应数据状态查询。
- 提供应用功能接口。
- 数据状态变化通知视图。

**控制器**

- 定义应用的功能。
- 映射用户动作到数据更新。
- 选择对应的视图。
- 一个控制器对应一组功能。

**视图**

- 通过视图展现应用数据。
- 向应用数据提交更新请求。
- 向控制器提交用户动作。
- 运行控制器选择不同的视图。



![1027054-20180521151956650-1028459743](image/1027054-20180521151956650-1028459743.png)

### Spring MVC 基本使用要点

- 在`web.xml`中配置`DispatcherServlet`，前端控制器。
- 在Bean定义中配置Web请求和`Controller`的对应关系。
- 配置各种视图的展现方式。
- 使用`Controller`的时候，会看到`ModelAndView`数据的生成，还会看到把`ModelAndView`数据交给相应的`View`来进行呈现。

## Web环境中的Spring MVC

Spring IoC是一个独立的模块，它并不是直接在Web容器中发挥作用的，如果要在Web环境中使用IoC容器，需要Spring为IoC设计一个启动过程，把IoC容器导入，并在Web容器中建立起来。

### Web环境下启动IoC容器

所谓的启动过程，就是在Web环容器的启动的时候完成IoC容器的启动。

- 启动Web容器环境。
- 使用特定的拦截器，将 IoC容器载入到Web环境中。
- 完成IoC容器初始化。

#### Tomcat的web.xml

- 首先定义了一个Servlet对象，它是Spring MVC的`DispatcherServlet`，是MVC中很重要的一个类，起着分发请求的作用。
- 为`DispatcherServlet`定义了**对应的URL映射**，这些URL映射为这个Servlet**指定了需要处理的HTTP请求**。
- `context-param`参数的配置用来指定Spring IoC容器读取**Bean定义**的XML文件的路径。
- 作为Spring MVC的启动类，`ContextLoaderListener`被定义为一个**监听器**，这个监听器是**与Web服务器的生命周期相关联的**，由`ContextLoaderListener`监听器负责**完成IoC容器在Web环境中的启动工作**。
- `DispatchServlet`和`ContextLoaderListener`提供了**在Web容器中对Spring的接口**，这些接口与Web容器耦合是通过`ServletContext`来实现的。
- `ServletContext`为Spring的IoC容器提供了一个**宿主环境**，在宿主环境中，Spring MVC建立起一个IoC容器的体系。
- IoC容器体系是通过`ContextLoaderListener`的初始化来建立的，在建立IoC容器体系后，把`DispatchServlet`作为Spring MVC处理**Web请求的转发器建立起来，从而完成响应HTTP请求的准备**。
- 

```xml
<!-- 配置 DispatcherServlet -->
<servlet>
  <servlet-name>dispatcherServlet</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <!-- 配置 DispatcherServlet 的一个初始化参数: 配置 SpringMVC 配置文件的位置和名称 -->
  <!-- 
            实际上也可以不通过 contextConfigLocation 来配置 SpringMVC 的配置文件, 而使用默认的.
            默认的配置文件为: /WEB-INF/<servlet-name>-servlet.xml
        -->
  <init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:springmvc.xml</param-value>
  </init-param>
  <load-on-startup>1</load-on-startup>
</servlet>

<!--前端控制器拦截的请求-->
<servlet-mapping>
  <servlet-name>dispatcherServlet</servlet-name>
  <url-pattern>/*</url-pattern>
</servlet-mapping>
<listener>
  <listener-class>
    org.springframework.web.context.ContextLoaderListener
  </listener-class>
</listener>
...
```

## 上下文在Web容器中的启动

### IoC容器启动的基本过程

>IoC容器的启动过程就是建立上下文的过程，该上下文是与ServletContext相伴而生的，同时也是IoC容器在Web应用环境中的具体表现之一。

- 由`ContextLoaderListener`启动的上下文为**根上下文**。
- 在根上下文的基础上，还有一个与Web MVC相关的上下文用来保存控制器`DispatcherServlet`需要的MVC对象，作为根上下文的**子上下文**，构成一个层次化的上下文体系。
- `ContextLoaderListener`是由Spring提供的用于在Web容器中建立IoC容器服务的类，实现了`ServletContextListener`接口。
  - `ServletContextListener`是在`Servlet API`中定义的，提供了**与Servlet生命周期结合的回调**，比如`contextInitialized`方法和`contextDestroyed`方法。
- 在`ContextLoader`中，完成了两个IoC容器建立的基本过程，一个是在Web容器中建立起**双亲IoC容器**，另一个是生成相应的`WebApplicationContext`并将其**初始化**。


```mermaid
sequenceDiagram
participant web as Web Contatiner
participant cll as ContextLoaderListener
participant cl as ContextLoader
participant xwac as XmlWebApplicationContext
web ->> cll: contextInitialized()
cll ->> cl: initWebApplicationContext()
cl ->> cl: loadParentContext()
cl ->> xwac: createWebApplicationContext()
xwac ->> xwac : refresh()
xwac -->> cll : WebApplicationContext
```

### Web容器中的上下文设计

- 为了方便在Web环境中使用IoC容器，Spring为Web应用提供了上下文的扩展接口`WebApplicationContext`来满足启动过程的需要。
- `WebApplicationContext`定义了一个`getServletContext`方法，通过这个方法可以得到当前Web容器的Servlet上下文环境，通过这个方法，相当于提供了一个**Web容器级别的全局环境**。
- **在启动过程中，Spring会使用一个默认的WebApplicationContext实现作为IoC容器**。
- `XmlWebApplicationContext`是从`ApplicationContext`继承下来的，在基本的ApplicationContext功能的基础上，**增加了对Web环境和XML配置定义的处理**。

#### WebApplicationContex结构层次

![ApplicationContext](image/ApplicationContext.png)

#### WebApplicationContext接口源码

```java
public interface WebApplicationContext extends ApplicationContext {

  // 定义用于在ServletContext中存取根上下文的常量
	String ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE = WebApplicationContext.class.getName() + ".ROOT";

	String SCOPE_REQUEST = "request";

	String SCOPE_SESSION = "session";

	String SCOPE_APPLICATION = "application";

	String SERVLET_CONTEXT_BEAN_NAME = "servletContext";

	String CONTEXT_PARAMETERS_BEAN_NAME = "contextParameters";

	String CONTEXT_ATTRIBUTES_BEAN_NAME = "contextAttributes";

	@Nullable
  // 获取web容器的ServletContext的方法。
	ServletContext getServletContext();

}
```

#### XmlWebApplicationContext源码

```java
public class XmlWebApplicationContext extends AbstractRefreshableWebApplicationContext {

	// 默认根容器的配置文件路径
	public static final String DEFAULT_CONFIG_LOCATION = "/WEB-INF/applicationContext.xml";

	// 默认的配置文件位置在/WEB-INF/目录下
	public static final String DEFAULT_CONFIG_LOCATION_PREFIX = "/WEB-INF/";

	// 默认的配置文件后缀名.xml文件
	public static final String DEFAULT_CONFIG_LOCATION_SUFFIX = ".xml";

	@Override
	protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {
		// 定义XmlBean定义解析器
		XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);
		// 设置环境变量
		beanDefinitionReader.setEnvironment(getEnvironment());
    // 设置资源加载器
		beanDefinitionReader.setResourceLoader(this);
    // 设置对象解析器
		beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));
		// 初始化Bean定义解析器
		initBeanDefinitionReader(beanDefinitionReader);
    // 加载Bean定义
		loadBeanDefinitions(beanDefinitionReader);
	}

	protected void initBeanDefinitionReader(XmlBeanDefinitionReader beanDefinitionReader) {
	}

	protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws IOException {
		String[] configLocations = getConfigLocations();
		if (configLocations != null) {
			for (String configLocation : configLocations) {
				reader.loadBeanDefinitions(configLocation);
			}
		}
	}

	@Override
	protected String[] getDefaultConfigLocations() {
		if (getNamespace() != null) {
			return new String[] {DEFAULT_CONFIG_LOCATION_PREFIX + getNamespace() + DEFAULT_CONFIG_LOCATION_SUFFIX};
		}
		else {
			return new String[] {DEFAULT_CONFIG_LOCATION};
		}
	}

}
```

### ContextLoader的设计与实现

- `ContextLoaderListener`通过使用`ContextLoader`来完成实际的`WebApplicationContext`初始化工作。
- `ContextLoader`就像Spring应用程序在Web容器中的启动器。

#### 启动过程

1. 从Servlet事件中得到`ServletContext`。

2. 读取配置在web.xml中的各个**相关的属性值**。

3. `ContextLoader`会实例化`WebApplicationContext`，并完成其载入和初始化过程。

4. 这个被初始化的第一个上下文作为**根上下文**而存在，这个根上下文载入后，被绑定到Web应用程序的`ServletContext`上。

   - 任何需要访问**根上下文**的应用程序代码都可以从`WebApplicationContextUtils`类的静态方法中得到。

     `WebApplicationContextUtils#getWebApplicationContext`

     ```java
     @Nullable
     public static WebApplicationContext getWebApplicationContext(ServletContext sc) {
       return getWebApplicationContext(sc, WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE);
     }
     ```

     

#### 根上下文的载入过程

- `ContextLoaderListener`监听器的回调方法响应Web容器创建和销毁的事件。
- 在`ContextLoaderListener#contextInitialized`方法中，做了IoC容器初始化的操作。

- `ContextLoaderListener#contextInitialized`中，调用了父类的`initWebApplicationContext`方法，由`ContextLoader#initWebApplicationContext`完成真正的初始化工作。
- 存取**根上下文**的路径是由Spring预先设置好的，在`ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE`的属性中定义这个路径。
- 通过`web.xml#contextClass`参数指定特定的IoC容器，如果没有，将使用默认的IoC容器`XmlWebApplicationContext`。


```java
String ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE = WebApplicationContext.class.getName() + ".ROOT";
```

```java
@Override
public void contextInitialized(ServletContextEvent event) {
   // 调用父类方法，传入ServletContext
   initWebApplicationContext(event.getServletContext());
}
```

```java
public WebApplicationContext initWebApplicationContext(ServletContext servletContext) {
  // 确保初始化的时候容器中没有根上下文，有则抛出异常。
  if (servletContext.getAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE) != null) {
    throw new IllegalStateException(
      "Cannot initialize context because there is already a root application context present - " +
      "check whether you have multiple ContextLoader* definitions in your web.xml!");
  }

  servletContext.log("Initializing Spring root WebApplicationContext");
  Log logger = LogFactory.getLog(ContextLoader.class);
  if (logger.isInfoEnabled()) {
    logger.info("Root WebApplicationContext: initialization started");
  }
  long startTime = System.currentTimeMillis();

  try {
    // 创建容器
    if (this.context == null) {
      this.context = createWebApplicationContext(servletContext);
    }
    // 为容器配置父容器
    if (this.context instanceof ConfigurableWebApplicationContext) {
      ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) this.context;
      if (!cwac.isActive()) {
        if (cwac.getParent() == null) {
          ApplicationContext parent = loadParentContext(servletContext);
          cwac.setParent(parent);
        }
        // 配置并调用容器的Refresh方法初始化容器
        configureAndRefreshWebApplicationContext(cwac, servletContext);
      }
    }
    // 设置当前容器为ServletContext的根容器
    servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, this.context);

    ClassLoader ccl = Thread.currentThread().getContextClassLoader();
    if (ccl == ContextLoader.class.getClassLoader()) {
      currentContext = this.context;
    }
    else if (ccl != null) {
      currentContextPerThread.put(ccl, this.context);
    }

    if (logger.isInfoEnabled()) {
      long elapsedTime = System.currentTimeMillis() - startTime;
      logger.info("Root WebApplicationContext initialized in " + elapsedTime + " ms");
    }

    return this.context;
  }
  catch (RuntimeException | Error ex) {
    logger.error("Context initialization failed", ex);
    servletContext.setAttribute(WebApplicationContext.ROOT_WEB_APPLICATION_CONTEXT_ATTRIBUTE, ex);
    throw ex;
  }
}

/**
 * 创建根上下文
 */
protected WebApplicationContext createWebApplicationContext(ServletContext sc) {
  // 决定使用什么样的类作为上下文
  Class<?> contextClass = determineContextClass(sc);
  if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
    throw new ApplicationContextException("Custom context class [" + contextClass.getName() +
                                          "] is not of type [" + ConfigurableWebApplicationContext.class.getName() + "]");
  }
  // 实例化根上下文
  return (ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);
}

/**
 * 决定上下文的类
 */
protected Class<?> determineContextClass(ServletContext servletContext) {
  // 从初始化参数中获取参数
  String contextClassName = servletContext.getInitParameter(CONTEXT_CLASS_PARAM);
  if (contextClassName != null) {
    try {
      return ClassUtils.forName(contextClassName, ClassUtils.getDefaultClassLoader());
    }
    catch (ClassNotFoundException ex) {
      throw new ApplicationContextException(
        "Failed to load custom context class [" + contextClassName + "]", ex);
    }
  }
  else {
    // 启用默认容器，XmlWebApplicationContext
    contextClassName = defaultStrategies.getProperty(WebApplicationContext.class.getName());
    try {
      return ClassUtils.forName(contextClassName, ContextLoader.class.getClassLoader());
    }
    catch (ClassNotFoundException ex) {
      throw new ApplicationContextException(
        "Failed to load default context class [" + contextClassName + "]", ex);
    }
  }
}
```

## Spring MVC的设计与实现

### Spring MVC的应用场景

- 使用Spring MVC，在web.xml中，需要配置`ContextLoaderListener`之外，还要对`DispatcherServlet`进行配置。
- `DispatcherServlet`实现的是Sun的J2EE核心模式中的**前端控制器模式**`Front Controller`。
- `DispatcherServlet`作为一个前端控制器，所有的Web请求都需要通过它来**处理**，**进行转发**、**匹配**、数据处理后，并转由页面进行展现，是Spring MVC实现中最为核心的部分。
- 对于不同的Web请求的映射需求，Spring MVC提供了不同的`HandlerMapping`的实现，可以让应用开发选取不同的映射策略。在默认的情况下，`DispatcherSevlet`选取了`BeanNameUrlHandlerMapping`作为映射策略实现。
- Spring MVC提供了各种Controller的实现来供应用扩展和使用，以应对不同的控制器使用场景，这些Controller控制器的需要实现`handleRequest`接口方法，并返回`ModelAndView`对象。
- Spring MVC提供了各种视图实现，比如常用的**JSP视图**、**Excel视图**、**PDF视图**等。
- Spring MVC提供了**拦截器**供应用使用，允许应用对Web请求进行拦截，以及**前置处理和后置处理**。
- 针对Web应用中常用到的国际化支持，Spring MVC提供了`LocalResolver`实现和接口，应用可以**定制自己的区域解析器**。


### Spring MVC 设计概览

- 在完成对`ContextLoaderListener`的初始化以后，Web容器开始初始化`DispatcherServlet`，这个初始化的启动与在web.xml中对**载入次序**的定义有关。
- `DispatcherServlet`会**建立自己的上下文**来持有Spring MVC的Bean对象，在建立这个自己持有的IoC容器时，会**从ServletContext中得到根上下文作为DispatcherServlet持有上下文的双亲上下文**。
- `DispatcherServlet`有了**根上下文**后对自己持有的上下文进行初始化，并将自己持有的这个上下文保存到`ServletContext`中，供以后检索和使用。

#### 类继承关系

`DispatcherServlet`通过继承`FrameworkServlet`和`HttpServletBean`而继承了`HttpServlet`，通过使用`Servlet` API来**对HTTP请求进行响应**，成为Spring MVC的前端处理器，同时成为**MVC模块与Web容器集成的处理前端**。

![DispatcherServlet](image/DispatcherServlet.png)

#### DispatcherServlet的处理过程

DispatcherServlet的工作大致分为两种

- 初始化

- 对HTTP请求进行响应

```mermaid
sequenceDiagram
participant hsb as HttpServletBean
participant fs as FrameworkServlet
participant ds as DispatcherServlet
hsb ->> fs: initServletBean()
fs ->> fs: initWebApplicationContext()
fs ->> ds: onRefresh()
ds ->> ds: initStrategies()
hsb ->> fs: doGet()/doPost()
fs ->> fs: processRequest()
fs ->> ds: doService()
ds ->> ds : doDispatch()
```



### DispatcherServlet的启动和初始化

- 作为Servlet, `DispatcherServlet`的启动与Servlet的启动过程是相联系的。
- 在Servlet的初始化过程中，Servlet的`init`方法会被调用，以进行初始化。
- `DispatcherServlet`的基类`HttpServletBean`中`init`方法实现了这个初始化过程。
- 在一个Web应用中，往往可以容纳多个Servlet存在。与此相对应，对于应用在Web容器中的上下体系，一个根上下文可以作为许多Servlet上下文的双亲上下文。

#### HttpServletBean的初始化

1. 读取配置在`ServletContext`中的Bean属性参数，这些属性参数设置在`web.xml`的Web容器**初始化参数**中。
2. 执行`DispatcherServlet`持有的IoC容器的初始化过程，一个新的上下文被建立起来，这个**DispatcherServlet持有的上下文被设置为根上下文的子上下文**。可以认为，根上下文是和Web应用相对应的一个上下文，而DispatcherServlet持有的上下文是和Servlet对应的一个上下文。
3. `DispatcherServlet`持有的上下文被建立起来以后，也需要和其他IoC容器一样完成**初始化**，这个初始化也是通过`refresh`方法来完成的。
4. `DispatcherServlet`给这个自己持有的上下文命名，并把它设置到Web容器的上下文中，这个名称和在`web.xml`中设置的DispatcherServlet的Servlet名称有关，从而**保证了这个上下文在Web环境上下文体系中的唯一性**。

`HttpServletBean#init`

```java
@Override
public final void init() throws ServletException {

  // 获取Servlet的初始化参数，对Bean属性进行配置
  PropertyValues pvs = new ServletConfigPropertyValues(getServletConfig(), this.requiredProperties);
  if (!pvs.isEmpty()) {
    try {
      BeanWrapper bw = PropertyAccessorFactory.forBeanPropertyAccess(this);
      ResourceLoader resourceLoader = new ServletContextResourceLoader(getServletContext());
      bw.registerCustomEditor(Resource.class, new ResourceEditor(resourceLoader, getEnvironment()));
      initBeanWrapper(bw);
      bw.setPropertyValues(pvs, true);
    }
    catch (BeansException ex) {
      if (logger.isErrorEnabled()) {
        logger.error("Failed to set bean properties on servlet '" + getServletName() + "'", ex);
      }
      throw ex;
    }
  }

  // 调用子类的initServletBean进行具体的初始化
  initServletBean();
}
```

`FrameworkServlet#initServletBean`

```java
@Override
protected final void initServletBean() throws ServletException {
  getServletContext().log("Initializing Spring " + getClass().getSimpleName() + " '" + getServletName() + "'");
  if (logger.isInfoEnabled()) {
    logger.info("Initializing Servlet '" + getServletName() + "'");
  }
  long startTime = System.currentTimeMillis();
  try {
    this.webApplicationContext = initWebApplicationContext();
    // 初始化上下文
    initFrameworkServlet();
  }
  catch (ServletException | RuntimeException ex) {
    logger.error("Context initialization failed", ex);
    throw ex;
  }
  if (logger.isDebugEnabled()) {
    String value = this.enableLoggingRequestDetails ?
      "shown which may lead to unsafe logging of potentially sensitive data" :
    "masked to prevent unsafe logging of potentially sensitive data";
    logger.debug("enableLoggingRequestDetails='" + this.enableLoggingRequestDetails +
                 "': request parameters and headers will be " + value);
  }
  if (logger.isInfoEnabled()) {
    logger.info("Completed initialization in " + (System.currentTimeMillis() - startTime) + " ms");
  }
}

protected WebApplicationContext initWebApplicationContext() {
  // 获取根容器
  WebApplicationContext rootContext =
    WebApplicationContextUtils.getWebApplicationContext(getServletContext());
  WebApplicationContext wac = null;
  if (this.webApplicationContext != null) {
    wac = this.webApplicationContext;
    if (wac instanceof ConfigurableWebApplicationContext) {
      ConfigurableWebApplicationContext cwac = (ConfigurableWebApplicationContext) wac;
      if (!cwac.isActive()) {
        if (cwac.getParent() == null) {
					// 设置根容器为父容器
          cwac.setParent(rootContext);
        }
        configureAndRefreshWebApplicationContext(cwac);
      }
    }
  }
  if (wac == null) {
    wac = findWebApplicationContext();
  }
  if (wac == null) {
    // No context instance is defined for this servlet -> create a local one
    wac = createWebApplicationContext(rootContext);
  }
  if (!this.refreshEventReceived) {
		// refresh Bean
    synchronized (this.onRefreshMonitor) {
      onRefresh(wac);
    }
  }
  if (this.publishContext) {
    // 获取配置的容器名字并放到ServletContext中
    String attrName = getServletContextAttributeName();
    getServletContext().setAttribute(attrName, wac);
  }
  return wac;
}
```

#### FrameworkServlet

- `DispatcherServlet`的上下文建立过程与前面建立**根上下文**的过程非常类似。
- 建立`DispatcherServlet`的上下文，需要把**根上下文**作为参数传递给FrameworkServlet。
- 通过反射技术来实例化上下文对象，并为它设置参数。
- 根据默认的配置，这个上下文对象也是`XmlWebApplicationContext`对象，这个类型是在`DEFAULT_CONTEXT_CLASS`参数中设置的。
- 在实例化结束以后，需要为这个上下文对象设置好一些**基本的配置**，这些配置包括它的**双亲上下文**、**Bean定义配置的文件位置**等。
- 通过调用IoC容器的refresh方法来完成IoC容器的最终初始化。
- 对**MVC**的初始化是在`DispatcherServlet`的`initStrategies`中完成的。

`FrameworkServlet#createWebApplicationContext`

```java
protected WebApplicationContext createWebApplicationContext(@Nullable ApplicationContext parent) {
  Class<?> contextClass = getContextClass();
  if (!ConfigurableWebApplicationContext.class.isAssignableFrom(contextClass)) {
    throw new ApplicationContextException(
      "Fatal initialization error in servlet with name '" + getServletName() +
      "': custom WebApplicationContext class [" + contextClass.getName() +
      "] is not of type ConfigurableWebApplicationContext");
  }
  ConfigurableWebApplicationContext wac =
    (ConfigurableWebApplicationContext) BeanUtils.instantiateClass(contextClass);
  // 设置环境
  wac.setEnvironment(getEnvironment());
  // 设置根容器
  wac.setParent(parent);
  String configLocation = getContextConfigLocation();
  if (configLocation != null) {
    wac.setConfigLocation(configLocation);
  }
  // 通过refresh来调用容器的初始化过程
  configureAndRefreshWebApplicationContext(wac);
  return wac;
}
```

`DispatcherServlet#onRefresh`

```java
@Override
protected void onRefresh(ApplicationContext context) {
  initStrategies(context);
}

protected void initStrategies(ApplicationContext context) {
  initMultipartResolver(context);
  // 初始化支持国际化
  initLocaleResolver(context);
  initThemeResolver(context);
  // 初始化支持请求映射
  initHandlerMappings(context);
  // 初始化处理器适配器
  initHandlerAdapters(context);
  // 初始化异常处理器
  initHandlerExceptionResolvers(context);
  initRequestToViewNameTranslator(context);
  // 初始化视图解析器
  initViewResolvers(context);
  initFlashMapManager(context);
}
```

### MVC处理HTTP分发请求

#### HandlerMapping的配置和设计原理

- 在初始化完成时，在上下文环境中已定义的所有`HandlerMapping`都已经被加载。
- 加载的`handlerMappings`被放在一个List中并**被排序**，存储着HTTP请求对应的映射数据。
- 每一个元素都对应着一个具体handlerMapping的配置，一般每一个`handlerMapping`可以持有**一系列从URL请求到Controller的映射**。

![HandlerMapping](image/HandlerMapping-6015313.png)

##### SimpleUrlHandlerMapping

- `SimpleUrlHandlerMapping`中，定义了一个map来持有一系列的映射关系。
- 映射关系是通过接口类`HandlerMapping`来封装的。
- 在`HandlerMapping`接口中定义了一个`getHandler`方法，通过这个方法，可以获得与HTTP请求对应的`HandlerExecutionChain`，在这个`HandlerExecutionChain`中，封装了具体的`Controller`对象。

```java
public interface HandlerMapping {

	String BEST_MATCHING_HANDLER_ATTRIBUTE = HandlerMapping.class.getName() + ".bestMatchingHandler";

	String LOOKUP_PATH = HandlerMapping.class.getName() + ".lookupPath";

	String PATH_WITHIN_HANDLER_MAPPING_ATTRIBUTE = HandlerMapping.class.getName() + ".pathWithinHandlerMapping";

	String BEST_MATCHING_PATTERN_ATTRIBUTE = HandlerMapping.class.getName() + ".bestMatchingPattern";

	String INTROSPECT_TYPE_LEVEL_MAPPING = HandlerMapping.class.getName() + ".introspectTypeLevelMapping";

	String URI_TEMPLATE_VARIABLES_ATTRIBUTE = HandlerMapping.class.getName() + ".uriTemplateVariables";

	String MATRIX_VARIABLES_ATTRIBUTE = HandlerMapping.class.getName() + ".matrixVariables";

	String PRODUCIBLE_MEDIA_TYPES_ATTRIBUTE = HandlerMapping.class.getName() + ".producibleMediaTypes";

  /**
  * 调用getHandler实际上返回的是一个HandlerExecutionChain，这是典型的Command的模式的使用，这个		 HandlerExecutionChain不但持有handler本身，还包括了处理这个HTTP请求相关的拦截器
  **/
  @Nullable
	HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception;

}
```

##### HandlerExecutionChain

- 持有一个`Interceptor`链和一个`handler`对象。
- handler对象实际上就是HTTP请求对应的Controller。
- 对拦截器链和handler的配置是在`HandlerExecutionChain`的初始化函数中完成的。
- 为了维护这个拦截器链和handler, HandlerExecutionChain还提供了一系列与拦截器链维护相关一些操作。

```java
public class HandlerExecutionChain {

	private static final Log logger = LogFactory.getLog(HandlerExecutionChain.class);

	private final Object handler;

	@Nullable
	private HandlerInterceptor[] interceptors;

	@Nullable
	private List<HandlerInterceptor> interceptorList;

	private int interceptorIndex = -1;

	public HandlerExecutionChain(Object handler) {
		this(handler, (HandlerInterceptor[]) null);
	}

	public HandlerExecutionChain(Object handler, @Nullable HandlerInterceptor... interceptors) {
		if (handler instanceof HandlerExecutionChain) {
			HandlerExecutionChain originalChain = (HandlerExecutionChain) handler;
			this.handler = originalChain.getHandler();
			this.interceptorList = new ArrayList<>();
			CollectionUtils.mergeArrayIntoCollection(originalChain.getInterceptors(), this.interceptorList);
			CollectionUtils.mergeArrayIntoCollection(interceptors, this.interceptorList);
		}
		else {
			this.handler = handler;
			this.interceptors = interceptors;
		}
	}

	public Object getHandler() {
		return this.handler;
	}

	public void addInterceptor(HandlerInterceptor interceptor) {
		initInterceptorList().add(interceptor);
	}

	public void addInterceptor(int index, HandlerInterceptor interceptor) {
		initInterceptorList().add(index, interceptor);
	}

	public void addInterceptors(HandlerInterceptor... interceptors) {
		if (!ObjectUtils.isEmpty(interceptors)) {
			CollectionUtils.mergeArrayIntoCollection(interceptors, initInterceptorList());
		}
	}

	private List<HandlerInterceptor> initInterceptorList() {
		if (this.interceptorList == null) {
			this.interceptorList = new ArrayList<>();
			if (this.interceptors != null) {
				// An interceptor array specified through the constructor
				CollectionUtils.mergeArrayIntoCollection(this.interceptors, this.interceptorList);
			}
		}
		this.interceptors = null;
		return this.interceptorList;
	}

	@Nullable
	public HandlerInterceptor[] getInterceptors() {
		if (this.interceptors == null && this.interceptorList != null) {
			this.interceptors = this.interceptorList.toArray(new HandlerInterceptor[0]);
		}
		return this.interceptors;
	}

	boolean applyPreHandle(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HandlerInterceptor[] interceptors = getInterceptors();
		if (!ObjectUtils.isEmpty(interceptors)) {
			for (int i = 0; i < interceptors.length; i++) {
				HandlerInterceptor interceptor = interceptors[i];
				if (!interceptor.preHandle(request, response, this.handler)) {
					triggerAfterCompletion(request, response, null);
					return false;
				}
				this.interceptorIndex = i;
			}
		}
		return true;
	}

	void applyPostHandle(HttpServletRequest request, HttpServletResponse response, @Nullable ModelAndView mv)
			throws Exception {

		HandlerInterceptor[] interceptors = getInterceptors();
		if (!ObjectUtils.isEmpty(interceptors)) {
			for (int i = interceptors.length - 1; i >= 0; i--) {
				HandlerInterceptor interceptor = interceptors[i];
				interceptor.postHandle(request, response, this.handler, mv);
			}
		}
	}

	void triggerAfterCompletion(HttpServletRequest request, HttpServletResponse response, @Nullable Exception ex)
			throws Exception {

		HandlerInterceptor[] interceptors = getInterceptors();
		if (!ObjectUtils.isEmpty(interceptors)) {
			for (int i = this.interceptorIndex; i >= 0; i--) {
				HandlerInterceptor interceptor = interceptors[i];
				try {
					interceptor.afterCompletion(request, response, this.handler, ex);
				}
				catch (Throwable ex2) {
					logger.error("HandlerInterceptor.afterCompletion threw exception", ex2);
				}
			}
		}
	}

	void applyAfterConcurrentHandlingStarted(HttpServletRequest request, HttpServletResponse response) {
		HandlerInterceptor[] interceptors = getInterceptors();
		if (!ObjectUtils.isEmpty(interceptors)) {
			for (int i = interceptors.length - 1; i >= 0; i--) {
				if (interceptors[i] instanceof AsyncHandlerInterceptor) {
					try {
						AsyncHandlerInterceptor asyncInterceptor = (AsyncHandlerInterceptor) interceptors[i];
						asyncInterceptor.afterConcurrentHandlingStarted(request, response, this.handler);
					}
					catch (Throwable ex) {
						logger.error("Interceptor [" + interceptors[i] + "] failed in afterConcurrentHandlingStarted", ex);
					}
				}
			}
		}
	}


	@Override
	public String toString() {
		Object handler = getHandler();
		StringBuilder sb = new StringBuilder();
		sb.append("HandlerExecutionChain with [").append(handler).append("] and ");
		if (this.interceptorList != null) {
			sb.append(this.interceptorList.size());
		}
		else if (this.interceptors != null) {
			sb.append(this.interceptors.length);
		}
		else {
			sb.append(0);
		}
		return sb.append(" interceptors").toString();
	}

}
```

##### 配置信息的注册过程

- handlerMap注册过程在容器对Bean进行依赖注入时发生，它实际上是通过一个Bean的postProcessor来完成的。

##### SimpleUrlHandlerMapping

```java
public class SimpleUrlHandlerMapping extends AbstractUrlHandlerMapping {

	private final Map<String, Object> urlMap = new LinkedHashMap<>();

	public SimpleUrlHandlerMapping() {
	}

	public SimpleUrlHandlerMapping(Map<String, ?> urlMap) {
		setUrlMap(urlMap);
	}

	public SimpleUrlHandlerMapping(Map<String, ?> urlMap, int order) {
		setUrlMap(urlMap);
		setOrder(order);
	}

	public void setMappings(Properties mappings) {
		CollectionUtils.mergePropertiesIntoMap(mappings, this.urlMap);
	}

	public void setUrlMap(Map<String, ?> urlMap) {
		this.urlMap.putAll(urlMap);
	}

	public Map<String, ?> getUrlMap() {
		return this.urlMap;
	}

	@Override
  // 初始化容器，注册handlers
	public void initApplicationContext() throws BeansException {
		super.initApplicationContext();
		registerHandlers(this.urlMap);
	}

	protected void registerHandlers(Map<String, Object> urlMap) throws BeansException {
		if (urlMap.isEmpty()) {
			logger.trace("No patterns in " + formatMappingName());
		}
		else {
      // 递归处理handler和映射规则
			urlMap.forEach((url, handler) -> {
				// Prepend with slash if not already present.
				if (!url.startsWith("/")) {
					url = "/" + url;
				}
				// Remove whitespace from handler bean name.
				if (handler instanceof String) {
					handler = ((String) handler).trim();
				}
				registerHandler(url, handler);
			});
			if (logger.isDebugEnabled()) {
				List<String> patterns = new ArrayList<>();
				if (getRootHandler() != null) {
					patterns.add("/");
				}
				if (getDefaultHandler() != null) {
					patterns.add("/**");
				}
				patterns.addAll(getHandlerMap().keySet());
				logger.debug("Patterns " + patterns + " in " + formatMappingName());
			}
		}
	}

}
```

##### AbstractUrlHandlerMapping

- `SimpleUrlHandlerMapping`注册过程的完成，很大一部分需要它的基类`AbstractUrlHandlerMapping`来配合。
- 如果使用Bean的名称作为映射，那么直接从容器中获取这个HTTP映射对应的Bean，然后还要对不同的URL配置进行解析处理。
- `handlerMap`是一个HashMap，其中保存了URL请求和Controller的映射关系。

`AbstractUrlHandlerMapping#registerHandler`

```java
protected void registerHandler(String urlPath, Object handler) throws BeansException, IllegalStateException {
  Assert.notNull(urlPath, "URL path must not be null");
  Assert.notNull(handler, "Handler object must not be null");
  Object resolvedHandler = handler;

  // 如果直接用bean名称进行映射，那就直接从容器中获取handler
  if (!this.lazyInitHandlers && handler instanceof String) {
    String handlerName = (String) handler;
    ApplicationContext applicationContext = obtainApplicationContext();
    if (applicationContext.isSingleton(handlerName)) {
      resolvedHandler = applicationContext.getBean(handlerName);
    }
  }

  Object mappedHandler = this.handlerMap.get(urlPath);
  if (mappedHandler != null) {
    if (mappedHandler != resolvedHandler) {
      throw new IllegalStateException(
        "Cannot map " + getHandlerDescription(handler) + " to URL path [" + urlPath +
        "]: There is already " + getHandlerDescription(mappedHandler) + " mapped.");
    }
  }
  else {
    if (urlPath.equals("/")) {
      if (logger.isTraceEnabled()) {
        logger.trace("Root mapping to " + getHandlerDescription(handler));
      }
      // 根处理器
      setRootHandler(resolvedHandler);
    }
    else if (urlPath.equals("/*")) {
      if (logger.isTraceEnabled()) {
        logger.trace("Default mapping to " + getHandlerDescription(handler));
      }
      // 默认处理器
      setDefaultHandler(resolvedHandler);
    }
    else {
      this.handlerMap.put(urlPath, resolvedHandler);
      if (logger.isTraceEnabled()) {
        logger.trace("Mapped [" + urlPath + "] onto " + getHandlerDescription(handler));
      }
    }
  }
}
```

------

#### 使用HandlerMapping完成请求的映射处理

- `AbstractHandlerMapping#getHandler`方法会根据初始化时得到的映射关系来生成DispatcherServlet需要的`HandlerExecutionChain`。
- 

`AbstractHandlerMapping#getHandler`

```java
@Override
@Nullable
public final HandlerExecutionChain getHandler(HttpServletRequest request) throws Exception {
  Object handler = getHandlerInternal(request);
  // 默认handler，也就是“/”对应的处理器
  if (handler == null) {
    handler = getDefaultHandler();
  }
  if (handler == null) {
    return null;
  }
  // 通过bean name获取指定的bean
  if (handler instanceof String) {
    String handlerName = (String) handler;
    handler = obtainApplicationContext().getBean(handlerName);
  }
	// 把Handler封装到HandlerExecutionChain中并加上拦截器
  HandlerExecutionChain executionChain = getHandlerExecutionChain(handler, request);
	// 
  if (logger.isTraceEnabled()) {
    logger.trace("Mapped to " + handler);
  }
  else if (logger.isDebugEnabled() && !request.getDispatcherType().equals(DispatcherType.ASYNC)) {
    logger.debug("Mapped to " + executionChain.getHandler());
  }

  if (hasCorsConfigurationSource(handler) || CorsUtils.isPreFlightRequest(request)) {
    CorsConfiguration config = (this.corsConfigurationSource != null ? this.corsConfigurationSource.getCorsConfiguration(request) : null);
    CorsConfiguration handlerConfig = getCorsConfiguration(handler, request);
    config = (config != null ? config.combine(handlerConfig) : handlerConfig);
    executionChain = getCorsHandlerExecutionChain(request, executionChain, config);
  }

  return executionChain;
}
```

##### AbstractUrlHandlerMapping

`AbstractUrlHandlerMapping#getHandlerInternal`

```java
@Override
@Nullable
protected Object getHandlerInternal(HttpServletRequest request) throws Exception {
  // 获取请求的URL路径
  String lookupPath = getUrlPathHelper().getLookupPathForRequest(request);
  request.setAttribute(LOOKUP_PATH, lookupPath);
  // 将得到的URL路径与Handler进行匹配，得到对应的Handler，如果没有对应的Hanlder，返回null，这样默认的Handler会被使用
  Object handler = lookupHandler(lookupPath, request);
  if (handler == null) {
    // We need to care for the default handler directly, since we need to
    // expose the PATH_WITHIN_HANDLER_MAPPING_ATTRIBUTE for it as well.
    Object rawHandler = null;
    // 根处理器
    if ("/".equals(lookupPath)) {
      rawHandler = getRootHandler();
    }
    // 默认处理器
    if (rawHandler == null) {
      rawHandler = getDefaultHandler();
    }
    if (rawHandler != null) {
      // 根据名称获取bean
      if (rawHandler instanceof String) {
        String handlerName = (String) rawHandler;
        rawHandler = obtainApplicationContext().getBean(handlerName);
      }
      validateHandler(rawHandler, request);
      handler = buildPathExposingHandler(rawHandler, lookupPath, lookupPath, null);
    }
  }
  return handler;
}

/**
 * 根据URL路径启动在handlerMap中对handler的检索，并最终返回handler对象
**/
@Nullable
protected Object lookupHandler(String urlPath, HttpServletRequest request) throws Exception {
  // Direct match?
  Object handler = this.handlerMap.get(urlPath);
  if (handler != null) {
    // Bean name or resolved handler?
    if (handler instanceof String) {
      String handlerName = (String) handler;
      handler = obtainApplicationContext().getBean(handlerName);
    }
    validateHandler(handler, request);
    return buildPathExposingHandler(handler, urlPath, urlPath, null);
  }

  // Pattern match?
  List<String> matchingPatterns = new ArrayList<>();
  for (String registeredPattern : this.handlerMap.keySet()) {
    if (getPathMatcher().match(registeredPattern, urlPath)) {
      matchingPatterns.add(registeredPattern);
    }
    else if (useTrailingSlashMatch()) {
      if (!registeredPattern.endsWith("/") && getPathMatcher().match(registeredPattern + "/", urlPath)) {
        matchingPatterns.add(registeredPattern + "/");
      }
    }
  }

  String bestMatch = null;
  Comparator<String> patternComparator = getPathMatcher().getPatternComparator(urlPath);
  if (!matchingPatterns.isEmpty()) {
    matchingPatterns.sort(patternComparator);
    if (logger.isTraceEnabled() && matchingPatterns.size() > 1) {
      logger.trace("Matching patterns " + matchingPatterns);
    }
    bestMatch = matchingPatterns.get(0);
  }
  if (bestMatch != null) {
    handler = this.handlerMap.get(bestMatch);
    if (handler == null) {
      if (bestMatch.endsWith("/")) {
        handler = this.handlerMap.get(bestMatch.substring(0, bestMatch.length() - 1));
      }
      if (handler == null) {
        throw new IllegalStateException(
          "Could not find handler for best pattern match [" + bestMatch + "]");
      }
    }
    // Bean name or resolved handler?
    if (handler instanceof String) {
      String handlerName = (String) handler;
      handler = obtainApplicationContext().getBean(handlerName);
    }
    validateHandler(handler, request);
    String pathWithinMapping = getPathMatcher().extractPathWithinPattern(bestMatch, urlPath);

    // There might be multiple 'best patterns', let's make sure we have the correct URI template variables
    // for all of them
    Map<String, String> uriTemplateVariables = new LinkedHashMap<>();
    for (String matchingPattern : matchingPatterns) {
      if (patternComparator.compare(bestMatch, matchingPattern) == 0) {
        Map<String, String> vars = getPathMatcher().extractUriTemplateVariables(matchingPattern, urlPath);
        Map<String, String> decodedVars = getUrlPathHelper().decodePathVariables(request, vars);
        uriTemplateVariables.putAll(decodedVars);
      }
    }
    if (logger.isTraceEnabled() && uriTemplateVariables.size() > 0) {
      logger.trace("URI variables " + uriTemplateVariables);
    }
    return buildPathExposingHandler(handler, bestMatch, pathWithinMapping, uriTemplateVariables);
  }

  // No handler found...
  return null;
}
```

#### Spring MVC对HTTP请求的分发处理

- 业务逻辑的调用入口是在handler的`handler`方法中实现的，这里是**连接Spring MVC和应用业务逻辑实现**的地方。
- `DispatcherServlet#doDispatch` 是对请求处理的实现。
  - 准备ModelAndView。
  - 调用getHandler来响应HTTP请求。
  - 通过执行Handler的处理来得到返回的ModelAndView结果。
  - 把ModelAndView对象交给相应的视图对象去呈现。

##### doDispatch协同模型和控制器过程

```mermaid
sequenceDiagram
participant ds as DispatcherServlet
participant hm as HandlerMapping
participant ha as HadnlerAdapter
participant hi as HandlerInterceptor
participant vr as ViewResolver
participant v as View

ds ->> ds: checkMulitipart()
ds ->> hm: getHandler()
hm -->> ds: HandlerExecutionChain
ds ->> hi: preHandler()
ds ->> ha: handler()
ha -->> ds: ModelAndView
ds ->> hi: postHandler()
ds ->> vr: resolveViewName()
vr ->> v: getView()
v -->> ds: View
ds ->> v: render()
```



`DispatcherServlet#doService`

```java
@Override
protected void doService(HttpServletRequest request, HttpServletResponse response) throws Exception {
  // 日志记录
  logRequest(request);

  // Keep a snapshot of the request attributes in case of an include,
  // to be able to restore the original attributes after the include.
  Map<String, Object> attributesSnapshot = null;
  if (WebUtils.isIncludeRequest(request)) {
    attributesSnapshot = new HashMap<>();
    Enumeration<?> attrNames = request.getAttributeNames();
    while (attrNames.hasMoreElements()) {
      String attrName = (String) attrNames.nextElement();
      if (this.cleanupAfterInclude || attrName.startsWith(DEFAULT_STRATEGIES_PREFIX)) {
        attributesSnapshot.put(attrName, request.getAttribute(attrName));
      }
    }
  }

  // 对HTTP请求参数进行快照处理
  request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
  request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
  request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
  request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());

  if (this.flashMapManager != null) {
    FlashMap inputFlashMap = this.flashMapManager.retrieveAndUpdate(request, response);
    if (inputFlashMap != null) {
      request.setAttribute(INPUT_FLASH_MAP_ATTRIBUTE, Collections.unmodifiableMap(inputFlashMap));
    }
    request.setAttribute(OUTPUT_FLASH_MAP_ATTRIBUTE, new FlashMap());
    request.setAttribute(FLASH_MAP_MANAGER_ATTRIBUTE, this.flashMapManager);
  }

  try {
    // 分发请求的入口
    doDispatch(request, response);
  }
  finally {
    if (!WebAsyncUtils.getAsyncManager(request).isConcurrentHandlingStarted()) {
      // Restore the original attribute snapshot, in case of an include.
      if (attributesSnapshot != null) {
        restoreAttributesAfterInclude(request, attributesSnapshot);
      }
    }
  }
}

/**
 * 分发请求
**/
protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
  HttpServletRequest processedRequest = request;
  HandlerExecutionChain mappedHandler = null;
  boolean multipartRequestParsed = false;

  WebAsyncManager asyncManager = WebAsyncUtils.getAsyncManager(request);

  try {
    // 定义视图
    ModelAndView mv = null;
    Exception dispatchException = null;

    try {
      processedRequest = checkMultipart(request);
      multipartRequestParsed = (processedRequest != request);

      // 根据请求得到对应的handler, handler的注册以及getHandler的实现
      mappedHandler = getHandler(processedRequest);
      if (mappedHandler == null) {
        noHandlerFound(processedRequest, response);
        return;
      }

      // 获取hanlder适配器
      HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

      String method = request.getMethod();
      boolean isGet = "GET".equals(method);
      if (isGet || "HEAD".equals(method)) {
        long lastModified = ha.getLastModified(request, mappedHandler.getHandler());
        if (new ServletWebRequest(request, response).checkNotModified(lastModified) && isGet) {
          return;
        }
      }
      // 前置处理器
      if (!mappedHandler.applyPreHandle(processedRequest, response)) {
        return;
      }

      // 调用并处理返回视图
      mv = ha.handle(processedRequest, response, mappedHandler.getHandler());

      if (asyncManager.isConcurrentHandlingStarted()) {
        return;
      }
			// 默认视图名称
      applyDefaultViewName(processedRequest, mv);
      // 后置处理
      mappedHandler.applyPostHandle(processedRequest, response, mv);
    }
    catch (Exception ex) {
      dispatchException = ex;
    }
    catch (Throwable err) {
      // As of 4.3, we're processing Errors thrown from handler methods as well,
      // making them available for @ExceptionHandler methods and other scenarios.
      dispatchException = new NestedServletException("Handler dispatch failed", err);
    }
    processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);
  }
  catch (Exception ex) {
    triggerAfterCompletion(processedRequest, response, mappedHandler, ex);
  }
  catch (Throwable err) {
    triggerAfterCompletion(processedRequest, response, mappedHandler,
                           new NestedServletException("Handler processing failed", err));
  }
  finally {
    if (asyncManager.isConcurrentHandlingStarted()) {
      // Instead of postHandle and afterCompletion
      if (mappedHandler != null) {
        mappedHandler.applyAfterConcurrentHandlingStarted(processedRequest, response);
      }
    }
    else {
      // Clean up any resources used by a multipart request.
      if (multipartRequestParsed) {
        cleanupMultipart(processedRequest);
      }
    }
  }
}
```

## Spring MVC视图的呈现

#### DispatcherServlet视图呈现的设计

- 在DispatcherServlet中，对视图呈现的处理是在`render`方法调用中完成的。
- 从`ModelAndView`对象中取得视图对象，然后调用视图对象的render方法，由这个视图对象来完成特定的视图呈现工作。
- 视图解析的过程
  - 在`ModelAndView`中寻找视图对象的逻辑名。
  - 已经在`ModelAndView`中设置了视图对象的名称，就对这个名称进行解析，得到实际要使用的视图对象。
  - ModelAndView中已经有了最终完成视图呈现的视图对象，可以直接使用该视图对象。
- 不同的视图类型，对应着不同视图对象的实现。


`DispatcherServlet#render`

```java
protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
  // 从request中读取locale信息，并设置response的locale值
  Locale locale =
    (this.localeResolver != null ? this.localeResolver.resolveLocale(request) : request.getLocale());
  response.setLocale(locale);

  View view;
  // 根据ModleAndView中设置的视图名称进行解析，得到对应的视图对象
  String viewName = mv.getViewName();
  if (viewName != null) {
    // 对视图名进行解析
    view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
    if (view == null) {
      throw new ServletException("Could not resolve view with name '" + mv.getViewName() +
                                 "' in servlet with name '" + getServletName() + "'");
    }
  }
  else {
    // 直接从ModelAndView对象中取得实际的视图对象
    view = mv.getView();
    if (view == null) {
      throw new ServletException("ModelAndView [" + mv + "] neither contains a view name nor a " +
                                 "View object in servlet with name '" + getServletName() + "'");
    }
  }

  // Delegate to the View object for rendering.
  if (logger.isTraceEnabled()) {
    logger.trace("Rendering view [" + view + "] ");
  }
  try {
    if (mv.getStatus() != null) {
      response.setStatus(mv.getStatus().value());
    }
    // 调用view实现对数据进行呈现，并通过HttpResponse把视图呈现给HTTP客户端
    view.render(mv.getModelInternal(), request, response);
  }
  catch (Exception ex) {
    if (logger.isDebugEnabled()) {
      logger.debug("Error rendering view [" + view + "]", ex);
    }
    throw ex;
  }
}
```

`DispatcherServlet#resolveViewName`

```java
@Nullable
protected View resolveViewName(String viewName, @Nullable Map<String, Object> model,
                               Locale locale, HttpServletRequest request) throws Exception {

  if (this.viewResolvers != null) {
    for (ViewResolver viewResolver : this.viewResolvers) {
      View view = viewResolver.resolveViewName(viewName, locale);
      if (view != null) {
        return view;
      }
    }
  }
  return null;
}
```

`BeanNameViewResolver#resolveViewName`

```java
@Override
@Nullable
public View resolveViewName(String viewName, Locale locale) throws BeansException {
  ApplicationContext context = obtainApplicationContext();
  if (!context.containsBean(viewName)) {
    // Allow for ViewResolver chaining...
    return null;
  }
  if (!context.isTypeMatch(viewName, View.class)) {
    if (logger.isDebugEnabled()) {
      logger.debug("Found bean named '" + viewName + "' but it does not implement View");
    }
    // Since we're looking into the general ApplicationContext here,
    // let's accept this as a non-match and allow for chaining as well...
    return null;
  }
  return context.getBean(viewName, View.class);
}
```

##### View的继承体系

- JSP/JSTL视图。
- FreeMaker视图。
- Velocity视图。
- PDF、Excel视图等。

![view](image/view.png)

#### JSP视图的实现

xxx

#### ExcelView视图的实现

xxx

#### PDF视图的实现

xxx