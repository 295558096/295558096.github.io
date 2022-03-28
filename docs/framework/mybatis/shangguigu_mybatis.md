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

#### OGNL

- `Object Graph Navigation Language`，一种**对象图导航语言**。
- OGNL 是一种强大的表达式语言，通过它可以非常方便的来操作对象属性。
  - 访问对象属性。
  - 调用方法。
  - 通过`@`调用静态属性和方法。
  - 可以调用构造方法。
- 支持`运算符`、`逻辑运算符`，`“`、`<`、`>`等特殊的序号需要进行转义操作。
- 支持访问集合的伪属性，`size`、`isEmpty`、`keys`、`values`、`iterator`、`next`、`hasNext` 等。
- `_parameter` 代表传入来的参数。
  - 单个参数代表参数。
  - 多个参数代表包含多个参数的 Map。
- `_databaseId`，代表当前数据库运行环境。

#### if

- 使用动态 SQL 最常见情景是根据条件包含 where 子句的一部分。

- if 标签中可以传入非常强大的判断条件，OGNL 表达式。

  ```sql
  <select id="findActiveBlogLike"
       resultType="Blog">
    SELECT * FROM BLOG WHERE state = ‘ACTIVE’
    <if test="title != null">
      AND title like #{title}
    </if>
    <if test="author != null and author.name != null">
      AND author_name like #{author.name}
    </if>
  </select>
  ```

#### where

- 使用 `where` 标签将条件语句 `if` 标签包裹起来。
- `where` 标签会自动去除拼接后 SQL 语句中**前面**多余的 `and` 。

#### trim

- `trim` 标签用来截取字符串，可以自定义截取规则。
- `prefix` 属性，为 SQL添加一个前缀。
- `prefixOverrides` 属性，去除整体字符串前面多余的串。
- `suffix` 属性，为 SQL添加一个后缀。
- `suffixOverrides` 属性，去除整体字符串后面多余的串。

#### choose

- `choose` 标签是用来进行语句分支选择的，**多个 `when` 分支只会命中最靠前的一个**。
- `when` 子标签，用来指定分支的逻辑。
  - `test` 属性，编写 OGNL 表达式，进行业务判断。
- `otherwise` 子标签，用来指定分支的其他逻辑情况。

#### foreach

- `foreach` 标签是对集合参数进行遍历的。
- `collection` 属性，指定要遍历的集合的在参数 Map 中对应的 key。
  - **List 类型的参数，如果没有指定命名参数，只能通过 `list`为 key 获取集合。**
- `item` 属性，指定集合中的每一个元素的变量名。
- `index` 属性，指定索引变量名。
  - 如果遍历的是一个 map，index 指的是当前 value 对应的 key。
- `separator` 属性，遍历分隔符。
- `open` 属性，指定循环开始字符。
- `close` 属性，指定循环结束字符。

#### set

- `set` 标签，用来解决更新操作中，拼接动态 SQL 后产生多余的逗号的问题。
- `set` 标签会删除动态语句中，在**最后面**多余出来的逗号。

#### bind

- `bind` 标签，绑定一个表达式的值到一个变量。

#### sql

- `sql` 标签，定义 sql 片段。

#### include

- `include` 标签，引用定义好的 sql 片段。

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

------

## 缓存

### 简介

- MyBatis 缓存机制使用的是 Map，用来缓存一些查询的结果。
- `一级缓存` 线程级别的缓存，本地缓存、SQLSession 级别的缓存。
- `二级缓存` 全局范围的缓存。

### 一级缓存

- 一级缓存默认存在且开启，不可以被关闭。
- 每次查询，先看一级缓存中有没有，如果没有就去发送新的 SQL 到数据库。
- 每个 SqlSession 拥有自己的一级缓存。
- 只要之前查询过的数据，mybatis 就会保存在一个缓存中（Map)，下次查询直接获取缓存中的查询结果。
- 一级缓存的使用的是 `PerpetualCache`。

#### 一级缓存失效的场景

- 不同的 SqlSession，使用不同的一级缓存，即使查询相同的 SQL 语句也不会使用缓存。
- 同一个方法，不同的参数，如果之前没有查询过，那么会发新的 Sql。
- 在一个 SqlSession 期间执行上任何一次增删改操作，增删改操作会把**缓存清空**。
- `sqlSession.clearCache()` 手动清空一级缓存。
- SqlSession 关闭，一级缓存失效。

### 二级缓存

- 二级缓存（second level cache），全局作用域缓存。
- 二级缓存默认不开启，需要手动配置。
- MyBatis 提供二级缓存的接口以及实现，缓存实现要求 POJO 实现 Serializable 接口。
- **二级缓存在 SqlSession 关闭或提交之后才会生效。**

