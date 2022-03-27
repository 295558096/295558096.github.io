# JavaWeb

## 简介

- 浏览器和服务器通讯的技术。

------

## 域对象

### PageContext

- 本页中能获取到相关属性。

### Request

- 本次请求中能获取到相关属性。

### Session

- 本次会话中能获取到相关属性。

### Application

- 本个应用中能获取到相关属性。

## 监听器

>Listener。

- Listener 是 JavaWeb 三大组件之一。
- JavaWeb 一种定义了八种监听器。

### ServletContextEvents

>ServletContext 事件。

- javax.servlet.ServletContextListener

  - 监听 ServletContext 生命周期的监听器。
  - 从创建到销毁的过程。
  - 服务器启动创建，做一些初始化操作。

  - 服务器停止销毁，做一些资源关闭的操作。
  - **使用频率最高的监听器**。

- javax.servlet.ServletContextAttributeListener

  - 监听 ServletContext 域中属性变化的监听器。

### HTTPSessionEvents

>HTTPSession 事件。

- javax.servlet.http.HttpSessionListener
  - 监听 HttpSession 的生命周期。
  - Session 第一次使用的时候创建，不是打开浏览器的时候创建。
  - Session 超时或者手动清空 Session 时销毁。
- javax.servlet.http.HttpSessionAttributeListener
  - 监听所有对象在 HttpSession 域中属性的变化的监听器。
- javax.servlet.http.HttpSessionActivationListener
  - 监听某个对象随着 HttpSession 活化钝化的监听器。
  - 不需要在 web.xml 中注册的监听器。
- javax.servlet.http.HttpSessionBindingListener
  - 监听某个对象保存（绑定）到 session 中和从 session 中移除（解绑）的监听器。
  - 不需要在 web.xml 中注册的监听器。

### ServletRequestEvents

>ServletRequest 事件。

- javax.servlet.ServletRequestListener
  - 监听 ServletRequest 的生命周期。
  - 请求进来的时候创建 Request 对象保存请求的详细信息。
  - 请求完成后销毁 Request 对象。
- javax.servlet.ServletRequestAttributeListener
  - 监听 ServletRequest 域中属性变化的监听器。

### 使用流程

- 定义类实现监听器接口。
- 实现监听器逻辑。
- 在 `web.xml` 配置监听器。

## Session

### 简介

xxx

### 生命周期

#### 创建

- 第一次使用 Session 的使用，进行创建。
- 如果根据 `JSESSIONID` 获取不到指定的 Session，服务器会创建一个新的 Session 并返回。

#### 销毁

- 调用 Session 对象的 invalidate 方法。
- 超时自动销毁。

### 活化钝化

- `活化`、`钝化`、`绑定`、`解绑` 都是操作具体的 Session 属性的，属性的类直接实现相关接口。
- Session 中的对象属性想要支持活化或者钝化功能，需要实现 `Serializable` 接口。
- Session 中属性对象随着 Session 一起活化的时候，钝化的对象通过反序列化创建一个一个新的活化对象。

------

## 国际化

### 简介

- 国际化也叫做 `i18n`，是 `Internationalization` 的简称。
- 为了让程序可以适配多国环境，包含计数法、货币单位、日期表示、语言等。

### Locale

#### 简介

- 区域、地点，是代表区域信息的类。
- Locale 类中包含很多静态属性，枚举了全部的区域信息。

#### 格式

- `语言代码_国家代码`。

### ResourceBundle

- 资源绑定，管理了国际化资源文件。
- 通过调用`ResourceBundle.getBundle()`加载获取指定区域信息的国际化配置。
- 通过调用 `ResourceBundle` 对象的 `getString()` 方法，获取到指定 `key` 的配置值。

### 国际化配置文件

- 创建国际化配置文件，`xxx_语言代码_国家代码.properties`。
- 配置文件通过 `key-value` 的方式存储国际化相关的信息。

### 浏览器语言设置

