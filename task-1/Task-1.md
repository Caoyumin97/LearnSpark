# 1 Spark安装

## 1.1 Java JDK部署

* 64位：https://pan.baidu.com/s/1EU1SM4h02Uj0fI-myzys9Q
* 官网下载：https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html

* 环境变量配置（JAVA_HOME, CLASS_PATH）
  * JAVA_HOME：安装地址
  * CLASS_PATH：;%JAVA_HOME%\lib; %JAVA_HOME%\lib\tools.jar

* 测试：cmd --> java -version



## 1.2 Spark部署

* 官网下载：http://spark.apache.org/
* 解压后配置环境变量：
  * SPARK_HOME：D:\spark\spark-2.4.1-bin-hadoop2.7
  * PATH：%SPARK_HOME%\bin
* 测试：cmd --> spark-shell



## 1.3 Hadoop部署

* 官网下载：http://mirror.bit.edu.cn/apache/hadoop/common/hadoop-3.2.0/
* Hadoop-3.2：https://pan.baidu.com/s/1WzPPZLDbXHUNTOqHPYxDrA，密码：9bv2
* 解压后配置环境变量：
  - HADOOP_HOME：D:\hadoop\hadoop-3.2.0
  - PATH：%HADOOP_HOME%\bin
* 测试：cmd --> spark-shell



## 1.4 python环境部署

* pyspark：cmd --> pip install pyspark
* findspark: cmd --> pip install findspark
* jupyter使用：

```python
import findspark
#可在环境变量中进行设置，即PATH中加入如下地址
findspark.init("D:\spark\spark-2.4.1-bin-hadoop2.7")
from pyspark.sql import SparkSession
from pyspark import SparkContext
from pyspark import SparkConf

# 创建sc
sc = SparkContext("local","Simple")
```





# 2 Spark的设计与运行原理

## 2.1 Spark简介

### 2.1.1 基本情况

* UCB的AMP实验室于2009年开发
* 是一种**<u>基于内存计算</u>**的**<u>数据并行计算框架</u>**
* 在Sort Benchmark记录中，Spark仅使用了十分之一的计算资源，获得了比Hadoop快3倍的速度
* 腾讯、淘宝、百度、亚马逊等公司均不同程度地使用了Spark来构建大数据分析应用，并应用到实际的生产环境中



### 2.1.2 Spark运行特点

* 运行速度快：使用**<u>DAG（Directed Acyclic Graph，有向无环图）</u>**执行引擎，以支持循环数据流与内存计算；
* 容易使用：Spark支持使用Scala、Java、Python和R语言进行编程，API简洁，可通过Spark Shell进行交互式编程；
* 通用性：完整技术栈，主要包括SQL查询、流式计算、机器学习和图算法组件；
* 运行模式多样：Spark可运行于独立的集群模式/Hadoop中，也可运行于Amazon EC2等云环境中，并且可以访问HDFS、Cassandra、HBase、Hive等多种数据源‘



### 2.1.3 Spark vs. Hadoop

* Hadoop存在的问题：
  * 表达能力有限：计算都必须要转化成Map和Reduce两个操作，但这并不适合所有的情况，难以描述复杂的数据处理过程；
  * 磁盘IO开销大：每次执行时都需要从磁盘读取数据，并且在计算完成后需要将中间结果写入到磁盘中，IO开销较大；
  * 延迟高：一次计算可能需要分解成一系列按顺序执行的MapReduce任务，任务之间的衔接由于涉及到IO开销，会产生较高延迟。而且，在前一个任务执行完成之前，其他任务无法开始，难以胜任复杂、多阶段的计算任务；
* Spark的优势：
  * 计算模式同属于MapReduce，但不局限于Map和Reduce操作，还提供了多种数据集操作类型，编程模型比MapReduce更灵活；
  * 中间结果直接放到内存中，带来了更高的迭代运算效率（对硬件要求稍高）；
  * **<u>基于DAG的任务调度执行机制，要优于MapReduce的迭代执行机制</u>**；
  * 对于实现相同功能的应用程序，Spark的代码量要比Hadoop少2-5倍，此外，Spark提供了实时交互式编程反馈，可以方便地验证、调整算法。



### 2.1.4 Spark生态系统

