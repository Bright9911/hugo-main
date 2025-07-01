+++
title = '【拓展】Docker'
date = 2025-07-01T09:42:46+08:00
description = ["描述"]
draft = true
comments= true
categories = [
    "docker",
    "笔记",
    "拓展",
]
image = "1.jpg"

+++

## 1 docker的使用

### 一、基础

##### 1、镜像

```
"https://hub.littlediary.cn"

  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "http://hub-mirror.c.163.com",
    "https://dockerhub.azk8s.cn",
    "https://mirror.ccs.tencentyun.com",
    "https://registry.cn-hangzhou.aliyuncs.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://docker.m.daocloud.io",  
    "https://noohub.ru", 
    "https://huecker.io",
    "https://dockerhub.timeweb.cloud" 
 ]
```

##### 2、进入容器

```
docker exec -it php-fpm bash
```

##### 3、清除缓存

```
docker system prune -a
```



### 二

##### 1、构建项目

```
docker-compose up --build
```

##### 2、重新构建项目

```
# 清理旧镜像缓存（重要！）
docker builder prune --force

# 重新构建
docker build -t laravel52-app .

# 运行容器
docker run -it -p 8000:8000 laravel52-app
```



### 三、启动

##### 1、启动第一个docker项目

```
docker build -t hello-docker .
docker run hello-docker
```

##### 2、启动默认教程的容器

```
docker run -d -p 8090:80 docker/getting-started
```

##### 3、启动laravel项目

```
php artisan serve --host=0.0.0.0
```





## 2 docker运行laravel5全流程

### 1、拉取 PHP 7.0.0 镜像

```
docker pull php:7.0-cli
```



### 2、运行容器

```
docker run -itd --name my-php-container -p 8000:8000 php:7.0-cli
```



### 3、进入容器进行操作

```
docker exec -it my-php-container bash
```



### 4、（额外）启动、停止、删除容器

```
docker run/stop/rm my-php-container
```



### 5、（额外）保存更改

如果在容器中进行了配置更改或安装了额外的软件包，可以将容器保存为新的镜像。

```
docker commit my-php-container php:custom-7.0.0
```



### 6、配置环境

```
# 替换软件源
echo > /etc/apt/sources.list

1、echo  "deb [trusted=yes] http://mirrors.tuna.tsinghua.edu.cn/debian buster main contrib non-free" >/etc/apt/sources.list

2、echo  "deb http://mirrors.tuna.tsinghua.edu.cn/debian/ buster-updates main contrib non-free" >>/etc/apt/sources.list

3、echo  "deb http://security.debian.org/debian-security buster/updates main contrib non-free" >>/etc/apt/sources.list
```

#### 1、在宿主机上下载 Composer 安装脚本

```
wget https://install.phpcomposer.com/installer -O composer-setup.php
```

#### 2、将脚本复制到容器

```
docker cp composer-setup.php my_container:/tmp/composer-setup.php
```

#### 3、在容器中安装 Composer

```
docker exec -it my_container bash
php /tmp/composer-setup.php
mv composer.phar /usr/local/bin/composer
```

#### 4、安装必要php拓展

mongodb拓展版本为1.5.0可以启动

```
# 更新软件包
apt-get update

# 下载拓展，要有以下拓展方可运行
PS C:\Users> php -m
[PHP Modules]
bcmath
bz2
calendar
Core
ctype
curl
date
dom
exif
fileinfo
filter
gd
gettext
gmp
hash
iconv
imap
intl
json
ldap
libxml
mbstring
mcrypt
mongodb
mysqli
mysqlnd
openssl
pcre
PDO
pdo_mysql
pdo_sqlite
Phar
Reflection
session
SimpleXML
soap
sockets
SPL
sqlite3
standard
tokenizer
wddx
xml
xmlreader
xmlrpc
xmlwriter
xsl
zip
zlib

[Zend Modules]
```



### 7、文件复制进容器

```
docker cp "D:\Misc\Docker\code\dns_web\." my-php-container:/var/www/html
```



### 8、运行项目

```
docker exec -it my-php-container bash
cd /var/www/html/...
composer install
php artisan serve --host=0.0.0.0
```



<blockquote class="alert-note">



做出修改后使用以下命令将文件更新 -- **此步骤应由挂载卷来操作，此法不方便**

</blockquote>

```
docker cp "D:/Misc/Docker/code/dns_web/." my-php-container:/var/www/html/code/dns_web
```


