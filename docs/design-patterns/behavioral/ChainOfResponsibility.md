# 责任链模式

>ChainOfResponsibility。

## 定义

xx

## 类图

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



