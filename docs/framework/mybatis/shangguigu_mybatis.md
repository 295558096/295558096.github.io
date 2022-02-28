# MyBatis

## 简介

- 专注于和数据库进行交互。
- 半自动的持久化层、SQL映射框架。

## 工具&框架

- 工具是一些功能的简单封装。
- 框架是某个领域的整体解决方案。

## JDBC

### 查询SQL流程

1. 获取连接。
2. 获取 `PreparedStatement`。
3. 编写 SQL。
4. 预编译参数。
5. 执行 SQL。
6. 封装结果。

### 缺点

- 使用麻烦。
- SQL语句硬编码在代码中，不够解耦。

## Hibernate

- ORM框架，对象关系映射，可以映射对象和表的关系。
- Hibernate 是一个**全映射、全自动的框架**。
- 不需要编写SQL，缺点也是不能自由定制SQL。
- HQL 存在额外的学习成本。

## 优点

- MyBatis 将重要的步骤抽取出来可以人工定制，其他步骤自动化。
- 重要步骤都是写在配置文件中，便于维护。
- 完全解决数据库的优化问题。
- MyBatis 底层就是对原生 JDBC 的一个简单封装，性能比较高。

## HelloWorld

- 创建数据库、表。

- 编写实体类、dao。

- 导入jar包。

  - mybatis-3.x.x.jar。
  - 数据库驱动包。
  - 需要 `mybatis` 打印日志的话，导入 `log4j` 日志包。

- 写配置。

  - 编写 `Mybatis` 全局配置文件，指导 `Mybatis` 如何初始化特性。从 XML 中构建 `SqlSessionFactory`。

  - 编写**SQL映射文件**，实现接口中具体方法对应的SQL语句。

  - **在全局配置文件中注册编写好的SQL映射文件**。

    ```xml
    <mappers>
      <mapper resource="org/mybatis/example/BlogMapper.xml"/>
    </mappers>
    ```

- 调用查询。

### 总结

- 获取到的`Dao`对象是接口的代理对象，是 Mybatis 帮我们自动生成的。

## SQL映射文件

- **增、删、改方法不用写返回值类型。Mybatis会根据方法的返回值类型自动封装。**

  - 如果是 `int` 或者 `long` 类型，返回受影响的行数。
  - 如果是 `boolean`，影响0行返回`false`，否则返回`true`。

- 通过`#{属性名}`取出参数中某个参数的值。

- `insert`、`update`、`delete` 标签可以引用的属性。

  | 属性                 | 描述                                                         |
  | -------------------- | ------------------------------------------------------------ |
  | **id**               | 在命名空间中唯一的标识符。                                   |
  | parameterType        | 将会传入这条语句的参数的类全限定名或别名。这个属性是可选的，因为 MyBatis 可以通过类型处理器（TypeHandler）推断出具体传入语句的参数，默认值为未设置（unset）。 |
  | parameterMap         | 用于引用外部 parameterMap 的属性，目前已被废弃。请使用行内参数映射和 parameterType 属性。 |
  | flushCache           | 将其设置为 true 后，只要语句被调用，都会导致本地缓存和二级缓存被清空，默认值：（对 insert、update 和 delete 语句）true。 |
  | **timeout**          | 这个设置是在抛出异常之前，驱动程序等待数据库返回请求结果的秒数。默认值为未设置（unset）（依赖数据库驱动）。 |
  | statementType        | 可选 STATEMENT，PREPARED 或 CALLABLE。这会让 MyBatis 分别使用 Statement，PreparedStatement 或 CallableStatement，默认值：PREPARED。 |
  | **useGeneratedKeys** | （仅适用于 insert 和 update）这会令 MyBatis 使用 JDBC 的 getGeneratedKeys 方法来取出由数据库内部生成的主键（比如：像 MySQL 和 SQL Server 这样的关系型数据库管理系统的自动递增字段），默认值：false。 |
  | **keyProperty**      | （仅适用于 insert 和 update）指定能够唯一识别对象的属性，MyBatis 会使用 getGeneratedKeys 的返回值或 insert 语句的 selectKey 子元素设置它的值，默认值：未设置（`unset`）。如果生成列不止一个，可以用逗号分隔多个属性名称。 |
  | keyColumn            | （仅适用于 insert 和 update）设置生成键值在表中的列名，在某些数据库（像 PostgreSQL）中，当主键列不是表中的第一列的时候，是必须设置的。如果生成列不止一个，可以用逗号分隔多个属性名称。 |
  | **databaseId**       | 如果配置了数据库厂商标识（databaseIdProvider），MyBatis 会加载所有不带 databaseId 或匹配当前 databaseId 的语句；如果带和不带的语句都有，则不带的会被忽略。 |

