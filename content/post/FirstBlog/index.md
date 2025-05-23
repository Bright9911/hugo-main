+++
title = 'Redis的使用'
date = 2025-05-23T10:23:18+08:00
draft = true
tags = [
    "markdown",
    "redis",
]
categories = [
    "redis",
    "笔记",
]
image = "pawel-czerwinski-8uZPynIu-rQ-unsplash.jpg"

+++

## 下载并解压

https://github.com/tporadowski/redis/releases

## 启动Redis服务器

```
.\redis-server.exe redis.windows.conf
```

默认端口为 6379，如果看到类似 Ready to accept connections 的消息，说明 Redis 服务已成功启动。

## 测试Redis

```
.\redis-cli.exe -h 127.0.0.1 -p 6379
```

执行 ping 命令，如果返回 PONG，说明 Redis 服务正常运行。

## 将 Redis 注册为 Windows 服务（可选）（没做）

在 Redis 安装目录中，运行以下命令将 Redis 注册为 Windows 服务：

```
redis-server.exe --service-install redis.windows.conf --service-name redisserver1 --loglevel verbose
```

启动服务：

```
redis-server.exe --service-start --service-name redisserver1
```

通过 services.msc 查看服务是否安装成功，并可以配置为开机自启动。



## 报错解决

当出现以下问题时：

```
127.0.0.1:6379> del name
(error) MISCONF Redis is configured to save RDB snapshots, but it is currently not able to persist on disk. Commands that may modify the data set are disabled, because this instance is configured to report errors during writes if RDB snapshotting fails (stop-writes-on-bgsave-error option). Please check the Redis logs for details about the RDB error.
```

##### 临时解决方法：

```
CONFIG SET stop-writes-on-bgsave-error no
```

##### 持久解决方法：

修改Redis配置文件

找到 Redis 配置文件 redis.conf（通常位于 Redis 安装目录或 /etc/redis/redis.conf）。

使用文本编辑器打开 redis.conf 文件，找到 stop-writes-on-bgsave-error 选项，并将其设置为 no：

```
stop-writes-on-bgsave-error no
```

保存文件并重启 Redis 服务。