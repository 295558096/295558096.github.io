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

## RequestParam 注解

### 简介

- Spring MVC默认方式获取参数，在方法上写一个和请求参数同名的参数来接受请求参数值。
  - 传入就封装。
  - 不传入参数为null。

- 通过`@RequestParam("xxx")`指定获取请求参数。
- `@RequestParam("xxx")` 等同于`request.getParamter("xxx")`。
- 默认是必须携带的参数。

### 参数

- `value` 指定要获取的参数的key。仅此一个属性的时候，可以省略。

- `required` 指定参数是否必填，默认是true。
- `defaultValue` 指明参数默认值。

## RequestHeader 注解

### 简介

- 获取请求头中某个key的值。
- `@RequestHeader("xxx")` 等同于 `request.getHeader("xxx")`。
- 默认是必须携带的请求头。

### 参数

- `value` 指定要获取的请求头的key。仅此一个属性的时候，可以省略。

- `required` 指定请求头是否必填，默认是true。
- `defaultValue` 指明请求头默认值。

## CookieValue 注解

### 简介

- 获取某个cookie的值。

- `@CookieValue("xxx")` 等同于 `request.getCookies()`后循环找出指定key的cookie。

### 参数

- `value` 指定要获取的请求头的key。仅此一个属性的时候，可以省略。

- `required` 指定请求头是否必填，默认是true。
- `defaultValue` 指明请求头默认值。

## POJO 封装参数

- POJO必须有无参的构造方法。
- 参数需要有set方法。
- SpringMVC 会自动为POJO进行赋值，从请求中获取属性，并封装到POJO的同名属性中。
- **支持级联封装**。

## 传入原生API

- 可以直接在请求方法的入参传入Servlet原生API。
- `HttpServletRequest` 请求
- `HttpServletResponse` 响应
- `HttpSession` session
- `java.security.Principal`  安全协议相关
- `Locale` 国际化相关的区域
- `InputStream` 输入字节流
- `OutputStream` 输出字节流
- `Reader` 输入字符流
- `Writer` 输出字符流

## 乱码的解决

### 请求乱码

#### GET请求

- 修改`server.xml`，添加配置`<Connector URIEncoding="UTF-8"/>`。

#### POST请求

- 在第一次获取请求参数之前设置`request.setCharacterEncoding("utf-8");`。
- 编写一个Filter，确保获取参数之前，设置`CharacterEncoding`。
- 配置SpringMVC提供的 `CharacterEncodingFilter` 也可以指定编码。
  - 配置Filter的initParam的`encoding`设置入参的编码集。
  - 配置Filter的initParam的`forceEncoding`设置是否强制转换响应编码。
- **字符编码Filter一般都在其他Filter之前，否则可能失效。**

### 响应乱码

- `response.setContextType("text/html;charset=UTF-8");`。

## 数据输出

### 原生 request

略

### 原生 session

略

### Map、Model、ModelMap

- 在方法入参传入`Map`、`Model` 或者 `ModelMap`这些参数里面保存的所有数据都会放在**请求域**中。
- SpringMVC中Map、Model、ModelMap的类型都是`BindingAwareModelMap`。
- 保存在`BindingAwareModelMap`中的数据最终都会被带到请求域中。

#### Map

- JDK的接口。

#### Model

- Spring 定义的接口。

#### ModelMap

- 类。
- 继承自`LinkedHashMap`。

#### BindingAwareModelMap

- 继承自`ExtendedModelMap`。
- `ExtendedModelMap` 继承自 `ModelMap` 并且实现`Model`接口。
- **BindingAwareModelMap 是 SpringMVC 保存在请求域中的共享容器对象。**

### ModelAndView

- 既包含视图信息，也包含模型信息。
- 向request域中带数据。

- 方法的返回值可以变为 `ModelAndView` 类型。

- `viewName`视图名，指向想要跳转的页面。
- `addObject()`方法，保存对象。

### SessionAttributes 注解

- 向Session域中携带数据。
- 是一个标记在类上的注解。
- 当给`BindingAwareModelMap` 中保存的数据的时候，同时给 `session` 中放一份。
- 不建议使用`@SessionAttributes`，可能会产生异常。
- 建议使用原生API。

