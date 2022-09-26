---
layout: mypost
title: Elasticsearch基础与实操
categories: [Elasticsearch]
---

*#音量注意：没有调节音量的功能，播放前请注意设备音量*

<iframe src="//music.163.com/outchain/player?type=2&id=1824751735&auto=1&height=66" frameborder="0" width="100%" height="86px" ></iframe>


## 序

> Elasticsearch-开源高扩展分布式全文搜索引擎，学到的都记录在下面了：

## 搭建7.4.2版本三节点ES集群过程

1. 配置防火墙

   ```shell
   cat /etc/sysconfig/iptables 
   #9200和9300端口都需要放开
   iptables -A INPUT -p tcp --dport 9200 -j ACCEPT
   iptables -A OUTPUT -p tcp --sport 9200 -j ACCEPT
   #重启执行两次
   service iptables restart
   iptables -L -n 
   #配完以后即时生效，有问题的话可以重启复原，telnet测试没问题后再保存：
   service iptables save
   #添加到自启动设置：
   chkconfig iptables on
   ```

2. 上传包到集群的第一台机器，随后分发到另外两台虚机

   > 下载地址：[下载 Elastic 产品：https://www.elastic.co/cn/downloads/](https://www.elastic.co/cn/downloads/)
   >
   > 1. 每个操作系统安装Elasticsearch的文件选择不同，参考：https://www.elastic.co/downloads/elasticsearch，选择对应的文件下载。
   > 2. 安装Kiabna需要根据操作系统做选择，参考：https://www.elastic.co/guide/en/kibana/current/install.html，选择对应的文件下载。
   > 3. 安装X-Pack需要根据Elasticsearch安装不同的方式提供不同的安装方法，参考：https://www.elastic.co/guide/en/x-pack/5.0/installing-xpack.html#installing-xpack。
   >
   > 一般情况下kibana的版本必须和Elasticsearch安装的版本一致。

   ```shell
   scp -r deployer@x.x.x.x:/home/deployer/elasticsearch-7.4.2-linux-x86_64.tar.gz /usr/share/
   cd /usr/share/elasticsearch-7.4.2
   tar -xzvf elasticsearch-7.4.2-linux-x86_64.tar.gz
   #为了安全，不允许用root用户启动，需要新建用户
   groupadd esuser 
   useradd -g esuser -p 此处密码 esuser
   chown -R esuser:esuser /usr/share/elasticsearch-7.4.2
   /usr/share/elasticsearch-7.4.2/bin/elasticsearch-plugin install x-pack
   /usr/share/elasticsearch-7.4.2/bin/kibana-plugin install x-pack
   ```

3. 检查旧版ES的设置，并和新的ES做对比

   ```shell
   cd /home/esuser/elasticsearch-2.4.1-master
   cd /usr/share/elasticsearch-7.4.2
   ```

4. 改esuser用户的profile配置`/etc/profile`

   ```shell
   export JAVA_HOME=/usr/share/elasticsearch-7.4.2/jdk
   export PATH=$JAVA_HOME/bin:$PATH
   if [ -x "$JAVA_HOME/bin/java" ]; then
   ​    JAVA="/usr/share/elasticsearch-7.4.2/jdk/bin/java"
   else
   ​    JAVA=`which java`
   fi
   ```

5. 修改elasticsearch.yml配置文件

   ```shell
   vim /usr/share/elasticsearch-7.4.2/config/elasticsearch.yml
   ```

   添加如下内容（注释内容不必添加）

   ```shell
   #集群名和节点名：
   #配置集群名称，默认是elasticsearch，es服务会通过广播方式自动连接在同一网段下的es服务，通过多播方式进行通信，同一网段下可以有多个集群，通过集群名称这个属性来区分不同的集群
   cluster.name: khzx-test-new
   #当前配置所在机器的节点名，你不设置就默认随机指定一个name列表中名字，该name列表在es的jar包中config文件夹里name.txt文件中，其中有很多作者添加的有趣名字。当创建ES集群时，保证同一集群中的cluster.name名称是相同的，node.name节点名称是不同的
   node.name: x.x.x.x-9300
   #初始主节点
   cluster.initial_master_nodes: ["x.x.x.x-9300"]
   #是否参与master选举和是否存储数据
   #指定该节点是否有资格被选举成为master（注意这里只是设置成有资格， 不代表该node一定就是master），默认是true，es是默认集群中的第一台机器为master，如果这台机挂了就会重新选举master。
   node.master: true
   #指定该节点是否存储索引数据，默认为true
   node.data: true
   #修改存储路径；索引数据的存储路径默认是es根目录下的data文件夹，可以设置多个存储路径，用逗号隔开，例：path.data: /path/to/data1,/path/to/data2
   path.data: /home/esuser/9300/datafile
   path.logs: /home/esuser/9300/logs
   #分片数和副本数，默认为五分片一副本，如果采用默认设置，而你集群只配置了一台机器，那么集群的健康度为yellow，也就是所有的数据都是可用的，但是某些复制没有被分配（健康度可用 curl 'localhost:9200/_cat/health?v' 查看，分为绿色、黄色或红色。绿色代表一切正常，集群功能齐全，黄色意味着所有的数据都是可用的，但是某些复制没有被分配，红色则代表因为某些原因，某些数据不可用）
   #index.number_of_shards: 5
   #index.number_of_replicas: 1
   #设置节点之间交互的tcp端口，默认是9300
   transport.tcp.port: 9300
   #设置是否压缩tcp传输时的数据，默认为false，不压缩。
   #transport.tcp.compress: true
   #设置对外服务的http端口，默认为9200
   #http.port: 9200
   #是否使用http协议对外提供服务，默认为true，开启。
   #http.enabled: false
   #允许其他网络访问
   network.host: x.x.x.x
   #设置集群中master节点的初始列表，可以通过这些节点来自动发现新加入集群的节点
   discovery.zen.ping.unicast.hosts: ["x.x.x.x:9300","x.x.x.x:9300","x.x.x.x:9300"]
   #discovery ping的超时时间，拥塞网络，网络状态不佳的情况下设置高一点
   discovery.zen.fd.ping_timeout: 120s
   discovery.zen.fd.ping_interval: 30s
   discovery.zen.fd.ping_retries: 3
   #master选举最少的节点数，这个一定要设置为整个集群节点个数的一半加1，即N/2+1
   discovery.zen.minimum_master_nodes: 2
   #是否锁定物理内存地址，默认配置为false，允许es使用swap交换分区，当jvm开始swapping时es的效率会降低，可能导致IOPS变高；要保证内存不swap，可以把ES_MIN_MEM和ES_MAX_MEM两个环境变量设置成同一个值，并且保证机器有足够的内存分配给es；同时也要允许elasticsearch的进程可以锁住内存，linux下启动es之前可以通过ulimit -l unlimited命令设置，后面会有具体介绍。
   bootstrap.memory_lock: false
   #是否支持过滤掉系统调用
   bootstrap.system_call_filter: false
   ```

   > 如何避免ElasticSearch发生脑裂（brain split）：http://blog.trifork.com/2013/10/24/how-to-avoid-the-split-brain-problem-in-elasticsearch/
   >
   > 注意，分布式系统整个集群节点个数N要为奇数个；但是即使集群节点个数为奇数，minimum_master_nodes为整个集群节点个数一半加1，也难以避免脑裂的发生，详情看讨论：https://github.com/elastic/elasticsearch/issues/2488
   >
   > 关于最后两项配置引用：[bootstrap.memory_lock简要说明](http://www.leiyawu.com/2019/10/17/bootstrap-memory-lock/)

6. 旧版本没有自带的免费的安全认证方案，采用了内网部署+NGINX+防火墙的方式做了认证策略限制。2019年5月21日开始，官方免费提供了x-pack实现Elastic Stack的核心安全功能。

   首先生成证书，在其中一个node节点执行即可，生成完证书传到集群其他节点

      ```shell
   /usr/share/elasticsearch-7.4.2/bin/elasticsearch-certutil ca
   /usr/share/elasticsearch-7.4.2/bin/elasticsearch-certutil cert --ca elastic-stack-ca.p12
   #修改权限
   ls elastic-*
   elastic-certificates.p12  elastic-stack-ca.p12
   chown esuser:esuser  elastic-*
      ```

   在上面elasticsearch.yml的末尾追加如下配置：

   ```shell
   #开启xpack认证机制
   xpack.security.enabled: true
   #此条不配将报错
   xpack.security.transport.ssl.enabled: true
   #证书相关，配置只进行证书验证
   xpack.security.transport.ssl.verification_mode: certificate
   xpack.security.transport.ssl.keystore.path: /usr/share/elasticsearch-7.4.2/bin/elastic-certificates.p12
   xpack.security.transport.ssl.truststore.path: /usr/share/elasticsearch-7.4.2/bin/elastic-certificates.p12
   ```

   重启es后，设置es默认用户的访问密码，重启后即可正常使用认证功能

      ```shell
   bin/elasticsearch-setup-passwrod interactive
      ```

   关于证书的两条命令

      ```shell
   #查看证书状态
   curl -XGET -u elastic:此处密码 "http://x.x.x.x:9200/_license"
   #查看证书信息
   curl -XGET -u elastic:此处密码 "http://x.x.x.x:9200/_xpack?pretty"
      ```

7. 修改jvm.options配置文件

   ```shell
   vim /usr/share/elasticsearch-7.4.2/config/jvm.options
   ```

   添加以下内容

      ```shell
   # Xms represents the initial size of total heap space
   # Xmx represents the maximum size of total heap space
   -Xms26g #初始化堆大小
   -Xmx26g #最大堆大小
      ```

   -xms和-xmx用来分配分配jvm堆内存，一般设置相同值，避免频繁扩容和GC释放堆内存造成的系统开销。最大可设置为30G，设置为26G是比较保险的值，这个值不要超过宿主机的内存的一半。

   -XX:HeapDumpPath=data设置当发生oom后生成的dump文件路径。

8. 修改kibana配置文件

   ```shell
   vim /usr/share/kibana-7.4.2-linux-x86_64/config/kibana.yml
   ```

   添加如下内容

   ```shell
   i18n.locale: "zh-CN"
   server.port: 5602
   server.host: "x.x.x.x"
   elasticsearch.hosts: ["http://x.x.x.x:9200","http://x.x.x.x:9200","http://x.x.x.x:9200"]
   elasticsearch.username: "elastic"
   elasticsearch.password: "此处密码"
   kibana.index: ".kibana"
   ```

9. 解除文件描述符等系统限制，改内核参数（不一定准确，回忆版）

   ```shell
   #修改配置文件“vi /etc/sysctl.conf”最后添加此行（或者用 echo "" >> /etc/sysctl.conf 追加）
   vm.max_map_count = 262144
   vm.swappiness = 1
   #文件配置后，让系统从配置文件“/etc/sysctl.conf”加载内核参数设置；
   /sbin/sysctl -p
   #修改配置文件“/etc/security/limits.d/90-nproc.conf”最后添加
   *          soft    nproc     unlimited
   root       soft    nproc     unlimited
   admin      soft    nproc     unlimited
   #root用户修改配置文件“/etc/security/limits.conf”
   *   soft    nofile     655350
   *   hard    nofile     655350
   *   soft    nproc      655350
   *   hard    nproc      655350
   *   soft    memlock    unlimited
   *   hard    memlock    unlimited
   esuser  soft  memlock  unlimited
   esuser  hard  memlock  unlimited
   #全部改完后重启
   #使用命令ulimit -a检查是否设置成功，显示：
   #max locked memory       unlimited
   ```

10. 启动，检查进程状态

    ```shell
     ./bin/elasticsearch -d
     ps -ef|grep elastic
     #如果是启动es2版本，则没有第七步的配置文件，需要指定环境变量启动
     export ES_HEAP_SIZE=6g
     /home/esuser/master-elasticsearch-2.4.1/bin/elasticsearch -d
     export ES_HEAP_SIZE=10g
     /home/esuser/data-elasticsearch-2.4.1/bin/elasticsearch -d
    ```

11. ES的目录结构

       ```shell
    [root@host-xx deployer]# cd /usr/share/elasticsearch-7.4.2
    [root@host-xx elasticsearch-7.4.2]# ll
    total 568
    #二进制文件，包括elasticsearch启动节点、elasticsearch-plugin安装插件
    drwxr-xr-x  2 esuser esuser   4096 May 21  2021 bin  
    #配置信息目录，包括elasticsearch.yml
    drwxr-xr-x  2 esuser esuser   4096 Aug 24 11:37 config  
    #自带jdk目录
    drwxr-xr-x  9 esuser esuser   4096 Oct 29  2019 jdk  
    #jar包存放目录
    drwxr-xr-x  3 esuser esuser   4096 Oct 29  2019 lib  
    -rw-r--r--  1 esuser esuser  13675 Oct 29  2019 LICENSE.txt
    #日志存放目录
    drwxr-xr-x  2 esuser esuser   4096 Feb 15 11:23 logs  
    #模块存放目录
    drwxr-xr-x 37 esuser esuser   4096 Oct 29  2019 modules  
    -rw-r--r--  1 esuser esuser 523209 Oct 29  2019 NOTICE.txt
    #插件安装目录，每个插件将包含在其子目录中
    drwxr-xr-x  3 esuser esuser   4096 May 21  2021 plugins  
    -rw-r--r--  1 esuser esuser   8500 Oct 29  2019 README.textile
       ```

## 基本概念

#### index 索引

>一个索引就是一个拥有几分相似特征的文档的集合。比如说，你可以有一个客户数据的索引，另一个产品目录的索引，还有一个订单数据的索引。一个索引由一个名字来标识（必须全部是小写字母的）。在一个集群中，可以定义任意多的索引。
>
>**可类比mysql中的表**

#### type 类型

> ES7之后废弃，略

#### filed 字段

> **可类比mysql中表的字段**，对文档数据根据不同属性进行的分类标识 

#### mapping 映射

> mapping是处理数据的方式和规则方面做一些限制，如某个字段的数据类型、默认值、分析器、是否被索引等等，这些都是映射里面可以设置的，其它就是处理es里面数据的一些使用规则设置也叫做映射，按着最优规则处理数据对性能提高很大，因此才需要建立映射，并且需要思考如何建立映射才能对性能更好。
>
> **相当于mysql中的创建表的过程，设置主键外键等等**

#### document 文档

> 一个文档是一个可被索引的基础信息单元。比如，你可以拥有某一个客户的文档，某一个产品的一个文档，当然，也可以拥有某个订单的一个文档。文档以 JSON（Javascript Object Notation）格式来表示，而JSON是一个到处存在的互联网数据交互格式。在一个index/type里面，你可以存储任意多的文档。
>
> **插入索引库以文档为单位，类比与数据库中的一行数据**

#### 集群 cluster

> 一个集群就是由一个或多个节点组织在一起，它们共同持有整个的数据，并一起提供索引和搜索功能。一个集群由 一个唯一的名字标识，这个名字默认就是“elasticsearch”。这个名字是重要的，因为一个节点只能通过指定某个集群的名字，来加入这个集群。

#### 节点 node

> 一个节点是集群中的一个服务器，作为集群的一部分，它存储数据，参与集群的索引和搜索功能。和集群类似，一 个节点也是由一个名字来标识的，默认情况下，这个名字是一个随机的漫威漫画角色的名字，这个名字会在启动的时候赋予节点。这个名字对于管理工作来说挺重要的，因为在这个管理过程中，你会去确定网络中的哪些服务器对应于Elasticsearch集群中的哪些节点。

> 一个节点可以通过配置集群名称的方式来加入一个指定的集群。默认情况下，每个节点都会被安排加入到一个叫 做“elasticsearch”的集群中，这意味着，如果你在你的网络中启动了若干个节点，并假定它们能够相互发现彼此， 它们将会自动地形成并加入到一个叫做“elasticsearch”的集群中。

> 在一个集群里，只要你想，可以拥有任意多个节点。而且，如果当前你的网络中没有运行任何Elasticsearch节点， 这时启动一个节点，会默认创建并加入一个叫做“elasticsearch”的集群。

> master管理节点资源配置：
>
> 管理节点主要负责存储索引的元数据、节点数据。佴是节点元数据变化是很小的，基本不会变，所以对CPU的要求不高，具备多线程处理能力即可。内存需要高一点即可，因为随着集群规模的增长，集群中索引的数量会越来越多。低配2C4GB，高点4C8GB。
>
> data数据节点资源配置:
>
> 数据节点需要承担综合能力，存储和查询都会在数据节点上，所以在生产环境中数据节点的CPU最好不要低于8C16GB。中配16C32GB，高配32C64GB（只分配32GB给ES)。CPU和内存的比例一般采用1:1或者1:2的比例。
>
> coordinating协调节点资源配置:
>
> 协调节点负责写入路由与查询路由、查询结果合并，数据节点只负责单节点数据写入/查询。当开始使用协调节点的时候，证明集群的规模较大，需要处理的任务比较多，所以建议配置大于等于集群中的数据节点的配置。

#### 分片和复制 shards&replicas

> 一个索引可以存储超出单个结点硬件限制的大量数据。比如，一个具有10亿文档的索引占据1TB的磁盘空间，而任一节点服务器都没有这样大的磁盘空间；或者单个节点处理搜索请求，响应太慢。为了解决这个问题，Elasticsearch提供了将索引划分成多份的能力，这些份就叫做分片。当你创建一个索引的时候，你可以指定你想要的分片的数量。每个分片本身也是一个功能完善并且独立的“索引”，这个“索引”可以被放置到集群中的任何节点上。分片很重要，主要有两方面的原因： 1）允许你水平分割/扩展你的内容容量。 2）允许你在分片（潜在地，位于多个节点上）之上进行分布式的、并行的操作，进而提高性能/吞吐量。

> 至于一个分片怎样分布，它的文档怎样聚合回搜索请求，是完全由Elasticsearch管理的，对于作为用户的你来说，这些都是透明的。

> 在一个网络/云的环境里，失败随时都可能发生，在某个分片/节点不知怎么的就处于离线状态，或者由于任何原因消失了，这种情况下，有一个故障转移机制是非常有用并且是强烈推荐的。为此目的，Elasticsearch允许你创建分片的一份或多份拷贝，这些拷贝叫做复制分片，或者直接叫复制。

> 复制之所以重要，有两个主要原因： 在分片/节点失败的情况下，提供了高可用性。因为这个原因，注意到复制分片从不与原/主要（original/primary）分片置于同一节点上是非常重要的。扩展你的搜索量/吞吐量，因为搜索可以在所有的复制上并行运行。总之，每个索引可以被分成多个分片。一个索引也可以被复制0次（意思是没有复制）或多次。一旦复制了，每个索引就有了主分片（作为复制源的原来的分片）和复制分片（主分片的拷贝）之别。分片和复制的数量可以在索引创建的时候指定。在索引创建之后，你可以在任何时候动态地改变复制的数量，但是你事后不能改变分片的数量。（修正：也可以通过收缩索引、拆分索引的方式实现改变分片数量）

> 默认情况下，Elasticsearch中的每个索引被分片5个主分片和1个复制，这意味着，如果你的集群中至少有两个节点，你的索引将会有5个主分片和另外5个复制分片（1个完全拷贝），这样的话每个索引总共就有10个分片。
>
> 索引分片数限制最大128个，单分片数据量推荐在20-40GB，不超过50GB，单个索引最大数据量6.25TB。实际上单索引数据量超过100GB就必须考虑拆分索引了。

## 从ES2到ES7的变化

#### 1.1版本差异

2.4.1版本与7.4.2版本差异较大，下面针对对应用影响较大的差异进行说明。

a)    字符串类型不再有string类型，使用keyword(不分词)或者text类型替代

