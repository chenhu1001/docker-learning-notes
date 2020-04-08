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
