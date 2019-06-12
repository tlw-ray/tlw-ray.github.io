# Docker私服使用说明

## 1. 概述

公司内部发布或使用Docker镜像需要使用到内部私有的镜像仓库。本文档叙述如何在docker中通过加入该私服并通过它登陆、下载、上传镜像。

## 2. Docker中加入私服

- CentOS：

    1. 编辑/etc/docker/daemon.json的insecure-registries中加入"172.16.0.183:8082"使之看起来如下: 
    
    ```JSON
    {
    "insecure-registries": ["172.16.0.183:5000", "172.16.0.183:8082", "https://registry.docker-cn.com"],
    "registry-mirrors": ["https://1wpwp4gi.mirror.aliyuncs.com"],
    "hosts": ["tcp://0.0.0.0:2375", "unix:///var/run/docker.sock"]
    }
    ```
    
    2. 加载配置变更

    ```shell
    systemctl daemon-reload
    ```

    3. 重启Docker服务

    ```shell
    systemctl restart docker
    ```

- Windows

	1. 系统托盘右键Docker图标 --> Settings --> Daemon --> Insecure registries --> 加入 172.16.0.183:8082

	2. 重启Docker服务

## 3. 登陆私服

无论下载、上传镜像都需要先登陆私服，通过以下命令可登陆。

```shell
docker login -u admin -p admin123 172.16.0.183:8082
```

## 4. 下载和安装镜像

- 4.1 MySQL

  - 概述: 需要执行创建数据卷，创建数据库容器两个步骤。

  - 参数:

    - 数据卷名: 数据卷的名称，当容器被删除时数据仍然被保存
    - 容器名: 容器的名称，建议由若干段构成，下划线分隔。可以是`团队名_容器名_软件名`, 例如:DRGs_Xinhua_MySQL5.5
    - 外部端口号: 对外的访问端口，例如3306
    - ROOT用户密码: 默认为root

  - 执行命令:
  
  ```shell
  docker volume create ${数据卷名}
  docker run --name ${容器名} -v ${数据卷名}:/var/lib/mysql -p ${外部端口号}:3306 -e MYSQL_ROOT_PASSWORD=root --restart always -d 172.16.0.183:5000/ptc/mysql:5.5
  ```

  - 以新华为例:
  
  ```shell
  docker volume create drg_xinhua_mysql5.5
  docker run --name DRGs_Xinhua_MySQL5.5 -v drg_xinhua_mysql5.5:/var/lib/mysql -p 3302:3306 -e MYSQL_ROOT_PASSWORD=root --restart always -d 172.16.0.183:5000/ptc/mysql:5.5
  ```

- 4.2 MongoDB

	- 概述: 以Compose方式安装MongoDB和Web客户端工具MongoExpress
	- 参数:
		- Mongo外部端口: MongoDB的外部端口，默认为27017，如果该服务器已经安装过MongoDB，为了避免端口冲突可修改为其它数值。
		- MongoExpress外部端口: MongoExpress外部端口，默认为8081，如果该端口已经被占用，可以使用其它端口号。
	- 步骤:
		- 创建内容如下的docker-compose.yaml文件:

        ```yaml
		version: '3.6'
		services:
		  mongo:
			restart: always
			image: 172.16.0.183:8082/ptc/mongo
			container_name: DW_Mongo
			ports:
			  - "${Mongo外部端口}:27017"
			volumes:
			  - "mongo_data_db:/data/db"
			  - "mongo_data_configdb:/data/configdb"
		  mongo_express:
			restart: always
			container_name: DW_Mongo_Express
			image: 172.16.0.183:8082/ptc/mongo-express
			depends_on:
			  - mongo
			ports:
			  - "${MongoExpress外部端口}:8081"
			environment:
			  - ME_CONFIG_MONGODB_SERVER=mongo
		volumes:
		  mongo_data_db:
		  mongo_data_configdb:
      ```
		
		- 启动容器:

        ``` shell
        docker-compse up -d
        ```
  
- 4.3 RabbitMQ

	- 概述: 安装中文版控制台的RabbitMQ
	- 条件: 参照之前第二步，Docker中加入私服的操作，需要加入172.16.0.183：8082的内网私服，并登陆入。
	- 参数: 
		- Web端口: 浏览器访问RabbitMQ管理界面的端口。默认: 15672
		- 协议端口: 外部程序访问RabbitMQ的端口。默认: 5672
		- 容器名称: 容器的名称，例如: ODS_Rabbitmq
	- 命令: 
		
	``` shell
	docker run -d --name ${容器名称} -p ${Web端口}:15672 -p ${协议端口}:5672 172.16.0.183:8082/dcs/rabbitmq:3.7.13-management-zh190530
	```
	
	- 示例: 
		
	``` shell
	docker run -d --name ODS_Rabbitmq -p 15672:15672 -p 5672:5672 172.16.0.183:8082/dcs/rabbitmq:3.7.13-management-zh190530
	```
	
- 4.4 Flink:

	- 概述: 安装Flink，包括jobmanager和taskmanager，横向扩容taskmanager，卸载。
	- 参数: 请根据需要修改下面yaml中的参数。
	- 步骤:
		- 编写docker-compose.yaml文件:
			
        ```yaml
        version: "2.1"
        services:
        jobmanager:
        image: 172.16.0.183:8082/dcs/flink:1.8.0-190602
        expose:
        - "6123"
        ports:
        - "8082:8081"
        command: jobmanager
        environment:
        - JOB_MANAGER_RPC_ADDRESS=jobmanager

        taskmanager:
        image: 172.16.0.183:8082/dcs/flink:1.8.0-190602
        expose:
        - "6121"
        - "6122"
        depends_on:
        - jobmanager
        command: taskmanager
        links:
        - "jobmanager:jobmanager"
        environment:
        - JOB_MANAGER_RPC_ADDRESS=jobmanager
        ```
			
		- 启动:
		
        ```shell
        docker-compose up -d
        ```
		
		- 扩容:
		
        ```shell
        docker-compose scale taskmanager=3
        ```
			
		- 停止:
		
        ```shell
        docker-compose down
        ```
	
- 4.5 岸桥

	- 概述: 安装岸桥管理Docker容器
	- 参数: 
		- Web端口: 浏览器访问岸桥的端口， 默认: 9000
		- 数据卷: 存放岸桥管理信息的文件卷名，例如: portainer_data
		- 容器名称: 容器的名称, 例如: DCS_Portainer
		
	- 命令:
		
		```shell
		docker volume create ${数据卷}
		docker run -d -p ${Web端口}:9000 --name ${容器名称} --restart always -v /var/run/docker.sock:/var/run/docker.sock -v ${数据卷}:/data portainer/portainer
		```
		
	- 示例:
		
		```shell
		docker volume create portainer_data
		docker run -d -p 9000:9000 --name portainer --restart always -v /var/run/docker.sock:/var/run/docker.sock -v portainer_data:/data portainer/portainer
		```
		