#### 参数

- `value` 指明要同步保存数据的key。
- `types` 指明要同步保存数据的类型。

### ModelAttribute 注解

- 方法入参标注该注解后，入参的对象就会放到数据模型中。
- **标记在方法上，该方法就会提前与目标方法先运行，可以进行校验或者数据获取**，将查询结果存入到`Model`、`Map` 或者 `ModelMap` 中。
- **标记在参数上**，取出保存在 `Model`、`Map` 或者 `ModelMap` 中保存的同名对象。
- `ModelAttribute` 注解如果**标记在方法上**那么可以**把方法运行后的返回值按照方法上指定的 key 放到隐含模型中。如果没有指定这个 key，就用返回值类型的首字母小写**。

## 视图解析

### 跳转方式

1. 通过视图解析器，`return "xx";` 跳转带拼接后的 `xx`路径下。
2. 通过相对路径，在返回值中拼上相对路径。
3. **转发 **`forward:路径`，转发到指定路径。客户端感知不到，认定是一次请求。
4. **重定向** `redirect:` 重定向的路径。
   - SpringMvc会自动拼接项目路径。
   - 如果通过Servlet原生的重定向，路径需要加上项目名才能成功。
   - 重定向浏览器会认定为多次请求。

5. 重定向和转发与配置的视图解析器没有关系。

## 国际化

### JavaWeb 国际化

- 得到一个 `Locale` 对象。
- 使用 `ResourceBundles` 绑定国际化资源文件。
- 使用 `ResourceBundle.getString("key");` 获取到国际化配置文件中的值。
- Web 页面的国际化，通过使用 **fmt 标签库**。
  - \<fmt:setLocale>
  - \<fmt:setBundle>
  - \<fmt:setmessage>

### SpringMvc 基于 JstlView 国际化

- 让 Spring 管理国际化资源，`ResourceBundleMessageSource`，bean 的 id 必须是 `messageSource`。
- 直接去页面使用即可。
- 直接访问资源文件或者是 `forward` 转发请求，国际化不会生效。

## ViewController 视图控制器

- 用来直接配置并指定跳转到视图的方法。
- 需要启用SpringMVC注解驱动的配置，否则其他的请求会报错。

```xml
<mvc:view-controller path="/xxx" view-name="xxx"/>

<mvc:annotation-driven></mvc:annotation-driven>
```

## 自定义视图和自定义视图解析器

- 编写自定义的视图解析器，实现 `ViewResolver` 接口和 `Ordered`接口。
  - 实现 `resolveViewName` 方法，返回指定的视图对象。
  - 实现 `order` 方法，用于指定视图解析器的优先级，**数字越小，优先级越高**。
- 编写自定义视图，实现 View 接口。
  - 实现 `render` 方法，设置响应的`ContentType`。
  - 可以获取到 `Model` 隐藏模型中的数据。
- 在 SpringMVC 配置文件中配置自定义视图解析器，放到 Ioc 容器中，指定自定义视图解析器的顺序。

## SpringMVC Restful

- 需要通过配置 `HttpHiddenMethodFilter` 来启用SpringMVC 对Restful 请求的支持。

- restful 请求规则 `请求类型 /资源名/资源标识`。

- 配置默认的处理请求Servlet。如果dispatcherServlet处理不了的请求，会转交给 Tomcat 默认的Servlet去处理。

  ```xml
  <!-- 启动默认处理器去处理静态资源-->
  <mvc:default-servlet-handler/>
  <!-- 解决启用默认处理器后，只有静态资源可以访问的问题-->
  <mvc:annotation-driven></mvc:annotation-driven>
  ```

  

### 表单标签

- 通过 SpringMVC 的表单标签前以实现**模型数据中的属性和 HTML 表单元素相绑定**，以实现表单数据更便捷编辑和表单值的回显。
- 表单标签库的地址 `http://www.springframework.org/tags/form`。

------

## SpringMVC 源码

### DispatcherServlet

#### 请求流程

