# spark集群启动
## 启动Hadoop集群
1. 格式化namenode：在spark1上执行以下命令，
`hdfs namenode -format`<br>
2. 启动hdfs集群：`start-dfs.sh'
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
##  启动ZooKeeper集群
1. 分别在三台机器上执行：zkServer.sh start。
2. 检查ZooKeeper状态：zkServer.sh status。
## 启动kafka集群
1. 在三台机器上分别进入kafka目录,执行以下命令：`nohup bin/kafka-server-start.sh config/server.properties &`
##  启动spark集群
1. 在spark目录下的sbin目录
2. 执行./start-all.sh
3. 使用jsp和8080端口可以检查集群是否启动成功
4. 进入spark-shell查看是否正常
