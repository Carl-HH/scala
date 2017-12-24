# kafka_2.9.2-0.8.1集群搭建
##  安装scala 2.11.4
1. 将scala-2.11.4.tgz使用SCP命令拷贝到spark1的/usr/local目录下。
2. 对scala-2.11.4.tgz进行解压缩：`tar -zxvf scala-2.11.4.tgz`。
3. 对scala目录进行重命名：`mv scala-2.11.4 scala`
4. 配置scala相关的环境变量
``` shell
vi .bashrc
export SCALA_HOME=/usr/local/scala
export PATH=$SCALA_HOME/bin
source .bashrc
```
5. 查看scala是否安装成功：`scala -version`
6. 按照上述步骤在spark2和spark3机器上都安装好scala。使用scp将scala和.bashrc拷贝到spark2和spark3上即可。

##  安装Kafka包
1. 将kafka_2.9.2-0.8.1.tgz使用SCP命令拷贝到spark1的/usr/local目录下。
2. 对kafka_2.9.2-0.8.1.tgz进行解压缩：`tar -zxvf kafka_2.9.2-0.8.1.tgz`。
3. 对kafka目录进行改名：`mv kafka_2.9.2-0.8.1 kafka`
4. 配置kafka
`vi /usr/local/kafka/config/server.properties`
`broker.id`：依次增长的整数，0、1、2、3、4，集群中Broker的唯一id
zookeeper.connect=192.168.0.191:2181,192.168.0.192:2181,192.168.0.193:2181
5. 安装slf4j
将slf4j-1.7.6.zip使用SCP命令拷贝到/usr/local目录下
`unzip slf4j-1.7.6.zip`<br>
把slf4j中的slf4j-nop-1.7.6.jar复制到kafka的libs目录下面
## 搭建kafka集群
1. 按照上述步骤在spark2和spark3分别安装kafka。用scp把kafka拷贝到spark2和spark3行即可。
2. 唯一区别的,就是`server.properties`中的`broker.id`，要设置为1和2
## 启动kafka集群
1. 在三台机器上分别执行以下命令：`nohup bin/kafka-server-start.sh config/server.properties &`
2. 解决'`kafka Unrecognized VM option 'UseCompressedOops`'问题
``` shell
vi bin/kafka-run-class.sh 
if [ -z "$KAFKA_JVM_PERFORMANCE_OPTS" ]; then
  KAFKA_JVM_PERFORMANCE_OPTS="-server  -XX:+UseCompressedOops -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -XX:+CMSClassUnloadingEnabled -XX:+CMSScavengeBeforeRemark -XX:+DisableExplicitGC -Djava.awt.headless=true"
fi
去掉-XX:+UseCompressedOops即可
```
3. 使用jps检查启动是否成功
## 测试kafka集群
使用基本命令检查kafka是否搭建成功
``` shell
bin/kafka-topics.sh --zookeeper 192.168.0.191:2181,192.168.0.192:2181,192.168.0.193:2181 --topic TestTopic --replication-factor 1 --partitions 1 --create

bin/kafka-console-producer.sh --broker-list 192.168.0.191:9092,192.168.0.192:9092,192.168.0.193:9092 --topic TestTopic

bin/kafka-console-consumer.sh --zookeeper 192.168.0.191:2181,192.168.0.192:2181,192.168.0.193:2181 --topic TestTopic --from-beginning
```