b)    5.x 支持多种type，6.x 只能有一种type，7.x中无type类型，全部采用默认type替代。8.x中将彻底删除type概念。本次升级需要多type的索引需要进行拆分处理

c)    弃用TCP协议的transport client全面转向java high level rest client，使用rest api可以拥有更加丰富的功能和更高的性能

d)    查询语法与返回值有差异

Elasticsearch 7.0 开始，内置了 Java 环境，因此安装 7.0+ 版本会方便很多

**重点说明：**

Elasticsearch2.4.X版本过于陈旧，ES社区早已不再进行维护，在2.x上遇到的问题社区不解决，提交issue也不处理，提交的代码也不被接收。从业内实践经验来看存在较多问题，例如：master 更新元数据超时导致内存泄露、tcp 协议字段溢出等。从长远的角度看，伴随业务的发展，升级到一个较新的版本是一个更好的选择。

#### 1.2节点规划原则

jvm具体大小根据该node要存储的数据量来估算，为了保证性能，在内存和数据量间建议的比例为：

- 搜索类项目的比例建议在1:16以内

- 日志类项目的比例建议在1:48~1:96

- 最大堆内存大小不超过32G

- 根据热数据大小预留足够的内存给操作系统，做文件缓存，建议一般分配总内存的一半给ES

