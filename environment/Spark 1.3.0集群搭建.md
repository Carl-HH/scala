# Spark 1.3.0集群搭建
## 安装spark包
1. 将spark-1.3.0-bin-hadoop2.4.tgz使用WinSCP上传到/usr/local目录下。
2. 解压缩spark包：tar zxvf spark-1.3.0-bin-hadoop2.4.tgz。
3. 更改spark目录名：mv spark-1.3.0-bin-hadoop2.4 spark
4. 设置spark环境变量
``` shell
vi .bashrc
export SPARK_HOME=/usr/local/spark
export PATH=$SPARK_HOME/bin
export CLASSPATH=.:$CLASSPATH:$JAVA_HOME/lib:$JAVA_HOME/jre/lib
source .bashrc
```
## 修改spark-env.sh文件
1. `cd /usr/local/spark/conf`
2. `cp spark-env.sh.template spark-env.sh`
3. `vi spark-env.sh`
``` shell
export JAVA_HOME=/usr/java/latest
export SCALA_HOME=/usr/local/scala
export SPARK_MASTER_IP=192.168.1.107
export SPARK_WORKER_MEMORY=1g
export HADOOP_CONF_DIR=/usr/local/hadoop/etc/hadoop
```

## 修改slaves文件
spark1
spark2
spark3

## 安装spark集群
在另外两个节点进行一模一样的配置，使用scp将spark和.bashrc拷贝到spark2和spark3即可

##  启动spark集群
1. 在spark目录下的sbin目录
2. 执行./start-all.sh
3. 使用jsp和8080端口可以检查集群是否启动成功
4. 进入spark-shell查看是否正常