### 参数取值

#### 单个参数

- `基本类型` `#{任意值}`
- `POJO`

#### 多个参数

- 使用 `0,1,2`参数索引信息。
- 使用 `param1，param2` 参数别名。
- 只要传入了多个参数，mybatis 会自动的将这些参数封装在一个 map 中，**默认的 key 就是参数位置的索引**。

#### 命名参数

- 通过 `@Param` 注解标记参数封装的 **key**，命名参数。
- 通过 `#{命名值}` 获取对应参数。

#### map

- 通过 `#{key}` 获取参数值。

#### POJO

- 通过 `#{POJO的属性名}` 获取参数值。

### 参数处理

- 通过 `JavaType` 指明参数的 Java 类型。
- 通过 `JdbcType` 指明参数转换成的 JDBC 数据类型，默认不指定，oracle 数据库不能处理 null，需要指定 `JdbcType`。

### 取值方式

#### #{属性名} 

- 预编译的方式。
- 参数对应位置都用 `？`占位，在预编译参数的时候，替换成对应的参数。
- **安全，可以避免 SQL 注入问题。**

#### ${属性名} 

- 不是预编译的方式。
- **直接将参数和SQL语句进行拼串**。
- **不安全，可能存在 SQL 注入问题。**
- **SQL 语句只有参数位置支持预编译，`表名`、`order by` 等处不支持预编译，只能通过 `${}`进行赋值。**

### 返回值 Map

#### 单条记录

- `key` 是列名。
- `value` 是列的值。

#### 多条记录

- 通过 `@MapKey`注解在方法上指定结果 Map 的 key 使用结果中的哪个属性。
- SQL映射文件中的方法的 `resultType` 写的是 map 中元素对象的类型。

### nameSpace

- 名称空间。
- 写接口的全类名，相当于告诉 MyBatis 这个配置文件是实现哪个接口的。
- 框架初始化的时候，根据全类名为接口绑定具体的SQL语句。

### select

- 用来定义一个查询操作。
- `id` 指定方法名，相当于这个配置是对于某个方法的实现。
- `resultType` 指定方法运行后的返回值类型，查询操作必须指定。

### insert

- 插入语句标签。
- 通过 `useGeneratedKeys` 和 `keyProperty` 获取到自增主键并封装到指定的属性中，**底层实现是调用JDBC原生的接口**。
- 通过 `selectKey` 标签**查询到非自增的主键的值**，其中 `order` 属性指定运行的时机。

### update

- 更新语句标签。

### delete

- 删除语句标签。

### cache

- 缓存相关。

### cache-ref

- 缓存引用相关。

### parameterMap

- 已经废弃。
- 来做复杂参数映射。

### resultMap

- 结果映射，自定义结果集的封装规则。
- 增删改查的方法中，通过指定 `resultMap` 属性绑定自定义的映射规则。
- `<resultType id>`属性为结果Map起一个唯一的别名，便于后续引用。
- `<resultType type>` 属性指定为哪一个 `JavaBean` 对象创建结果映射。
- `id` 子标签，为主键字段创建映射规则。
  - `column` 指定数据库中的主键的列名。
  - `property` 指定 JavaBean 的属性名。