#### 1.3分片规划原则

根据官方团队以及社区的经验，分片规划建议如下：

- 单搜索场景： Shard大小 <= 15GB

- 日志场景，单 Shard大小 <= 50GB

- 分片数尽量是集群节点数量的倍数，集群负载会比较均衡

#### 1.4数据建模原则

为提升整个集群的使用效率，需要合理设置索引mapping，常见的配置如下所示：

- enabled: true | false 设置为false仅存储，不做搜索或聚合分析

- index: true | false是否构建倒排索引，不需要进行搜索的字段设置为false节省存储空间

- Index_options: docs | freqs | positions | offsets存储倒排索引的哪些信息

- norms: true | false是否存储归化相关参数，如果字段仅用于过滤和聚合分析，可关闭

- doc_values: true | false是否启用 doc_values，用于排序和聚合分析

- field_data: true | false是否为text类型启用 fielddata，实现排序和聚合分析

- store: true | false是否存储该字段值，默认为false即可

- coerce: true | false是否开启自动数据类型转换功能，比如字符串转为数字、浮点转为整型等

**一般性流程如下：**

1. 确定源数据类型
2. 检查是否需要检索
3. 是否需要排序和聚合分析
4. 是否需要另行存储

## 集群运维

#### 1.日志

1. 日志分类：

   集群名_deprecation.log：用以打印使用的过期api

   集群名_index_indexing_slowlog.log: 写慢日志

   集群名_index_search_slowlog.log：查询慢日志

   集群名.log：集群日志

