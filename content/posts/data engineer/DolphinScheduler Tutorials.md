---
title: 'DolphinScheduler Tutorials'
date: 2023-03-28T20:18:25+08:00
draft: false
author: "Me" #作者
categories: 
- work
tags: 
- Bigdata
- Tutorials
weight:  # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
---
# DolphinScheduler 教程

---

## 简介

- 系统架构图
![image-20220326114546396](https://homjay.oss-cn-shanghai.aliyuncs.com/image-20220326114546396.png)
- 启动流程图
![image-20220326114622069](https://homjay.oss-cn-shanghai.aliyuncs.com/image-20220326114622069.png)
### 架构说明:
1. **MasterServer** # 主要负责 DAG 任务切分、任务提交监控，并同时监听其它MasterServer和WorkerServer的健康状态
2. **WorkerServer** # 主要负责任务的执行和提供日志服务
3. ZooKeeper # 系统中的MasterServer和WorkerServer节点都通过ZooKeeper来进行集群管理和容错
4. Task Queue # 提供任务队列的操作，目前队列也是基于Zookeeper来实现
5. Alert # 提供告警相关接口，接口主要包括告警两种类型的告警数据的存储、查询和通	知功能
6. **API** # API接口层，主要负责处理前端UI层的请求。
7. UI # 系统的前端页面，提供系统的各种可视化操作界面。

---

### 去中心化 & 中心化
1. 中心化思想 : 分布式集群中的节点按照角色分工 MASTER + SLAVE
   - 问题 : 一旦Master出现了问题，则群龙无首，整个集群就会崩溃。
   - 解决 : 多数Master/Slave架构模式都采用了主备Master的设计方案，可以是热备或者冷备，
2. 去中心化思想 : 通常没有Master/Slave的概念，所有的角色都是一样的，地位是平等的，全球互联网就是一个典型的去中心化的分布式系统，

	
#### DolphinScheduler的去中心化
- Master/Worker注册到"Zookeeper"中，实现Master集群和Worker集群无中心，并使用Zookeeper"分布式锁来选"举其中的一台Master或Worker为"管理者"来执行任务。

- 中心化
![image-20220326115024058](https://homjay.oss-cn-shanghai.aliyuncs.com/image-20220326115024058.png)

## 案例

### shell

![image-20220326140502551](https://homjay.oss-cn-shanghai.aliyuncs.com/image-20220326140502551.png)

![image-20220326140526631](https://homjay.oss-cn-shanghai.aliyuncs.com/image-20220326140526631.png)

![image-20220326140622547](https://homjay.oss-cn-shanghai.aliyuncs.com/image-20220326140622547.png)

![image-20220326140704969](https://homjay.oss-cn-shanghai.aliyuncs.com/image-20220326140704969.png)

![image-20220326140717585](https://homjay.oss-cn-shanghai.aliyuncs.com/image-20220326140717585.png)

### spark

![image-20220326140755121](https://homjay.oss-cn-shanghai.aliyuncs.com/image-20220326140755121.png)

![image-20220326144745391](https://homjay.oss-cn-shanghai.aliyuncs.com/image-20220326144745391.png)

![image-20220326144831244](https://homjay.oss-cn-shanghai.aliyuncs.com/image-20220326144831244.png)

### hive

![image-20220326150021133](https://homjay.oss-cn-shanghai.aliyuncs.com/image-20220326150021133.png)

### dataX

![image-20220326150054701](https://homjay.oss-cn-shanghai.aliyuncs.com/image-20220326150054701.png)

![image-20220326150411669](https://homjay.oss-cn-shanghai.aliyuncs.com/image-20220326150411669.png)

### 作业调度

![image-20220326150520310](https://homjay.oss-cn-shanghai.aliyuncs.com/image-20220326150520310.png)

# --------------------✂------------------------

# DolphinScheduler 安装

---

```SHELL
cd /opt/software # 上传tar
tar -zxvf apache-dolphinscheduler-incubating-1.3.3-dolphinscheduler-bin.tar.gz -C /opt/module && \
cd /opt/module && \
mv apache-dolphinscheduler-incubating-1.3.3-dolphinscheduler-bin dolphinscheduler-bin

# 分别给hadoop101,hadoop102,hadoop103三台机器创建dolphinscheduler用户 要配置sudo免密
useradd dolphinscheduler && \
echo "dolphinscheduler123" | passwd --stdin dolphinscheduler && \
echo 'dolphinscheduler  ALL=(ALL)  NOPASSWD: NOPASSWD: ALL' >> /etc/sudoers && \
sed -i 's/Defaults    requirett/#Defaults    requirett/g' /etc/sudoers

# 配置密码登录 密码 : dolphinscheduler123
su dolphinscheduler
ssh-keygen -t rsa -P '' -f ~/.ssh/id_rsa
cat ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
for ip in hadoop102 hadoop103;
do
ssh-copy-id $ip
done

# 数据库初始化 ： 添加mysql-connector-java驱动包到dolphinscheduler的lib目录下
cp mysql-connector-java.jar /opt/module/dolphinscheduler/dolphinscheduler-bin/lib/

# 创建dolphinscheduer所需数据库
mysql -uroot -p123456
CREATE DATABASE dolphinscheduler DEFAULT CHARACTER SET utf8 DEFAULT COLLATE utf8_general_ci;
GRANT ALL PRIVILEGES ON dolphinscheduler.* TO 'dolphinscheduler'@'%' IDENTIFIED BY '123456';
flush privileges;

# 修改配置数据库文件
cd /opt/module/dolphinscheduler-bin/conf && \
vim datasource.properties

spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://hadoop102:3306/dolphinscheduler?useUnicode=true&characterEncoding=UTF-8&allowMultiQueries=true 
spring.datasource.username=dolphinscheduler
spring.datasource.password=123456

# 执行脚本 创建表
cd /opt/module/dolphinscheduler-bin && \
sh script/create-dolphinscheduler.sh 

# 修改 dolphinscheduler_env.sh 环境变量
cd /opt/module/dolphinscheduler-bin && \
vim conf/env/dolphinscheduler_env.sh

export HADOOP_HOME=/opt/module/hadoop-3.1.3
export HADOOP_CONF_DIR=/opt/module/hadoop-3.1.3/etc/hadoop
#export SPARK_HOME1=/opt/soft/spark1
export SPARK_HOME2=/opt/module/spark
#export PYTHON_HOME=/opt/soft/python
export JAVA_HOME=/opt/module/java
export HIVE_HOME=/opt/module/hive
#export FLINK_HOME=/opt/soft/flink
#export DATAX_HOME=/opt/soft/datax/bin/datax.py

# 将jdk软链到/usr/bin/java下
ln -s /opt/module/java/bin/java /usr/bin/java

# 修改安装参数
cd /opt/module/dolphinscheduler-bin && \
vim conf/config/install_config.conf

dbtype="mysql"
dbhost="hadoop102:3306"
username="dolphinscheduler"
dbname="dolphinscheduler"
password="123456"
zkQuorum="hadoop102:2181,hadoop103:2181,hadoop104:2181"

# ds安装目录 不同于/opt/module/dolphinscheduler
installPath="/opt/module/ds"

deployUser="dolphinscheduler"
mailServerHost="smtp.qq.com"
mailServerPort="25"

# sender,配置了和mailUser一样就行
mailSender="841747290@qq.com"

# user
mailUser="841747290@qq.com"

#邮箱密码
mailPassword="xxxxxx"

#starttlsEnable和sslEnable不能同时为true
starttlsEnable="true"
sslEnable="false"
sslTrust="smtp.qq.com"

resourceStorageType="HDFS"
defaultFS="hdfs://hadoop102:9820"

#resourcemanager HA对应的地址
yarnHaIps="hadoop101,hadoop103"

#因为使用了resourcemaanger Ha所以保持默认,如果是单resourcemanager配置对应ip
singleYarnIp="yarnIp1"

#资源上传根路径，支持hdfs和s3
resourceUploadPath="/data/dolphinscheduler"

hdfsRootUser="root"

#需要部署ds的机器
ips="hadoop102,hadoop103,hadoop104"
sshPort="22"

#指定master
masters="hadoop104"

#指定workers，并且可以指定组名，default为默认组名
workers="hadoop102:default,hadoop103:default"

#报警服务器地址
alertServer="hadoop102"

#后台api服务器地址
apiServers="hadoop102"

# 切换用户,一键部署
cd /opt/module && \
chown -R dolphinscheduler:dolphinscheduler dolphinscheduler-bin/
cd /opt/module/dolphinscheduler-bin
su dolphinscheduler
sh install.sh 

# 开启
cd /opt/module/ds && \
sh bin/start-all.sh

# web页面
http://hadoop102:12345/

# 用户名 和 密码
admin 
dolphinscheduler123
```

