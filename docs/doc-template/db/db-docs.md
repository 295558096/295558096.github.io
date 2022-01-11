# 数据库设计模板

## 数据库设计

### 新增表

#### xx表

| 字段        | 类型     | 长度 | 约束                        | 默认值            | 索引 | 注释                               |
| :---------- | :------- | :--- | :-------------------------- | :---------------- | :--- | :-----------------------------|
| id          | bigint   | 20   | AUTO_INCREMENT              |                   | PK   | ID                            |
| fk_order_id | bigint   | 20   | NOT NULL                    |                   |      | 订单ID                         |
| version     | int      | 11   | NOT NULL                    | 0                 |      | 乐观锁                         |
| create_time | datetime | 0    |                             | CURRENT_TIMESTAMP |      | 创建时间                        |
| modify_time | datetime | 0    | ON UPDATE CURRENT_TIMESTAMP | CURRENT_TIMESTAMP |      | 更新时间                        |

```sql
CREATE TABLE `xxxx_table` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT,
  `version` int(11) NOT NULL DEFAULT '0' COMMENT '乐观锁',
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `modify_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '更新时间',
  PRIMARY KEY (`id`),
  KEY `idx_order_id` (`fk_order_id`) USING BTREE
) ENGINE=InnoDB;
```
### 修改表

略