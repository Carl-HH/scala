# Hadoop 2.4.1集群搭建
## 安装hadoop包
1. 使用课程提供的hadoop-2.4.1.tar.gz，使用SCP上命令上传到CentOS的/usr/local目录下。
2. 将hadoop包进行解压缩：tar -zxvf hadoop-2.4.1.tar.gz
3. 对hadoop目录进行重命名：mv hadoop-2.4.1 hadoop
4. 配置hadoop相关环境变量
```shell
vi .bashrc
export HADOOP_HOME=/usr/local/hadoop
export PATH=$HADOOP_HOME/bin:$HADOOP_HOME/sbin
source .bashrc
```
## 修改core-site.xml
```xml
<property>
  <name>fs.default.name</name>
  <value>hdfs://spark1:9000</value>
</property>
```
## 修改hdfs-site.xml
``` xml 
<property>
  <name>dfs.name.dir</name>
  <value>/usr/local/data/namenode</value>
</property>
<property>
  <name>dfs.data.dir</name>
  <value>/usr/local/data/datanode</value>
</property>
<property>
  <name>dfs.tmp.dir</name>
  <value>/usr/local/data/tmp</value>
</property>
<property>
  <name>dfs.replication</name>
  <value>3</value>
</property>
```
## 修改mapred-site.xml
``` xml
<property>
  <name>mapreduce.framework.name</name>
  <value>yarn</value>
</property>
```
## 修改yarn-site.xml
``` xml 
<property>
  <name>yarn.resourcemanager.hostname</name>
  <value>spark1</value>
</property>
<property>
  <name>yarn.nodemanager.aux-services</name>
  <value>mapreduce_shuffle</value>
</property>
```
## 修改slaves文件
spark1<br>
spark2<br>
spark3<br>
## 在另外两台机器上搭建hadoop
1. 使用如上配置在另外两台机器上搭建hadoop，可以使用scp命令将spark1上面的hadoop安装包和.bashrc配置文件都拷贝过去。
2. 要记得对.bashrc文件进行source，以让它生效。
3. 记得在spark2和spark3的/usr/local目录下创建data目录。
## 启动hdfs集群
1. 格式化namenode：在spark1上执行以下命令，
`hdfs namenode -format`
2. 启动hdfs集群：`start-dfs.sh'
3. 验证启动是否成功：jps、50070端口 <br>
spark1：namenode、datanode、secondarynamenode<br>
spark2：datanode<br>
spark3：datanode<br>
## 启动yarn集群
1. 启动yarn集群：`start-yarn.sh`
2. 验证启动是否成功：jps、8088端口<br>
spark1：resourcemanager、nodemanager<br>
spark2：nodemanager<br>
spark3：nodemanager