- `result` 子标签，为普通字段创建映射规则。
  - `column` 指定数据库中的列名。
  - `property` 指定 JavaBean 的属性名。

#### 级联映射

- 级联映射，属性的属性。
- 可以通过 result 子标签指定 `属性.属性` 的方式，完成对级联属性的赋值。
- JavaBean中的一个属性也是 JavaBean 的情况下，通过 `association` 子标签定义级联属性的映射规则，推荐。
  - `<association property>` 指定级联属性在结果集中对应的是属性名称。
  - `<association javaType>` 指定级联属性的 Java 类型。
  - `<association select>` 指定通过哪个唯一的标识语句进行级联数据查询。
  - `<association colomn>` 指定查询级联属性的时候，使用的参数。
    - 传值多个的情况下使用 `{key=列名, key=列名}` 的方式。
  - `<association fetchType>` 指定加载时机，可以覆盖全局配置。
    - `eager` 不延迟。
    - `lazy` 延迟。 
- 通过开启全局延迟加载策略，`lazyLoadingEnabled=true、aggressiveLazyLoading=flase`，可以似的Mybatis在使用到级联属性的时候，才执行分布的SQL语句。

#### 集合属性

- 通过 `collection` 子标签定义 `JavaBean` 的集合属性的封装规则。
  - `<collection property>` 指定集合属性在结果集中对应的是属性名称。
  - `<collection ofType>` 指定集合属性元素的 Java 类型。
  - `<collection select>` 指定通过哪个唯一的标识语句进行关联集合数据查询。
  - `<collection colomn>` 指定查询关联集合属性的时候，使用的参数。
    - 传值多个的情况下使用 `{key=列名, key=列名}` 的方式。
- 通过开启全局延迟加载策略，`lazyLoadingEnabled=true、aggressiveLazyLoading=flase`，可以似的Mybatis在使用到集合属性的时候，才执行分布的SQL语句。

### resultType

- 指明查询到的结果和哪个 JavaBean 映射，使用默认的映射规则。
- 默认的映射规则是属性、列名一一对应。

### sql

- 抽取可重用的 sql 片段。

### 动态SQL

- 动态 SQL 是 MyBatis 强大特性之一，极大简化了拼装 SQL 的操作。
- 动态 SQL 元素和使用 `JSTL` 或其他类似基于 `XML` 的文本处理器相似。
- MyBatis 采用功能强大的基于 `OGNL` 的表达式来简化操作。
  - **if**
  - **choose** `when`、`otherwise`
  - **trim** `where`、`set`
  - **foreach**

------

## SqlSession&SqlSessionFactory

- 每个基于 MyBatis 的应用都是以一个 `SqlSessionFactory` 的实例为核心的。
- `SqlSessionFactory` 是 `SqlSession` 工厂，负责创建 `SqlSession` 对象。
- `SqlSession` 代表一次和数据库的会话。

### 根据全局配置文件获取 SqlSessionFactory

```java
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
```

### 获取 SqlSession

- `sqlSessionFactory.openSession()`。
- `sqlSession.getMapper()`获取到Mapper映射文件。

## 全局配置文件

>指导 Mybatis 正确运行的全局配置的文件

Mybatis 的顶级节点属性是 `configuration`，下面可以配置众多标签。

### properties

- 定义属性配置。
- 类似于 Spring 的 `context-place-holder`，可以引用外部的配置文件的。这些属性可以在外部进行配置，并可以进行动态替换。
- 既可以在典型的 Java 属性文件中配置这些属性，也可以在 properties 元素的子元素中设置。

```xml
<properties resource="org/mybatis/example/config.properties">
  <property name="username" value="${user}"/>
  <property name="password" value="${pwd}"/>
</properties>
```

### settings

- 定义设置信息。
- 极为重要的调整设置，会改变 MyBatis 的运行时行为。