#### 属性

- `eviction`
  - 缓存回收策略，默认 LRU。
  - `LRU` 最近最少使用的，移除最长时问不被使用的对象。
  - `FIFO` 先进先出，按对象进入缓存的顺序来移除它们。
  - `SOFT` 软引用，移除基于垃圾回收器状态和软引用规则的对象。
  - `WEAK` 弱引用，更积极地移除基于垃圾收集器状态和弱引用规则的对象。
- `flushInterval`
  - 刷新间隔，单位毫秒。
  - 默认情况是不设置，也就是没有刷新间隔，缓存仅仅调用语句时刷新。
- `size`
  - 引用数目，正整数。
  - 代表缓存最多可以存储多少个对象，太大容易导致内存溢出。

- `readOnly`
  - `true` 只读缓存，会给所有调用者返回缓存对象的相同实例。因此这些对象不能被修改。这提供了很重要的性能优势。
  - `false` 读写缓存，会返回缓存对象的拷贝（通过序列化）。这会慢一些，但是安全，因此**默认是 false**。
- `type`
  - 指定使用的缓存类型。
  - 默认使用 Mybatis 自己内置的缓存 Map。

#### 使用步骤

- 全局配置文件中开启二级缓存。

  ```xml
  <setting name="cacheEnabled" value="true" />
  ```

- 需要使用二级缓存的映射文件处使用 `cache` 配置缓存。

  ```xml
  <cache/>
  ```

- POJO 需要实现 Serializable 接口。

### 整合第三方缓存

- Mybatis 的默认缓存使用的是 Map，过于简陋。
- Mybatis 的缓存定义成 Cache 接口，可以通过实现 Cache 接口，扩展缓存实现方案。
- EhCache、Redis 等第三方缓存中间件和框架都可以和 Mybatis 进行整合，进而实现更强大的缓存功能。
- 在 SQL映射文件中的 `cache` 标签的 `type` 属性指定使用的缓存实现。
- 在 SQL映射文件中的 `cache-ref`  标签指定和其他的名称空间使用同一块缓存。

### 总结

- **不会出现一级缓存和二级缓存中有同一个数据。**
- **任何时候都是先看二级缓存、再看一级缓存，如果都没有就去查询数据库。**
- 一级缓存默认开启且不能关闭，Mybatis中的配置和标签属性都控制的是二级缓存是否生效。
- `select` 标签的 `useCache` 属性是指定是否使用二级缓存的。
- 增删改标签的属性 `flushCache` 同时清空一级缓存和二级缓存，默认是true。
- 查询标签的属性 `flushCache` 同时清空一级缓存和二级缓存，默认是false。
- `sqlSession.clearCache()` 只清空一级缓存。

------

## 插件

### pageHelper

#### 简介

- 一个开源的 Mybatis 通用的分页插件。
- 该插件是通过物理分页实现SQL分页查询的。

#### 使用方式

##### 引入 jar 包

- jsqlparser.jar
- pagehelper.jar

##### 引入依赖

```xml
<dependency>
    <groupId>com.github.pagehelper</groupId>
    <artifactId>pagehelper</artifactId>
    <version>最新版本</version>
</dependency>
```

##### 配置Mybatis插件

- 特别注意，新版拦截器是 `com.github.pagehelper.PageInterceptor`。

-  `com.github.pagehelper.PageHelper` 现在是一个特殊的 `dialect` 实现类，是分页插件的默认实现类，提供了和以前相同的用法。

  ```xml
  <!-- 
      plugins在配置文件中的位置必须符合要求，否则会报错，顺序如下:
      properties?, settings?, 
      typeAliases?, typeHandlers?, 
      objectFactory?,objectWrapperFactory?, 
      plugins?, 
      environments?, databaseIdProvider?, mappers?
  -->
  <plugins>
      <!-- com.github.pagehelper为PageHelper类所在包名 -->
      <plugin interceptor="com.github.pagehelper.PageInterceptor">
          <!-- 使用下面的方式配置参数，后面会有所有的参数介绍 -->
          <property name="param1" value="value1"/>
  	</plugin>
  </plugins>
  ```

  ##### 配置Spring插件

  使用 spring 的属性配置方式，可以使用 `plugins` 属性像下面这样配置。

  ```xml
  <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    <!-- 注意其他配置 -->
    <property name="plugins">
      <array>
        <bean class="com.github.pagehelper.PageInterceptor">
          <property name="properties">
            <!--使用下面的方式配置参数，一行配置一个 -->
            <value>
              params=value1
            </value>
          </property>
        </bean>
      </array>
    </property>
  </bean>
  ```

#### 参数介绍

