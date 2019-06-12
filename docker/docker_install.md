# Docker安装

# 基本步骤:

1. 设定yum源
2. 升级linux内核
3. 安装Docker-ce
4. 配置daemon.json

## 1. 设定会被用到的yum源

### 1.1 设置阿里云作为源

    mv /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
    wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo

### 1.2 添加阿里epel源 

    yum-config-manager --add-repo http://mirrors.aliyun.com/repo/epel-7.repo

### 1.3 更新yum缓存，安装后面会用到的软件，并进行系统更新

    yum install -y telnet

### 1.4 添加ELRepo源

    rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
    rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm

### 1.5 添加docker-ce的源

    yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

### 1.6 清理缓存并生成新的缓存

    yum clean all
    yum makecache
    yum update -y

## 2. 升级Linux内核到较新的长效维护版(lt)内核

### 2.1 由于存在冲突，先卸载现有的kernel-headers

    yum erase kernel-headers -y

### 2.2 安装kernel-lt, kernel-lt-devel, kernel-lt-headers

    yum --enablerepo=elrepo-kernel install kernel-lt kernel-lt-devel kernel-lt-headers -y

### 2.3 设置GRUB默认的内核版本, 将第一个内核作为默认启动内核

    sed -i s/GRUB_DEFAULT=saved/GRUB_DEFAULT=0/ /etc/default/grub

### 2.4 重新创建内核配置

    grub2-mkconfig -o /boot/grub2/grub.cfg

### 2.5 重启

    reboot

### 3. 安装docker-ce18.3

### 3.1 安装依赖的包

    yum install -y device-mapper-persistent-data lvm2

### 3.2 安装docker-ce

    yum install -y docker-ce

### 3.3 修改SELinux从enforcing为permissive模式

    sed -i s/SELINUX=enforcing/SELINUX=permissive/ /etc/selinux/config

### 3.4 设定服务自启动

    systemctl enable docker

### 3.5 配置防火墙

建议直接关闭，否则后面可能遇到其它问题，如果不想关闭可以:

        firewall-cmd --permanent --zone=public --add-interface=docker0 
        firewall-cmd --permanent --zone=public --add-port=2375/tcp
        firewall-cmd --reload


### 3.6 配置/etc/docker/daemon.json

- 用到的阿里云码是我注册的，后面请替换为您自己的。
- 添加了内网的仓库，阿里云镜像，开放2375端口。

    {
      "insecure-registries": ["172.16.0.183:5000"],
      "registry-mirrors": ["https://1wpwp4gi.mirror.aliyuncs.com"],
      "hosts": ["tcp://0.0.0.0:2375", "unix:///var/run/docker.sock"]
    }
    
### 3.7 配置/usr/lib/systemd/system/docker.service

- 修改docker.service文件，避免上边daemon.json修改后导致docker服务无法启动的问题

~~~sheel
sed -i 's/-H fd:\/\/ //' /usr/lib/systemd/system/docker.service
~~~

该命令的效果是，对于文件docker.service的第14行， [Service]段落内如果为: 

	ExecStart=/usr/bin/dockerd -H df:// --containerd=/run/containerd/containerd.sock

那么，需要去掉其中的 -H df:// 改为如下: 

	ExecStart=/usr/bin/dockerd --containerd=/run/containerd/containerd.sock

## 4. 启动服务

	systemctl start docker