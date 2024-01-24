---

---

## docker部署Debian10的zabbix监控系统

### 一.Debian安装并启用docker

#### 1.安装docker

##### 下载（[组件地址](https://download.docker.com/linux/debian/dists/buster/pool/stable/amd64/))

containerd.io_1.6.4-1_amd64.deb

docker-buildx-plugin_0.10.2-1~debian.10~buster_amd64.deb

docker-ce-cli_24.0.0-1~debian.10~buster_amd64.deb

docker-ce_24.0.0-1~debian.10~buster_amd64.deb

docker-compose-plugin_2.18.1-1~debian.10~buster_amd64.deb

上述四个docker组件（因**相关依赖下载不了**的问题，所以选择通过下载组件的方式来启动docker)

##### 安装

```
sudo dpkg -i /path/to/package.deb   把/path/to/换为自己文件所在的路径
```

**PS：在下载插件过程中，若出现版本不对的问题以及插件顺序的问题请参考终端报错信息来解决**

**更多方法请参考[docker官网](https://dockerdocs.cn/engine/install/debian/#os-requirements)**

#### 2.启动docker

systemctl start docker **（启动doker）**

systemctl enable docker**（设置开机自启）**

systemctl status docker**（查看docker运行状况）**

### 二.安装部署zabbix

#### 1.拉取相关镜像

##### 拉取mysql5.7镜像
docker pull mysql:5.7
##### 拉取zabbix/zabbix-java-gateway:centos-5.4-latest镜像
docker pull zabbix/zabbix-java-gateway:centos-5.4-latest
##### 拉取zabbix/zabbix-server-mysql:centos-5.4-latest镜像
docker pull zabbix/zabbix-server-mysql:centos-5.4-latest
##### 拉取zabbix/zabbix-web-nginx-mysql:centos-5.4-latest镜像
docker pull zabbix/zabbix-web-nginx-mysql:centos-5.4-latest
##### 拉取zabbix/zabbix-snmptraps:centos-5.4-latest镜像（告警镜像）
docker pull zabbix/zabbix-snmptraps:centos-5.4-latest
##### 查看镜像
docker images



#### 2.安装zabbix-agent2

##### Debian10 安装 zabbix-agent2
wget https://mirrors.aliyun.com/zabbix/zabbix/5.4/debian/pool/main/z/zabbix/zabbix-agent2_5.4.9-1+debian10_amd64.deb
dpkg -i zabbix-agent2_5.4.9-1+debian10_amd64.deb

##### 编辑配置文件允许容器网段访问
##### 修改 Server=172.22.0.0/24  具体网段和docker-compose.yml的容器配置一致
vi /etc/zabbix/zabbix_agent2.conf
##### 重启
systemctl restart zabbix-agent2
##### 设置开机自动启动
systemctl enable zabbix-agent2


#### 3.创建程序目录（名字可自取）
mkdir -p /opt/zabbix
##### 进入目录
cd /opt/zabbix
##### 创建snmp trap告警目录
mkdir -p ./snmptraps
##### 设置snmptraps目录权限
chown -R 1997 ./snmptraps
##### 创建mib目录
mkdir -p ./mibs
##### 创建自定义告警脚本目录
mkdir -p ./alertscripts
##### 创建自定义外部检查脚本目录
mkdir -p ./externalscripts
##### 创建mysql配置文件目录
mkdir -p ./mysql/conf
##### 创建mysql数据库文件目录
mkdir -p ./mysql/data
##### 创建日志文件目录
mkdir -p ./mysql/logs

##### 设置mysql目录权限
chown -R 999.999 ./mysql/logs
chown -R 999.999 ./mysql/data

**PS：告警目录和mib库看自己是否需要来创建（在后面的容器配置中可决定是否挂载这些目录。如要挂挂载，目录必须存在）**

#### **4.创建并编辑docke**r-compose.yml配置文件

nano docker-compose.yml

#### 5.docker-compose.yml文件内容

version: '3'
services:
  mysql:
    image: mysql:5.7
    container_name: mysql
    volumes:
      - ./mysql/data:/var/lib/mysql
      - ./mysql/conf:/etc/mysql/conf.d
      - ./mysql/logs:/var/log/mysql
      - /etc/localtime:/etc/localtime
    restart: always
    privileged: true
    environment:
      # root密码
      - MYSQL_ROOT_PASSWORD=root@zabbix
      # 新建数据库
      - MYSQL_DATABASE=zabbix
      # 新建用户
      - MYSQL_USER=zabbix
      # 新用户密码
      - MYSQL_PASSWORD=admin@zabbix
      - TZ=Asia/Shanghai
      - LANG=en_US.UTF-8
    ports:
      - "3306:3306"
    networks:
      zabbix-net:
    command: --character-set-server=utf8 --collation-server=utf8_bin
  zabbix-gateway:
    image: zabbix/zabbix-java-gateway:centos-5.4-latest
    container_name: zabbix-gateway
    volumes:
      - /etc/localtime:/etc/localtime
    restart: always
    privileged: true
    ports:
      - "10052:10052"
    networks:
      zabbix-net:
  zabbix-snmptraps:
    image: zabbix/zabbix-snmptraps:centos-5.4-latest
    container_name: zabbix-snmptraps
    volumes:
      - /etc/localtime:/etc/localtime
      - ./snmptraps:/var/lib/zabbix/snmptraps
    restart: always
    privileged: true
    ports:
      - "162:1162/udp"
    networks:
      zabbix-net:
  zabbix-server:
    image: zabbix/zabbix-server-mysql:centos-5.4-latest
    container_name: zabbix-server
    volumes:
      - /etc/localtime:/etc/localtime
      - ./snmptraps:/var/lib/zabbix/snmptraps
      - ./alertscripts:/usr/lib/zabbix/alertscripts
      - ./externalscripts:/usr/lib/zabbix/externalscripts
    restart: always
    privileged: true
    environment:
      # 监听端口
      - ZBX_LISTENPORT=10051
      # 数据库地址
      - DB_SERVER_HOST=mysql
      # 数据库端口
      - DB_SERVER_PORT=3306
      # 数据库名
      - MYSQL_DATABASE=zabbix
      # 数据库用户
      - MYSQL_USER=zabbix
      # 数据库密码
      - MYSQL_PASSWORD=admin@zabbix
      # 数据库root密码
      - MYSQL_ROOT_PASSWORD=root@zabbix
      # 用于存储主机 监控项 触发器数据的共享内存大小
      - ZBX_CACHESIZE=1G
      # 历史缓存数据大小
      - ZBX_HISTORYCACHESIZE=512M
      # 历史索引缓存大小
      - ZBX_HISTORYINDEXCACHESIZE=16M
      # 用于存储趋势数据的共享内存大小
      - ZBX_TRENDCACHESIZE=256M
      # 历史数据缓存大小
      - ZBX_VALUECACHESIZE=256M
      # ICMP pingers进程数
      - ZBX_STARTPINGERS=64
      # IPMI进程数
      - ZBX_IPMIPOLLERS=1
      # 开启Traps告警
      - ZBX_ENABLE_SNMP_TRAPS=true
      # Traps进程数
      - ZBX_STARTTRAPPERS=1
      # 开启zabbix java gateway
      - ZBX_JAVAGATEWAY_ENABLE=true
      # zabbix java gateway地址
      - ZBX_JAVAGATEWAY=zabbix-gateway
      # java gateway进程数
      - ZBX_STARTJAVAPOLLERS=1
    ports:
      - "10051:10051"
    networks:
      zabbix-net:
    links:
      - mysql
      - zabbix-gateway
  zabbix-web:
    image: zabbix/zabbix-web-nginx-mysql:centos-5.4-latest
    container_name: zabbix-web
    volumes:
      - /etc/localtime:/etc/localtime
    restart: always
    privileged: true
    environment:
      # Web页面左上角程序名
      - ZBX_SERVER_NAME=Zabbix 5.4
      # zabbix server地址
      - ZBX_SERVER_HOST=zabbix-server
      # zabbix server端口
      - ZBX_SERVER_PORT=10051
      # 数据库地址
      - DB_SERVER_HOST=mysql
      # 数据库端口
      - DB_SERVER_PORT=3306
      # 数据库名
      - MYSQL_DATABASE=zabbix
      # 数据库用户
      - MYSQL_USER=zabbix
      # 数据库密码
      - MYSQL_PASSWORD=admin@zabbix
      # 数据库root密码
      - MYSQL_ROOT_PASSWORD=root@zabbix
      # 时区
      - PHP_TZ=Asia/Shanghai
    ports:
      - "80:8080"
    networks:
      zabbix-net:
    links:
      - mysql
      - zabbix-server
networks:
  zabbix-net:
    driver: bridge
    ipam:
      config:
        # 配置容器网段
        - subnet: 172.22.0.0/24
          gateway: 172.22.0.1

**PS:所有的容器的映射端口须得查看是否被占用，若被占用，请更换为未被占用的端口，MySQL密码用户名保持一致links对象请保证前后容器名一致**

#### 6.启动容器

##### 进入目录
cd /opt/zabbix
##### 创建并后台启动容器
docker compose up -d
##### 查看mysql日志
docker logs mysql
##### 查看zabbix java gateway日志
docker logs zabbix-gateway
##### 查看zabbix snmptraps日志
docker logs zabbix-snmptraps
##### 查看zabbix server日志
docker logs zabbix-server
##### 查看zabbix nginx日志
docker logs zabbix-web
##### 以root权限进入容器zabbix-server
#docker exec -u root -ti zabbix-server /bin/bash
##### 复制mibs下的所有文件至容器zabbix-server的/var/lib/zabbix/mibs目录
#docker cp /opt/zabbix/mibs/. zabbix-server:/var/lib/zabbix/mibs/

PS：mibs文件根据自己的需求来，[下载地址1](https://support.huawei.com/enterprise/zh/software/253350229-ESW2000397776),[下载地址2](https://www.h3c.com/cn/d_200905/635750_30003_0.htm),[了解mib库](https://www.zabbix.com/documentation/5.0/zh/manual/config/items/itemtypes/snmp/mibs?hl=MIB%2Cmib)

##### 从容器zabbix-server复制zabbix配置文件至宿主机当前目录
#docker cp zabbix-server:/etc/zabbix/zabbix_server.conf ./

#### 7.访问测试

浏览器访问：[http地址](172.22.181.132:80)
用户名：Admin
密码：zabbix

PS:账号手写字母不可为小写

#### 8.使用手册

[更多](https://www.zabbix.com/documentation/5.0/zh/manual/it_services)

#### 9.添加新的监控主机

1.下载新的zabbix-agent2

2.更改zabbix-agent2.cof的配置文件改变server(ip地址加网段或者0.0.0.0/0),host name（为创建主机时的主机名称）,serveractive（zabbix-agent2所在服务器IP）

3.启动zabbix-agent2服务并设置开机自启

启动agent2

systemctl start zabbix-agent2

设置开机自启动agent2   

systemctl enable zabbix-agent2

![image-20231209191116943](C:\Users\Vinci\AppData\Roaming\Typora\typora-user-images\image-20231209191116943.png)

登录进web页面找到配置，然后选择主机

![image-20231209191213754](C:\Users\Vinci\AppData\Roaming\Typora\typora-user-images\image-20231209191213754.png)

再选择创建主机

![image-20231209191752498](C:\Users\Vinci\AppData\Roaming\Typora\typora-user-images\image-20231209191752498.png)

主机名称为hostname,可见名称自定义，群组自定义选择，再点击interfaces下的添加选择客户端

![image-20231209191918974](C:\Users\Vinci\AppData\Roaming\Typora\typora-user-images\image-20231209191918974.png)

点击模板，先在主机群组里选择Templates再选择Linux by zabbix agent ,systemd by zabbix agent2

![image-20231209192227795](C:\Users\Vinci\AppData\Roaming\Typora\typora-user-images\image-20231209192227795.png)

最后点击添加，即可添加新的监控服务器

### 三.zabbix接入grafana数据平台

#### 1.docker部署grafana

**下载grafana镜像,该命令下载最新镜像，也可指定版本**

docker pull grafana/grafana:latset

docker pull grafana/grafana:tag

**创建挂载目录**

mkdir /path/to/your/grafana/{data,plugins,config} -p**（分别在grafana下创建data,plugins,confi目录）**

**进入grafana目录**

cd /path/to/your/grafana/

**查看grafana目录**

ls

**分别给data,plugins,config添加权限（不添加可能会报错）**

chmod 777 data/

chmod 777 plugins/

chmod 777 config/

**启动一个临时grafana的容器，为了cp配置文件**

docker run --name grafana-tmp -d -p 3000:3000 grafana/grafana:latest

**复制文件到容器所在服务器得挂载目录**

docker cp grafana-tmp:/etc/grafana/grafana.ini /path/to/your/grafana/config/

**移除临时容器**

**停止容器**

docker stop grafana-tmp 

**删除容器**

docker rm grafana-tmp

**重新启动一个正式的grafana容器**

docker run -d \
     -p 3000:3000 \
     --name=grafana \
     --restart=always \
     -v /etc/localtime:/etc/localtime:ro \
     -v /path/to/your/grafana/data:/var/lib/grafana \
     -v /path/to/your/grafana/plugins/:/var/lib/grafana/plugins \
     -v /path/to/your/grafana/config/grafana.ini:/etc/grafana/grafana.ini \
     -e "GF_SECURITY_ADMIN_PASSWORD=admin" \ (临时密码为admin，登进平台后会要求修改密码)
     -e "GF_INSTALL_PLUGINS=grafana-clock-panel,grafana-simple-json-datasource,grafana-piechart-panel" \
     grafana/grafana:latest

**查看该容器日志**

docker logs grafana

![image-20231211164104960](C:\Users\Vinci\AppData\Roaming\Typora\typora-user-images\image-20231211164104960.png)

上述日志情况出现**downloaded**字样，**等待**容器下载完所需插件**一段时间**后，即**可正常登入web页面**

#### **2.下载zabbix数据源**（默认无数据源）

**进入容器**

docker exec -it grafana bash

**下载插件**

grafana-cli plugins install alexanderzobnin-zabbix-app

**成功下载后**

**退出容器**

exit

**重启容器**

docker restart grafana

同**查看该日志**那里**一样**

#### 3.登入grafana的 web页面

账号：admin

密码：admincqupt

**home**那里找到**dashboards**

然后里面的仪表盘即可使用





