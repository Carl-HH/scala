# Hive 0.13搭建
## 安装hive包
1. 将apache-hive-0.13.1-bin.tar.gz使用SCP命令上传到spark1的/usr/local目录下。
2. 解压缩hive安装包：<br>`tar -zxvf apache-hive-0.13.1-bin.tar.gz`。
3. 重命名hive目录：<br> `mv apache-hive-0.13.1-bin hive`
4. 配置hive相关的环境变量
``` shell
vi .bashrc
export HIVE_HOME=/usr/local/hive
export PATH=$HIVE_HOME/bin
source .bashrc
```
## 安装mysql
1. 在spark1上安装mysql。
2. 使用yum安装mysql server。
``` shell
yum install -y mysql-server
service mysqld start
chkconfig mysqld on
```
3. 使用yum安装mysql connector<br>
`yum install -y mysql-connector-java`
4. 将mysql connector拷贝到hive的lib包中
``` shell
cp /usr/share/java/mysql-connector-java-5.1.17.jar /usr/local/hive/lib
```
5. 在mysql上创建hive元数据库，并对hive进行授权
``` sql 
create database if not exists hive_metadata;
grant all privileges on hive_metadata.* to 'hive'@'%' identified by 'hive';
grant all privileges on hive_metadata.* to 'hive'@'localhost' identified by 'hive';
grant all privileges on hive_metadata.* to 'hive'@'spark1' identified by 'hive';
flush privileges;
use hive_metadata;
```
## 配置hive-site.xml
``` shell
mv hive-default.xml.template hive-site.xml
vi hive-site.xml
```
``` xml
<property>
  <name>javax.jdo.option.ConnectionURL</name>
  <value>jdbc:mysql://spark1:3306/hive_metadata?createDatabaseIfNotExist=true</value>
</property>
<property>
  <name>javax.jdo.option.ConnectionDriverName</name>
  <value>com.mysql.jdbc.Driver</value>
</property>
<property>
  <name>javax.jdo.option.ConnectionUserName</name>
  <value>hive</value>
</property>
<property>
  <name>javax.jdo.option.ConnectionPassword</name>
  <value>hive</value>
</property>
<property>
  <name>hive.metastore.warehouse.dir</name>
  <value>/user/hive/warehouse</value>
</property>
```
## 配置hive-env.sh和hive-config.sh
``` shell 
mv hive-env.sh.template hive-env.sh

vi /usr/local/hive/bin/hive-config.sh
export JAVA_HOME=/usr/java/latest
export HIVE_HOME=/usr/local/hive
export HADOOP_HOME=/usr/local/hadoop
```
##  验证hive是否安装成功
直接输入hive命令，可以进入hive命令行