- 设置浏览器的语言信息的项目和优先级可以影响获取到的国际化配置的结果。
- 请求头中的 `Accept-Language` 包含了请求对于区域的优先级排序情况。

### 语言国际化流程

1. 获取到当前浏览器带来的区域信息。
2. 使用 ResourceBundle 管理国际化资源文件。
3. 动态获取配置文件中值。

### 日期格式化

- `DateFormat` 是对**日期**进行格式化的接口。
- 通过调用 `DateFormat.getXXXInstance` 静态方法通过传入**风格**和**区域信息**可以获得对应的格式化器实例。
- 通过调用日期格式化实例对象的 `format`方法，传入日期对象，将日期格式化特定区域的特定风格展示。

### 数字格式化

- `NumberFormat` 是对**数字**进行格式化的接口。
- 使用方式和 `DateFormat` 一样。
- `NumberFormat` 支持对**数字**和**货币**进行格式化操作。

### 自定义消息格式化

- `MessageFormat` 是对自定义消息进行格式化的接口。
- 在国际化配置文件中，通过 `{0}`、`{1}` 的方式预置一个占位符，索引从 0 开始。
- `MessageFormat#format` 方法可以对预置参数的国际化配置传入具体的信息，替换占位符。

### fmt标签库

- `fmt` 标签库是JSTL中定义用来专门为处理国际化信息的标签。
- 通过 `<%@ taglib prefix="fmt" uri="http://java.sun.com/jsp/jstl/fmt" %>`引入标签。
- 通过 `<fmt: setLocale="xxx">` 设置区域信息。
- 通过 `<fmt: setBundle basename="xxx"/>` 设置绑定的国际化配置的基础路径。
- 在需要引用国际化文件的地方使用 `<fmt: message key="xxx"/>` 即可引用对应的配置。
- 通过 `<fmt: param>` 标签给指定的预置国际化配置绑定对应的参数值。

------

## 文件上传下载

- **上传**，浏览器将本地的文件上传到服务器上，交给服务器保存。
- **下载**，浏览器把服务器保存的内容下载来本地。

### 上传

#### 简介

- 文件上传必须是 `POST` 类型的表单提交。
- 文件是以流的方式交给服务器的。
- 文件上传的表单是一种新的解析方式。
- 文件上传不能使用普通的 `<form action="" method="post"/>`表单。
- `request#getParameter`方法不能获取到参数信息。
- `enctype`，`encodeType` 的缩写，编码类型，规定表单数据在发送到服务器之前应该如何编码。
  - `application/x-www-form-urlencoded`
    - 在发送前编码所有字符。
    - 默认。
  - `multipart/form-data` 
    - 不对字符编码。
    - 在使用包含文件上传控件的表单时，必须使用该值。
  - `text/plain`
    - 空格转换为”+”加号，但不对特殊字符编码。
- IE浏览器获取到的文件名是带路径的，需要特殊处理。

#### 表单内容

- 多部件上传文件的表单内容不同于普通的表单内容。

- 格式

  ```xml
  分隔符
  部件1头信息
  空行
  部件1内容
  分隔符
  部件2头信息
  空行
  部件2内容
  ```

  ![image-20220316202932247](image/image-20220316202932247.png)

#### 解析流程

- 导入 `commons-fileupload.jar`、`commons-io.jar`。
- 通过`ServletFileUplead#isMultipartContent(request)`方法用来判断请求是否是**多部件表单请求**。
- 通过 `ServletFileUpload#parseRequest` 解析多部件请求，返回一个包含全部表单项的 `FileItem` 集合。
- 通过 `FileItem#isFormField`方法判断是普通的表单项还是文件项。
- `IOUtil.copy(is, os)` 方法可以进行流拷贝。

### 下载

- 将服务器中文件的流交给浏览器。
- 浏览器收到流之后的默认行为。
  - 认识的打开。
  - 不认识的下载。

#### 流程

- 通过`response#getOutputStream` 方法获取到输出流。

-  设置响应头，让浏览器下载文件，而不是打开。

  ```java
  Response.setHeader("Content-Disposition", "attachment;filename=xxx");
  ```