1. 请求来到`org.springframework.web.servlet.DispatcherServlet#doDispatch`方法。
2. 检查请求是否属于文件上传请求，是的话，包装原请求。
3. 根据请求找到目标处理器，`HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());`。
   - `HandlerAdapter` 中包含了**拦截器链**和**目标控制器**，在IOC容器启动的时候，初始化的这些`Controller`信息。
   - `handlerMappings`中对象的`handlerMapping`集合中保存了所有的**请求地址和对应处理器的信息**。
   - 根据请求的地址，来匹配出对应的处理器。
   - 返回的对象类型是 `HandlerExecutionChain`。

4. 找不到处理就抛404异常。
5. 拿到能执行这个类的所有方法的适配器。
   - `HttpRequestHandlerAdapter`、`SimpleControllerHandlerAdapter`、`AnnotationMethodHandlerAdapter`。
   - `AnnotationMethodHandlerAdapter` 解析注解方法的适配器。
   - 遍历`handlerAdapters`，通过support方法判断是否适配。

6. `mv = ha.handle(processedRequest, response, mappedHandler.getHandler());`调用适配器处理当前请求，得到`ModelAndView`对象。
   - 根据反射调用目标方法。
   - 方法解析器根据当前请求地址找到真正的目标方法。
   - 创建一个方法执行器，包装原生的`Request`、`Response`。
   - 构建一个`BindingAwareModelMap`。
   - 触发方法执行器的 `invokeHandlerMethod` 完成目标方法的调用，目标方法利用反射执行期间确定参数值。
   - `resolveHandlerArguments()`用来确定方法的参数。
     - 定义一个和`method.getParaterTypes()`结果等长的数组。
     - 遍历入参从中为每个位置的参数设置值。
     - 首先获取方法参数上标记的**注解**情况并分析注解。
       - 如果注解是 `SpringMVC` 注解则获取参数名称、默认值、是否必填的信息。
       - 根据参数的key调用`String[] paramValues = request.getParamterValues(paramName);` 获取全部该参数名称的参数值。
     - 如果没有**标记注解**，按照普通参数解析。
       - 分析参数类型是否是原生API，如果是，直接将原生 API 赋值给当前参数。
       - 如果不是，就根据参数名直接获取对应的参数值。
       - 如果参数类型是 `Map` 或者 `Model`，会把**隐含模型**传入。
     - 如果参数是自定义类型时，标记了@ModelAttribute注解，那么attrName等于注解的value，否则为空。按照如下顺序取值：
       - 如果隐含模型中有这个 `attrName`，就从隐含模型中获取对应的值。**如果 `attrName` 为空，从隐含模式中获取参数类名小写的 key 对应的值。**
       - 如果满足 `@SessionAttribute` 的规则，满足则从`SessionAttribute`中取值。
       - 如果匹配不到，创建一个空对象。
   - 先通过反射调用@ModelAttribute标记的方法，再通过反射调用的真正的目标方法。
   
7. 调用`processDispatchResult(processedRequest, response, mappedHandler, mv, dispatchException);`方法处理请求结果，转发到目标页面。

#### 请求流程图

![image-20220119211002836](image/image-20220119211002836.png)

### SpringMVC 九大组件

- `DispatcherServlet` 中有九大引用对象，在IOC容器启动阶段被注入，被我们称为九大组件。

- SpringMVC 在工作的时候，关键位置的业务都是由这些组件完成的。

- 九大组件都是**接口**，定义了规范，支持进行扩展。

- 可以在 `web.xml` 中修改 `DispatcherServlet` 某些属性的默认配置。

- **九大组件的初始化都是去容器中找对应的组件实现，如果没有找到，就使用SpringMVC提供的默认组件策略**。

- 有的组件是通过类型找到的，有的是通过ID找到的。

- `DispatcherServlet#onRefresh` 实现了父类的方法，在IOC容器启动的时候被调用，完成组件的注册。

  ```java
  @Override
  protected void onRefresh(ApplicationContext context) {
    initStrategies(context);
  }	
  
  protected void initStrategies(ApplicationContext context) {
    initMultipartResolver(context);
    initLocaleResolver(context);
    initThemeResolver(context);
    initHandlerMappings(context);
    initHandlerAdapters(context);
    initHandlerExceptionResolvers(context);
    initRequestToViewNameTranslator(context);
    initViewResolvers(context);
    initFlashMapManager(context);
  }
  ```