- 分页插件提供了多个可选参数。
- `dialect` 
  - 默认情况下会使用 PageHelper 方式进行分页。
  - 如果想要实现自己的分页逻辑，可以实现 `Dialect`(`com.github.pagehelper.Dialect`) 接口，然后配置该属性为实现类的全限定名称。
- `helperDialect` 
  - 分页插件会自动检测当前的数据库链接，自动选择合适的分页方式。 
  - 你可以配置`helperDialect`属性来指定分页插件使用哪种方言。配置时，可以使用下面的缩写值。
    - oracle
    - mysql
    - mariadb
    - sqlite
    - hsqldb
    - postgresql
    - db2
    - sqlserver
    - informix
    - h2
    - sqlserver2012
    - derby
  - **特别注意：**使用 SqlServer2012 数据库时，需要手动指定为 `sqlserver2012`，否则会使用 SqlServer2005 的方式进行分页。你也可以实现 `AbstractHelperDialect`，然后配置该属性为实现类的全限定名称即可使用自定义的实现方法。
- `offsetAsPageNum`
  - 默认值为 **false**，该参数对使用 `RowBounds` 作为分页参数时有效。
  -  当该参数设置为 **true** 时，会将 `RowBounds` 中的 `offset` 参数当成 `pageNum` 使用，可以用页码和页面大小两个参数进行分页。
- `rowBoundsWithCount` 
  - 默认值为**false**。
  - 该参数对使用 RowBounds 作为分页参数时有效。
  - 当该参数设置为true时，使用 RowBounds 分页会进行 count 查询。
- `pageSizeZero`
  -  默认值为 **false**。
  - 当该参数设置为 true 时如果 `pageSize=0` 或者 `RowBounds.limit=0` 就会查询出全部的结果（相当于没有执行分页查询，但是返回结果仍然是 Page 类型）。
- `reasonable`
  - 分页合理化参数，默认值为false。
  - 当该参数设置为 true 时，`pageNum<=0` 时会查询第一页， pageNum>pages（超过总数时），会查询最后一页。
  - 默认 false 时，直接根据参数进行查询。
- `params`
  - 为了支持`startPage(Object params)`方法，增加了该参数来配置参数映射。
  - 用于从对象中根据属性名取值， 可以配置 pageNum、pageSize、count、pageSizeZero、reasonable。
  - 不配置映射的用默认值， 默认值为`pageNum=pageNum;pageSize=pageSize;count=countSql;reasonable=reasonable;pageSizeZero=pageSizeZero`。
- `supportMethodsArguments`
  - 支持通过 Mapper 接口参数来传递分页参数，默认值`false`。
  - 分页插件会从查询方法的参数值中，自动根据上面 `params` 配置的字段中取值，查找到合适的值时就会自动分页。 
- `autoRuntimeDialect`
  - 默认值为 false。
  - 设置为 true 时，允许在运行时根据多数据源自动识别对应方言的分页 。
  - 不支持自动选择sqlserver2012，只能使用sqlserver。
- `closeConn`
  - 默认值为 true。
  - 当使用运行时动态数据源或没有设置 `helperDialect` 属性自动获取数据库类型时，会自动获取一个数据库连接， 通过该属性来设置是否关闭获取的这个连接。
  - 默认 true 关闭，设置为 false 后，不会关闭获取的连接，这个参数的设置要根据自己选择的数据源来决定。
- `aggregateFunctions`(5.1.5+)
  - 默认为所有常见数据库的聚合函数，允许手动添加聚合函数（影响行数），所有以聚合函数开头的函数，在进行 count 转换时，会套一层。
  - 其他函数和列会被替换为 count(0)，其中count列可以自己配置。

#### 使用方式

