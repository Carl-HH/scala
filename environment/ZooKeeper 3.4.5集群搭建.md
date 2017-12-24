# ZooKeeper 3.4.5集群搭建
## 安装ZooKeeper包
1. 将zookeeper-3.4.5.tar.gz使用SCP命令拷贝到spark1的/usr/local目录下。
2. 对zookeeper-3.4.5.tar.gz进行解压缩：<br>
`tar -zxvf zookeeper-3.4.5.tar.gz`。
3. 对zookeeper目录进行重命名：<br>
`mv zookeeper-3.4.5 zk`。
4. 配置zookeeper相关的环境变量<br>
```shell
vi .bashrc
export ZOOKEEPER_HOME=/usr/local/zk
export PATH=$ZOOKEEPER_HOME/bin
source .bashrc
```
## 配置zoo.cfg
```
cd zk/conf
mv zoo_sample.cfg zoo.cfg

vi zoo.cfg
修改：dataDir=/usr/local/zk/data
新增：
server.0=spark1:2888:3888	
server.1=spark2:2888:3888
server.2=spark3:2888:3888
```
## 设置zk节点标识
``` shell
cd zk
mkdir data
cd data

vi myid
0
```
## 搭建zk集群
1. 在另外两个节点上按照上述步骤配置ZooKeeper，使用scp将zk和.bashrc拷贝到spark2和spark3上即可。
2. 唯一的区别是spark2和spark3的标识号分别设置为1和2。

##  启动ZooKeeper集群
1. 分别在三台机器上执行：zkServer.sh start。
2. 检查ZooKeeper状态：zkServer.sh status。


