+++
title = '【拓展】Redis的使用'
date = 2025-05-28T18:23:18+08:00
draft = true
categories = [
    "redis",
    "笔记",
    "拓展",
]
image = "pawel-czerwinski-8uZPynIu-rQ-unsplash.jpg"

+++

## 1、下载并解压

https://github.com/tporadowski/redis/releases

## 2、启动Redis服务器

```
.\redis-server.exe redis.windows.conf
```

默认端口为 6379，如果看到类似 Ready to accept connections 的消息，说明 Redis 服务已成功启动。

## 3、测试Redis

```
.\redis-cli.exe -h 127.0.0.1 -p 6379
```

执行 ping 命令，如果返回 PONG，说明 Redis 服务正常运行。

## 4、将 Redis 注册为 Windows 服务（可选）

在 Redis 安装目录中，运行以下命令将 Redis 注册为 Windows 服务：

```
redis-server.exe --service-install redis.windows.conf --service-name redisserver1 --loglevel verbose
```

启动服务：

```
redis-server.exe --service-start --service-name redisserver1
```

通过 services.msc 查看服务是否安装成功，并可以配置为开机自启动。



## 5、报错解决

当出现以下问题时：

```
127.0.0.1:6379> del name
(error) MISCONF Redis is configured to save RDB snapshots, but it is currently not able to persist on disk. Commands that may modify the data set are disabled, because this instance is configured to report errors during writes if RDB snapshotting fails (stop-writes-on-bgsave-error option). Please check the Redis logs for details about the RDB error.
```

**临时解决方法：**

```
CONFIG SET stop-writes-on-bgsave-error no
```

**持久解决方法：**

修改Redis配置文件

找到 Redis 配置文件 redis.conf（通常位于 Redis 安装目录或 /etc/redis/redis.conf）。

使用文本编辑器打开 redis.conf 文件，找到 stop-writes-on-bgsave-error 选项，并将其设置为 no：

```
stop-writes-on-bgsave-error no
```

保存文件并重启 Redis 服务。

## 6、Redis 常用命令总结

### 6.1、连接与服务器相关命令

- **连接命令**
  - `redis - cli`：启动 Redis 客户端，用于与 Redis 服务器交互。
- **服务器信息和状态命令**
  - `INFO`：提供 Redis 服务器的详细信息和统计指标。
  - `DBSIZE`：返回当前数据库中键值对的数量。
- **服务器配置命令**
  - `CONFIG GET `：获取 Redis 服务器指定配置项的值。
  - `CONFIG SET  `：设置 Redis 服务器指定配置项的值。

### 6.2、键（Key）相关命令

- **键的基本操作命令**
  - `DEL key1 [key2 ...]`：删除一个或多个键。
  - `EXISTS key`：检查键是否存在。
  - `EXPIRE key seconds`：为键设置过期时间，以秒为单位。
  - `TTL key`：查看键的剩余生存时间。
  - `RENAME key newkey`：将键重命名。
  - `RENAME NX key newkey`：只有在新键不存在时，才会将键重命名。
- **键的遍历命令**
  - `KEYS pattern`：查找所有符合给定模式的键。
  - `SCAN cursor [MATCH pattern] [COUNT count]`：迭代地遍历数据库中的键。

### 6.3、字符串（String）类型命令

- **基本赋值和获取命令**
  - `SET key value`：将键值对存储到 Redis 中。
  - `GET key`：获取指定键的值。
  - `GETSET key value`：将键的旧值返回，并设置新值。
- **字符串操作命令**
  - `APPEND key value`：将值追加到指定键的字符串值后面。
  - `STRLEN key`：获取指定键的字符串值的长度。
  - `INCR key`：将键的数值值增加 1。
  - `INCRBY key increment`：将键的数值值增加指定的增量。
  - `DECR key`：将键的数值值减 1。
  - `DECRBY key decrement`：将键的数值值减少指定的减量。

### 6.4、哈希（Hash）类型命令

