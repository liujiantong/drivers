# 安装步骤

## 安装前提
* 所有集群里的服务器都能访问 archive.cloudera.com
* 修改每个节点上的/etc/hosts, 为集群中的每个节点添加主机名

## install mysql

### 在namenode1安装mysql
* yum install mariadb mariadb-server
* systemctl start mariadb
* systemctl enable mariadb

### 设置mysql
* 按照[说明安装](https://www.cloudera.com/documentation/enterprise/latest/topics/install_cm_mariadb.html)

### 创建hadoop组件所需数据库
```
create database cmf DEFAULT CHARACTER SET utf8;
grant all on cmf.* TO 'cmf'@'%' IDENTIFIED BY 'cmf_password';

create database amon DEFAULT CHARACTER SET utf8;
grant all on amon.* TO 'amon'@'%' IDENTIFIED BY 'amon_password';

create database rman DEFAULT CHARACTER SET utf8;
grant all on rman.* TO 'rman'@'%' IDENTIFIED BY 'rman_password';

create database metastore DEFAULT CHARACTER SET utf8;
grant all on metastore.* TO 'hive'@'%' IDENTIFIED BY 'hive_password';

create database nav DEFAULT CHARACTER SET utf8;
grant all on nav.* TO 'nav'@'%' IDENTIFIED BY 'nav_password';

create database hue DEFAULT CHARACTER SET utf8;
grant all on hue.* TO 'hue'@'%' IDENTIFIED BY 'hue_password';

create database oozie DEFAULT CHARACTER SET utf8;
grant all on oozie.* TO 'oozie'@'%' IDENTIFIED BY 'oozie_password';
```

### 设置 cloudera-scm-server 数据库
[安装说明](https://www.cloudera.com/documentation/enterprise/latest/topics/cm_ig_installing_configuring_dbs.html#cmig_topic_5)
- /usr/share/cmf/schema/scm_prepare_database.sh mysql cmf cmf cmf_password

## 安装CDH v5.10.x
- [安装说明](https://www.cloudera.com/documentation/enterprise/latest/topics/cm_ig_install_path_b.html#cmig_topic_6_6_1__section_gkr_z31_v5)
```
wget https://archive.cloudera.com/cm5/redhat/7/x86_64/cm/cloudera-manager.repo
cp cloudera-manager.repo /etc/yum.repos.d/
yum install cloudera-manager-daemons cloudera-manager-server
systemctl start cloudera-scm-server
systemctl enable cloudera-scm-server
```

### 安装过程
1. 添加集群
2. 搜索主机, root/ikang.com@88



# Issues

1. JAVA_HOME NOT FOUND
- export JAVA_HOME="/usr/local/java"
- add java home to /usr/share/cmf/bin/cmf-server
- /etc/default/bigtop-utils
- yum install bigtop-utils

[ref here](https://www.cloudera.com/documentation/enterprise/latest/topics/cdh_ig_jdk_installation.html#topic_29)

2. Could not load requested class : com.mysql.jdbc.Driver

- wget https://dev.mysql.com/get/Downloads/Connector-J/mysql-connector-java-5.1.41.tar.gz
- cp mysql-connector-java-5.1.41-bin.jar /usr/share/java/mysql-connector-java.jar

[ref here](https://www.cloudera.com/documentation/enterprise/latest/topics/cm_ig_mysql.html#cmig_topic_5_5_1)

3. Connections could not be acquired from the underlying database
- vim /etc/cloudera-scm-server/db.properties
- com.cloudera.cmf.db.setupType=EXTERNAL

4. install mysql
- sudo systemctl enable mariadb
- sudo systemctl start mariadb

5. java home
- export JAVA_HOME="/usr/local/java"
- /usr/lib64/cmf/service/common/cloudera-config.sh

6. debug error log
- /var/log/cloudera-scm-server/cloudera-scm-server.log
- /var/log/cloudera-scm-server/cloudera-scm-server.out

7. Error, CM server guid updated | Could not find a HOST_MONITORING
- systemctl stop cloudera-scm-agent
- rm -f /var/lib/cloudera-scm-agent/cm_guid
- systemctl start cloudera-scm-agent

8. zookeeper start error
- change dir to /hadoop/zookeeper
- chown -R zookeeper:zookeeper /var/lib/zookeeper

9. oozie start error
- oozie need to be installed with yarn

10. httpfs on namenode1
- chown -R httpfs:hadoop /var/lib/hadoop-httpfs
- chmod 0755 /var/lib/hadoop-httpfs

11. Permission denied
- chmod 0755 /var/lib/hadoop-yarn/yarn-nm-recovery/LOCK

12. spark start error: applicationHistory not exists
- sudo -u hdfs hadoop fs -mkdir /user/spark
- sudo -u hdfs hadoop fs -mkdir /user/spark/applicationHistory
- sudo -u hdfs hadoop fs -chown -R spark:spark /user/spark
- sudo -u hdfs hadoop fs -chmod 1777 /user/spark/applicationHistory

13. hive metastore not init
- create database metastore DEFAULT CHARACTER SET utf8;
- use metastore;
- source /etc/hive/hive-schema-1.1.0.mysql.sql

14. oozie table oozie.validate_conn not exist, oozie-cmf-oozie-OOZIE_SERVER-namenode2.log.out
- you need to use "Create Database" from Cloudera Manager
- Click on Oozie, then in the Actions menu on the right, select Create Database

15. yarn namenode2 error
- chown -R yarn:hadoop /var/lib/hadoop-yarn
- chmod 0755 /var/lib/hadoop-yarn

16. pstree command not FOUND
- yum install psmisc

17. oozie db schema exists
- drop database oozie;
- re-create database oozie;

18. Failed to add storage directory [DISK]file:/home/dfs/dn/
- rm -rf /home/dfs/dn

19. Versions of CDH that are too new for this version of Cloudera Manager will not be shown
- HASH value in PARCELS is NULL
- reset cmf database

20. Failed to create spark client. Permission denied: user=anonymous, access=WRITE
- sudo -u hdfs hadoop fs -chmod g+w /user
- beeline -n hdfs -u jdbc:hive2://namenode2:10000

21. add impala module
- add impala service using cloudera manager

22. install anaconda
- to support python api for spark ml

23. sqoop: Could not load db driver class: oracle.jdbc.OracleDriver
- copy ojdbc6.jar to /var/lib/sqoop

24. sqoop: Container is running beyond physical memory limits
- yarn.scheduler.minimum-allocation-mb: 2g
- mapreduce.map.memory.mb: 4g
- mapreduce.reduce.memory.mb: 8g
- mapreduce.map.java.opts.max.heap: 3g
- mapreduce.reduce.java.opts.max.heap: 6g

25. Enabling the Oozie Web Console
- Download [ext-2.2](). Extract the contents of the file to /var/lib/oozie/ on the same host as the Oozie Server
- Enable Oozie server web console in Cloudera Manager Admin Console

25. cloudera manager 提示 DNS解析错误
- hostnamectl set-hostname flume-node1
