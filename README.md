# docker-learning-notes

# 1、安装MySQL
## 下载镜像
```
docker pull mysql:5.6
```

## 配置文件
```
mkdir -p /opt/docker-mysql/conf.d
```
增加并修改配置文件config-file.cnf
内容如下,设置表名不区分大小写; linux下默认是区分的，windows下默认不区分
```
[mysqld]
# 表名不区分大小写
lower_case_table_names=1 
#server-id=1
datadir=/var/lib/mysql
#socket=/var/lib/mysql/mysqlx.sock
#symbolic-links=0
# sql_mode=NO_ENGINE_SUBSTITUTION,STRICT_TRANS_TABLES 
[mysqld_safe]
log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
```

## 启动

增加数据文件夹
```
mkdir -p /opt/docker-mysql/var/lib/mysql
```
启动，设置默认密码 123456-abc

```
$ docker run --name mysql \
    --restart=always \
    -p 3306:3306 \
    -v /opt/docker-mysql/conf.d:/etc/mysql/conf.d \
    -v /opt/docker-mysql/var/lib/mysql:/var/lib/mysql \
    -e MYSQL_ROOT_PASSWORD=123456-abc \
    -d mysql:5.6
 ```
## 常用命令
 
 进入容器
 ```
 docker exec -it mysql bash
 ```
 查看日志
 ```
 docker logs -f mysql
 ```