* 大数据处理主要类型（3种）：
  * 复杂的批量数据处理：时间跨度通常在数十分钟到数小时之间；
  * 基于历史数据的交互式查询：时间跨度通常在数十秒到数分钟之间；
  * 基于实时数据流的数据处理：时间跨度通常在数百毫秒到数秒之间。
* 伯克利数据分析软件栈BDAS（Berkeley Data Analytics Stack）架构

![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2016/10/%E5%9B%BE-BDAS%E6%9E%B6%E6%9E%84.jpg)

	> Spark专注于数据的处理分析，而数据的存储还是要借助于Hadoop分布式文件系统HDFS、Amazon S3等来实现

* Spark生态系统：

  * **<u>Spark Core</u>**：Spark Core包含Spark的基本功能，如内存计算、任务调度、部署模式、故障恢复、存储管理等。Spark建立在统一的抽象RDD之上，使其可以以基本一致的方式应对不同的大数据处理场景；通常所说的Apache Spark，就是指Spark Core；

    > RDD: Resilient Distributed Dataset，弹性分布式数据集，是分布式内存的一个抽象概念，RDD提供了一种高度受限的共享内存模型，即RDD是只读的记录分区的集合，只能通过在其他RDD执行确定的转换操作（如map、join和group by）而创建，然而这些限制使得实现容错的开销很低。对开发者而言，RDD可以看作是Spark的一个对象，它本身运行于内存中，如读文件是一个RDD，对文件计算是一个RDD，结果集也是一个RDD ，不同的分片、 数据之间的依赖 、key-value类型的map数据都可以看做RDD。
    >
    > ——Spark Programming Guide ．阿帕奇官网

  * **<u>Spark SQL</u>**：Spark SQL允许开发人员直接处理RDD，同时也可查询Hive、HBase等外部数据源。Spark SQL的一个重要特点是其能够统一处理关系表和RDD，使得开发人员可以轻松地使用SQL命令进行查询，并进行更复杂的数据分析；

    > Hive：基于Hadoop的一个数据仓库工具，可以将结构化的数据文件映射为一张数据库表，并提供简单的sql查询功能，可以将sql语句转换为MapReduce任务进行运行；
    >
    > HBase：一个分布式的、面向列的开源数据库，其不同于一般的关系数据库，它是一个适合于非结构化数据存储的数据库，另一个不同的是HBase基于列的而不是基于行的模式。

  * **<u>Spark Streaming</u>**：Spark Streaming支持高吞吐量、可容错处理的实时流数据处理，其核心思路是将流式计算分解成一系列短小的==批处理作业==。Spark Streaming支持多种数据输入源，如Kafka、Flume和TCP套接字等；

  * **<u>MLlib（机器学习）</u>**：MLlib提供了常用机器学习算法的实现，包括聚类、分类、回归、协同过滤等，降低了机器学习的门槛，开发人员只要具备一定的理论知识就能进行机器学习的工作；

  * **<u>GraphX（图计算）</u>**：GraphX是Spark中用于图计算的API，其性能良好，拥有丰富的功能和运算符，能在海量数据上自如地运行复杂的图算法。



## 2.2 Spark运行架构

### 2.2.1 基本概念

* **<u>RDD</u>**：是弹性分布式数据集（Resilient Distributed Dataset）的简称，是分布式内存的一个抽象概念，提供了一种高度受限的共享内存模型；
* **<u>DAG</u>**：是Directed Acyclic Graph（有向无环图）的简称，反映RDD之间的依赖关系；
* **<u>Executor</u>**：是运行在工作节点（Worker Node）上的一个进程，负责运行任务，并为应用程序存储数据；
* **<u>应用</u>**：用户编写的Spark应用程序；
* **<u>任务</u>**：运行在Executor上的工作单元；
* **<u>作业</u>**：一个作业包含多个RDD及作用于相应RDD上的各种操作；
* **<u>阶段</u>**：是作业的基本调度单位，一个作业会分为多组任务，每组任务被称为“阶段”，或者也被称为“任务集”。



### 2.2.2 架构设计

* 运行架构组成部分：
  * 集群资源管理器（Cluster Manager）
  * 运行作业任务的工作节点（Worker Node）
  * 每个应用的任务控制节点（Driver）
  * 每个工作节点上负责具体任务的执行进程（Executor）