#### MultipartResolver

- 文件上传解析器。

#### LocaleResolver

- 区域信息、国际化解析器。

#### ThemeResolver

- 主题解析器。
- 支持主题更换功能。

#### HandlerMapping

- 存储 Handler 的映射信息。

#### HandlerAdapter

- Handler 的适配器。

#### HandlerExceptionResolver

- 异常解析器。
- SpringMVC 支持强大的异常解析功能。

#### RequestToViewNameTranslator

- 请求到视图名的转换器。

#### FlashMapManager

- SpringMVC 中允许重定向携带数据的管理器。

#### ViewResolver

- 视图解析器。
- `InternalResourceViewResolver`。



### 视图解析器原理

- 方法执行后的**返回值**会作为页面地址参考，转发或者重定向到页面。
- 视图解析器可能会进行页面地址的拼串。

- 任何方法的返回值最终都会被包装成`ModelAndView`对象。
- `org.springframework.web.servlet.DispatcherServlet#processDispatchResult`方法会处理ModelAndView对象，并最终渲染页面。

<img src="image/%E6%9C%AA%E5%91%BD%E5%90%8D%E6%96%87%E4%BB%B6%20(1).png" alt="未命名文件 (1)" style="zoom:50%;" />

#### 视图渲染流程

- `org.springframework.web.servlet.DispatcherServlet#render`真正进行页面渲染的方法。

  ```java
  protected void render(ModelAndView mv, HttpServletRequest request, HttpServletResponse response) throws Exception {
  		// Determine locale for request and apply it to the response.
  		Locale locale =
  				(this.localeResolver != null ? this.localeResolver.resolveLocale(request) : request.getLocale());
  		response.setLocale(locale);
  
  		View view;
  		String viewName = mv.getViewName();
  		if (viewName != null) {
  			// We need to resolve the view name.
  			view = resolveViewName(viewName, mv.getModelInternal(), locale, request);
  			if (view == null) {
  				throw new ServletException("Could not resolve view with name '" + mv.getViewName() +
  						"' in servlet with name '" + getServletName() + "'");
  			}
  		}
  		else {
  			// No need to lookup: the ModelAndView object contains the actual View object.
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
        // 调用View对象的render方法进行视图渲染。
  			view.render(mv.getModelInternal(), request, response);
  		}
  		catch (Exception ex) {
  			if (logger.isDebugEnabled()) {
  				logger.debug("Error rendering view [" + view + "]", ex);
  			}
  			throw ex;
  		}
  	}

- `org.springframework.web.servlet.view.AbstractView#render` render方法的抽象实现。

  ```java
  	@Override
  	public void render(@Nullable Map<String, ?> model, HttpServletRequest request,
  			HttpServletResponse response) throws Exception {
  
  		if (logger.isDebugEnabled()) {
  			logger.debug("View " + formatViewName() +
  					", model " + (model != null ? model : Collections.emptyMap()) +
  					(this.staticAttributes.isEmpty() ? "" : ", static attributes " + this.staticAttributes));
  		}
  
  		Map<String, Object> mergedModel = createMergedOutputModel(model, request, response);
  		prepareResponse(request, response);
      // 渲染
  		renderMergedOutputModel(mergedModel, getRequestToExpose(request), response);
  	}
  ```

