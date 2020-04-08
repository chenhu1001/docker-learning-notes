## 一、引言  
最近有个项目需求在内网环境的RedHat6.8服务器上部署，因不能联网，就需要离线安装docker、docker-compose 。在可联网使用yum工具安装是非常方便，不过离线安装的话也可以使用rpm来完成。  
## 二、下载离线安装需要docker、docker-compose安装包以及其依赖包  
在docker官网查看对RedHat6.8兼容性较好的版本为Docker version 1.7.1，再查 Docker Compose Github Docs，发现docker-compose 1.5.2 版本是兼容Docker 1.7.1 的，不然安装高版本的docker-compose后当运行docker-compose build的时候，就会提示我们的Dcoker版本太低要求升级Docker。然后下载相关的安装包以及依赖包：6个docker安装依赖包为lxc-libs-1.0.9-1.el6.x86_64.rpm、lxc-1.0.9-1.el6.x86_64.rpm、lua-lxc-1.0.9-1.el6.x86_64.rpm、lua-filesystem-1.4.2-1.el6.x86_64.rpm、lua-alt-getopt-0.7.0-1.el6.noarch.rpm、libcgroup-0.40.rc1-12.el6.x86_64.rpm，docker安装包docker-io-1.7.1-2.el6.x86_64.rpm，docker-compose 1.5.2二进制文件docker-compose-Linux-x86_64。  
## 三、依次安装需要docker依赖包，以及docker  
安装命令rpm -ivh xxx，6个docker依赖包安装顺序为1、lua-filesystem-1.4.2-1.el6.x86_64.rpm，2、lua-alt-getopt-0.7.0-1.el6.noarch.rpm ，3、lxc-libs-1.0.9-1.el6.x86_64.rpm，4、lua-lxc-1.0.9-1.el6.x86_64.rpm，5、lxc-1.0.9-1.el6.x86_64.rpm，6、libcgroup-0.40.rc1-12.el6.x86_64.rpm；最后安装docker：rpm -ivh docker-io-1.7.1-2.el6.x86_64.rpm，验证安装结果：docker --version 出来“Docker version 1.7.1, build 786b29d/1.7.1”即安装完成。安装完成后，需要启动Docker守护进程：service docker start；最后，可选的操作（但建议启用），让Docker在服务器系统启动时启动：chkconfig docker on。  
## 四、安装docker-compose  
重命名文件：cp docker-compose-Linux-x86_64 docker-compose；赋予可执行权限：chmod +x docker-compose；移动到可执行程序目录mv docker-compose /usr/local/bin/docker-compose，验证安装结果：docker-compose --version 出来“docker-compose version 1.5.2, build 7240ff3”即安装完成。  
## 五、部署脚本  
为了方便部署多台，写成批量部署脚本，与所有下载文件放同一目录即可，运行脚本需先赋予可执行权限chmod +x docker-setup。所有安装文件和脚本在附件中docker.rar中。  
## 六、在可以访问外网的服务器上，下载Docker镜像，通过镜像迁移的方式，使离线服务器获取Docker镜像
### 1、环境准备
*服务器node01、node02  
*node01可以访问外网，node02不能访问外网，但node01与node02之间是互通的  
*node01和node02均已成功安装并启动Docker  
### 2、在node01上，从远程仓库获取oracle-12c镜像
```
	[root@node01 ~]# docker pull sath89/oracle-12c
	Using default tag: latest
	Trying to pull repository docker.io/sath89/oracle-12c ...
	latest: Pulling from docker.io/sath89/oracle-12c

	863735b9fd15: Pull complete
	4fbaa2f403df: Pull complete
	44be94a95984: Pull complete
	a3ed95caeb02: Pull complete
	b8bc6e8767ee: Pull complete
	c918da326197: Pull complete
	448e1619a038: Pull complete
	faadd00cf98e: Pull complete
	48a252b66251: Pull complete
	0be1ba285f23: Pull complete
	Digest: sha256:0d075905ca2243f0c60397e49eaae6edd62afcce43528df77768f346ece7c49b
```
### 3、查看下载好的镜像
```
	[root@node01 ~]# docker images
	REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
	docker.io/sath89/oracle-12c   latest              7508a4d8d54f        13 days ago         5.703 GB
```
### 4、将oracle-12c镜像保存成 tar 归档文件
```
[root@node01 ~]# docker save -o oracle-12c.tar sath89/oracle-12c
[root@node01 ~]# ls
oracle-12c.tar
```
### 5、将保存好的oracle-12c.tar上传至服务器node02上
```
	[root@node01 ~]# scp oracle-12c.tar node02:~
	oracle-12c.tar                              100% 5469MB  43.8MB/s   02:05
```
### 6、登录node02，加载oracle-12c.tar
```
	[root@node02 ~]# docker load -i oracle-12c.tar
	227021bc9aa6: Loading layer [==================================================>] 196.8 MB/196.8 MB
	80ec921b230b: Loading layer [==================================================>] 208.9 kB/208.9 kB
	1f253a0703ec: Loading layer [==================================================>] 4.608 kB/4.608 kB
	5f70bf18a086: Loading layer [==================================================>] 1.024 kB/1.024 kB
	0488c9e2de86: Loading layer [==================================================>]   983 kB/983 kB
	105daf83e29c: Loading layer [==================================================>] 2.048 kB/2.048 kB
	032e683277c4: Loading layer [==================================================>] 350.7 kB/350.7 kB
	e276bf45da60: Loading layer [==================================================>] 5.524 GB/5.524 GB
	37bb0779ab40: Loading layer [==================================================>] 6.656 kB/6.656 kB
	f32071949555: Loading layer [==================================================>] 12.18 MB/12.18 MB
	Loaded image: docker.io/sath89/oracle-12c:latest
```
### 7、oracle-12c镜像加载完成，查看
```
	[root@node02 ~]# docker images
	REPOSITORY                    TAG                 IMAGE ID            CREATED             SIZE
	docker.io/sath89/oracle-12c   latest              7508a4d8d54f        2 weeks ago         5.703 GB
```
