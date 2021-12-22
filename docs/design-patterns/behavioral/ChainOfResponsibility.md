# 责任链模式

>ChainOfResponsibility。

## 定义

- 责任链模式为请求创建了一个接收者对象的链。这种模式给予请求的类型，对请求的发送者和接收者进行解耦。这种类型的设计模式属于行为型模式。

- 在这种模式中，通常每个接收者都包含对另一个接收者的引用。如果一个对象不能处理该请求，那么它会把相同的请求传给下一个接收者，依此类推。

## 应用

- Structs的Filter和Spring的拦截器都是责任链模式的具体实现。

## 类图

![责任链模式的 UML 图](image/2021-chain-of-responsibility.svg)



## 代码

### 接口

```java
public interface Filter {

    boolean doFilter(MsgInfo info);

}
```

### Filter实现类

```java
public class FaceFilter implements Filter {

    @Override
    public boolean doFilter(MsgInfo info) {
        if (info.getMsg().contains("***")) {
            return false;
        }
        info.setMsg(info.getMsg().replace("T-T", "(^v^)"));
        return true;
    }
}

public class HtmlFilter implements Filter {

    @Override
    public boolean doFilter(MsgInfo info) {
        if (info.getMsg().contains("<script>")) {
            return false;
        }
        info.setMsg(info.getMsg().replace("<html>", "【HTML】"));
        return true;
    }
}

public class KeyWordFilter implements Filter {

    @Override
    public boolean doFilter(MsgInfo info) {
        if (info.getMsg().contains("mayun")) {
            return false;
        }
        info.setMsg(info.getMsg().replace("996", "955"));
        return true;
    }
}
```

### 责任链

```java
public class FilterChain implements Filter {

    List<Filter> filters = new ArrayList<>();

    public FilterChain addFilter(Filter filter) {
        filters.add(filter);
        return this;
    }

    @Override
    public boolean doFilter(MsgInfo info) {
        for (Filter filter : filters) {
            if (!filter.doFilter(info)) {
                return false;
            }
        }
        return true;
    }
}
```

### 客户端调用代码

```java
public class CORTest {

    public static void main(String[] args) {

        String msg = "程序员996真的好不开心啊，想哭T-T，<html>!!mayun!!";
        MsgInfo msgInfo = new MsgInfo(msg);
        FilterChain f1 = new FilterChain();
        f1.addFilter(new HtmlFilter()).addFilter(new KeyWordFilter());
        FilterChain f2 = new FilterChain();
        f2.addFilter(new FaceFilter());
        f2.addFilter(f1);
        f2.doFilter(msgInfo);
        System.out.println(msgInfo.getMsg());
    }

}
```



## ServletFilter代码实现

### Filter接口及实现

```java
public interface Filter {

    void doFilter(Request request, Response response, FilterChain chain);

}

public class DBFilter implements Filter {

    @Override
    public void doFilter(Request request, Response response, FilterChain chain) {
        System.out.println("deal request by db");
        chain.doFilter(request, response);
        System.out.println("deal response by db");

    }
}

public class HtmlFilter implements Filter {

    @Override
    public void doFilter(Request request, Response response, FilterChain chain) {
        System.out.println("deal request by html");
        chain.doFilter(request, response);
        System.out.println("deal response by html");

    }
}

public class MsgFilter implements Filter {

    @Override
    public void doFilter(Request request, Response response, FilterChain chain) {
        System.out.println("deal request by msg");
        if ("error".equals(request.msg)) {
            return;
        }
        chain.doFilter(request, response);
        System.out.println("deal response by msg");

    }
}
```



### FilterChain接口及实现

```java
public interface FilterChain {

    void doFilter(Request request, Response response);

}

public class CommonFilterChain implements FilterChain {

    int index = 0;

    List<Filter> filterList = new ArrayList<>();

    public CommonFilterChain addFilter(Filter filter) {
        this.filterList.add(filter);
        return this;
    }

    @Override
    public void doFilter(Request request, Response response) {
        if (index + 1 > filterList.size()) {
            return;
        }
        Filter f = filterList.get(index);
        index++;
        f.doFilter(request, response, this);
    }
}
```

### Request类和Response类

```java
public class Request {

    String msg;

}

public class Response {

    String msg;

}
```

### 客户端调用

```java
public class ServletFilterTest {

    public static void main(String[] args) {
        Request request = new Request();
        request.msg = "error";
        Response response = new Response();
        CommonFilterChain chain = new CommonFilterChain();
        chain.addFilter(new HtmlFilter()).addFilter(new MsgFilter()).addFilter(new DBFilter());
        chain.doFilter(request, response);
    }

}
```