- 不指定文件名的情况下，文件名默认会使用请求名称。

- 通过 `new String(filename.getBytes("GBK"), "ISO-8859-1");` 给文件名转码解决中文文件名乱码问题。

  - 利用了GBK、UTF-8、ISO8859-1字符集的兼容性解决的问题。
  - 存在轻微漏洞，如果文件名碰巧叫**图片**，会出现乱码。

------

## Tomcat

### 简介

- Apache 下的顶级项目。
- 应用广泛的 Web 服务器。

### 目录结构

- `bin` 可执行程序。
  - Tomcat的各种脚本在这个目录下。
  - `startup.bat`、`startup.sh` 启动脚本。
  - `shutdown.bat`、`shutdown.sh` 关闭脚本。
- `conf` Tomcat配置文件的目录。
  - `server.xml` Tomcat服务器配置集中的地方。
  - `web.xml` Tomcat 所有 web 项目都需要遵循的一个 xml。
- `lib` Tomcat运行期间需要使用的 jar 包。
- `logs` 运行日志。
- `temp` 临时目录。
- `webapps` 运行项目的集合。
  - 每一个项目都是一个独立的文件夹的形式。
- `work` 存在运行期间编译的一些文件。
  - JSP页面。

------

## Servlet

- Servlet 是 Java 规范中的一个接口，是 JavaWeb 的三大组件之一。
- 一般不直接使用 Servlet 接口，通过实现 `HttpServlet` 并实现 `doGet`、`doPost` 方法来实现 `Servlet` 的功能。

### 域对象

- PageContext。
- ServletRequest。
- HttpSession。
- ServletContext。

### 使用

-  编写 Servlet 类。
- 在 `web.xml` 中进行配置，声明拦截器及其工作的映射路径。

### 重点

- 一个 `Servlet` 对应一个 `ServletConfig` 封装消前 Servlet 的配置信息。
- ServletContext，是四大域对象之一，在域中保存一些数据，跨页面获取到保存的数据。
  - 域对象。
  - 持有当前项目的信息。
  - 一个项目对应一个 ServletContext，只是代表当前项目。

### HttpServletRequest

- Servlet 请求对象，是浏览器的请求数据，包含请求头和请求体。
- 每一次发送请求，请求的详细信息会被 Tomcat 息动封装成个 Request 对象。

#### 作用

- `request.getParameter("xxx")` 获取请求参数。
- 作为域对象保存数据，同一次请求期间可以共享数据。
- `request.getSession()` 获取到 HttpSession 对象。

#### 格式

```json
请求首行
请求头
空行
请求体
```

### HttpServletResponse

- Servlet 响应对象，是交给浏览器解析的数据，包含响应头，响应首行，响应体。

#### 作用

- 交给浏览器的数据，服务器给浏览器发送数据，都是写出字符流或者字节流。
  - 响应首行。
  - 响应头，对浏览器的一些命令。
  - 响应体，浏览器收到要解析的数据。
- 进行请求重定向，`response.sendRedirect("重定向的地址")`。
  - 路径需要以 `/` 开始，但是不能以 `/` 结束。
  - 浏览器会受到一次**302**响应，并再次请求新地址，相当于两次不同的请求。
- 对请求进行转发，`request.getRequestDispatcher("/index.jsp").forward(request,response)`。
  - 将当次请求和响应交给另外一个程序处理，是在服务器内部发送的，浏览器感知不到。

#### 格式

```json
响应首行
响应头
空行
响应体
```

### 乱码问题

#### 请求乱码

**GET请求**

##### 原因

- Tomcat 服务器默认的编解码和解码格式都是 `ISO8859-1`。
- 浏览器发送给服务器的数据，服务器收到解析出来乱码。
- 所有的请求参数是带在 url地址上的，Tomcat 收到这个请求就会**调用默认的编解码格式将其解码完成**，并封装到 `HttpServletRequest` 对象中。