| 设置名                           | 描述                                                         | 有效值                                                       | 默认值                                                |
| -------------------------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ----------------------------------------------------- |
| cacheEnabled                     | 全局性地开启或关闭所有映射器配置文件中已配置的任何缓存。     | true \| false                                                | true                                                  |
| lazyLoadingEnabled               | 延迟加载的全局开关。当开启时，所有关联对象都会延迟加载。 特定关联关系中可通过设置 `fetchType` 属性来覆盖该项的开关状态。 | true \| false                                                | false                                                 |
| aggressiveLazyLoading            | 开启时，任一方法的调用都会加载该对象的所有延迟加载属性。 否则，每个延迟加载属性会按需加载（参考 `lazyLoadTriggerMethods`)。 | true \| false                                                | false （在 3.4.1 及之前的版本中默认为 true）          |
| multipleResultSetsEnabled        | 是否允许单个语句返回多结果集（需要数据库驱动支持）。         | true \| false                                                | true                                                  |
| useColumnLabel                   | 使用列标签代替列名。实际表现依赖于数据库驱动，具体可参考数据库驱动的相关文档，或通过对比测试来观察。 | true \| false                                                | true                                                  |
| useGeneratedKeys                 | 允许 JDBC 支持自动生成主键，需要数据库驱动支持。如果设置为 true，将强制使用自动生成主键。尽管一些数据库驱动不支持此特性，但仍可正常工作（如 Derby）。 | true \| false                                                | False                                                 |
| autoMappingBehavior              | 指定 MyBatis 应如何自动映射列到字段或属性。 NONE 表示关闭自动映射；PARTIAL 只会自动映射没有定义嵌套结果映射的字段。 FULL 会自动映射任何复杂的结果集（无论是否嵌套）。 | NONE, PARTIAL, FULL                                          | PARTIAL                                               |
| autoMappingUnknownColumnBehavior | 指定发现自动映射目标未知列（或未知属性类型）的行为。`NONE`: 不做任何反应`WARNING`: 输出警告日志（`'org.apache.ibatis.session.AutoMappingUnknownColumnBehavior'` 的日志等级必须设置为 `WARN`）`FAILING`: 映射失败 (抛出 `SqlSessionException`) | NONE, WARNING, FAILING                                       | NONE                                                  |
| defaultExecutorType              | 配置默认的执行器。SIMPLE 就是普通的执行器；REUSE 执行器会重用预处理语句（PreparedStatement）； BATCH 执行器不仅重用语句还会执行批量更新。 | SIMPLE REUSE BATCH                                           | SIMPLE                                                |
| defaultStatementTimeout          | 设置超时时间，它决定数据库驱动等待数据库响应的秒数。         | 任意正整数                                                   | 未设置 (null)                                         |
| defaultFetchSize                 | 为驱动的结果集获取数量（fetchSize）设置一个建议值。此参数只可以在查询设置中被覆盖。 | 任意正整数                                                   | 未设置 (null)                                         |
| defaultResultSetType             | 指定语句默认的滚动策略。（新增于 3.5.2）                     | FORWARD_ONLY \| SCROLL_SENSITIVE \| SCROLL_INSENSITIVE \| DEFAULT（等同于未设置） | 未设置 (null)                                         |
| safeRowBoundsEnabled             | 是否允许在嵌套语句中使用分页（RowBounds）。如果允许使用则设置为 false。 | true \| false                                                | False                                                 |
| safeResultHandlerEnabled         | 是否允许在嵌套语句中使用结果处理器（ResultHandler）。如果允许使用则设置为 false。 | true \| false                                                | True                                                  |
| mapUnderscoreToCamelCase         | 是否开启驼峰命名自动映射，即从经典数据库列名 A_COLUMN 映射到经典 Java 属性名 aColumn。 | true \| false                                                | False                                                 |
| localCacheScope                  | MyBatis 利用本地缓存机制（Local Cache）防止循环引用和加速重复的嵌套查询。 默认值为 SESSION，会缓存一个会话中执行的所有查询。 若设置值为 STATEMENT，本地缓存将仅用于执行语句，对相同 SqlSession 的不同查询将不会进行缓存。 | SESSION \| STATEMENT                                         | SESSION                                               |
| jdbcTypeForNull                  | 当没有为参数指定特定的 JDBC 类型时，空值的默认 JDBC 类型。 某些数据库驱动需要指定列的 JDBC 类型，多数情况直接用一般类型即可，比如 NULL、VARCHAR 或 OTHER。 | JdbcType 常量，常用值：NULL、VARCHAR 或 OTHER。              | OTHER                                                 |
| lazyLoadTriggerMethods           | 指定对象的哪些方法触发一次延迟加载。                         | 用逗号分隔的方法列表。                                       | equals,clone,hashCode,toString                        |
| defaultScriptingLanguage         | 指定动态 SQL 生成使用的默认脚本语言。                        | 一个类型别名或全限定类名。                                   | org.apache.ibatis.scripting.xmltags.XMLLanguageDriver |
| defaultEnumTypeHandler           | 指定 Enum 使用的默认 `TypeHandler` 。（新增于 3.4.5）        | 一个类型别名或全限定类名。                                   | org.apache.ibatis.type.EnumTypeHandler                |
| callSettersOnNulls               | 指定当结果集中值为 null 的时候是否调用映射对象的 setter（map 对象时为 put）方法，这在依赖于 Map.keySet() 或 null 值进行初始化时比较有用。注意基本类型（int、boolean 等）是不能设置成 null 的。 | true \| false                                                | false                                                 |
| returnInstanceForEmptyRow        | 当返回行的所有列都是空时，MyBatis默认返回 `null`。 当开启这个设置时，MyBatis会返回一个空实例。 请注意，它也适用于嵌套的结果集（如集合或关联）。（新增于 3.4.2） | true \| false                                                | false                                                 |
| logPrefix                        | 指定 MyBatis 增加到日志名称的前缀。                          | 任何字符串                                                   | 未设置                                                |
| logImpl                          | 指定 MyBatis 所用日志的具体实现，未指定时将自动查找。        | SLF4J \| LOG4J(deprecated since 3.5.9) \| LOG4J2 \| JDK_LOGGING \| COMMONS_LOGGING \| STDOUT_LOGGING \| NO_LOGGING | 未设置                                                |
| proxyFactory                     | 指定 Mybatis 创建可延迟加载对象所用到的代理工具。            | CGLIB \| JAVASSIST                                           | JAVASSIST （MyBatis 3.3 以上）                        |
| vfsImpl                          | 指定 VFS 的实现                                              | 自定义 VFS 的实现的类全限定名，以逗号分隔。                  | 未设置                                                |
| useActualParamName               | 允许使用方法签名中的名称作为语句参数名称。 为了使用该特性，你的项目必须采用 Java 8 编译，并且加上 `-parameters` 选项。（新增于 3.4.1） | true \| false                                                | true                                                  |
| configurationFactory             | 指定一个提供 `Configuration` 实例的类。 这个被返回的 Configuration 实例用来加载被反序列化对象的延迟加载属性值。 这个类必须包含一个签名为`static Configuration getConfiguration()` 的方法。（新增于 3.2.3） | 一个类型别名或完全限定类名。                                 | 未设置                                                |
| shrinkWhitespacesInSql           | 从SQL中删除多余的空格字符。请注意，这也会影响SQL中的文字字符串。 (新增于 3.5.5) | true \| false                                                | false                                                 |
| defaultSqlProviderType           | Specifies an sql provider class that holds provider method (Since 3.5.6). This class apply to the `type`(or `value`) attribute on sql provider annotation(e.g. `@SelectProvider`), when these attribute was omitted. | A type alias or fully qualified class name                   | Not set                                               |
| nullableOnForEach                | Specifies the default value of 'nullable' attribute on 'foreach' tag. (Since 3.5.9) | true \| false                                                | false                                                 |