- `InternalResourceView#renderMergedOutputModel`视图解析器根据合并后的输出模型渲染参数的方法。

  ```java
  @Override
  	protected void renderMergedOutputModel(
  			Map<String, Object> model, HttpServletRequest request, HttpServletResponse response) throws Exception {
  
  		// 暴露属性到请求域中。
  		exposeModelAsRequestAttributes(model, request);
  
  		// Expose helpers as request attributes, if any.
  		exposeHelpers(request);
  
  		// 获取转发路径
  		String dispatcherPath = prepareForRendering(request, response);
  
  		// 获取转发器
  		RequestDispatcher rd = getRequestDispatcher(request, dispatcherPath);
  		if (rd == null) {
  			throw new ServletException("Could not get RequestDispatcher for [" + getUrl() +
  					"]: Check that the corresponding file exists within your web application archive!");
  		}
  
  		// If already included or response already committed, perform include, else forward.
  		if (useInclude(request, response)) {
  			response.setContentType(getContentType());
  			if (logger.isDebugEnabled()) {
  				logger.debug("Including [" + getUrl() + "]");
  			}
  			rd.include(request, response);
  		}
  
  		else {
  			// Note: The forwarded resource is supposed to determine the content type itself.
  			if (logger.isDebugEnabled()) {
  				logger.debug("Forwarding to [" + getUrl() + "]");
  			}
        // 进行页面转发
  			rd.forward(request, response);
  		}
  	}
  ```

  

#### View 与 ViewResolver

- `org.springframework.web.servlet.ViewResolver#resolveViewName`，视图解析器通过解析视图名返回 **View** 对象。

  ```java
  	@Override
  	@Nullable
  	public View resolveViewName(String viewName, Locale locale) throws Exception {
  		if (!isCache()) {
  			return createView(viewName, locale);
  		}
  		else {
  			Object cacheKey = getCacheKey(viewName, locale);
  			View view = this.viewAccessCache.get(cacheKey);
  			if (view == null) {
  				synchronized (this.viewCreationCache) {
  					view = this.viewCreationCache.get(cacheKey);
  					if (view == null) {
  						// 没有缓存的情况下，创建View对象。
  						view = createView(viewName, locale);
  						if (view == null && this.cacheUnresolved) {
  							view = UNRESOLVED_VIEW;
  						}
  						if (view != null && this.cacheFilter.filter(view, viewName, locale)) {
  							this.viewAccessCache.put(cacheKey, view);
  							this.viewCreationCache.put(cacheKey, view);
  						}
  					}
  				}
  			}
  			else {
  				if (logger.isTraceEnabled()) {
  					logger.trace(formatKey(cacheKey) + "served from cache");
  				}
  			}
  			return (view != UNRESOLVED_VIEW ? view : null);
  		}
  	}
  ```

- `UrlBasedViewResolver#createView`创建 View 对象。

  ```java
  @Override
  	protected View createView(String viewName, Locale locale) throws Exception {
  		// If this resolver is not supposed to handle the given view,
  		// return null to pass on to the next resolver in the chain.
  		if (!canHandle(viewName, locale)) {
  			return null;
  		}
  
  		// 重定向的返回值，创建一个重定向的View对象。
  		if (viewName.startsWith(REDIRECT_URL_PREFIX)) {
  			String redirectUrl = viewName.substring(REDIRECT_URL_PREFIX.length());
  			RedirectView view = new RedirectView(redirectUrl,
  					isRedirectContextRelative(), isRedirectHttp10Compatible());
  			String[] hosts = getRedirectHosts();
  			if (hosts != null) {
  				view.setHosts(hosts);
  			}
  			return applyLifecycleMethods(REDIRECT_URL_PREFIX, view);
  		}
  
  		// 转发请求的返回值，创建一个转发的View对象。
  		if (viewName.startsWith(FORWARD_URL_PREFIX)) {
  			String forwardUrl = viewName.substring(FORWARD_URL_PREFIX.length());
  			InternalResourceView view = new InternalResourceView(forwardUrl);
  			return applyLifecycleMethods(FORWARD_URL_PREFIX, view);
  		}
  
  		// 调用父类的loadView，创建并加载View对象。
  		return super.createView(viewName, locale);
  	}
  ```

#### 视图渲染及视图的总结