##### 解决方案

- 修改 Tomcat 服务器的配置文件，在 `server.xml` 中找到 8080 端口的配置处，添加 `URIEncoding="UTF-8"`，**推荐**。
- `new String (xxx.getBytes("ISO-8859-1"), "UTF-8");`，通过重新解码和编码解决乱码。

**POST请求**

##### 原因

- POST 请求带来的数据都在**请求体**中存放，不会立即被 Tomcat 服务器解析。
- 只有当通过 `request.getParamter("xxx")`方法获取参数时，Tomcat 才将整个请求体使用默认编码格式全部解析完成。

##### 解决方案

- 通过给服务器配置 `CharsetEncodingFilter` 过滤器，**推荐**。
- 在获取 POST 请求参数之前，设置 `request.setcharacterEncoding ("utf-8");`。

#### 响应乱码

##### 原因

- 直接写出去的数据，浏览器并不知道数据的内容类型以及编码格式等。
- 浏览器会使用默认的格式解析响应信息。

#### 解决方案

- 通过设置 `response.setContentType("text/html;charset=utf-8");`。
- charset 指定的是字符集的编码格式，只针对字符生效。如果响应体是字节流，不需要设置 charset。

### 路径问题

#### 相对路径

- 不以 `/` 开始的路径就是相对路径。
- 相对于当前资源所在的路径为标准。
- 相对路径存在参考标准问题，不建议使用。

#### 虚拟绝对路径

- 在程序中，以 `/` 开始的路径。
- 推荐写绝对路径，加上项目名。

##### 服务器解析的路径

- **就是当前项目下的地址，会自动加上项目名。**

##### 浏览器解析的路径

- **浏览器在解析的时候会在前面拼接上服务器的地址。**
- **页面要写以 `/` 开始的路径需要加上项目名。**

##### 区分

- **区分服务器解析的路径和浏览器解析的路径，决定在 / 后面是否追加项目名。**
- `response.sendRedriect("/xxx")`，因为这个路径是要浏览器访问的，需要加上项目名。
- 只要是 **html** 标签里面写的路径都是浏览器解析，除此之外都是服务器解析。

#### 绝对路径

- 指定磁盘地址、服务器地址的路径。

### 其他

- `org.apache.catalina.servlets.DefaultServlet` 是 Tomcat 默认生效的 Servlet，用来处理静态资源的。

------

## JSP

### 简介

- JSP（全称JavaServer Pages）是由 Sun Microsystems 公司主导创建的一种**动态网页**技术标准。
- JSP部署于网络服务器上，可以响应客户端发送的请求，并根据请求内容动态地生成HTML、XML或其他格式文档的Web网页，然后返回给请求者。
- `org.apache.jasper.servlet.JspServlet`，解析 JSP 页面的 Servlet。

### 资源分类

#### 静态请求

- html 页面。
- css。
- img。
- 视频。

#### 动态请求

- Java 代码（servlet 程序）。

### 处理流程

1. 向服务器发送请求 `xxx.jsp`。
2. `JspServlet` 拦截到请求。
3. JspServlet 找到 `xxx.jsp` 文件，第一次请求，将其翻译成 `xxx_jsp.java`，编译成 `index_jsp.class`，后续请求直接调用对应的 `.class` 文件。
4. 利用反射调用 class 文件中的 `jspService` 方法。
5. 将 jsp 文件中的数据通过响应流写出去。

### 原理

- `.jsp`文件在被编译之后会成为一个 `.class` 文件。

### 九大隐含对象

#### 四个域对象

- PageContext
  - 当前页面共享的数据在当前页面能取出来。
- Request
  - 同一次请求共享的数据，同一请求期间可以共享（转发、重定向）。
  - 一但 response 了响应就完成，当次请求就结束了。
- Session。
  - 同一次会话期间数据共享。
  - 浏览器打开开始会话。
  - 浏览器关闭结束会话。
- Application。
  - 同一个 web 应用中共享数据。
  - 只要服务器不关都可以使用。