```xml
<settings>
  <setting name="cacheEnabled" value="true"/>
  <setting name="lazyLoadingEnabled" value="true"/>
  <setting name="multipleResultSetsEnabled" value="true"/>
  <setting name="useColumnLabel" value="true"/>
  <setting name="useGeneratedKeys" value="false"/>
  <setting name="autoMappingBehavior" value="PARTIAL"/>
  <setting name="autoMappingUnknownColumnBehavior" value="WARNING"/>
  <setting name="defaultExecutorType" value="SIMPLE"/>
  <setting name="defaultStatementTimeout" value="25"/>
  <setting name="defaultFetchSize" value="100"/>
  <setting name="safeRowBoundsEnabled" value="false"/>
  <setting name="mapUnderscoreToCamelCase" value="false"/>
  <setting name="localCacheScope" value="SESSION"/>
  <setting name="jdbcTypeForNull" value="OTHER"/>
  <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>
</settings>
```

------

### typeAliases

- 设置**类型别名**配置。

- 为常用的 Java 类型起别名。

- **别名默认是类名，不区分大小写。**

- `@Aliase` 通过注解给指定的 JavaBean 起别名。

- 指定别名。

  ```xml
  <typeAliases>
    <typeAlias alias="Author" type="domain.blog.Author"/>
    <typeAlias alias="Blog" type="domain.blog.Blog"/>
    <typeAlias alias="Comment" type="domain.blog.Comment"/>
    <typeAlias alias="Post" type="domain.blog.Post"/>
    <typeAlias alias="Section" type="domain.blog.Section"/>
    <typeAlias alias="Tag" type="domain.blog.Tag"/>
  </typeAliases>

- 批量起别名。

  ```xml
  <typeAliases>
    <package name="domain.blog"/>
  </typeAliases>
  ```

#### 默认别名

- Mybatis为常见的 Java 类型内建的类型别名。

- 都是不区分大小写的。

  | _byte      | byte       |
  | ---------- | ---------- |
  | _long      | long       |
  | _short     | short      |
  | _int       | int        |
  | _integer   | int        |
  | _double    | double     |
  | _float     | float      |
  | _boolean   | boolean    |
  | string     | String     |
  | byte       | Byte       |
  | long       | Long       |
  | short      | Short      |
  | int        | Integer    |
  | integer    | Integer    |
  | double     | Double     |
  | float      | Float      |
  | boolean    | Boolean    |
  | date       | Date       |
  | decimal    | BigDecimal |
  | bigdecimal | BigDecimal |
  | object     | Object     |
  | map        | Map        |
  | hashmap    | HashMap    |
  | list       | List       |
  | arraylist  | ArrayList  |
  | collection | Collection |
  | iterator   | Iterator   |

------

### typeHandlers

- 配置**类型处理器**信息。

- MyBatis 在设置预处理语句 `PreparedStatement` 中的参数或从结果集中取出一个值时， 都会用类型处理器将获取到的值**以合适的方式转换成 Java 类型**。

- 自定义 `TypeHandler`，实现 `org.apache.ibatis.type.TypeHandler`接口，或者继承 `org.apache.ibatis.type.BaseTypeHandler`。

  ```xml
  <typeHandlers>
    <typeHandler handler="org.mybatis.example.ExampleTypeHandler"/>
  </typeHandlers>
  ```

#### 默认的类型处理器

| 类型处理器                 | Java 类型                     | JDBC 类型                                                    |
| -------------------------- | ----------------------------- | ------------------------------------------------------------ |
| BooleanTypeHandler         | java.lang.Boolean, boolean    | 数据库兼容的 `BOOLEAN`                                       |
| ByteTypeHandler            | java.lang.Byte, byte          | 数据库兼容的 `NUMERIC` 或 `BYTE`                             |
| ShortTypeHandler           | java.lang.Short, short        | 数据库兼容的 `NUMERIC` 或 `SMALLINT`                         |
| IntegerTypeHandler         | java.lang.Integer, int        | 数据库兼容的 `NUMERIC` 或 `INTEGER`                          |
| LongTypeHandler            | java.lang.Long, long          | 数据库兼容的 `NUMERIC` 或 `BIGINT`                           |
| FloatTypeHandler           | java.lang.Float, float        | 数据库兼容的 `NUMERIC` 或 `FLOAT`                            |
| DoubleTypeHandler          | java.lang.Double, double      | 数据库兼容的 `NUMERIC` 或 `DOUBLE`                           |
| BigDecimalTypeHandler      | java.math.BigDecimal          | 数据库兼容的 `NUMERIC` 或 `DECIMAL`                          |
| StringTypeHandler          | java.lang.String              | `CHAR, VARCHAR`                                              |
| ClobReaderTypeHandler      | java.io.Reader                | `-`                                                          |
| ClobTypeHandler            | java.lang.String              | `CLOB, LONGVARCHAR`                                          |
| NStringTypeHandler         | java.lang.String              | `NVARCHAR, NCHAR`                                            |
| NClobTypeHandler           | java.lang.String              | `NCLOB`                                                      |
| BlobInputStreamTypeHandler | java.io.InputStream           | -                                                            |
| ByteArrayTypeHandler       | byte[]                        | 数据库兼容的字节流类型                                       |
| BlobTypeHandler            | byte[]                        | `BLOB`, `LONGVARBINARY`                                      |
| DateTypeHandler            | java.util.Date                | `TIMESTAMP`                                                  |
| DateOnlyTypeHandler        | java.util.Date                | `DATE`                                                       |
| TimeOnlyTypeHandler        | java.util.Date                | `TIME`                                                       |
| SqlTimestampTypeHandler    | java.sql.Timestamp            | `TIMESTAMP`                                                  |
| SqlDateTypeHandler         | java.sql.Date                 | `DATE`                                                       |
| SqlTimeTypeHandler         | java.sql.Time                 | `TIME`                                                       |
| ObjectTypeHandler          | Any                           | `OTHER` 或未指定类型                                         |
| EnumTypeHandler            | Enumeration Type              | VARCHAR 或任何兼容的字符串类型，用来存储枚举的名称（而不是索引序数值） |
| EnumOrdinalTypeHandler     | Enumeration Type              | 任何兼容的 `NUMERIC` 或 `DOUBLE` 类型，用来存储枚举的序数值（而不是名称）。 |
| SqlxmlTypeHandler          | java.lang.String              | `SQLXML`                                                     |
| InstantTypeHandler         | java.time.Instant             | `TIMESTAMP`                                                  |
| LocalDateTimeTypeHandler   | java.time.LocalDateTime       | `TIMESTAMP`                                                  |
| LocalDateTypeHandler       | java.time.LocalDate           | `DATE`                                                       |
| LocalTimeTypeHandler       | java.time.LocalTime           | `TIME`                                                       |
| OffsetDateTimeTypeHandler  | java.time.OffsetDateTime      | `TIMESTAMP`                                                  |
| OffsetTimeTypeHandler      | java.time.OffsetTime          | `TIME`                                                       |
| ZonedDateTimeTypeHandler   | java.time.ZonedDateTime       | `TIMESTAMP`                                                  |
| YearTypeHandler            | java.time.Year                | `INTEGER`                                                    |
| MonthTypeHandler           | java.time.Month               | `INTEGER`                                                    |
| YearMonthTypeHandler       | java.time.YearMonth           | `VARCHAR` 或 `LONGVARCHAR`                                   |
| JapaneseDateTypeHandler    | java.time.chrono.JapaneseDate | `DATE`                                                       |

------

### objectFactory

- 设置**对象工厂**配置。

### plugins

- 设置**插件**配置。
- 插件功能是 Mybatis 提供的特别强大的功能。
- **插件是 MyBatis 提供的一个非常强大的机制，可以通过插件来修改 MyBatis 的一些核心行为。**
- **插件通过动态代理机制，可以个入四大对象的任何一个方法的执行。**
  - `Executor (update, query, flushStatements, commit, rollback, getTransaction, close, isClosed)`
  - `ParameterHandler (getParameterObject, setParameters)`
  - `ResultSetHandler (handleResultSets, handleOutputParameters)`
  - `StatementHandler (prepare, parameterize, batch, update, query)`

### environments

- 设置**环境配置**。
- MyBatis 可以配置成适应多种环境，这种机制有助于将 SQL 映射应用于多种数据库之中， 现实情况下有多种理由需要这么做。
- **尽管可以配置多个环境，但每个 SqlSessionFactory 实例只能选择一种环境。**
- **id 是当前环境的唯一标识，default 指定默认的环境信息。**
- 为了指定创建哪种环境，只要将它作为可选的参数传递给 `SqlSessionFactoryBuilder` 即可。

```xml
<environments default="development">
  <environment id="development">
    <transactionManager type="JDBC">
      <property name="..." value="..."/>
    </transactionManager>
    <dataSource type="POOLED">
      <property name="driver" value="${driver}"/>
      <property name="url" value="${url}"/>
      <property name="username" value="${username}"/>
      <property name="password" value="${password}"/>
    </dataSource>
  </environment>
