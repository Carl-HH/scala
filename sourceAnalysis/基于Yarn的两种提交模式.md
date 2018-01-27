# 基于Yarn的两种提交模式剖析
## spark的三种提交模式
1. spark内核架构，standlone模式，基于Spark自己的Master-Worker集群。
2. 基于Yarn的yarn-cluster模式
3. 基于Yarn的yarn-client模式
4. 如果你要切换到第二种和第三种模式，很简单，将我们之前用于提交spark应用程序的spark-submit脚本，加上--master参数，设置为yarn-cluster，或yarn-client，即可。如果你没设置，那么，就是standalone模式。
>> yarn-cluster vs yarn-client
1. yarn-client用于测试，因为，driver运行在本地客户端
2. yarn-cluster，用于生产环境
