#Flume介绍

## 1.为何用Flume

### 1.1 问题的产生 

HDFS、MapReduce、HBase 数据都是老师给你的

你在工作中需要代码处理一个业务，老板只会提需求，你要解决 首要是知道数据类型 数据长什么样子

订单数据、用户数据、商品数据都是存储在mysql中，

效率高，是因为select*from goods where name like %s%

某个商品男的看到多,还是女的看的多，数据库里面没有！

也就是说数据库会存储数据，但有些业务也没有数据！

**所以我们要收集数据！**
### 1.2 收集数据
数据来源：

- 文件
- 数据库
- 爬虫-只针对公共数据，就是提供共享大家使用
- 合作、购买（微信 + 京东）流量吸引、数据共享，**导流**

*问题1：微信里面为什么没有淘宝？*

遇到的问题：

一个人看一个商品的次数   
一个人购买一个商品的次数  
哪个多？  
看商品的数据（**冷数据**）数据量大的时候，不会放入数据库中,因为需要计算

> 冷数据：数据一般不会变，历史数据，日志数据

格式种类

- csv comma
- tsv table
- json
- xml
- text
- 行式
- 列式
- 压缩

### 1.3 解决思路
**最终结论：必须要解决数据格式和存储地不统一  
希望：一个组件能解决所有问题**

[http://hadoop.apache.org/](http://hadoop.apache.org/ "Hadoop官网")  
[http://flume.apache.org/](http://flume.apache.org/ "Flume官网")  
主页上没有，是cloudera捐献的

### 1.4 现存问题

大数据处理流程：

- **数据采集**
- 数据ETL
- 数据存储
- 数据计算、分析
- 数据可视化

数据采集难点：  

- 数据源多种多样
- 数据量大，变化快，**流数据**
- 避免重复数据
- 保证数据的质量
- 数据采集的性能

命名：

- flume OG（original generation）版本1.0以前
- flume NG（next generation）1.0之后


##2. Flume介绍

优点：可靠性、横向性

一般性步骤：  

1. Flume数据采集  
2. MapReduce清洗，计算
3. 存入HBase
4. Hive统计、分析
5. 存入Hive表
6. Sqoop导出
7. MySQL
8. Web可视化

### 2.1 flume的组件

- Event：是Flume数据传输的基本单元，以事件的形式将数据从源头送到最终的目的
- Client：将一个原始log包装成为Events并发送到一个或多个实体
- Agent：将Events从一个节点传输到另外一个节点或最终目的地

其中Agent包含Source，Channel，Sink

- Source：用于对接数据源，接受Event或收集到的Data包装成Event
- Channel：包含Event驱动和轮询两种类型。
> source必须至少和一个channel关联    
> 可以和任何数量的sourc、sink工作  
> 用来缓存进来的Event，将source和sink连接起来，在内存中运行

- sink：存储Event到最终的目的地终端如HDFS、HBase
> 类似JDBC数据缓存池，一条一条没有一次插入性能高

### 2.2 Flume部署模式

####2.2.1 单一Agent采集数据
![](http://flume.apache.org/_images/DevGuide_image00.png)
####2.2.2 多Agent串联采集数据
![](http://flume.apache.org/_images/UserGuide_image03.png)
####2.2.3 多Agent合并串联采集数据
![](http://flume.apache.org/_images/UserGuide_image02.png)
####2.2.4 多Agent合并串联采集数据
![](http://flume.apache.org/_images/UserGuide_image01.png)


## 3. Flume安装配置
### 3.1 上传安装包至CentOS下

>解压安装包到hadoop目录下

	tar -zxvf apache-flume-1.9.0-bin.tar.gz -C /usr/hadoop

### 3.2 配置环境变量

	vi /etc/profile

在末尾添加以下代码，保存退出

	export FLUME_HOME=/usr/hadoop/apache-flume-1.9.0-bin
	export PATH=$FLUME_HOME/bin:$PATH

> 生效配置

	source /etc/profile

### 3.3 验证环境

	flume-ng version

> 出现以下结果配置正确

	Flume 1.9.0
	Source code repository: https://git-wip-us.apache.org/repos/asf/flume.git
	Revision: d4fcab4f501d41597bc616921329a4339f73585e
	Compiled by fszabo on Mon Dec 17 20:45:25 CET 2018
	From source with checksum 35db629a3bda49d23e9b3690c80737f9

### 3.4 配置Flume文件

规则：  
1. 指定Agent的名称以及指定Agent的各个组件的名称  
2. 指定Source  
3. 指定Channel  
4. 指定Sink  
5. 指定Source、Channel、Sink之间的关系  


[https://centos.pkgs.org/](https://centos.pkgs.org/)  

>下载 telnet 安装包并进行安装    

	rpm -ivh your-package  

> 在Flume下创建和修改配置文件

	mkdir agent
	vi netcat-logger.properties

> 配置Agent名称、Source、Channel、Sink的名称

	# 配置Agent名称、Source、Channel、Sink的名称
	a1.sources=r1
	a1.channels=c1
	a1.sinks=k1
	
	# 配置Channel组件属性c1
	a1.channels.c1.type=memory
	a1.channels.c1.capacity=1000
	
	# 配置Source组件属性r1
	a1.sources.r1.type=netcat
	a1.sources.r1.bind=localhost
	a1.sources.r1.port=8888

	# 配置Sink组件属性k1
	a1.sinks.k1.type=logger

	#连接关系
	a1.sources.r1.channels=c1
	a1.sinks.k1.channel=c1

> 启动Agent去采集数据
	
	-c conf:指定flume自身配置文件所在目录
	-n a1：指定agent的名字
	-f agent/netcat-logger.properties：指定采集规则目录
	-D：Java配置参数
	
#
>输入以下命令：
	
	flume-ng agent -c conf -n a1 -f ../agent/netcat-logger.properties -Dflume.root.logger=INFO,console

![TIM截图20191130005444.png](https://i.loli.net/2019/11/30/EvLnzGOMsKStR6Q.png)

>使用telnet命令，输入“hello”

![TIM截图20191130005604.png](https://i.loli.net/2019/11/30/8NtlnIWJ97cQUoi.png)

>生成event

![TIM截图20191130005651.png](https://i.loli.net/2019/11/30/mhtDPAX6sQfebiw.png)