</environments>
```

#### environment

- 具体的环境信息。

#### transactionManager

- 事务管理器。
- 在 MyBatis 中有两种类型的事务管理器 。
- `JDBC` 配置直接使用了 JDBC 的提交和回滚设施，它依赖从数据源获得的连接来管理事务作用域。
- `MANAGED` 这个配置几乎没做什么。它从不提交或回滚一个连接，而是让容器来管理事务的整个生命周期（比如 JEE 应用服务器的上下文）。默认情况下它会关闭连接。然而一些容器并不希望连接被关闭，因此需要将 `closeConnection` 属性设置为 false 来阻止默认的关闭行为。

#### dataSource

- dataSource 元素使用标准的 JDBC 数据源接口来配置 JDBC 连接对象的资源。
- Mybatis 有三种内建的数据源类型。
  - `UNPOOLED` 这个数据源的实现会每次请求时打开和关闭连接。
  - `POOLED` 这种数据源的实现利用“池”的概念将 JDBC 连接对象组织起来，避免了创建新的连接实例时所必需的初始化和认证时间。 这种处理方式很流行，能使并发 Web 应用快速响应请求。
  - `JNDI` 这个数据源实现是为了能在如 EJB 或应用服务器这类容器中使用，容器可以集中或在外部配置数据源，然后放置一个 JNDI 上下文的数据源引用。

### databaseIdProvider

- 数据库厂商标识。
- Mybatis 用来做数据库移植的。
- 先在配置文件中为每个数据库厂商起一个名称，之后再SQL映射文件中编写 SQL 的时候通过 `databaseId` 指定方法的执行数据库信息。
- 不指定 `databaseId`的 SQL 语句代表不区分数据库环境。
- 通过这样的配置，实现不同类型的数据库走不同语句。
- 精确匹配的语句优先级高于不指定数据库环境的语句。

### mappers

- 配置编写好的映射器信息。
- `url` 可以从磁盘或者网络路径引用。
- `resource` 在类路径下找 SQL 映射文件。
- `class` 直接引用接口的全类名。
  - 注册基于注解的 DAO 接口。
  - 注册全类名的话，需要保证配置文件和 DAO 类在一个路径下。
- 可以通过 `package` 进行批量注册，但是需要保持 SQL 映射文件和 DAO 文件在一个包下。
  - 可以在 `resources` 目录下创建和 DAO 文件所在目录同名的文件夹统一管理 SQL 映射文件，编译后，会合并在 `bin` 目录的同一个文件下。

## 杂谈

### 数据关联关系

- 一对多的情况，外键一定放在多的一端。
- 多对多的情况，需要单独的中间表来维护外键的关系。
