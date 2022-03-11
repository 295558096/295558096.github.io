# JavaWeb

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
  - 服务器启动创建。
  - 服务器停止销毁。
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
- javax.servlet.http.HttpSessionBindingListener
  - 监听某个对象保存（绑定）到 session 中和从 session 中移除（解绑）的监听器。

### ServletRequestEvents

>ServletRequest 事件。

- javax.servlet.ServletRequestListener
  - 监听 ServletRequest 的生命周期。
  - 请求进来的时候创建 Request 对象保存请求的详细信息。
  - 请求完成后销毁 Request 对象。
- javax.servlet.ServletRequestAttributeListener
  - 监听 ServletRequest 域中属性变化的监听器。