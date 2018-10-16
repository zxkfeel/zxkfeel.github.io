#### centos7安装rabbitmq

#### 1.所需安装包

_rabbitmq-server-3.6.5-1.noarch.rpm_

_socat-1.7.3.2-2.el7.x86_64.rpm_

_erlang-18.3-1.el7.centos.x86_64.rpm_

下载地址：

` wget http://www.rabbitmq.com/releases/erlang/erlang-18.3-1.el7.centos.x86_64.rpm` 

` wget http://www.rabbitmq.com/releases/rabbitmq-server/v3.6.5/rabbitmq-server-3.6.5-1.noarch.rpm`

` wget http://www.rpmfind.net/linux/centos/7.5.1804/os/x86_64/Packages/socat-1.7.3.2-2.el7.x86_64.rpm`



#### 2.安装

2.1 安装erlang:

` rpm -ivh erlang-18.3-1.el7.centos.x86_64.rpm`

2.2 安装socat

`rpm -ivh socat-1.7.3.2-2.el7.x86_64.rpm`

2.3 安装rabbitmq

`rpm -ivh rabbitmq-server-3.6.5-1.noarch.rpm`

#### 3.配置

3.1 配置管理台用户名密码

`vim /usr/lib/rabbitmq/lib/rabbitmq_server-3.6.5/ebin/rabbit.app`

修改loopback_users中的<<"guest">>，只保留guest

3.2 服务的启动和停止

启动：`rabbitmq-server start &`

停止：`rabbitmqctl app_stop`

kill服务：`ps -ef|grep rabbit`

找到daemon进程，以及rabbitmq进程pid，kill掉

3.3 启动管理插件

`rabbitmq-plugins enable rabbitmq_management`

3.4 访问管理台

192.168.0.202:15672

#### 4.集群配置：

需要在/etc/hosts/中配置，例如：

192.168.0.201  first-server

192.168.0.202 second-server

192.168.0.203 third-server

first-server/serond-server/third-server为主机名，可以在/etc/hostname中查看

#### 5.卸载

/sbin/service rabbitmq-server stop

yum list | grep rabbitmq

yum -y remove rabbitmq-server.noarch

#### 6.开机自启

`chkconfig rabbitmq-server on`

#### 7.创建root用户并赋administrator角色

`rabbitmqctl add_user root root`

`rabbitmqctl set_user_tags root administrator`

#### 8.主从配置

例如：192.168.0.201，hostname为server201;192.168.0.202,hostname为server202，以201为主，202为从，（其实没有很明显的主从的概念，两台rabbitmq其实都是一样的，使用默认的硬盘存储）

##### 8.1 配置hosts（/etc/hosts）

两台机器hosts分别配置：

192.168.0.201 server201

192.168.0.202 server202

##### 8.2 复制cookie

201和202机器，/var/lib/rabbitmq/.erlang.cookie内容保持一致，将201的cookie内容复制到202中。

##### 8.3 将子节点加入主节点

202上关闭rabbitmq服务，并执行：

`rabbitmqctl join_cluster rabbit@server201`

然后启动202的服务，查看状态

`rabbitmqctl cluster_status`

201和202管理台中的nodes栏也会显示两个mq服务。