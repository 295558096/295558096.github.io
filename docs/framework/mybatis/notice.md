# 注意事项

>mybatis使用过程中遇到问题的一些总结。

## 批量插入

>实现自增主键数据的批量插入，并且可以正常回填主键。

### 官网文档地址

[批量插入](https://mybatis.org/mybatis-3/zh/sqlmap-xml.html)

### 解决方案

#### 不指定param参数

- **接口参数不加@Param注解。**

- collection的值固定写为list。
- 注意拼接后SQL的长度导致的执行报错。

```xml
<insert id="insertAuthor" useGeneratedKeys="true"
    keyProperty="id">
  insert into Author (username, password, email, bio) values
  <foreach item="item" collection="list" separator=",">
    (#{item.username}, #{item.password}, #{item.email}, #{item.bio})
  </foreach>
</insert>
```

#### 使用SQL Session

- mybatis推荐的方式，可以避免单条SQL超长的问题。

```java
public class Test() {
    public void doBatchInsert(List<User> list) {
        SqlSession sqlSession = null;
        try {
            sqlSession = sqlSessionFactory.openSession(ExecutorType.BATCH, false);
            for (User user : list) {
                userMapper.insert(user);
            }
            sqlSession.commit();
        } catch (Exception e) {
            e.printStackTrace();
            sqlSession.rollback();
        } finally {
            if (sqlSession != null) {
                sqlSession.close();
            }
        }
    } 
}
```

