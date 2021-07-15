# docker-learning-notes

# 常用命令
```
// 安装docker
curl -fsSL https://get.docker.com -o get-docker.sh  && \
 bash get-docker.sh

// 查看容器状态信息
docker container inspect myNginx

// 查看容器日志信息
docker container logs myNginx

// 停止容器
docker stop myNginx

// 启动容器
docker start myNginx

// 进入容器
docker exec -it containerId bash

// 删除镜像
docker rmi ed9c93747fe1

// 宿主机和容器文件相互复制
docker cp  mytomcat:/usr/local/tomcat/conf/server.xml  server.xml
docker cp  server.xml  mytomcat:/usr/local/tomcat/conf/server.xml  

Dockerfile
// 从哪里下载镜像
FROM nginx
// 执行构建时的指令
RUN echo '<h1>chenhu</h1>' > /usr/share/nginx/html/index.html

docker build -t clangnginx:1.0 -f Dockfile .
```
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
 
## 安装mysql8.0
拉取镜像
```
$ docker pull mysql:8.0.16
```

创建数据目录和配置文件
在宿主机创建放置mysql的配置文件的目录和数据目录，并且进行授权
```
$ mkdir -p /usr/mysql/conf /usr/mysql/data
$ chmod -R 755 /usr/mysql/
```

创建配置文件
在上面创建的配置文件目录/usr/mysql/conf下创建MySQL的配置文件my.cnf
```
$ vim /usr/mysql/conf/my.cnf
```

添加以下内容到上述创建的配置文件中
```
[client]
#socket = /usr/mysql/mysqld.sock
default-character-set = utf8mb4
[mysqld]
#pid-file        = /var/run/mysqld/mysqld.pid
#socket          = /var/run/mysqld/mysqld.sock
#datadir         = /var/lib/mysql
#socket = /usr/mysql/mysqld.sock
#pid-file = /usr/mysql/mysqld.pid
datadir = /usr/mysql/data
character_set_server = utf8mb4
collation_server = utf8mb4_bin
secure-file-priv= NULL
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0
# Custom config should go here
!includedir /etc/mysql/conf.d/
```

启动创建容器
```
$ docker run --restart=unless-stopped -d --name mysql -v /usr/mysql/conf/my.cnf:/etc/mysql/my.cnf -v /usr/mysql/data:/var/lib/mysql -p 3306:3306 -e MYSQL_ROOT_PASSWORD=123456 mysql:8.0.16
```

问题
上述虽然安装好了mysql，但是使用远程的Navicat连接时提示错误，不能正确连接mysql，此时需要修改按照下面说的步骤修改一下mysql的密码模式以及主机等内容才可以。

修改mysql密码以及可访问主机
进入容器内部
```
$ docker exec -it mysql /bin/bash
```

连接mysql
```
$ mysql -uroot -p
```

使用mysql库
```
$ mysql> use mysql
```

修改访问主机以及密码等，设置为所有主机可访问
```
$ mysql> ALTER USER 'root'@'%' IDENTIFIED WITH mysql_native_password BY '新密码';
```
注意：
mysql_native_password，mysql8.x版本必须使用这种模式，否则navicate无法正确连接

刷新
```
$ mysql> flush privileges
```
经过以上步骤，再次远程使用Navicat连接数据库时就可以正常连接了

# 2、安装Redis
## 下载镜像
```
docker pull redis
```

## 启动
```
docker run --name redis -d -p 6379:6379 redis --requirepass "123456-abc"
```

## 自定义配置文件
```
创建一个文件夹用来存放redis的配置文件、存储数据
mkdir -p /usr/local/redis/conf /usr/local/redis/data
```

```
执行命令
docker run -p 6379:6379 -v /usr/local/redis/conf/redis.conf:/etc/redis/redis.conf -v /usr/local/redis/data:/data --name redis -d redis:6.2.4 redis-server /etc/redis/redis.conf
```