#### 输入输出隐含对象

- response
  - HttpServletResponse。
  - 处理JSP生成的响应，然后将响应结果发送给客户端。
  - service方法的 response 参数。
- out
  - JspWriter。
  - 表示输出流，将作为请求的响应发送给客户端。
  - 是 PrintWriter 的一个实例。

#### Servlet对象

- page
  - 提供对网页上定义的所有对象的访问。
  - 属于一个Object对象，是此 Servlet 的 一个引用。如 `Object page = this;`。
- config
  - ServletConfig。
  - 存储 Servlet 的一些初始信息是 ServletConfig 的一个实例。

#### 异常对象

- exception
  - Throwable。
  - 此对象负责处理程序执行过程中引发的异常。

### JSTL

- Java server pages standarded tag library，即JSP标准标签库。
- 主要提供给 Java Web 开发人员一个标准通用的标签库。
- 利用 JSTL 的标签取代 JSP 页面上的 Java 代码，从而提高程序的可读性，降低程序的维护难度。

### EL

- 简化取值操作的。
- el 表达式 只取出 10 个对象中的值。

#### 取值

- 四个域对象
  - PageScope
  - ReguestScope
    - 请求域是 Request 对象中的一个 map 属性。
  - SessionScope
  - ApplicationScope
- 其他
  - Param
    - 获取请求参数的。
  - ParamValues
    - 获取请求参数的，复选框，一个参数名对应多个值。
  - Header
    - 获取请求头。
  - HeaderValues
    - 获取多个值的请求头。
  - Cookie
    - 获取 cookie。
  - InitParam
    - 获取 `web.xml` 中配置的 `context-param` 初始化参数。

------

## Cookie

### 简介

- 浏览器端保存少量数据的一种技术。

### 特点

- 每个 cookie 不能超过 4KB。
- cookie 的值都是纯文本。
- 默认不支持中文，保存中文需要对内容进行编码。
- **保存的当前网站的 cookie 在每次请求访问这个网站时都会携带。**

### 使用

#### 设置 cookie

- 服务器通过调用两个参数的构造器创建 Cookie 对象，得到一个 cookie 实例。
- 调用 cookie 对象的方法，可以添加属性。
- 调用 `Response` 对象的 `addCookie` 方法，向浏览器写入 cookie。
- 浏览器收到的响应中，在响应头中会包含一个 `Set-Cookie` 的命令让浏览器设置对应的 cookie。

#### 读取 cookie

- 通过 `request#getCookies` 获取全部的 cookie。

### 有效时间

- 默认有效时间是会话级别的，浏览器关闭，cookie 失效。
- 通过 `setMaxAge` 方法设置 `cookie` 的有效时间，单位为秒。
  - 正数表示多少秒超时。
  - 负数表示随浏览器关闭而失效。
  - 0 表示 cookie 立即失效。
- 通过同名 cookie 覆盖，修改 cookie 的有效期时间。

## Session

### 简介

- 服务器端保存当前会话大量数据的一种技术。
- Session 保存的数据可以在同一次会话期间共享。

### 核心

- 浏览器和服务器进行交互期间，可能需要保存一些数据，服务器就为每个会话专门创建一个用来保存数据的 map，这个 map 就是 session。
- 每个会话拥有独立的一个 map，在每次创建 map 的时候，会有一个唯一标识，JSESSIONID。
- session 在服务器首次通过 `request.getSession` 获取 Session 对象的时候被创建。

## 会话机制

### 核心原理

- Session生成的唯一标识 JSESSIONID 被设置到浏览器的Cookie中，key 为 JSESSIONID。
- 浏览器每次发起请求都会携带全部的 Cookie。

### 会话关闭

- cookie 过期或者失效，可以通过 Cookie 持久化技术找到之前对应的 Session。
- session 过期或者失效，不可找回，和 session 的有效期有关。

### 令牌机制

- 客户端携带的参数或者cookie的值与服务器端Session的值进行比对，如果一致，认定为请求合法。