- 请求处理方法执行完成后，最终返回一个 `ModelAndView` 对象。对于那些返回 `String`、`View` 或 `ModelMap` 等类型的处理方法，SpringMVC 也会在内部将它们装配成一个 `ModelAndView` 对象，它包含了**逻辑名**和**模型对象**的视图。
- SpringMVC 借助视图解析器`ViewResolver`得到最终的视图对象`View`，最终的视图可以是 `JSP`，也可能是 `Excel`、`Jfree Chart` 等各种表现形式的视图。
- 对于最终究竟采取何种视图对象对模型数据进行渲染，处理器并不关心，处理器工作重点聚焦在生产模型数据的工作上，从而实现 MVC 的充分解耦。
- 视图的作用是渲染模型数据，将模型里的数据以棊种形式呈现给客户。
- 为了实现视图模型和具体实现技术的解耦，Spring 在 `org.springframework.web.servlet` 包中定义了一个高度抽象的 `View` 接口。
- **视图对象由视图解析器负责实例化**。由于视图是**无状态**的，所以他们不会有线程安全的问题。
- SpringMVC 为**逻辑视图名的解析提供了不同的策略**，可以在 Spring WEB 上下文中**配置一种或多种解析策略，并指定他们之间的先后顺序**。**每一种映射策略对应一个具体的视图解析器实现类**。
- **视图解析器的作用比较单一，将逻辑视图解析为一个具体的视图对象**。
- 所有的视图解析器都必须实现 `ViewResoler` 接口。

#### 常用视图实现类

| 类型        | 视图类型                      | 说明                                                         |
| ----------- | ----------------------------- | ------------------------------------------------------------ |
| URL资源视图 | **IntemalResourceView**       | 将 JSP 或其它资源封装成一个视图，是InternalResourceViewResolver。默认使用的视图实现类。 |
| URL资源视图 | **JstlView**                  | 如果 JSP 文件中使用了 JSTL 国际化标签的功能，则要使用该视图类。 |
| 文档视图    | AbstractExcelView             | Excel 文档视图的抽象类。该视图类基于 POI 构造 Excel 文档。   |
| 文档视图    | AbstractPdfView               | PDF 文档视图的抽象类。该视图类基于 iText 构造 PDF 文档。     |
| 报表视图    | ConfigurableJsperReports View | 使用 JasperReports 报表技术的视图。                          |
| 报表视图    | JasperReportsCsvView          | 使用 JasperReports 报表技术的视图。                          |
| 报表视图    | JasperReportsMultiFormat View | 使用 JasperReports 报表技术的视图。                          |
| 报表视图    | JasperReportsHtmlView         | 使用 JasperReports 报表技术的视图。                          |
| 报表视图    | JasperReportsPdfView          | 使用 JasperReports 报表技术的视图。                          |
| 报表视图    | JasperReportsXlsView          | 使用 JasperReports 报表技术的视图。                          |
| JSON视图    | MappingJacksonJsonView        | 将模型数据通过 `Jackson` 开源框架的 `ObjectMapper` 以 JSON 方式输出。 |

#### 常用的视图解析器实现类

- 程序员可以选择一种视图解析器或混用多种视图解析器。
- 每个视图解析器都实现了 `Ordered` 接口并开放出一个 order 属性，可**以通过 order 属性指定解析器的优先顺序，order 越小优先级越高**。
- SpringMVC 会按视图解析器顺序的优先顺序对逻辑视图名进行解析，直到解析成功并返回视图对象，否则将抛出 `ServletException` 异常。



| 类型               | 视图类型                         | 说明                                                         |
| ------------------ | -------------------------------- | ------------------------------------------------------------ |
| 解析为 Bean 的名字 | **BeanNameViewResolver**         | 将逻辑视图名解析为一个 Bean， Bean 的 id 等于逻辑视图名。    |
| 解析为 URL 文件    | **InternalResourceViewResolver** | 将视图名解析为个 URL 文件，一般使用该解析器将视图名映射为一个保存在 **WEB-INF** 目录下的程序文件（如 JSP)。 |
| 解析为 URL 文件    | JasperReportsViewResolver        | JasperReports 是一个基于 Java 的开源报表工具，该解析器将视图名解析为报表文件对应的 URL。 |
| 模版文件视图       | FreeMarkerViewResolver           | 解析为基于 FreeMarker 模版技术的模版文件。                   |
| 模版文件视图       | VelocityViewResolver             | 解析为基于 Velocity 模版技术的模版文件。                     |
| 模版文件视图       | VelocityLayoutViewResolver       | 解析为基于 Velocity 模版技术的模版文件。                     |