# 3、Docker pull网络错误解决
```
[root@localhost chenhu]# dig @114.114.114.114 registry-1.docker.io
; <<>> DiG 9.9.4-RedHat-9.9.4-72.el7 <<>> @114.114.114.114 registry-1.docker.io
; (1 server found)
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 10006
;; flags: qr rd ra; QUERY: 1, ANSWER: 8, AUTHORITY: 0, ADDITIONAL: 1
;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 512
;; QUESTION SECTION:
;registry-1.docker.io.          IN      A
;; ANSWER SECTION:
registry-1.docker.io.   39      IN      A       34.201.196.144
registry-1.docker.io.   39      IN      A       34.197.189.129
registry-1.docker.io.   39      IN      A       3.224.11.4
registry-1.docker.io.   39      IN      A       3.221.133.86
registry-1.docker.io.   39      IN      A       34.192.182.239
registry-1.docker.io.   39      IN      A       34.228.211.243
registry-1.docker.io.   39      IN      A       34.199.40.84
registry-1.docker.io.   39      IN      A       34.199.77.19
;; Query time: 38 msec
;; SERVER: 114.114.114.114#53(114.114.114.114)
;; WHEN: Wed Sep 11 03:47:59 PDT 2019
;; MSG SIZE  rcvd: 177
```
然后添加下面这条host
```
34.199.77.19 registry-1.docker.io
```

# 4、安装Nginx
```
docker pull nginx
```

```
mkdir -p /nginx/{conf,conf.d,html,logs}
```
编写nginx.conf配置文件
```
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;
events {
    worker_connections 1024;
}
http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log  /var/log/nginx/access.log  main;
    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;
    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;
    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;
    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  120.77.96.184;
        root         /usr/share/nginx/html;
        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;
       # location / {
       # proxy_pass http://pic;
       # }
        location /report/{
                proxy_pass http://119.3.194.28:39032/;
                proxy_redirect off;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header Host $http_host;
        }
        error_page 404 /404.html;
            location = /40x.html {
        }
        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
}
```

```
docker run --name nginx -d -p 80:80 -v /nginx/conf/nginx.conf:/etc/nginx/nginx.conf  -v /nginx/logs:/var/log/nginx -v /nginx/html:/usr/share/nginx/html nginx
```

# 5、CentOS7安装Docker
```
yum install -y yum-utils device-mapper-persistent-data lvm2
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
yum-config-manager --enable docker-ce-edge
yum-config-manager --enable docker-ce-test
yum-config-manager --disable docker-ce-edge
yum install docker-ce
```

# 6、Docker-Compose安装

```
curl -L https://github.com/docker/compose/releases/download/1.18.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
# 可能需要赋予执行权限(755,777,等按需自行填写)
chmod 755 /usr/local/bin/docker-compose
```

# 7、docker~使用阿里加速器
https://yq.aliyun.com/articles/325299

# 8、安装oracle
选择的docker镜像其实是一个"自动配置安装"oracle 11g的镜像, 所以你需要自己下载oracle 11g的安装包.
```
wget http://mirror.xrk.org/oracle/linux/linux.x64_11gR2_database_1of2.zip
wget http://mirror.xrk.org/oracle/linux/linux.x64_11gR2_database_2of2.zip
```
创建一个目录
```
mkdir -p /install/database
```
将下载的oracle安装压缩文件解压到/install/database
```
unzip linux.x64_11gR2_database_1of2.zip -d /install/database
unzip linux.x64_11gR2_database_2of2.zip -d /install/database
```
拉取镜像
```
docker pull jaspeen/oracle-11g
```
运行容器
```
docker run --privileged --name oracle11g -p 1521:1521 -v /install/database:/install jaspeen/oracle-11g
```
其他的错误如果没有自动跳出docker的run启动过程, 就不用管, 这个过程有点慢  
进入oracle11g容器
```
docker exec -it oracle11g /bin/bash
```
切换到image的oracle用户
```
su - oracle
```
进入oracle数据库
```
sqlplus / as sysdba
```
解锁scott用户
```
SQL> alter user scott account unlock;
User altered.
SQL> commit;
Commit complete.
SQL> conn scott/tiger
ERROR:
ORA-28001: the password has expired
Changing password for scott
New password:
Retype new password:
Password changed
Connected.
SQL> 
```
即可通过数据库管理工具连接oracle数据库

## 9、Xfce安装
```
docker run -d -p 6080:6080 -e VNC_RESOLUTION=1920x1080 yangxuan8282/alpine-xfce4-novnc:amd64
```
完成后访问：http://ip:6080 默认的vnc密码是:alpinelinux

## 10、安装Portainer
```
docker run -d -p 9000:9000 -v /var/run/docker.sock:/var/run/docker.sock --restart=always --name portainer portainer/portainer
```