- **哈希的基本操作命令**
  - `HSET key field value`：将哈希表中字段的值设置为指定的值。
  - `HGET key field`：获取哈希表中指定字段的值。
  - `HSETNX key field value`：只有在字段不存在时，才将哈希表中字段的值设置为指定的值。
  - `HMSET key field1 value1 field2 value2 ...`：同时将多个字段值设置到哈希表中。
  - `HMGET key field1 [field2 ...]`：获取哈希表中多个字段的值。
  - `HDEL key field1 [field2 ...]`：删除哈希表中的一个或多个字段。
- **哈希的其他命令**
  - `HEXISTS key field`：检查哈希表中指定字段是否存在。
  - `HLEN key`：获取哈希表中字段 - 值对的数量。
  - `HKEYS key`：获取哈希表中所有的字段。
  - `HVALS key`：获取哈希表中所有的值。
  - `HGETALL key`：获取哈希表中所有的字段和值。

### 6.5、列表（List）类型命令

- **列表的基本操作命令**
  - `LPUSH key value1 [value2 ...]`：将一个或多个值插入到列表头部。
  - `RPUSH key value1 [value2 ...]`：将一个或多个值插入到列表尾部。
  - `LPOP key`：移出并获取列表的第一个元素。
  - `RPOP key`：移出并获取列表的最后一个元素。
  - `RPOPLPUSH source destination`：移出并获取 source 列表的最后一个元素，然后将其添加到 destination 列表的头部。
- **列表的其他命令**
  - `LRANGE key start end`：获取列表中指定区间内的元素。
  - `LINDEX key index`：获取列表中指定位置的元素。
  - `LLEN key`：获取列表的长度。
  - `LREM key count value`：根据参数 count 的值，移除列表中与 value 相等的元素。

### 6.6、集合（Set）类型命令

- **集合的基本操作命令**
  - `SADD key member1 [member2 ...]`：将一个或多个成员添加到集合中。
  - `SREM key member1 [member2 ...]`：移除集合中的一个或多个成员。
  - `SMEMBERS key`：获取集合中的所有成员。
  - `SISMEMBER key member`：判断成员是否是集合中的元素。
- **集合的其他命令**
  - `SCARD key`：获取集合中元素的数量。
  - `SPOP key [count]`：移除并返回集合中的一个或多个随机元素。
  - `SRANDMEMBER key [count]`：返回集合中的一个或多个随机元素。

### 6.7、有序集合（Sorted Set）类型命令

- **有序集合的基本操作命令**
  - `ZADD key score1 member1 [score2 member2 ...]`：将成员及其分数添加到有序集合中。
  - `ZREM key member1 [member2 ...]`：移除有序集合中的一个或多个成员。
  - `ZSCORE key member`：获取有序集合中成员的分数。
  - `ZRANGE key start end [WITHSCORES]`：获取有序集合中指定区间内的成员，按照分数从小到大排序。
  - `ZREVRANGE key start end [WITHSCORES]`：与 ZRANGE 类似，但成员是按照分数从大到小排序。
- **有序集合的其他命令**
  - `ZCARD key`：获取有序集合中元素的数量。
  - `ZCOUNT key min max`：计算有序集合中分数在指定区间内的成员数量。
  - `ZINCRBY key increment member`：将有序集合中成员的分数增加指定的步长。

### 6.8、事务相关命令

- **事务开始和执行命令**
  - `MULTI`：开始一个事务。
  - `EXEC`：执行事务中的所有命令。
- **事务控制命令**
  - `DISCARD`：放弃事务中所有的命令操作。
  - `WATCH key1 [key2 ...]`：在执行事务之前，监视一个或多个键。

### 6.9、持久化相关命令

- **持久化方式控制命令**
  - `SAVE`：强制 Redis 进行数据持久化操作。
  - `BGSAVE`：异步进行数据持久化操作。
- **持久化状态查询命令**
  - `LASTSAVE`：返回上次成功持久化操作的时间戳。