2. 查看日志命令：查看尾部200行-搜索关键字aaa

   ```shell
   tail -n 200 catalina.out | grep -C 5 aaa
   ```

3. 节点缩容

   1. 排除节点

      先通过分配过滤器迁移数据，将此节点上分片数据移动至其它节点，防止直接kill节点导致分片和集群状态异常。

      PUT _cluster/settings?pretty

      {

      ​	"transient": {

      ​		"cluster.routing.allocation.exclude._ip": "node.ip"

      ​	}

      }

   2. 移除节点

      待该节点数据迁移完后，kill掉节点进程。

#### 2. 查询常用命令

（1）获取所有索引

GET  _cat/indices?format=json 

（2）获取所有mapping

GET  _mapping 

（3）查询集群健康状态

GET  _cluster/health 

（4）查询集群临时/永久配置

GET  _cluster/settings?pretty 

（5）查询节点信息

GET  _nodes/stats?pretty 

（6）查询节点个数

GET  _cat/nodes?v 

（6）查询索引文档

GET indexName/_search

{

  "query": {

   "term" : {

​    "name": "spring"

   }

  }

 }

#### 3.修改常用命令

（1）创建索引

PUT /indexName 

{

 "settings": {

  "index": {

   "number_of_shards": 1,  

   "number_of_replicas": 1

  }

 },

"mappings": {

  "properties": {

   "age":   { "type": "integer" },  

   "email":  { "type": "keyword"  }, 

   "name":  { "type": "text"  }   

  }

 }

}