![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2016/11/%E5%9B%BE9-5-Spark%E8%BF%90%E8%A1%8C%E6%9E%B6%E6%9E%84.jpg)

> HDFS: Hadoop Distributed File System，其设计成适合运行在通用硬件(commodity hardware)上的分布式文件系统。它和现有的分布式文件系统有很多共同点。但同时，它和其他的分布式文件系统的区别也是很明显的。HDFS是一个高度容错性的系统，适合部署在廉价的机器上。HDFS能提供高吞吐量的数据访问，非常适合大规模数据集上的应用。

* 应用组成：

  一个应用（Application）由一个任务控制节点（Driver）和若干个作业（Job）构成，一个作业由多个阶段（Stage）构成，一个阶段由多个任务（Task）组成。当执行一个应用时，任务控制节点会向集群管理器（Cluster Manager）申请资源，启动Executor，并向Executor发送应用程序代码和文件，然后在Executor上执行任务，运行结束后，执行结果会返回给任务控制节点，或者写到HDFS或者其他数据库中

![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2016/11/%E5%9B%BE9-6-Spark%E4%B8%AD%E5%90%84%E7%A7%8D%E6%A6%82%E5%BF%B5%E4%B9%8B%E9%97%B4%E7%9A%84%E7%9B%B8%E4%BA%92%E5%85%B3%E7%B3%BB.jpg)



### 2.2.3 Spark运行基本流程

![img](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2016/11/%E5%9B%BE9-7-Spark%E8%BF%90%E8%A1%8C%E5%9F%BA%E6%9C%AC%E6%B5%81%E7%A8%8B%E5%9B%BE.jpg)

1. 当一个Spark应用被提交时，首先需要为这个应用构建起基本的运行环境，即由任务控制节点（Driver）创建一个SparkContext，由SparkContext负责和资源管理器（Cluster Manager）的通信以及进行资源的申请、任务的分配和监控等。SparkContext会向资源管理器注册并申请运行Executor的资源；
2. 资源管理器为Executor分配资源，并启动Executor进程，Executor运行情况将随着“心跳”发送到资源管理器上；
3. SparkContext根据RDD的依赖关系构建DAG图，DAG图提交给DAG调度器（DAGScheduler）进行解析，将DAG图分解成多个“阶段”（每个阶段都是一个任务集），并且计算出各个阶段之间的依赖关系，然后把一个个“任务集”提交给底层的任务调度器（TaskScheduler）进行处理；Executor向SparkContext申请任务，任务调度器将任务分发给Executor运行，同时，SparkContext将应用程序代码发放给Executor；
4. 任务在Executor上运行，把执行结果反馈给任务调度器，然后反馈给DAG调度器，运行完毕后写入数据并释放所有资源。



# 3 RDD的设计与运行原理

> Resilient Distributed Datasets: A Fault-Tolerant Abstraction for In-Memory Cluster Computing

## 3.1 RDD设计背景

* MapReduce的缺陷

  实际应用中，存在许多迭代式算法（比如机器学习、图算法等）和交互式数据挖掘工具，这些应用场景的共同之处是，不同计算阶段之间会重用中间结果，即一个阶段的输出结果会作为下一个阶段的输入。而目前的MapReduce框架都是把中间结果写入到HDFS中，带来了大量的数据复制、磁盘IO和序列化开销。

* RDD需求

  **<u>提供一个抽象的数据架构</u>**，我们不必担心底层数据的分布式特性，只需将具体的应用逻辑表达为一系列转换处理，不同RDD之间的转换操作形成依赖关系，可以实现**<u>管道化</u>**，从而避免了中间结果的存储，大大降低了数据复制、磁盘IO和序列化开销。



## 3.2 RDD概念

### 3.2.1 本质与基本属性

