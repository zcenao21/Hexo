---
title: Airflow安装使用
date: 2021-03-04 20:16:00
tags: 
    - airflow
    - 调度
categories: 
    - other
---

airflow是一个优秀的开源调度软件，可视化做的很棒，操作方便。airflow本身是python编写，需要安装python环境。<!--more--> 安装环境为CentOS7.3，work权限（非root）。具体安装的软件如下：

**Python 3.7.1**

**MySQL 5.7.33**

**Caddy 0.10.11** 

**airflow 1.10.5**



## Mysql安装

### 下载配置

mysql下载完成后上传到当前普通用户目录下解压，依次执行以下命令

```
$ tar -zxvf mysql-5.7.29-linux-glibc2.12-x86_64.tar.gz         #解压文件包
$ mv mysql-5.7.29-linux-glibc2.12-x86_64 /home/apps/mysql      #移动到指定目录并重命名
```

编辑my.cnf配置文件，放在当前mysql安装目录下，依次执行以下命令

```
$ cd mysql	         #进入安装目录
$ vim my.cnf	      #编辑配置文件
```

编辑my.cnf文件，我这里mysql的路径是/home/apps/mysql，需根据自己的路径进行修改

```
[client]
port=3306					#服务端口
socket=/home/apps/mysql/mysql.sock		#指定套接字文件

[mysqld]
port=3306					#服务端口
basedir=/home/apps/mysql			#mysql安装路径
datadir=/home/apps/mysql/data                   #数据目录
pid-file=/home/apps/mysql/mysql.pid		#指定pid文件
socket=/home/apps/mysql/mysql.sock		#指定套接字文件
log_error=/home/apps/mysql/error.log            #指定错误日志
server-id=100                                   #Mysql主从唯一标识
```



### 安装启动mysql
安装：依次执行以下命令，指定配置文件安装并初始化mysql，没有报错即安装成功

```
$ cd bin
$ ./mysqld --defaults-file=/home/apps/mysql/my.cnf --initialize --user=apps --basedir=/home/apps/mysql --datadir=/home/apps/mysql/data		#安装并初始化mysql
```

启动：依次执行以下命令，没有报错并能成功监听3306端口即表示启动成功

```
$ ./mysqld_safe --defaults-file=/home/apps/mysql/my.cnf --user=apps &      #启动mysql
$ netstat -tln | grep 3306		                              #查看是否成功监听3306端口
```



### 登入mysql
获取初始密码，初始密码在error.log日志文件内，执行以下命令

```
$ cd ..
$ less error.log | grep root@localhost		#查找root用户的初始登录密码
```

登录mysql，直接输入登录命令 bin/mysql -u root -p 有可能会报mysql没有找到/tmp/mysqk.sock文件
有两种解决方法
如果本机上没有其他数据库，可以通过软连接方式将寻找sock文件的路径指向我们mysql安装目录下的sock文件
也可以直接指定mysql.sock文件启动，执行以下命令：

```
$ ./mysql -u root -p -S /home/apps/mysql/mysql.sock 	#指定sock文件登录
```

成功登入mysql后，修改登录密码，执行以下sql语句

```
SET PASSWORD FOR 'root'@'localhost' = PASSWORD('123456');	--设置登录密码为123456
flush privileges;
```





## Python安装

```
wget https://www.python.org/ftp/python/3.7.1/Python-3.7.1.tgz // 下载
tar -zxvf *.tgz // 解压
到解压目录中
mkdir /home/work/soft/python37 //创建目录，作为工作目录
./configure --prefix=/home/work/soft/python37
make && make install
```



## Airflow安装

```
pip install apache-airflow==1.10.5
```

这个过程中可能会出现依赖冲突等各种问题，逐一解决。

安装完成后

先export AIRFLOW_HOME=xxx，export AIRFLOW_GPL_UNIDECODE=yes

再到 /home/work/soft/python37/bin目录下，执行./airflow，是为了在AIRFLOW_HOME下生成airflow.cfg文件



### 配置airflow

配置AIRFLOW_HOME目录下的airflow.cfg文件

```
[core]
dags_folder = /home/work/airflow/dags # 配置dag文件夹
executor = CeleryExecutor  # celeryExecutor方便分布式部署
sql_alchemy_conn =  mysql://airflow:airflow@localhost/airflow  # 配置mysql
sql_engine_encoding = utf-8
task_runner = BashTaskRunner

[celery]
broker_url = sqla+mysql://airflow:airflow@localhost/airflow
result_backend = db+mysql://airflow:airflow@localhost/airflow 
```



### airflow添加用户登录

```
在 airflow.cfg 文件中 [webserver] 下添加如下配置
[webserver]
authenticate = True
auth_backend = airflow.contrib.auth.backends.password_auth

然后执行：

import airflow
from airflow import models, settings
from airflow.contrib.auth.backends.password_auth import PasswordUser
user = PasswordUser(models.User())
user.username = 'in_airflow'  # 用户名
user.password = 'mifi_in'   # 用户密码
session = settings.Session()
session.add(user)
session.commit()
session.close()
exit()
```



### 替换UTC时间

airflow右上角显示的默认为UTC时间，需要更改源码

1.在airflow家目录下修改airflow.cfg，设置

