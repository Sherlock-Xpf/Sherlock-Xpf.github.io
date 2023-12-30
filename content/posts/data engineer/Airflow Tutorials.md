---
title: 'Airflow Tutorials'
date: 2023-03-26T20:18:25+08:00
draft: false
author: "Me" #作者
categories: 
- work
tags: 
- Bigdata
- Tutorials
weight:  # 输入1可以顶置文章，用来给文章展示排序，不填就默认按时间排序
---

# Airflow Tutorials

---

```SHELL
## What Airflow？
    Airflow是一个以编程方式编写，安排和监视工作流的平台
    Airflow将工作流编写任务的有向无环图(DAG)
```

- 有向无环图(DAG)
![image-20220326161236333](https://homjay.oss-cn-shanghai.aliyuncs.com/image-20220326161236333.png)
## 添加Dag任务

```shell
# 编写.py脚本，创建work-py目录用于存放python调度脚本
mkdir -p /opt/module/work-py && \
cd /opt/module/work-py/ && \
vim test.py
```
```python
#!/usr/bin/python
from airflow import DAG
from airflow.operators.bash_operator import BashOperator
from datetime import datetime, timedelta

default_args = {
    'owner': 'test_owner',
    'depends_on_past': True,
    'email': ['2473196869@qq.com'],
    'start_date':datetime(2020,12,15),
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 1,
    'retry_delay': timedelta(minutes=5),
    # 'queue': 'bash_queue',
    # 'pool': 'backfill',
    # 'priority_weight': 10,
    # 'end_date': datetime(2016, 1, 1),
}
dag = DAG('test', default_args=default_args, schedule_interval=timedelta(days=1))
 
t1 = BashOperator(
    task_id='dwd',
    bash_command='ssh hadoop103 "spark-submit  --master yarn --deploy-mode client --driver-memory 1g --num-executors 2 --executor-cores 2 --executor-memory 2g --class com.atguigu.member.controller.DwdMemberController --queue spark /root/work/com_atguigu_warehouse-1.0-SNAPSHOT-jar-with-dependencies.jar "',
    retries=3,
    dag=dag)

t2 = BashOperator(
    task_id='dws',
    bash_command='ssh hadoop103 "spark-submit  --master yarn --deploy-mode client --driver-memory 1g --num-executors 2 --executor-cores 2 --executor-memory 2g --class com.atguigu.member.controller.DwsMemberController --queue spark /root/work/com_atguigu_warehouse-1.0-SNAPSHOT-jar-with-dependencies.jar "',
    retries=3,
    dag=dag)

t3 = BashOperator(
    task_id='ads',
    bash_command='ssh hadoop103 "spark-submit  --master yarn --deploy-mode client --driver-memory 1g --num-executors 2 --executor-cores 2 --executor-memory 2g --class com.atguigu.member.controller.AdsMemberController --queue spark /root/work/com_atguigu_warehouse-1.0-SNAPSHOT-jar-with-dependencies.jar "',
    retries=3,
    dag=dag)

t2.set_upstream(t1)
t3.set_upstream(t2)


 # 把任务放到仓库中
vim ~/airflow/airflow.cfg # 查看Airflow配置文件
mkdir ~/airflow/dags
cp test.py ~/airflow/dags/

# 查看任务列表
airflow list_dags
```

<img src="https://homjay.oss-cn-shanghai.aliyuncs.com/image-20220326161538510.png" alt="image-20220326161538510" style="zoom:50%;" />

- 查看任务dag

![image-20220326161946079](https://homjay.oss-cn-shanghai.aliyuncs.com/image-20220326161946079.png)

- 开启任务

![image-20220326162005959](https://homjay.oss-cn-shanghai.aliyuncs.com/image-20220326162005959.png)

- 查看dag图、甘特图

![image-20220326162031310](https://homjay.oss-cn-shanghai.aliyuncs.com/image-20220326162031310.png)

![image-20220326162037449](https://homjay.oss-cn-shanghai.aliyuncs.com/image-20220326162037449.png)

## 删除Dag任务

![image-20220326162102885](https://homjay.oss-cn-shanghai.aliyuncs.com/image-20220326162102885.png)

## 调度频率	Crontab

<img src="https://homjay.oss-cn-shanghai.aliyuncs.com/Crontab%E6%A0%BC%E5%BC%8F%E8%AF%B4%E6%98%8E%E7%9A%84%E5%B1%95%E7%A4%BA%E5%9B%BE.png" alt="Crontab格式说明的展示图" style="zoom: 50%;" />

<img src="https://homjay.oss-cn-shanghai.aliyuncs.com/Crontab%E5%91%BD%E4%BB%A4%E7%9A%84%E5%B7%A5%E4%BD%9C%E6%B5%81%E7%A8%8B%E5%9B%BE.png" alt="Crontab命令的工作流程图" style="zoom:50%;" />

# -----✂-----

# Airflow 安装

- 1、安装 **Miniconda**
- 2、安装 **python**
- 3、安装 **Airflow** 

```shell
# 下载 Miniconda
mkdir -p /opt/software && cd /opt/software/ && \
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

# 安装 Miniconda3
bash Miniconda3-latest-Linux-x86_64.sh
	>>>	/opt/module/Miniconda3
source ~/.bashrc 	
conda config --set auto_activate_base false

# 安装python
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/main
conda config --set show_channel_urls yes
conda create --name airflow2 python=3.7.4
    # 创建环境：conda create -n env_name
    # 查看所有环境：conda info --envs
    # 删除一个环境：conda remove -n env_name --all

# 安装 Airflow 
conda activate airflow2
cd /opt/software/
export SLUGIFY_USES_TEXT_UNIDECODE=yes
pip install apache-airflow

airflow db init # 初始化airflow
airflow version # 查看版本

# 启动airflow web服务,启动后浏览器访问	http://hadoop103:8087
airflow webserver -p 8087 -D
airflow scheduler -D # 启动airflow调度

```

![image-20220326160642254](https://homjay.oss-cn-shanghai.aliyuncs.com/image-20220326160642254.png)



