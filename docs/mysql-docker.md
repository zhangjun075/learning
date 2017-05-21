# /mysql/mysql-server镜像

启动后：docker run --name mysql -e MYSQL_ROOT_PASSWORD=123456   -d -p 3306:3306  mysql/mysql-server:latest

本地telnet是不通的。

这个时候如下操作：

docker exec -it mysql mysql -uroot -p

USE mysql;CREATE USER 'root'@'%' IDENTIFIED BY '密码';GRANT ALL PRIVILEGES ON . TO 'root'@'%';FLUSH PRIVILEGES;

以上两步操作即可。

# mysql创建docker镜像

Dockerfile
```
FROM mysql:latest

MAINTAINER  braveheart <zhangjunheling@163.com>
LABEL Descripttion="This image is build for MAC to use mysql" Vendor="GitHub" Version="latest"
RUN apt-get update
RUN apt-get -y install vim
RUN usermod -u 1000 mysql
RUN mkdir -p /var/run/mysqld
RUN chmod -R 777 /var/run/mysqld
```

执行命令：
```
docker build -t zhangjun05/mysql .
```

启动docker命令
```
docker run -p 3309:3306 -v /Users/junzhang/data/mysqldata/:/var/lib/mysql -e MYSQL_ROOT_PASSWORD=123456 -d --name mysql zhangjun05/mysql
```
