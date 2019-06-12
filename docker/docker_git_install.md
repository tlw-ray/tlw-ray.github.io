# GOGS+DRONE安装

## 运行环境

- CentOS7
- Docker1.8+
- DockerCompose 1.2+
- GIT2.0+

## 简介

## 安装包介绍

- docker-compose.yml 在Docker环境下打包安装软件的脚本
- .env 是docker-compose.yml用到的环境变量
- install.sh 根据本机配置生成环境变量，并调用安装过程
- uninstall.sh 从Docker上卸载已经安装的Drone包
- images 是readme中的截图
- mysql 为GOGS建立数据库的脚本
- project 带有CI脚本.drone.yml的GIT示例项目

## 安装步骤

1. 解压缩drone_install_190118;

~~~
tar -vxf drone_install_190118
~~~

2. 进入drone目录执行安装脚本;

~~~
  cd drone
  ./install.sh
~~~

  如图:
  ![执行install.sh脚本](images\install.jpg)


3. 从岸桥访问能够观察到GOGS+Drone软件已被安装, 包含Mysql, GOGS, Drone-Server, Drone-Agent四个部分。访问该服务器的10080会显示初次访问的配置页面。需要配置mysql数据库在容器内的连接库名为"db",密码为"root"以及该GIT服务的外部访问URL为"http://IP:10080", 这些内容已先在docker-compose.xml中定义, 如需修改请编辑该文件中的对应内容。

  如图: 
  ![岸桥中查看已经安装的内容](images\droneInPortainer.jpg)
  ![编辑默认配置信息](images\configGit01.jpg)
  ![编辑默认配置信息](images\configGit02.jpg)

4. 配置完毕后转跳到注册用户界面，请根据提示注册用户，本示例会创建用户名密码均为root的用户，邮箱为root@163.com;

  ![注册用户界面](images\registUser.jpg)

5. 注册用户并登陆后，可在服务端和客户端分别创建名为project的git仓库;

  ![服务端创建仓库](images\createRepository.jpg)
  ![服务端创建仓库1](images\createRepository1.jpg)

6. 访问drone, 并为project项目注册持续集成;

  ![注册用户并登陆](images\droneLogin.jpg)
  ![转跳到的页面中开启对project项目的CI](images\droneCI.jpg)
  ![设定CI的触发条件](images\droneHook.jpg)

7. 客户端创建project仓库，其中包含的.drone.yml是持续集成脚本。执行下列代码可将project目录建立为git仓库，并提交到刚才建立的project仓库中，在push时会根据刚才的设定触发.drone.yml内脚本的执行。

~~~
cd project
git init
git config --local user.name "root"
git config --local user.email "root@163.com"
git add --a
git commit -m "first commit"
git remote add origin http://172.16.0.175:10080/root/project.git
git push -u origin master
~~~

8. 在Drone中查阅持续集成脚本运行效果;

![项目提交后自动执行.drone.yml中的内容](images\droneCIResult.jpg)
