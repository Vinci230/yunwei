**一般的python的dokerfile的编写在外面完成好了**



FROM python:3.9我拉取的需要用到的镜像程序

WORKDIR  /app在容器中创建app的工作目录

COPY .   /app将我的项目路径复制到这里来，注意路径的正确,点代表在当前文件下的目录

RUN pip install  -r requirements.txt下载依赖项

EXPOSE 8080暴露端口

CMD ["python", "项目那么.py"]后面的指的是我的程序叫什么名字就什么名字

注意：注意路劲正确以及依赖项，dokerfile文件在同一个目录啦

**带有js文件的dockerfile文件的编写：**

FROM python:3.9
RUN apt-get update \
    && apt-get install -y curl \
    && curl -fsSL https://deb.nodesource.com/setup_14.x | bash - \
    && apt-get install -y nodejs
WORKDIR /app

COPY hongrendj.py /app
COPY hongrendj_log.js /app
COPY hongrendj_sign.js /app
COPY requirements.txt /app
RUN pip install -r requirements.txt
RUN npm install
EXPOSE 8080

CMD ["python", "hongrendj.py"]

////////////////////这个是最好的选择

FROM python:3.9

##### 设置工作目录
WORKDIR /app

##### 复制Python应用程序所需的文件
COPY hongrendj.py /app
COPY requirements.txt /app

##### 安装Python依赖
RUN pip install -r requirements.txt
RUN pip install requests

##### 安装Node.js
RUN apt-get update \
    && apt-get install -y curl \
    && curl -fsSL https://deb.nodesource.com/setup_14.x | bash - \
    && apt-get install -y nodejs

##### 复制Node.js应用程序所需的文件
COPY hongrendj_log.js /app
COPY hongrendj_sign.js /app
COPY package.json package-lock.json /app

##### 安装Node.js依赖
//RUN npm install

##### 暴露端口
EXPOSE 8080

##### 设置启动命令
CMD ["python", "hongrendj.py"]

docker rmi -f 镜像id 强制删除某个不需要的镜像

docker build --tag <镜像名字>:<标签> <Dockerfile目录路径>，这个目录路径指的是dockerfile文件在的目录路径而不是dockerfile的路径

然后**开启harbor**，

cd /harbor

要在harbor目录中进行终端

docker-compose up -d

登录harbor：

docker login --username admin http://8.130.101.163:8080

docker login -u admin -p Harbor12345 http://8.130.101.163:8080

登录成功后

docker tag 镜像名字：版本 8.130.101.163:8080/我的项目目录名字/镜像名字：新版本

docker push 8.130.101.163:8080/我的项目目录名字/镜像名字：新版本

就推送到harbor仓库成功啦，推送完后关闭harbor，依然在harbor里进行命令

docker-compose stop

别人拉取：

docker pull  8.130.101.163:8080/我的项目目录名字/镜像名字：新版本

**就可以拉取成功**

**运行gielab_runner**

docker run -d --name gitlab-runner --restart always \
 -v /var/run/docker.sock:/var/run/docker.sock \
 -v /etc/gitlab-runner/config:/etc/gitlab-runner \
 --env TZ=Asia/Beijing \
 gitlab/gitlab-runner:latest

**在服务器上生成runner**

**\# -url 输入你的gitlab地址 --registration-token 输入你的token**
sudo docker exec -it gitlab-runner gitlab-runner register --url http://gitlab.com --registration-token GR1348941oLThQ2BHjiL9otfxWd2P
Runtime platform                  arch=amd64 os=linux pid=36 revision=44a1c2be version=15.6.0
WARNING: The 'register' command has been deprecated in GitLab Runner 15.6 and will be replaced with a 'deploy' command. For more information, see https:/
/gitlab.com/gitlab-org/gitlab/-/issues/380872                
Running in system-mode.)例如：https://gitlab.com/):
https://gitlab.com]: # 直接敲Enter或替换输入你的gitlab
Enter the registration token:
[7xBMLJpZsc-oKS5BpCjc]: # 直接敲Enter或替换输入你的token
Enter a description for the runner:

[c8b010cd1bc9]: deploy	"直接敲回车"

Enter tags for the runner (comma-separated):

这个的tags就是后面yml编写里面的tags

Enter optional maintenance note for the runner:

Registering runner... succeeded           runner=7xBMLJpZ
Enter an executor: kubernetes, docker, parallels, ssh, docker+machine, docker-ssh+machine, instance, custom, docker-ssh, shell, virtualbox:
docker
Enter the default Docker image (for example, ruby:2.7):
osrf/ros:humble-simulation-jammy
Runner registered successfully. Feel free to start it, but if it's running already the config should be automatically reloaded!

Configuration (with the authentication token) was saved in "/etc/gitlab-runner/config.toml"

**.**

**gitlab-ci.yml文件编写**

一定要注意需不需要禁用

\## 定义pipeline流程：verify->build->dockerpush->deploy（自己看需要几个阶段，有几个阶段就有几个任务）
stages:
 \- verify
 \- build
 \- dockerpush
 \- deploy

\#单元测试
unit-test:
 stage: verify # 属于哪个流程
 tags:
  \- test-cicd # 在哪个runner上面执行，在注册runner可以自定义
 script:
  \- echo unit-test # 执行脚本

\#java编译
java-package:
 stage: build
 tags:
  \- test-cicd
 script:
  \- echo build

\#push镜像
docker-push:
 stage: dockerpush
 tags:
  \- test-cicd
 script:
  \- echo docker-push

\#deploy
service-1:
 stage: deploy
 tags:
  \- test-cicd
 script:
  \- echo deploy

上传到gitlab即可

gitlab-ctl resta

 wget  https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml  --no-check-certificate  
--2023-07-16 01:17:33--  https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

[root@localhost /]# cd etc
[root@localhost etc]# kubectl apply -f kube-flannel.yml

kubectl get node