* 一个RDD就是一个分布式对象集合，本质上是一个只读的分区记录集合，每个RDD可以分成多个分区，每个分区就是一个数据集片段，并且一个RDD的不同分区可以被保存到集群中不同的节点上，从而可以在集群中的不同节点上进行并行计算；
* RDD提供了一种高度受限的共享内存模型，即RDD是只读的记录分区的集合，不能直接修改，只能基于稳定的物理存储中的数据集来创建RDD，或者通过在其他RDD上执行确定的转换操作（如map、join和groupBy）而创建得到新的RDD；
* RDD提供了一组丰富的操作以支持常见的数据运算，分为“行动”（Action）和“转换”（Transformation）两种类型，前者用于执行计算并指定输出的形式，后者指定RDD之间的相互依赖关系。两类操作的主要区别是，转换操作（比如map、filter、groupBy、join等）接受RDD并返回RDD，而行动操作（比如count、collect等）接受RDD但是返回非RDD（即输出一个值或结果）；
* Spark用Scala语言实现了RDD的API，程序员可以通过调用API实现对RDD的各种操作。

### 3.2.2 执行过程

1. RDD读入外部数据源（或者内存中的集合）进行创建；
2. RDD经过一系列的“转换”操作，每一次都会产生不同的RDD，供给下一个“转换”使用；
3. 最后一个RDD经“行动”操作进行处理，并输出到外部数据源（或者变成Scala集合或标量）。

![å¾9-8 Sparkçè½¬æ¢åè¡å¨æä½](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2016/11/%E5%9B%BE9-8-Spark%E7%9A%84%E8%BD%AC%E6%8D%A2%E5%92%8C%E8%A1%8C%E5%8A%A8%E6%93%8D%E4%BD%9C.jpg)

示例：

![å¾9-9 RDDæ§è¡è¿ç¨çä¸ä¸ªå®ä¾](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2016/11/%E5%9B%BE9-9-RDD%E6%89%A7%E8%A1%8C%E8%BF%87%E7%A8%8B%E7%9A%84%E4%B8%80%E4%B8%AA%E5%AE%9E%E4%BE%8B.jpg)

<u>Spark只是记录了RDD之间的生成和依赖关系。当F要进行输出时，也就是当F进行“行动”操作的时候，Spark才会根据RDD的依赖关系生成DAG，并从起点开始真正的计算</u>

// 类似于tensorflow中的computation graph构图和session执行？

* 上述这一系列处理称为一个“血缘关系（Lineage）”，即DAG拓扑排序的结果。采用惰性调用，通过血缘关系连接起来的一系列RDD操作就可以实现管道化（pipeline）

* 在MapReduce的设计中，为了尽可能地减少MapReduce过程，在单个MapReduce中会写入过多复杂的逻辑



## 3.3 RDD特性

总体而言，Spark采用RDD以后能够实现高效计算的主要原因如下：

1. 高效的容错性。现有的分布式共享内存、键值存储、内存数据库等，为了实现容错，必须在集群节点之间进行数据复制或者记录日志，也就是在节点之间会发生大量的数据传输，这对于数据密集型应用而言会带来很大的开销。在RDD的设计中，数据只读，不可修改，如果需要修改数据，必须从父RDD转换到子RDD，由此在不同RDD之间建立了血缘关系。所以，RDD是一种天生具有容错机制的特殊集合，不需要通过数据冗余的方式（比如检查点）实现容错，而只需通过RDD父子依赖（血缘）关系重新计算得到丢失的分区来实现容错，无需回滚整个系统，这样就避免了数据复制的高开销，而且重算过程可以在不同节点之间并行进行，实现了高效的容错。此外，RDD提供的转换操作都是一些粗粒度的操作（比如map、filter和join），RDD依赖关系只需要记录这种粗粒度的转换操作，而不需要记录具体的数据和各种细粒度操作的日志（比如对哪个数据项进行了修改），这就大大降低了数据密集型应用中的容错开销；
2. 中间结果持久化到内存。数据在内存中的多个RDD操作之间进行传递，不需要“落地”到磁盘上，避免了不必要的读写磁盘开销；
3. 存放的数据可以是Java对象，避免了不必要的对象序列化和反序列化开销。



## 3.4 RDD间的依赖关系