（2）修改索引配置

PUT indexName /_settings

{

  "refresh_interval": "30s",

  "number_of_replicas":0

}

（3）索引创建别名

PUT indexName /_alias/test1

（4）暂停集群索引自动分片

PUT _cluster/settings?flat_settings

{

  "transient" : {

​    "cluster.routing.allocation.enable" : "none"

  }

}

（5）暂停集群自动负载均衡

PUT _cluster/settings

{

  "transient" : {

​    "cluster.routing.rebalance.enable" : "none"

  }

}

（6）reindex迁移数据

POST _reindex?wait_for_completion=false

{

  "index": "test3",

  "size": 5000

 },

 "dest": {

  "index": "test3"

 }

}

（7）修改集群为只读

PUT _all/_settings

{

​	"index": {

​		"blocks": {

​			"read_only": "false"

​		}

​	}

}

（8）增加慢日志打印

PUT _settings

{

  "index.search.slowlog.threshold.query.debug": "200ms",

  "index.search.slowlog.threshold.query.info": "1s",

  "index.search.slowlog.threshold.query.warn": "3s"

}

#### 4.故障排查相关

1. 查看当前任务，分析耗时

   GET _cat/tasks/?v

2. 查看当前线程队列，队列是否占满，是否有拒绝队列

   GET _cat/thread_pool?v

   GET _cat/thread_pool/search?v

