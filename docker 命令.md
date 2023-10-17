systemctl start docker 启动docker服务

systemctl stop docker 停止服务

systemctl restart docker 重启docker服务

systemctl status docker 查看docker状态

systemctl enable docker 设置docker服务开机自启

镜像（image）相关命令

查看镜像

docker images

docker images -q 查看所有的镜像

搜索镜像（从网络中查找需要的镜像）

docker search 镜像名称

拉取镜像：

docker pull 镜像名称  #名称：版本号

#可以去hub.docker.com查看对应镜像的版本号

删除镜像

docker  rmi  镜像id#删除指定的id的镜像

docker  rmi   ‘docker images -q'组合命令，先查找所有镜像id然后再删除

容器命令：

查看容器：

docker ps 显示正在运行的容器

docker ps -a 显示历史所有容器

创建并启动容器

docker run -it --name=c1 镜像名字：版本  /bin/bash #创建一个容器名为c1，并分配一个终端，进入 /bin/bash,退出容器后容器自动关闭

docker run -id --name==c2 镜像名字：版本 #创建一个容器名为c2，后台运行，并且进入容器之后退出也不会关闭容器

进入容器

docker exec -it 容器名称 /bin/bash #退出容器，容器不会关闭

停止容器

docker stop 容器名称

启动容器

docker start 容器名称

删除容器

docker rm 容器名称 #不能删除运行中的容器

查看容器信息

docker inspect 容器名称

# 查看所有的容器（不加-a：查看当前正在运行的容器的详情）
[root@localhost ~]# docker ps -a

# 删除容器：
[root@localhost ~]# docker rm 容器ID

# 进入容器
[root@localhost ~]# docker exec -it 容器ID /bin/bash
[root@localhost ~]# docker exec -it a6e3963fd253 /bin/bash
[root@localhost ~]# docker attach 容器名或容器ID

# 退出容器
# 使用Ctrl + P + Q退出容器，就不会中断工程，等于退出容器后，还可访问容器的工程，再进入，也是使用命令：docker attach  容器ID
# ctrl+d   退出容器且关闭, docker ps 查看无
# ctrl+p+q 退出容器但不关闭

# 附加：进入容器中,安装jdk和tomcat的步骤和在linux中安装步骤一致，你可以把容器当成一个linux虚拟机。

数据卷

概念：

是宿主机中的一个目录或文件

当容器目录和数据卷目录绑定之后，双方的修改是同步的

容器 和 数据卷 之间是多对多的关系，即一个容器可以挂载多个数据卷，一个数据卷也可以同时被多个容器所挂载

作用：容器数据的持久化（不会因为容器出现故障删除后丢失其中的数据）

外部机器和容器间接通信

容器之间的数据交换

配置

创建启动容器时， 使用 -v 参数来设置数据卷容器 和 数据卷

docker run -it  --name=xx \

-v 宿主机目录：容器内目录

-v宿主机目录：容器内目录#挂载的目录

……

centos：7 #要创建的容器的镜像名

注意：目录必须是绝对目录

目录不存在会自动创建

可以挂在多个数据卷