* RDD中的依赖关系分为窄依赖（Narrow Dependency）与宽依赖（Wide Dependency）

  ![å¾9-10 çªä¾èµä¸å®½ä¾èµçåºå«](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2016/11/%E5%9B%BE9-10-%E7%AA%84%E4%BE%9D%E8%B5%96%E4%B8%8E%E5%AE%BD%E4%BE%9D%E8%B5%96%E7%9A%84%E5%8C%BA%E5%88%AB.jpg)

  * 窄依赖表现为一个父RDD的分区对应于一个子RDD的分区，或多个父RDD的分区对应于一个子RDD的分区；
  * 宽依赖则表现为存在一个父RDD的一个分区对应一个子RDD的多个分区；
  * 总体而言，如果父RDD的一个分区只被一个子RDD的一个分区所使用就是窄依赖，否则就是宽依赖。窄依赖典型的操作包括map、filter、union等，宽依赖典型的操作包括groupByKey、sortByKey等



## 3.5 阶段划分

Spark通过分析各个RDD的依赖关系生成了DAG，再通过分析各个RDD中的分区之间的依赖关系来决定如何划分阶段：

* 在DAG中进行反向解析，遇到宽依赖就断开，遇到窄依赖就把当前的RDD加入到当前的阶段中；
* 将窄依赖尽量划分在同一个阶段中，可以实现流水线计算

![å¾9-11æ ¹æ®RDDååºçä¾èµå³ç³»ååé¶æ®µ](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2016/11/%E5%9B%BE9-11-%E6%A0%B9%E6%8D%AERDD%E5%88%86%E5%8C%BA%E7%9A%84%E4%BE%9D%E8%B5%96%E5%85%B3%E7%B3%BB%E5%88%92%E5%88%86%E9%98%B6%E6%AE%B5.jpg)

* 把一个DAG图划分成多个“阶段”以后，每个阶段都代表了一组关联的、相互之间没有Shuffle依赖关系的任务组成的任务集合。每个任务集合会被提交给任务调度器（TaskScheduler）进行处理，由任务调度器将任务分发给Executor运行



## 3.6 RDD运行过程

1. 创建RDD对象；
2. SparkContext负责计算RDD之间的依赖关系，构建DAG；
3. DAGScheduler负责把DAG图分解成多个阶段，每个阶段中包含了多个任务，每个任务会被任务调度器分发给各个工作节点（Worker Node）上的Executor去执行。

![å¾9-12 RDDå¨Sparkä¸­çè¿è¡è¿ç¨](http://dblab.xmu.edu.cn/blog/wp-content/uploads/2016/11/%E5%9B%BE9-12-RDD%E5%9C%A8Spark%E4%B8%AD%E7%9A%84%E8%BF%90%E8%A1%8C%E8%BF%87%E7%A8%8B.jpg)



# 4 Spark的三种部署模式

## 4.1 Standalone模式

独立部署到一个集群中，而不需要依赖其他系统来为其提供资源管理调度服务。在架构的设计上，Spark与MapReduce1.0完全一致，都是由一个Master和若干个Slave构成，并且以槽（slot）作为资源分配单位。不同的是，Spark中的槽不再像MapReduce1.0那样分为Map 槽和Reduce槽，而是只设计了统一的一种槽提供给各种任务来使用



## 4.2 Spark on Mesos模式

Mesos是一种资源调度管理框架，可以为运行在它上面的Spark提供服务。Spark on Mesos模式中，Spark程序所需要的各种资源，都由Mesos负责调度。由于Mesos和Spark存在一定的血缘关系，因此，Spark这个框架在进行设计开发的时候，就充分考虑到了对Mesos的充分支持，因此，相对而言，Spark运行在Mesos上，要比运行在YARN上更加灵活、自然。目前，Spark官方推荐采用这种模式，所以，许多公司在实际应用中也采用该模式。



## 4.3 Spark on YARN模式

Spark可运行于YARN之上，与Hadoop进行统一部署，即“Spark on YARN”，资源管理和调度依赖YARN，分布式存储则依赖HDFS。



# 5 Word Count

```python
from pyspark import SparkContext
sc = SparkContext( 'local', 'test')
logFile = "file:///usr/local/spark/README.md"
logData = sc.textFile(logFile, 2).cache()
numAs = logData.filter(lambda line: 'a' in line).count()
numBs = logData.filter(lambda line: 'b' in line).count()
print('Lines with a: %s, Lines with b: %s' % (numAs, numBs))
```