3. 查看热点线程，哪些线程占用cpu多

   GET _nodes/hot_threads

4. 缓存占用查看

   查看request缓存

   _stats/request_cache?pretty&human

   查看fielddata缓存

   _stats/fielddata?fields=*&pretty

   清除缓存

   POST /_cache/clear?fielddata=true

   POST /_cache/clear?request_cache=true

5. 分片不自动分配排查

   一般来说，ElasticSearch会自动分配 那些 unassigned shards，当发现某些shards长期未分配时，首先看下是否是因为：为索引指定了过多的primary shard 和 replica 数量，然后集群中机器数量又不够。另一个原因就是由于故障，shard自动分配达到了最大重试次数了，这时执行 reroute 就可以了（POST _cluster/reroute?retry_failed=true）。

   查看执行计划，查看分配失败原因

   GET _cluster/allocation/explain
    {
   "index": "",
     "shard": 2,
     "primary": false 
   }

   分配一个过期的主分片
   {
     "commands": [
       {
         "allocate_stale_primary": {
           "index": "index",
           "shard": 1,
           "node": "es.ip.3",
           "accept_data_loss": true
         }
       }
     ]
   }

   分配一个空的主分片
   allocate_empty_primary

   分配一个副本分片
   allocate_replica
   去掉 "accept_data_loss": true