```java
//第一种，RowBounds方式的调用
List<User> list = sqlSession.selectList("x.y.selectIf", null, new RowBounds(0, 10));

//第二种，Mapper接口方式的调用，推荐这种使用方式。
PageHelper.startPage(1, 10);
List<User> list = userMapper.selectIf(1);

//第三种，Mapper接口方式的调用，推荐这种使用方式。
PageHelper.offsetPage(1, 10);
List<User> list = userMapper.selectIf(1);

//第四种，参数方法调用
//存在以下 Mapper 接口方法，你不需要在 xml 处理后两个参数
public interface CountryMapper {
    List<User> selectByPageNumSize(
            @Param("user") User user,
            @Param("pageNum") int pageNum, 
            @Param("pageSize") int pageSize);
}
//配置supportMethodsArguments=true
//在代码中直接调用：
List<User> list = userMapper.selectByPageNumSize(user, 1, 10);

//第五种，参数对象
//如果 pageNum 和 pageSize 存在于 User 对象中，只要参数有值，也会被分页
//有如下 User 对象
public class User {
    //其他fields
    //下面两个参数名和 params 配置的名字一致
    private Integer pageNum;
    private Integer pageSize;
}
//存在以下 Mapper 接口方法，你不需要在 xml 处理后两个参数
public interface CountryMapper {
    List<User> selectByPageNumSize(User user);
}
//当 user 中的 pageNum!= null && pageSize!= null 时，会自动分页
List<User> list = userMapper.selectByPageNumSize(user);

//第六种，ISelect 接口方式
//jdk6,7用法，创建接口
Page<User> page = PageHelper.startPage(1, 10).doSelectPage(new ISelect() {
    @Override
    public void doSelect() {
        userMapper.selectGroupBy();
    }
});
//jdk8 lambda用法
Page<User> page = PageHelper.startPage(1, 10).doSelectPage(()-> userMapper.selectGroupBy());

//也可以直接返回PageInfo，注意doSelectPageInfo方法和doSelectPage
pageInfo = PageHelper.startPage(1, 10).doSelectPageInfo(new ISelect() {
    @Override
    public void doSelect() {
        userMapper.selectGroupBy();
    }
});
//对应的lambda用法
pageInfo = PageHelper.startPage(1, 10).doSelectPageInfo(() -> userMapper.selectGroupBy());

//count查询，返回一个查询语句的count数
long total = PageHelper.count(new ISelect() {
    @Override
    public void doSelect() {
        userMapper.selectLike(user);
    }
});
//lambda
total = PageHelper.count(()->userMapper.selectLike(user));
```

#### 官方文档

