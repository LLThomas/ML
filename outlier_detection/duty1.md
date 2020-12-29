# Application Areas of Outlier Detection

### 1. Financial fraud detection(金融欺诈检测，发现信用卡异常交易)
Fraud detection is very normal that occurs with credit cards. Credit card fraud shows unauthorized use of credit card, which shows significant different patterns of transaction. Such patterns are useful to detect outliers in transactions of the credit card which is called as fraud detection.  
信用卡公司维护不同用户的交易记录，每笔交易都有用户标识、交易金额、交易位置
### 2. Medical Applications（监控病人身体状况）
A patient who have some dieses, there are different test like blood test, MRI scan, PET scan are conducted. From that test data are collected and which shows some unusual data which is detected by outlier detection methods for medical diagnosis and to identify types of dieses and other related information.  
通过传感器来获取病人的身体健康数据，用来判断病人的身体健康状况
### 3. Web Log Applications(网络日志检测)
通过分析该网站访问的日志，对某ip下的访问次数进行监控，如果发现某时段突然有很多次访问，就出现了异常
### 4. Sensor networks（传感器网络，发现设备故障和动物车辆位置突然变化）
Use of sensor network is increased in various activities of regular life. Sensor network is used for record and monitor various locations and also monitors environment. Sometimes it is happen that equipment becomes faulty which cause wrong data is generated. Outlier detection is used to detect such faulty data or erroneous data. Sensor network is also used to track location of vehicles or animals. Sudden pattern change in the location is detected by outlier detection for track location of animal or vehicle.
### 6. Quality Control（质量控制）
通过单个物体的特征或者制造过程的总体特征来把控制造业生产物品的质量
### 7.  Earth Science Applications(地球科学方面的异常检测)
[应用到了flink,点击跳转](#jump1)  
通过追踪海平面温度的数据，来发现温度的异常值，可以与对应天气建立关系
# what can be solved by flink
<span id="jump1"></span>
### 1. Hydrologic Time Series Anomaly Detection （水文时间序列异常检测 ）
    1. 解决的问题：判断水文时间序列与一般规律相差较大的异常现象，如日降雨量、日径流量（日降雨量是一个随时间变化的时间序列数据）
    2. 基于flink平台做数据处理
### 2. Mux(一个提供视频api和数据分析的公司)
    1. 用flink来实时检测用户在视频上传或者播放过程中出现的错误并报警
    2. 该功能的需求：
        1. Ability to support an unbounded number of customer properties and video titles (i.e. must scale easily without sacrificing performance)
        2. No reliance on database polling（不依赖数据库）
        3. Ability to support alerting for infrequently-played video-titles (could take days to accumulate enough views to make an accurate determination)
        4. Low latency
        5. Fault tolerant, because things happen
# what cannot
### 1. 
<div>
    <a href="https://www.hindawi.com/journals/mpe/2020/3187697/">点击链接查看论文（Hydrologic Time Series Anomaly Detection）</a>
</div>
    

outlier detection stream data  异常值检测流数据  
outlier detection time series data  异常值检测时间序列数据

# Flink笔记
### 是什么
    flink是一款高吞吐量、低延迟的针对流数据和批数据的分布式实时处理引擎
### flink处理流程
    Job Client接受用户程序代码 -> 然后创建数据流 -> 将数据流交给Job Manager -> 在执行完成后Job Client将结果返回给用户
### 小知识1
    1. Table API：（降低学习成本，方便非开发人员获取数据，而不用考虑如何获取数据）
       1. flink同时支持批任务与流任务，如何让API层统一
       2. 需要熟悉两套API：DataStream/DataSetAPI，API有一定难度，开发人员无法focus on business
       3. 我们还需要提供一种关系型的API来实现Flink API层的流与批的统一
       4. 首先Table & SQL API是一种关系型API，用户可以像操作mysql数据库表一样的操作数据，而不需要写java代码完成Flink Function，更不需要手工的优化java代码调优（想查就查而不用考虑如何优化查询来提高查询速度，学习成本低）
       5. SQL作为一个非程序员可操作的语言，学习成本很低，如果一个系统提供SQL支持，将很容易被用户接受。
    2. Data Source：
       1. 套接字：套接字上联应用进程，下联网络协议栈，是应用程序通过网络协议进行通信的接口，是应用程序与网络协议根进行交互的接口
       2. 添加数据源的方法
          1. 从集合中（有界数据集，更偏向于本地测试用）
          2. 从文件中（适合监听文件修改并读取其内容）
          3. 从socket中（监听主机的 host port，从 Socket 中获取数据）
          4. 自定义（例如可以从kafka中获取，大多数场景需要）

<div>
    <img src="https://zhisheng-blog.oss-cn-hangzhou.aliyuncs.com/images/p92UrK.jpg">
</div>

### 小知识2
    1. Job Manager（作业管理器）：
       1） JobManager是master主节点，它主要负责资源的分配、任务的调度以及集群的管理。TaskManager是从节点，如果单独运行在一台机器上则可以称作一个slave节点。主从节点之间通过心跳机制保持联系，当我们从client提交一个job的时候，会将job提交到jobmanager主节点，然后由主节点分配到各个slave节点。
       2） 各个节点之间消息的通信使用akka框架来完成，数据的传输基于Netty框架来完成
    2. Task Manager（任务管理器）：
        1. Flink的TM就是运行在不同节点上的JVM进程（process）,这个进程会拥有一定量的资源。比如内存，cpu，网络，磁盘等
        2. Task Manager 的一个 Slot（） 代表一个可用线程，该线程具有固定的内存(平均分TM的内存)，注意 Slot 只对内存隔离，没有对 CPU 隔离，slot之间可以共享JVM资源, 可以共享Dataset和数据结构，也可以通过多路复用共享TCP连接和心跳消息
    3. Actor模型：
        1. Actor模型用于处理并发计算，它定义了一系列 系统组件 应该如何动作和交互的 通用规则
        2. 一个Actor指的是一个 最基本的计算单元 ，它能接收一个消息并且基于其执行计算。
        3. Actors一大重要特征在于 actors之间相互隔离 ，它们 并不互相共享内存。也就是说，一个actor能维持一个 私有的状态 ，并且这个状态 不可能被另一个actor所改变。
        4. 一个actor接收到消息后，它能做如下三件事中的一件：
            1）Create more actors; 创建其他actors
            2）Send messages to other actors; 向其他actors发送消息
            3）Designates what to do with the next message. 指定下一条消息到来的行为
    4. Flink Deployment Modes（Flink部署方式）:
       1. Session Mode：会话模式假设一个已经运行的集群，并使用该集群的资源来执行任何提交的应用程序。
          1. 优势：这样做的好处是，您不必为每个提交的作业支付启动整个集群所需的资源开销
          2. 缺点：如果其中一个作业行为不当或使一个任务管理器崩溃，那么在该任务管理器上运行的所有作业都会受到故障的影响
       2. Per-Job Mode（生产模式首选）：为了提供更好的资源隔离保障，Per-Job模式使用可用的集群管理器框架（如YARN、Kubernetes）为每个提交的作业旋转一个集群，这个集群只对该作业可用
          1. 优势：当作业完成后，集群会被拆掉，任何滞留的资源（文件等）都会被清理掉。这提供了更好的资源隔离，因为一个行为不端的作业只能使自己的任务管理器瘫痪
       3. Application Mode：在上述所有模式中，应用程序的main()方法都是在客户端执行的。这个过程包括在本地下载应用程序的依赖关系，执行main()以提取Flink运行时能够理解的应用程序的表示（即JobGraph），并将依赖关系和JobGraph（s）运送到集群。这使得客户端成为一个沉重的资源消耗者，因为它可能需要大量的网络带宽来下载依赖关系和将二进制文件传送到集群，以及CPU周期来执行main()。当客户机在不同用户之间共享时，这个问题会更加明显。         在此观察的基础上，应用模式为每个提交的应用创建一个集群，但这次，应用的main()方法是在JobManager上执行的。为每个应用创建集群可以看做是创建一个只在特定应用的作业之间共享的会话集群，并在应用结束后被拆掉。通过这种架构，应用模式提供了与Per-Job模式相同的资源隔离和负载均衡保证，但其粒度是整个应用。在JobManager上执行main()可以节省所需的CPU周期，也可以节省下载本地依赖关系所需的带宽。此外，由于每个应用程序只有一个JobManager，因此它可以更均匀地分散集群中下载应用程序依赖关系的网络负载。
    5. Flink Deployment Target：
       1. Local：本地运行模式，用单机的多个线程（单个进程，区分local-cluster模式）来模拟Spark的分布式计算，通常用于验证程序的逻辑是否有问题
       2. Standalone：
           1. pros： 
                1. no dependency on external components;
                2. easy to add/remove TaskManager in the cluster;
                3. easy for debug, and log retrieve;
           2.  cons：
                1.  No job isolation as slots share the same JVM, refer to Job Isolation on Flink;
                2.  Need to have a zookeeper for node failure recovery;
                3.  jobmanager 发布job不均（如果当前集群总共有2台8核的机器用以部署TaskManager，每台机器上一个TaskManager实例，每个TaskManager的TaskSlot为8，而我们的job的并行度为12，就会出现某台机器slot被占满而另一台机器却使用了很少slot）
       3. Yarn：
           1. pros：
                1. job isolation provided by YARN;
                2. node failure auto-recovery;
                3. flexible resource capacity per TaskManager for different jobs;
           2. cons：
                1. external cost for YARN;
                2. So far YARN is tied closed with a distribution file system, HDFS/AWS/GoogleCloud;
### 相关知识
    1. Kafka是一种高吞吐量的分布式发布订阅消息系统，它可以处理消费者在网站中的所有动作流数据。Kafka的目的是通过Hadoop的并行加载机制来统一线上和离线的消息处理，也是为了通过集群来提供实时的消息。
    2. 批处理：
       1. 输入：是一段时间内已经存收集保存好的数据
       2. 输出：可以作为下一个批处理的输入
    3. yarn：Apache Hadoop YARN 是一种新的 Hadoop 资源管理器，它是一个通用资源管理系统，可为上层应用提供统一的资源管理和调度，它的引入为集群在利用率、资源统一管理和数据共享等方面带来了巨大好处
    4. case object 所定义的对象能用于序列化，而普通的object不能，这在使用Actor编写网络程序时尤其要注意(下方链接有序列化的作用)
    5. Scala 方法声明格式如下：def functionName ([参数列表]) : [return type] = { function body   return [expr]}
    6. flatmap = flatten + map（flatmap过程就像是先 map, 然后再将 map 出来的这些列表首尾相接 (flatten).）
    7. map: map对列表中的每个元素应用一个函数，返回应用后的元素所组成的列表。(var list=List(1,2,3,4)    list.map((x:Int)=>x*2)   list(2,4,6,8))
    8. flatten：flatten将嵌套结构扁平化为一个层次的集合。(压平)
    9. QuorumPeerMain是zookeeper集群的启动入口类，是用来加载配置启动QuorumPeer线程
    10. Rest接口：就是用URL定位资源，用HTTP动词（GET,POST,DELETE,PUT）描述操作。
        比如，我们有一个friends接口，对于“朋友”我们有增删改查四种操作，怎么定义REST接口？

        增加一个朋友，uri: generalcode.cn/v1/friends 接口类型：POST
        删除一个朋友，uri: generalcode.cn/va/friends 接口类型：DELETE
        修改一个朋友，uri: generalcode.cn/va/friends 接口类型：PUT
        查找朋友，uri: generalcode.cn/va/friends 接口类型：GET

        上面我们定义的四个接口就是符合REST协议的，请注意，这几个接口都没有动词，只有名词friends，都是通过Http请求的接口类型来判断是什么业务操作。

        举个反例：generalcode.cn/va/deleteFriends 该接口用来表示删除朋友，这就是不符合REST协议的接口。
        
        那这种风格的接口有什么好处呢？前后端分离。前端拿到数据只负责展示和渲染，不对数据做任何处理。后端处理数据并以JSON格式传输出去，定义这样一套统一的接口，在web，ios，android三端都可以用相同的接口
    11. Hadoop: 大家说的Hadoop，指的是提供海量数据存储能力的HDFS，管理大量机器运算资源的Yarn以及一个编程框架MapReduce(Hadoop = HDFS+Yarn+MapReduce)
    12. ZooKeeper: 统一配置管理、统一命名服务、分布式锁、集群管理
    
[java继承（implements与extends）总结](https://blog.csdn.net/weixin_39938767/article/details/80056922?utm_medium=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.compare&depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromMachineLearnPai2-2.compare)  
[序列化的作用](https://blog.csdn.net/ccecwg/article/details/17141337)

### 编译源码错误记录
    1. import org.apache.flink.queryablestate.client.state.serialization.KvStateSerializer包不存在的错误
        解决方法：<dependency>
                        <groupId>org.apache.flink</groupId>
                        <artifactId>flink-queryable-state-client-java_2.11</artifactId>
                        <version>1.4.1</version>
		            </dependency>
    1. object streaming is not a member of package org.apache.flink.test
        import org.apache.flink.test.streaming.runtime.util.TestListResultSink
        解决方法：




# 看源码的步骤
    1. 先看低版本的源码
    2. 学会使用
    3. 看官方文档和架构
    4. 看最新版源码
    5. 用编辑器调试（先找到入口）