6. 查看、合并segment

   ES默认段达到5g不再合并，可以进行手动段合并，减少内存和磁盘占用，提高读写效率。查询会查询所有段，包含删除状态段，结果合并的时候再过滤掉删除状态的段，段合并可以减少查询段的数量提高查询效率。

   按索引总览segment

   GET /_cat/indices?=segmentsCount:desc&v&h=index,segmentsCount,segmentsMemory,memoryTotal,mergesCurrent,mergesCurrentDocs,storeSize,p,r

   删除状态的段

   GET _cat/segments?v&s=docs.deleted:desc,size

   删除状态段合并
   POST mytest/_forcemerge?only_expunge_deletes=true
   only_expunge_deletes=true 只merge已删除状态的  max_num_segments=1合并为一个段

   合并全部段
   POST _forcemerge

   查看各个节点forceMerge的线程数
   GET _cat/thread_pool/force_merge?v&s=name
   查看forceMerge任务详情
   GET _tasks?detailed=true&actions=*forcemerge

7. 查询慢

   - 增大刷新时间间隔 index.refresh_interval，缓存失效是自动的，当索引 refresh 时就会失效
   - 尽量使用缓存
   - 增加副本数
   - 控制分片大小，建议30G左右
   - 段合并，减少段数量
   - 优化慢sql语句

8. 写入慢

   - 增大刷新时间间隔 index.refresh_interval
   - bulk批量写入
   - 使用ssd盘
   - 修改translog异步写入，有丢失数据风险，评估数据重要性后使用
   - 控制分片大小，写入多场景每个分片不超过50G
   - 大批量写入，可以暂时修改副本数为0

## 唠叨环节

下雪啦，下雪啦！雪地里来了一群小画家

小鸡画竹叶，小狗画梅花，小鸭画枫叶，小马画月牙

不用颜料不用笔，几步就成一副画

青蛙为什么没参加? 他在洞里睡着啦