[官方文档地址](https://github.com/pagehelper/Mybatis-PageHelper/blob/master/wikis/zh/HowToUse.md)

------

## SSM整合

### 导包

#### Spring

##### aop核心

- com.springsource.net.sf.cglib.jar
- com.springsource.org. aopalliance.jar
- com.springsource.org.aspectj.weaver.jar
- spring-aspects.jar

##### ioc核心

- commons-logging.jar
- spring-aop.jar
- spring-beans.jar
- spring-context.jar
- spring-core.jar
- spring-expression.jar

##### jdbc核心

- spring-jdbc.jar
- spring-orm.jar
- spring-tx.jar

##### 单元测试

- spring-test.jar

#### SpringMVC

##### mvc核心

- spring-web.jar
- spring-webmvc.jar

##### 文件上传

- commons-io.jar
- commons-fileupload.jar

##### 数据校验

- hibernate-validator.jar
- hibernate-validator-annotation-processor.jar
- classmate.jar
- jboss-logging.jar
- validation-api.jar

#### Mybatis

##### mybatis核心

- mybatis.jar
- mybatis-spring.jar

#### 其他

##### 数据库驱动

- mysql-connector-java.jar

##### 数据源

- c3p0.jar

### 写配置

#### Web

- 编写 `web.xml`。

#### Spring

- 在 `web.xml` 中注册监听器 `org.springframework.web.context.ContextLoaderListener` 来启动 Spring 容器。
- 通过在 `context-param`中配置`ContextConfigLocation`属性来指定 Spring 配置文件的位置。

```xml
<!-- Spring加载的xml文件,不配置默认为applicationContext.xml -->
<context-param>
  <param-name>contextConfigLocation</param-name>
  <param-value>/WEB-INF/springConfig.xml</param-value>
</context-param>
<!-- 该类作为spring的listener使用，它会在创建时自动查找web.xml配置的applicationContext.xml文件 -->
<listener>
  <listener-class>
    org.springframework.web.context.ContextLoaderListener
  </listener-class>
</listener>
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context" xmlns:tx="http://www.springframework.org/schema/tx"
    xmlns:aop="http://www.springframework.org/schema/aop"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans-3.0.xsd
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context-3.0.xsd http://www.springframework.org/schema/tx http://www.springframework.org/schema/tx/spring-tx.xsd http://www.springframework.org/schema/aop https://www.springframework.org/schema/aop/spring-aop.xsd">

    <!-- 扫描组件，除了 Controller-->
    <context:component-scan base-package="xx">
        <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <!-- 导入外部配置-->
    <context:property-placeholder location="classpath: jdbc.properties"/>

    <!-- 配置数据源-->
    <bean id="dataSource" class="com.alibaba.druid.pool.DruidDataSource">
        <property name="username" value="${jdbc.user}"/>
        <property name="password" value="${jdbc.password}"/>
        <property name="url" value="${jdbc.url}"/>
        <property name="driverClassName" value="${jdbc.driver}"/>
    </bean>

    <!-- 配置事务管理器-->
    <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
        <property name="dataSource" ref="dataSource"/>
    </bean>

    <!--配置事务切面-->
    <aop:config>
        <aop:pointcut id="transactionPointcut" expression="execution(* com.xxx.*.*.service.*.*(..))"/>
        <aop:advisor pointcut-ref="transactionPointcut" advice-ref="transactionAdvice"/>
        <aop:aspect ref="tddlrwRounter">
            <aop:around pointcut-ref="transactionPointcut" method="determineReadOrWriteDB"/>
        </aop:aspect>
    </aop:config>

    <!-- 配置事务路由器-->
    <bean id="tddlrwRounter" class="com.tiangou.tgframe.aop.TDDLRWRounter"/>

    <!-- 配置事务切面规则-->
    <tx:advice id="transactionAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="add*" propagation="REQUIRED" rollback-for="java.lang.Exception"/>
            <tx:method name="edit*" propagation="REQUIRED" rollback-for="java.lang.Exception"/>
            <tx:method name="remove*" propagation="REQUIRED" rollback-for="java.lang.Exception"/>
            <tx:method name="insert*" propagation="REQUIRED" rollback-for="java.lang.Exception"/>
            <tx:method name="save*" propagation="REQUIRED" rollback-for="java.lang.Exception"/>
            <tx:method name="upd*" propagation="REQUIRED" rollback-for="java.lang.Exception"/>
            <tx:method name="modify*" propagation="REQUIRED" rollback-for="java.lang.Exception"/>
            <tx:method name="delete*" propagation="REQUIRED" rollback-for="java.lang.Exception"/>
            <tx:method name="*" propagation="SUPPORTS" read-only="true"/>
        </tx:attributes>
    </tx:advice>
</beans>
```



#### SpringMVC

- 在 `web.xml`注册 SpringMVC 的 `DispatcherServlet` 前端控制器。
- 在 `web.xml`注册编码过滤器，`CharacterEncodingFilter`。
- 在 `web.xml`注册支持 Rest 风格请求的过滤器，`HiddenHttpMethodFilter`。

```xml
<!--spring mvc配置-->
<!-- 配置Sping MVC的DispatcherServlet,也可以配置为继承了DispatcherServlet的自定义类,这里配置spring mvc的配置(扫描controller) -->
<servlet>
  <servlet-name>springmvcservlet</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <!-- spring MVC的配置文件 -->
  <init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>/WEB-INF/springmvc.xml</param-value>
  </init-param>
  <!-- 下面值小一点比较合适，会优先加载 -->
  <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mapping>
  <servlet-name>springmvcservlet</servlet-name>
  <url-pattern>/</url-pattern>
</servlet-mapping>

<!-- 配置请求过滤器，编码格式设为UTF-8，避免中文乱码 -->
<filter>
  <filter-name>charsetfilter</filter-name>
  <filter-class>
    org.springframework.web.filter.CharacterEncodingFilter
  </filter-class>
  <init-param>
    <param-name>encoding</param-name>
    <param-value>UTF-8</param-value>
  </init-param>
  <init-param>
    <param-name>forceEncoding</param-name>
    <param-value>true</param-value>
  </init-param>
</filter>
<filter-mapping>
  <filter-name>charsetfilter</filter-name>
  <url-pattern>/*</url-pattern>
</filter-mapping>
```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xmlns:context="http://www.springframework.org/schema/context"
    xmlns:mvc="http://www.springframework.org/schema/mvc"
    xsi:schemaLocation="http://www.springframework.org/schema/beans
        http://www.springframework.org/schema/beans/spring-beans.xsd   
        http://www.springframework.org/schema/context
        http://www.springframework.org/schema/context/spring-context.xsd  
        http://www.springframework.org/schema/mvc  
        http://www.springframework.org/schema/mvc/spring-mvc.xsd">

    <!-- 只扫描控制器-->
    <context:component-scan base-package="xxx" use-default-filters="false">
        <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
    </context:component-scan>

    <!--文件解析器-->
    <bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
        <property name="defaultEncoding" value="UTF-8"/>
        <property name="maxUploadSize" value="20971520"/>
    </bean>

    <!-- 视图解析器-->
    <bean id="internalResourceViewResolver"
        class="org.springframework.web.servlet.view.InternalResourceViewResolver">
        <property name="prefix" value="/WEB-INF/pages/"/>
        <property name="suffix" value=".html"/>
    </bean>

    <!-- 扫描静态-->
    <mvc:default-servlet-handler/>
    <!-- 扫描动态-->
    <mvc:annotation-driven/>
</beans>
```

#### Mybatis

```xml
<!-- sqlSessionFactory -->
<bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
  <!-- mybatis配置文件位置-->
  <property name="configLocation" value="mybatis-config.xml"/>
  <property name="dataSource" ref="dataSource"/>
  <!-- 自动扫描entity目录, 省掉 Configuration.xml 里的手工配置 -->
  <property name="typeAliasesPackage" value="com.xxx.*.entity"/>
  <!-- 显式指定Mapper文件位置 -->
  <property name="mapperLocations" value="classpath:mybatis/**/*Mapper.xml"/>
</bean>

<!-- 指定Mybatis扫描的Mapper.java位置并注入到IOC容器中-->
<bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
  <property name="basePackage" value="com.tgou.unionPayShop"/>
  <property name="sqlSessionFactoryBeanName" value="sqlSessionFactory"/>
</bean>
```

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
  <settings>
    <setting name="mapUnderscoreToCamelCase" value="true"/>
    <!-- 打印sql日志 -->
    <setting name="logImpl" value="STDOUT_LOGGING" />
  </settings>
</configuration>
```

------

## 工作原理

### 分层情况

#### 接口层

- 配置信息维护接口。
- 接口调用方式。
- 基于 StatementID 进行唯一接口调用。
- 基于 Mapper 接口。

#### 数据处理层

- 参数映射 `ParameterHandler`
  - 参数映射配置
  - 参数映射解析
  - 参数类型解析
- SQL解析 `SqlSource`
  - SQL语句配置
  - SQL语句解析
  - SQL语句动态生成
- SQL执行 `Executor`
  - `SimpleExecutor` 简单执行器
  - `BatchExecutor` 批量执行器
  - `ReuseExecutor` 可复用执行器
- 结果处理和映射 `ResultSetHandler`
  - 结果映射配置
  - 结果类型转换
  - 结果类型转换

#### 框架支撑层

- SQL 语句配置方式
  - 基于XML配置
  - 基于注解
- 事务管理
- 连接池管理
- 缓存机制

#### 引导层

- 基于 XML 配置方式
- 基于 Java API 方式

### 主体流程

- 获取 sqlSessionFactory 对象。
  - 解析配置文件的每一个信息保存在 Configuration 中，返回包含 Configuration 的 `DefaultSqlSession`。
- 获取 sqlSession 对象。
  - 返回 `SqlSession` 的实现类 `DefaultSqlSession` 对象。他里面包含了 Executor 和 Configuration。
- 获取接口的代理对象，MapperProxy。
  - 使用 MapperProxyFactory 创建一个 MapperProxy 的代理对象。
- 执行增则改查方法。

### 获取 SqlSessionFactory 的流程

>根据配置文件，创建 SqlSessionFactory。

- `SqlSessionFactoryBuilder` 会加载mybaitis全局配置文件，并通过XPath解析的方式，解析出全局配置信息。
- `org.apache.ibatis.session.Configuration` 类的对象保存了全部的全局配置信息。
- 解析全局配置文件的过程中，会加载 Mappers 的配置，根据类名或者url地址找到 mapper.xml 文件，然后解析 `mapper.xml` 文件。
- 通过校验和解析 mapper.xml，获取该 xml 中的全部配置信息和 增删改查的SQL 语句块。
- 所有的增删改查标签都会封装成为一个 `org.apache.ibatis.mapping.MappedStatement` 对象，并保存在 `configuration` 中。
- 通过传入 `Configuration` 对象并调用 `XmlConfigBuilder` 的 `build`方法，得到默认的 `DefaultSqlSessionFactory` 实例。

#### Configuration

- Mybatis 中来封装服务启动后解析全部配置和SQL映射文件获得的全部属性。
- `Configuration` 对象中包含了全部的 `MappedStatement` 对象，属性名为 `mappedStatements`。

#### MappedStatement

- Mybatis 中用来映射在 xml 映射文件中一个SQL语句的全部信息。
- 包含缓存、配置、唯一ID、来源、缓存使用情况以及全部的标签属性。
- Mybatis 工作工作过程中通过id确定要调用的 `MappedStatement`，基于 `MappedStatement` 进行进一步的绑定参数、执行SQL、映射结果等操作。

### 通过 OpenSession 获取 SqlSession对象流程

- 调用 `DefaultSqlSessionFactory` 的 `openSession` 方法，底层调用的是 `Configuration` 的 `openSessionFromDataSource` 方法。
- Configuration 获取事务的信息以及 `ExecutorType` 的信息构建出对应的 `Executor` 对象。
- 如果有二级缓存配置开启，创建 `CachingExecutor(executor)`。
- 获取到全部的拦截器插件信息，调用每一个拦截器重新包装 executor 并返回。
- 通过 `configuration`、`executor` 以及是否自动提交属性构建出一个 `DefaultSqlSession` 对象。

#### Executor

- Mybatis四大对象之一，代表着SQL语句的执行器。
- Executor 中包含 `Configuration` 对象和 `Transaction` 对象。

### 通过getMapper获取到接口的代理对象流程

- 调用 `DefaultSqlSession` 的 `getMapper` 方法获取接口的代理对象，实际上调用的是 `configuration` 对象的 `getMapper` 方法。
- configuration底层是调用的 `MapperRegistry#getMapper` 方法。
- MapperRegistry 的 getMapper 方法中是从 `knownMappers` 根据类型获取对应 Mapper 接口代理对象的工厂对象，即 `MapperProxyFactory`。
- 通过调用 `MapperProxyFactory#newInstance` 方法，传入 sqlSession 参数，获得 `MapperProxy`对象。
- `MapperProxy `是 `InvocationHandler` 的实现类，可以创建动态代理对象，实际上也是通过 jdk 的动态代理接口创建的 **Mapper 代理对象**。

#### MapperProxyFactory

#### MapperProxy

#### MapperRegistry

### 执行增删改查流程

>以查询为例。

- 通过返回的代理对象调用查询方法，实际上是通过 `MapperProxy` 调用他的 `invoke` 方法。

- invoke 方法中会根据方法是来自于接口还是来 Object 类走不同的执行流程。

  - Object 的方法直接执行。
  - 接口的方法会调用 `MapperMethod.execute(sqlSession, args)`。

- `MapperMethod.execute` 方法会判断要执行的方法的增删改查的类型。

- 判断返回值类型，执行调用不用的执行方法。

- 调用 `MapperMethod#convertArgsToSqlCommandParam (args)`方法转换参数。

- 调用 `DefaultSqlSession` 的 `selecX` 方法进行数据查询。

- 根据 id 在 configuration 中获取到 MappedStatement 对象。

- 调用 `Executor` 的 query 方法，传入查询参数、MappedStatement 等信息。

- 调用 `MappedStatement` 的 `getBoundSql` 方法获取到 `BoundSql`的对象。

- 通过解析参数和指定的语句，根据规则生成 Mybatis 中缓存使用的 key。

- 尝试着根据 Key 从二级缓存中获取查询结果。

- 如果没有在缓存中查到结果，直接调用 `executor#query` 方法查询结果。

- `Executor#query` 方法会先从一级缓存中查询结果，如果没有的话，再走数据库查询，并且将查询的结果缓存到一级缓存。

  - 获取 configuration 对象。

  - 调用 `Configuration#newStatementHandler`方法创建 `StatementHandler` 对象。

  - 创建 `ParamterHandler`，并为 `ParamterHandler` 对象绑定全部的插件。

  - 创建 `ResultSetHandler`，并为 `ResultSetHandler` 对象绑定全部的插件。

  - 通过 `StatementHandler` 预编译SQL生成JDBC执行需要使用的 `PrepareStatement` 对象并为 `PrepareStatement` 对象绑定全部的插件。

  - 通过调用 `TypeHandler` 为 `PrepareStatement` 预编译参数、设置参数。

  - 执行

  - 通过 `ResultSetHandler` 封装查询出的结果，使用 `TypeHandler` 处理值。

    

#### StatementHandler

- Mybatis四大对象之一。
- 处理SQL语句预编译、设置参数等相关工作。

#### BoundSql

- Mybatis中的具体SQL信息的封装对象。

#### TypeHandler

- 类型处理器。
- 在参数设置和结果集封装阶段被调用。

#### ParamterHandler

- Mybatis 四大对象之一。
- 用于设置预编译参数。
- 在生成 Statement 阶段被 StatementHandler 调用。

#### ResultSetHandler

- Mybatis 四大对象之一。
- 用于结果集的封装处理。

------

## 插件开发

### 简介

- MyBatis 在四大对象的创建过程中，都会有插件进行介入。插件可以利用动态代理机制一层层的包装目标对象，需实现在目标对家执荇目标方法之前迸行拦截的效果。
- **MyBatis 允许在已映射语句执行过程中的某一点进行拦截调用**。
- Mybatis 插件的接口是 `org.apache.ibatis.plugin.Interceptor`。
- **插件机制是通过调用 `Interceptor#plugin` 方法使用插件为目标对象创建一个代理对象**。
- 默认情况下，MyBatis 允许使用插件来拦截的方法调用包括。
  - Executor
    - update
    - query
    - flushStatements
    - commit
    - rollback
    - getTransaction
    - close
    - isClosed
  - ParameterHandler
    - getParameterObject
    - setParameters
  - ResultSetHandler
    - handleResultSets
    - handleOutputParameters
  - StatementHandler
    - prepare
    - parameterize
    - batch
    - update
    - query

### 插件的绑定

- 在四大对象创建后不会直接返回，而是会绑定插件，`interceptorChain.pluginAll(xxxHandler)`。

- `org.apache.ibatis.plugin.InterceptorChain#pluginAll`

  ```java
    public Object pluginAll(Object target) {
      for (Interceptor interceptor : interceptors) {
        target = interceptor.plugin(target);
      }
      return target;
    }
  ```

### 插件开发流程

- 自定义插件类实现 `org.apache.ibatis.plugin.Interceptor`接口并实现相关方法。
  - `intercept` 拦截方法后的处理操作，可以在目标方法执行前后添加自定义的业务逻辑。
    - `MetaObject metaObject = SystemMetaObject.forobject(target)` 可以获取到元数据对象。
    - 调用元数据对象的`setValue`和`getValue`可以设置和获取在元数据对象中的具体参数信息。
  - `plugin` 为目标对象生成代理类。
    - Mybatis 提供了工具类可以用于生成代理对象`Plugin#wrap`。
  - `setProperties` 在插件注册时设置属性。
- 通过将注解`@Intercepts` 标注在自定义插件类上，完成插件签名。
  - `@Signature` 注解标记具体的拦截对象类型、方法名称和参数列表。
- 在 Mybatis 的全局配置文件中注册编写好的插件。
- 插件会在四大对象的创建过程中被`InterceptorChain#pluginAll`方法调用，每个插件会根据自己的签名匹配情况为对应的对象创建代理对象或者返回原对象。

### 多插件运行流程

- 多个插件在四大对象创建的过程中，会按照在全局配置文件中的注册顺序进行拦截。
- 多个插件在目标方法的调用过程中，会按照在全局配置文件中的注册顺序的逆序进行拦截。
- 多插件的创建过程，会为目标对象的代理对象继续创建代理对象。

------

## 实用场景

### 批量操作

#### 简介

- 操作数据库往往会又需要批量插入数据或者批量更新数据的功能。

#### 常规方案

- 通过 `forEach` 标签在 insert 的方法中动态生成多组要插入的 `values`。
- 通过 `forEach` 标签生成**多条SQL语句**，一次性插入，需要**开启数据库的支持多条语句功能**。

#### 风险

- 通过一条长语句发送数据进行批量操作的时候，可能会出现SQL超长的现象，导致数据库操作失败。

#### BatchExecutor

- Mybatis 提供了多种类型的 Executor，其中 `BatchExecutor` 是用来处理批量操作的。

- 利用 `BatchExecutor` 执行的批量语句，不会把SQL语句直接逐条发送给数据库去执行，而是把预编译好的SQL语句发送给数据库，然后不断的给数据库发送要插入的参数信息，省去了在服务器端多次预编译的时间，传输给数据库的数据是更少的，数据库会在设置好参数之后，一次性执行全部的SQL，而不是多次执行。

- `BatchExecutor` 的批量处理效率很高，在1w条数据的情况下，效率是普通执行器的三倍。

- 与Spring的整合方式。

  ```xml
  <bean id="batchSqlSession" class="org.mybatis.spring.SqlSessionTemplate">
    <constructor-arg name="sqlSessionFactory" ref="sqlSessionFactory"/>
    <constructor-arg name="executorType" value="BATCH"/>
  </bean>
  ```

### 存储过程

- 实际开发中，操作数据库会使用一些存储过程。MyBatis 支持对存储过程的调用。

#### 调用方式

- `select` 标签中 `statementType="CALLABLE"`。
- 标签体中调用语法 `{call procedure_name (#(param1_info), #(param2_info})}`。

### 自定义 TypeHandler 处理枚举

- 通过自定义 `TypeHandler` 的形式来在**设置参数**或者**取出结果集的**时候**自定义参数封装策略**。
- 默认情况下 Mybatis 在处理枚举对象的时候是通过 `EnumTypeHandler` 处理的是枚举的名字，即枚举的 `name()`。
-  `EnumOrdinalTypeHandler` 处理器是使用枚举的索引当做结果。
- 保存方法和查询方法都可以指定自定义的 TypeHandler，注意存储和读取的时候要使用相同的处理器，避免数据混淆。

#### 流程

- 实现 `TypeHandler`接口或者继承 `BaseTypeHandler`。
- 使用 `@MappedTypes`  定义处理的 java 类型。
- 使用 `@MappedJdbcTypes` 定义 jdbcType 类型。
- 在自定义结果集标签或者参数处理的时候声明使用自定义 TypeHandler 进行处理或者在全局配置 `TypeHandler` 要处理的 javaType。

## 杂谈

### 数据关联关系

- 一对多的情况，外键一定放在多的一端。
- 多对多的情况，需要单独的中间表来维护外键的关系。