```
 default_timezone = Asia/Shanghai
```

2.进入airflow包的安装位置,也就是site-packages的位置,以下修改文件均为相对位置

```
cd /root/.virtualenvs/af/lib/python3.4/site-packages/ 
```

3.修改airflow/utils/timezone.py

在 utc = pendulum.timezone(‘UTC’) 这行(第27行)代码下添加,

```
from airflow import configuration as conf
try:
	tz = conf.get("core", "default_timezone")
	if tz == "system":
		utc = pendulum.local_timezone()
	else:
		utc = pendulum.timezone(tz)
except Exception:
	pass
```

修改utcnow()函数 (在第69行)

```
原代码 d = dt.datetime.utcnow() 
修改为 d = dt.datetime.now()
```

4.修改airflow/utils/sqlalchemy.py
在utc = pendulum.timezone(‘UTC’) 这行(第37行)代码下添加

```
from airflow import configuration as conf
try:
	tz = conf.get("core", "default_timezone")
	if tz == "system":
		utc = pendulum.local_timezone()
	else:
		utc = pendulum.timezone(tz)
except Exception:
	pass
```

注释cursor.execute(“SET time_zone = ‘+00:00’”) (第124行)

5.修改airflow/www/templates/admin/master.html(第31行)

```
把代码 var UTCseconds = (x.getTime() + x.getTimezoneOffset()*60*1000); 
改为 var UTCseconds = x.getTime();

把代码 "timeFormat":"H:i:s %UTC%",
改为  "timeFormat":"H:i:s",
```

最后重启airflow-webserver即可

### 初始化数据库

```
airflow resetdb
airflow initdb
```

### 启动airflow

使用脚本检查，如果没有相应服务则启动。总共四个服务，webserver，scheduler，servelog，worker

```
#!/bin/sh
source ~/.bashrc
time=`date "+%Y-%m-%d-%H-%M-%S"`
echo "start check webserver" $time
ps -fe|grep "airflow webserver\\|airflow-webserver" |grep -v grep
#不存在
if [ $? -ne 0 ]
then
    echo $time "不存在"
    cd /home/work/soft/airflow
    rm -rf /home/work/soft/airflow/airflow-webserver.*
    nohup airflow webserver > /home/work/soft/airflow/logs/airflow-webserver_${time}.log &
    ps -fe|grep "airflow webserver\\|airflow-webserver" |grep -v grep
#存在
else
    echo $time "存在"
fi
```

```
#!/bin/sh
source ~/.bashrc
time=`date "+%Y-%m-%d-%H-%M-%S"`
echo "start check scheduler" $time
ps -fe|grep "airflow scheduler" |grep -v grep
#不存在
if [ $? -ne 0 ]
then
    echo $time "不存在"
    cd /home/work/soft/airflow
    pwd
    nohup /home/work/soft/python37/bin/airflow scheduler > /home/work/soft/airflow/logs/airflow-scheduler_${time}.log &
    ps -fe|grep "airflow scheduler" |grep -v grep
#存在
else
    echo $time "存在"
fi
```

```
#!/bin/sh
source ~/.bashrc
time=`date "+%Y-%m-%d-%H-%M-%S"`
echo "start check airflow servelog" $time
ps -fe|grep "airflow serve_logs" |grep -v grep |grep -v root
#不存在
if [ $? -ne 0 ]
then
    echo $time "不存在"
    cd /home/work/soft/airflow
    nohup airflow serve_logs> /home/work/soft/airflow/logs/airflow-servelog_${time}.log &
    ps -fe|grep "airflow serve_logs" |grep -v grep |grep -v root
#存在
else
    echo $time "存在"
fi
```

```
#!/bin/sh
source ~/.bashrc
time=`date "+%Y-%m-%d-%H-%M-%S"`
echo "start check airflow worker" $time
ps -fe|grep "celeryd: xxxx:MainProcess" |grep -v grep |grep -v root
#不存在
if [ $? -ne 0 ]
then
    echo $time "不存在"
    cd /home/work/soft/airflow
    nohup airflow worker > /home/work/soft/airflow/logs/airflow-worker_${time}.log &
    ps -fe|grep "celeryd: xx:MainProcess" |grep -v grep |grep -v root
    #存在
else
    echo $time "存在"
fi
```





## caddy安装

caddy在这里是为了自动部署代码，可以直接关联到git。每次git代码修改，都会自动部署到airflow上

下载linux版本，解压，然后配置 Caddyfile

```
0.0.0.0:6666 {
    root /home/work/bin/xx_scripts
    log /home/work/soft/caddy/access.log
    git {
        repo git@git.xxx-scripts.git
        path /home/work/bin/xx_scripts
        key /home/work/.ssh/id_rsa
        hook /deploy
        hook_type generic
        then rm -v /home/work/airflow/dags
        then rm -v /home/work/airflow/sqls
        then rm -v /home/work/airflow/tests
        then ln -sv /home/work/bin/xx_scripts/dags /home/work/airflow/dags
        then ln -sv /home/work/bin/xx_scripts/sqls /home/work/airflow/sqls
        then ln -sv /home/work/bin/xx_scripts/tests /home/work/airflow/tests
        interval 3600
    }
}
```

启动caddy

```
nohup ./caddy -config Caddyfile &
```

