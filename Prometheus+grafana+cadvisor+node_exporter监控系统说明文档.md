## 容器化实现Prometheus+grafana+cadvisor+node_exporter监控系统

一.安装部署Prometheus容器



#### 1.Prometheus（监控平台）

账号：**admin**

密码：**admin**

账号密码都为默认，因为容器开启后不需要输入账号密码，版本更新

地址：**202.202.43.139:9090**

#### 2.grafana（仪表盘平台）

账号：**admin**

密码：**cquptlanshan**

地址：**202.202.43.139:3000**

#### 3.cadvisor（监控容器信息的组件）

地址：***:8081**(所有的服务器都是8081端口)

#### 4.node_exporter（监控服务器如cpu,流量等主机信息的组件）

地址：***:9111**(所有的服务器都是9111端口)

注意：node_exporter开启方式为systemctl start node_exporter(因为9100端口占用，换端口容器还是没解决)

#### 5.监控平台使用方法及注意事项

##### 1.监控新服务器

先在新服务器上运行**node_exporter**组件

然后在**mastera**服务器上，**cd /home/cqupt**后使用**vi $PWD/prometheus/prometheus.yml**，将**- job_name**为**node_exporter**字段下的**targets**字段中添加**新服务器IP加组件运行端口**，保存退出后，**docker restart prometheus**命令后，再到Prometheus平台刷新即可看见新服务器已被监控

##### 2.监控新服务器上的docker容器信息

先在新服务器上运行**cadvisor**组件,**cd /home/cqupt**后使用**vi $PWD/prometheus/prometheus.yml**，将**- job_name**为cadvisor字段下的**targets****字段中添加**新服务器IP加组件运行端口**，保存退出后，**docker restart prometheus**命令后，再到Prometheus平台刷新即可看见新服务器下的docker容器信息已被监控

**3.grafana现存仪表盘的介绍和使用**

点击主页**home旁边的三杠**，找到**Dashboards**字样，名为**主机监控**以及**node_exporter**的仪表盘为监控主机信息的仪表盘，其余的均为监控docker容器信息的仪表盘，两类仪表盘均import了中文版

##### 如何import新仪表盘

查询你所需要功能的仪表盘的**ID**，点击**Dashboards**中的**new**,下拉选择**import**，输入**ID**，点击**load,**再点击**最下面的load**，再选择**数据源Prometheus（default)**,最后点击**import**，新仪表盘即可使用

**注意事项**：

编辑Prometheus的命令**不建议**修改，组件运行时请确**保组件默认端口未被占用**





