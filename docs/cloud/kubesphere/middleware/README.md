# 中间件部署实战

## 部署三要素

- 应用的部署方式。
- 应用的数据挂载（数据，配置文件）。
- 应用的可访问性。

![key](image/key.png)

## 部署MySQL

### mysql容器启动

```bash
docker run -p 3306:3306 --name mysql-01 \
-v /mydata/mysql/log:/var/log/mysql \
-v /mydata/mysql/data:/var/lib/mysql \
-v /mydata/mysql/conf:/etc/mysql/conf.d \
-e MYSQL_ROOT_PASSWORD=root \
--restart=always \
-d mysql:5.7 
```

### mysql配置示例

```bash
[client]
default-character-set=utf8mb4
 
[mysql]
default-character-set=utf8mb4
 
[mysqld]
init_connect='SET collation_connection = utf8mb4_unicode_ci'
init_connect='SET NAMES utf8mb4'
character-set-server=utf8mb4
collation-server=utf8mb4_unicode_ci
skip-character-set-client-handshake
skip-name-resolve
```

### mysql部署分析

- 集群内部

  直接通过应用的【服务名.项目名】直接访问

  ```bash
  mysql -uroot -h**his-mysql-glgf.his** -p
  ```

- 集群外部

![mysql](image/mysql.png)

## 部署Redis

### redis容器启动

```bash
# 创建配置文件
## 1、准备redis配置文件内容
mkdir -p /mydata/redis/conf && vim /mydata/redis/conf/redis.conf

## 配置示例
appendonly yes
port 6379
bind 0.0.0.0

# docker启动redis
docker run -d -p 6379:6379 --restart=always \
-v /mydata/redis/conf/redis.conf:/etc/redis/redis.conf \
-v  /mydata/redis-01/data:/data \
 --name redis-01 redis:6.2.5 \
 redis-server /etc/redis/redis.conf
```

### redis部署分析

![redis](image/redis.png)

## 部署ElasticSearch

- 指定子路径，覆盖指定的配置文件，保留其他的配置文件。

### es容器启动

```bash
# 创建数据目录
mkdir -p /mydata/es-01 && chmod 777 -R /mydata/es-01

# 容器启动
docker run --restart=always -d -p 9200:9200 -p 9300:9300 \
-e "discovery.type=single-node" \
-e ES_JAVA_OPTS="-Xms512m -Xmx512m" \
-v es-config:/usr/share/elasticsearch/config \
-v /mydata/es-01/data:/usr/share/elasticsearch/data \
--name es-01 \
elasticsearch:7.13.4
```

### es部署分析

>子路径挂载，配置修改后，k8s不会对其Pod内的相关配置文件进行热更新，需要自己重启Pod。

![es](image/es.png)

## 应用商店

- 可以登录应用商店，从应用商店部署。


## 应用仓库

- 使用企业空间管理员登录，设置应用仓库。
- 使用应用仓库需要`Helm`相关的知识点。