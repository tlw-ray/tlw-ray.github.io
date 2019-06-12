# 基于Docker的SQLServer安装

- 撰写: 唐力伟
- 时间: 20190424

## 概述

- Docker环境下挂卷安装Linux版本的SQLServer
- Linux版本下SQLServer建库操作
- 建库异常处理

## 运行环境

- CentOS7 +
- MSSQL镜像: https://hub.docker.com/r/microsoft/mssql-server-linux
- NavicatSQLServer

## 1. 安装MSSQL镜像
- 建立数据卷
	~~~shell
	docker volume create dos_mssql_data
	~~~
- 拉取并执行镜像创建数据库容器,(注意: 密码包含符号，数字，大小写字母，长度大于8位。）
	~~~shell
	docker run --name ${DB_CONTAINER_NAME} -v dos_mssql_data:/var/opt/mssql  -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=${STRONG_PASSWORD}'  -p 1433:1433 -d ptc/mssql-server-linux:latest
	~~~
## 2. 建立数据库
	- 打开navicat(SQLServer版)
	- 建立连接: 文件-->新建连接-->输入IP,用户名，密码，测试并连接。
	- 建立数据库: 
		- 双击已经建立的连接，使之变为绿色。
		- 右键该连接，新建数据库。
			- 数据库名: 常规--> 输入`数据库名`并选择`兼容级别`为100。如图:![](images/createDatabase1.png)
			- 数据文件(重要)如图:![](images/createDatabase2.png)
				- 弹出对话框中选择"文件"页，输入数据库名称!
				- 数据库文件:选中第一行的.ndf文件，设定"文件目录"为"/var/opt/mssql"。
				- 数据库文件:选中第二行的.ldf文件，设定"文件目录"为"/var/opt/mssql"。
				- 设定的路径是上面挂载卷的位置，也是mssql在linux下默认放置数据文件的位置。
				- 可以通过选择"SQL预览"页面查看创建数据库的SQL语句。
				- 点击确定按钮建立数据库。

## 3. 建库异常处理

当建库失败后再建库会提示如下图的错误信息。

![lock](images/lock.png)

由于model上的锁导致无法继续建立数据库。此时，请尝试执行以下语句，杀掉所有锁后可以继续建立数据库:

- 查询锁
~~~SQL
select spid ,PROGRAM_NAME from master..sysprocesses
where DB_NAME(dbid) = 'model' 
~~~
- 杀死锁
~~~SQL
kill 54
~~~


