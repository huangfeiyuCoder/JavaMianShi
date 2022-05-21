<a name="ZXisM"></a>
# MYSQL的初步原理了解
初步了解数据库的底层结构设计，和常见的一些关键词。比如数据库的分层， 客户端层，服务层，查询优化器，语法解析器，执行引擎，存储引擎，RedoLog，undoLog，RedoBuffer缓存池，数据缓存池，OSCache，磁盘等等。知道这些组件在上面是干啥的。
<a name="fa533"></a>
## 01-系统如何跟MySQL打交道的？
数据库驱动，数据库连接池，JDBC，MYSQL连接层<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1635321477555-f7b545bb-10f2-432b-9adb-b2d5ac01150a.png#clientId=ue833fb38-ced1-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=316&id=DmPJf&margin=%5Bobject%20Object%5D&name=image.png&originHeight=632&originWidth=1328&originalType=binary&ratio=1&rotation=0&showTitle=false&size=447590&status=done&style=none&taskId=u23dccbf4-8bf0-4d92-b173-e03d07604fa&title=&width=664)
<a name="CwpYR"></a>
### SQL连接驱动
在底层跟数据库建立网络连接，发送SQL指令。
<a name="SKiq6"></a>
### 数据库连接池
如果多个线程并发处理多个请求的时，去抢夺一个连接去访问数据库的话，那效率很低下的。<br />而且连接的创建销毁也是一个很浪费性能的问题。<br />每次还要去做账号，权限连接的校验，也是浪费。<br />所以，使用一个池子，把连接池存起来，要用的时候拿出来用。维持一个核心连接数



<a name="f6gas"></a>
## 02-Mysql用架构设计？
MySQL的架构设计中，SQL接口、SQL解析器、查询优化器其实都是通用的，他就是一套组件而已。<br />但是存储引擎，就是插拔式的，可以选择不同的存储引擎。常见的InnoDB、MyISAM、Memory等等<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1635322397192-33db015f-80c0-455b-b108-8f910b40ac10.png#clientId=ue833fb38-ced1-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=350&id=uad95f83d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=506&originWidth=1100&originalType=binary&ratio=1&rotation=0&showTitle=false&size=435893&status=done&style=none&taskId=uf8657311-5a2f-4794-b1a5-080fdffe041&title=&width=761)

<a name="BP7Hn"></a>
### 一个不变的原则：网络连接必须让线程来处理
说白了就是一个 Mysql的客户端连接层，别整的这么高大上。。<br />Mysql内部有工作线程池，监听客户端发送过来的SQL语句。<br />Tomcat不也是这种设计。。。

<a name="YUZzM"></a>
### SQL接口：SQL运行接口封装
可以理解为：用来执行SQL语句的一个接口，也就是真正开始干事的API

<a name="hgi5Q"></a>
### 查询解析器：解析SQL语法
SQL接口中先要经过查询解析器，负责对SQL语句进行解析。<br />就相当于动作拆解，先要打开冰箱，然后把大象放进去，然后关门。就是先解析动作。

<a name="Ii4cd"></a>
### 查询优化器：查询最优路径
会经过查询优化器，找到最优的查询路径。<br />因为一条SQL语句可能会走不同的索引，去拿到数据，所以需要一个最优的方案。

<a name="PsQke"></a>
### 存储引擎：真正执行SQL语句
底层的存储引擎去真正的执行语句，去操作磁盘文件，获取，修改你要的结果。<br />不通存储引擎有着不同的作用？这里要思考一下，后面也会讲讲。

<a name="h0wG9"></a>
### 执行器：按计划调用存储引擎接口

- [x] 存储引擎可以帮助我们去访问内存以及磁盘上的数据，那么是谁来调用存储引擎的接口呢？

执行器会根据**优化器**选择的执行方案，**调用存储引擎的接口**按照顺序和步骤，把SQL语句的逻辑给执行了。

说白了，就相当于执行器才是最后的指挥长。上面介绍的那些，只是封装的功能。<br />负责跟存储引擎配合完成一个SQL语句在磁盘与内存层面的全部数据更新操作。

<a name="ChqDz"></a>
## 03-初步了解innoDB的存储引擎架构设计
下面以一条更新语句为例子：<br />注意：（在写入Redo日志前，执行器还会记录一条binlog日志，然后在redo文件里，记录一个commit标记，告诉他binlog日志以同步完成，可提交事物。）

在存储引擎中先后经历了如下：<br /> ->数据从磁盘中加载到数据缓存池<br /> ->记录Undo日志，方便回滚<br /> ->修改数据缓存池中的值<br /> ->记录Redo日志，记录修改的内容<br /> ->把Redo日志池数据，刷到磁盘中持久化。<br /> -> commit 提交事务。
<a name="QvJWs"></a>
### 缓冲池：InnoDB的重要内存结构

- 内存缓冲池里有数据，就可以不用去查磁盘了。空间换时间，比扫描磁盘高效。
- 如果直接在缓冲池中修改记录，会对这条记录加**独占锁**。 后面单独介绍锁

<a name="cNsLe"></a>
### undo日志文件：回滚更新的数据

- 为了考虑到可能要回滚数据的需要，这里会把你更新前的值写入undo日志文件。

<a name="XmogV"></a>
### 更新缓冲池的数据：持久化到磁盘
把要更新的记录从磁盘文件加载到缓冲池，同时加独占锁后，且要把更新前的旧值写入undo日志文件。<br />之后，正式开始更新这行记录了，更新时，先会更新缓冲池中的记录，此时这个数据就是脏数据了。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1635324045972-77344814-79ea-4c74-bbd4-c66767a0c4d5.png#clientId=ue833fb38-ced1-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=410&id=u65df4b01&margin=%5Bobject%20Object%5D&name=image.png&originHeight=820&originWidth=1342&originalType=binary&ratio=1&rotation=0&showTitle=false&size=705900&status=done&style=none&taskId=uadcf42de-d521-43d4-af1e-ebc33a32ae8&title=&width=671)
<a name="m5Gav"></a>
### Redo Log Buffer：系统宕机，避免数据丢失

- 用来存放Redo日志的日志缓存池
- 因为上面已经对你的 记录缓冲池 做了更新，修改值还没有落到你的磁盘上，宕机容易丢失数据。
- 所以需要一个Redo日志去记录你的修改值，避免宕机后，修改数据丢失。
- Innodb存储引擎特有。
> Redo日志，就是记录下来你对数据做了什么修改，比如对“id=10这行记录修改了name字段的值为xxx”，这
> 就是一条日志。偏向于物理性质的 重做日志


<a name="nakjN"></a>
### 提交事务：将修改值存到磁盘
上面说到Redo日志也是先存放内存中的，还没有去提交事物，此时如果宕机也是会造成修改数据丢失的！！<br />但是开发中，一般是会以提交事务为准，才算修改成功，没提交事务就当做失败处理。

<a name="B9pjr"></a>
#### 提交事务时，将Redo日志写入磁盘

- 想提交一个事务，会按设置的策略刷入到磁盘，或者OS缓存，或者不刷就提交。
- 这个策略是通过innodb_flush_log_at_trx_commit来配置的。

<a name="juGDP"></a>
#### Redo刷盘策略

- [x] **innodb_flush_log_at_trx_commit持久化到磁盘的三个策略？**
- 设置为 0 时：不持久化，直接提交事务，会丢失数据。
- 设置为 1 时：会持久化到磁盘，提交事物，不会丢失数据。（通常做法）
- 设置为 2 时：持久化到OS系统内存中，后面会定期刷到磁盘上，

提交事物。MySQL宕机不会丢失，但是操作系统宕机后就会丢失数据。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1635325431699-70a6277b-e76b-4d55-99da-cafb7308de42.png#clientId=ue833fb38-ced1-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=308&id=uad531bdb&margin=%5Bobject%20Object%5D&name=image.png&originHeight=510&originWidth=1098&originalType=binary&ratio=1&rotation=0&showTitle=false&size=411866&status=done&style=none&taskId=u172a85ce-2f93-43c5-add8-b72099c02d3&title=&width=663.991455078125)

<a name="w2rp8"></a>
## 04-初步聊更新语句中BinLog日志

<a name="ejghU"></a>
### MySQL binlog到底是什么东西？
简单来说也就是，**归档日志**。是MYSQL server服务层面的日志，不属于存储引擎特有。<br />记录的意思是：“那个数据页做了什么修改”

<a name="WdbDC"></a>
### 提交事务：同时也会写入BinLog日志
在提交事务时，执行器会把binlog日志写入到磁盘中，做日志归档，和主从同步。

<a name="eSicn"></a>
### binlog日志的刷盘策略分析
Binlog日志也有刷盘策略，看是刷到OS Cache中，还是磁盘上。sync_binlog参数设置为1，还是0

- 设置为 0 时：刷到OS Cache中，如果系统机器宕机的话，会丢失日志。（默认的设置）
- 设置为 1 时：刷到磁盘文件中，宕机也不会丢失，就是效率低一点。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1635327349232-b542a4b4-1e5b-4007-913f-7d691377cf32.png#clientId=ue833fb38-ced1-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=583&id=u17ba8bb7&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1166&originWidth=1490&originalType=binary&ratio=1&rotation=0&showTitle=false&size=850449&status=done&style=none&taskId=u985cb2ca-530b-4b2b-8f10-1c886943327&title=&width=745)
<a name="kwcbl"></a>
### <br />基于binlog和redo log完成事务的提交
上面执行器记录完binlog日志后，会在Redo日志里记录 ：<br />本次binlog日志文件名，和文件位置，以及写入一个commit标记（可提交事物的标记）

<a name="rV7Ur"></a>
### 思考
<a name="D1Bmj"></a>
#### 在redo日志中写入commit标记的意义是什么？
用来保持redo log日志与binlog日志一致的，防止写完binlog日志后就宕机了。

<a name="J6HJB"></a>
#### 后台IO线程随机将内存更新后的脏数据刷回磁盘
上面说到的提交事务和记录日志，但是最终 修改的值 还是没有落到磁盘上。<br />所以需要有后台IO线程，定时把内存缓存池中的修改值，刷到磁盘中。此时内存和磁盘上的数据才一致。

<a name="xgwhf"></a>
#### 修改语句经历的事情
引擎 ->数据从磁盘中加载到数据缓存池<br />引擎 ->记录Undo日志，方便回滚<br />引擎 ->修改数据缓存池中的值    ......> 后台IO线程异步刷回磁盘<br />   执行器 ->MySQL Server服务层会记录binlog日志，并刷到OSCache 或 磁盘中 <br />引擎 ->然后记录Redo日志，记录修改的内容，同时记录binLog日志信息，和commit可提交标记。<br />引擎 ->把Redo日志池数据，刷到磁盘中持久化。<br />引擎 -> commit 提交事务。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1635327669627-fc996a12-678e-44d4-ac74-558847f84504.png#clientId=ue833fb38-ced1-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=576&id=sPZwC&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1152&originWidth=1318&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1029569&status=done&style=none&taskId=u56dfae65-f9f2-4af6-8ff9-96596be52cb&title=&width=659)

<a name="aMy7V"></a>
## 05-数据库机器配置和性能压测
<a name="Ps07b"></a>
### 普通的Java应用系统部署在机器上能抗多少并发？

   - Java应用系统部署的时候常选用的机器配置大致是2核4G和4核8G的较多一些
   - 数据库部署的时候常选用的机器配置最低在8核16G以上，正常在16核32G。
- 对Java系统而言，如果仅仅只是系统内部运行一些普通的业务逻辑，纯粹在自己内存中完成一些业务逻辑，这个性能是极高的，如果有IO，网络连接，数据库连接的话，性能就差一些。
<a name="g1dXH"></a>
#### 配置 对应的 并发数

   - 4C8G的机器每秒500左右并发量是OK的
<a name="OYANl"></a>
### 高并发下，数据库应该用什么配置
（我们财务系统是4C8G，压测环境的数据库是8C16G）<br />通常机器下：8核16G的MySQL数据库，QPS达到3千左右，TPS达到2千，是没问题的。<br />当然也看机器的CPU型号和是否为固态磁盘等。

<a name="AiKra"></a>
### 生产环境的数据库压测
<a name="YokEg"></a>
#### 架构师该有逼数的几个点

- 运维部署完MYSQL之后，里面的复杂参数让DBA自己去调整，比较复杂，不建议自己调。
- 当然，有部分高阶参数的调整会影响性能的，我们也要知道有这么个东西。
- 其次：大概知道几核几G的机器能够扛下多少量的并发，这也要知道个大概。

<a name="JO3rJ"></a>
#### 数据库服务先要压测
基于工具，模拟每秒发出1000个请求到数据库，观察一下他的CPU负载、磁盘IO负载、网络，IO负载、内存复杂，看能否每秒处理掉这1000个请求，或处理500个请求？ 这个过程，就是压测。<br />说白了，就是你的知道这个数据库的负载情况，心里有点数。不至于新系统刚上线就炸了。

<a name="WPwvT"></a>
#### QPS和TPS的区别

- **QPS**：queryPerSecond,每秒可以处理多少个SQL语句。
- TPS：TransactionPerSecond，每秒可以处理多少个事务。 可能有多条SQL组成一个事务。

注意：QPS不仅仅是所有查询，也包含修改语句。

<a name="MNWZ3"></a>
#### 压测性能指标
**IO相关的**

- IOPS：机器IO的并发读写请求能力
- 吞吐量：机器磁盘每秒存储多少字节的量，一般redolog,binlog日志都会直接和磁盘交互的，会有影响。
- Iatency：这个指标说的是**往磁盘**写入一条数据的延迟。

**其他性能指标**

- CPU负载：看机器的运算能力，CPU是否发热宕机。
- 网络负载：每秒网卡能输出多少MB数据
- 内存负载：看你数据库服务占用的内存情况有多少，过高可能会被操作系统踢掉进程。或者死机。

<a name="gWtNN"></a>
### 数据库压测经验和工具
<a name="nlXkT"></a>
#### 数据库压测工具：sysbench

- sysbench，这个工具可以自动帮你在数据库里**构造出来大量的数据**，
- 可以模拟几千个**线程并发访问**数据库，模拟各种SQL语句来访问数据库，
- **模拟各种事务提交**到数据库里去，甚至模拟出几十万的TPS去压测数据库。
<a name="YIkTD"></a>
#### sysbench压测工具的安装使用
> curl -s [https://packagecloud.io/install/repositories/akopytov/sysbench/script.rpm.sh](https://packagecloud.io/install/repositories/akopytov/sysbench/script.rpm.sh) | sudo bash
> sudo yum -y install sysbench
> sysbench --version

这里看看就好，安装过程和使用过程到时候去百度就OK了/

- [ ] 自己可以动手实践一下。

压测中，要会观察Mysql服务能抗下多少TPS，QPS，数量由少变多，增加到服务卡顿为止。
> 在压测的过程中，必须是不停的增加sysbench的线程数量，持续的让数据库承载更高的QPS，同时密切关注机器的CPU、内存、磁盘和网络的负载情况，在硬件负载情况比较正常的范围内，哪怕负载相对较高一些，也还是可以继续增加线程数量和提高数据库的QPS的。


<a name="VyNab"></a>
#### Linux的Top命令分析CPU负载情况
> top - 15:52:00 up 42:35, 1 user, load average: 0.15, 0.05, 0.01

结果解释**：**

- 15:52:00 = 当前时间
- up 42:35 = 机器运行了多长时间
- 1 user = 当前有一个用户正在使用
- load average: 0.15, 0.05, 0.01 = CPU在一分钟，五分钟内的负载情况。（重点关注）

解释CPU负载是什么意思<br />假设我们是一个4核的CPU，此时如果你的CPU负载是0.15，这就说明，4核CPU中连一个核都没用满，4核CPU基本都很空闲，没啥人在用。<br />如果你的CPU负载是1，那说明4核CPU中有一个核已经被使用的比较繁忙了，另外3个核还是比较空闲一些。要是CPU负载是1.5，说明有一个核被使用繁忙，另外一个核也在使用，但是没那么繁忙，还有2个核可能还是空闲的。<br />发现4核CPU的load average已经基本达到3.5，4了，那么说明几个CPU基本都跑满了。

<a name="QmwIJ"></a>
####  Linux上观察内存负载情况
> Mem: 33554432k total, 20971520k used, 12268339 free, 307200k buffers

结果解释：

- Mem: 33554432k tota = 总内存大概有32个G
- 20971520k used  =  已经使用大概20G
- 12268339 free  =  还有10G+是空闲的
- 307200k buffers  =  还有300M用作OS内核缓冲区（OsCache）

<a name="fuGdo"></a>
#### Linux上观察磁盘IO负载情况
查看磁盘IO命令：  dstat -d<br />结果：<br />-dsk/total -<br />read writ<br />103k 211k<br />0   11k<br />分析： 

- 存储的IO吞吐量是每秒钟读取103kb的数据，每秒写入211kb的数据。
- 普通机械硬盘都是每秒上百M的读写，所以这个很正常。

查看磁盘IOP，IOPS命令：		dstat -r<br />结果：<br />--io/total-<br />read writ<br />0.25 31.9<br />      0   253<br />0   39.0<br />分析：意思就是读IOPS和写IOPS分别是多少，也就是说随机磁盘读取每秒钟多少次，随机磁盘写入每秒钟执行多少次，一般来说，随机磁盘读写每秒在两三百次都是可以承受的。

<a name="SCm0K"></a>
#### Linux上观察网卡流量情况
网卡流量命令:		dstat -n <br />结果：<br />-net/total-<br />recv send<br />16k  17k<br />分析:<br />每秒钟网卡接收到流量有多少kb，每秒钟通过网卡发送出去的流量有多少kb<br />通常来说，机器使用的是千兆网卡，那么每秒钟网卡的总流量也就在100MB左右，甚至更低一些。

<a name="hFuwn"></a>
#### 数据库监控部署
简单介绍一下Prometheus和Grafana是什么<br />普罗米修斯<br />Prometheus其实就是一个监控数据采集和存储系统，他可以利用监控数据采集组件（比如mysql_exporter）从你指定的MySQL数据库中采集他需要的监控数据，然后他自己有一个时序数据库，他会把采集到的监控数据放入自己的时序数据库中，其实本质就是存储在磁盘文件里。

安装和启动Prometheus<br />部署Grafana，对数据库的监控系统做可视化报表<br />[http://cactifans.hi-www.com/prometheus/](http://cactifans.hi-www.com/prometheus/)<br />安装过程自己搜资料，这里不多BB。

<a name="HeANg"></a>
# MYSQL原理回顾深入
专栏的讲解顺序，MySQL经过压测的、有完善监控的数据库之后，你开发时使用数据库的顺序来讲解，先讲解系统中执行各种增删改操作时背后对应的内幕原理，以及事务的原理，包括锁的底层机制，然后讲解执行各种复杂的查询操作的时候，涉及到的索引底层原理，查询优化的底层原理。<br />再来讲解平时我们在开发系统的时候，如何进行数据库的建模，在数据库建模的时候，应该如何注<br />意字段类型、索引类型的一些问题，如何保证数据库避免死锁、高性能的运行。<br />最后说说主从架构设计以及分库分表架构设计，包括一些生产实践的案例。

<a name="lyIrQ"></a>
## 06-回顾BufferPool在数据库中的重要性
BufferPool:里面缓存了磁盘里的真实数据，为了提高操作数据的效率，把数据从磁盘存储到了内存上。我们后面把它称之为“数据缓存池”

<a name="oR4aK"></a>
### BufferPool如何配置大小

   - 默认情况下是128MB
   - 数据库如果是16核32G的机器，可以分配个2G
   - 设置参数：innodb_buffer_pool_size = 2147483648
<a name="PekYa"></a>
### 存储的数据结构是什么？

- Pool里面存内存数据页（缓存页）；  
- 数据页里面存有缓存数据和 页的表空间，数据页编号等描述信息。

<a name="EkaWM"></a>
#### 数据页：抽象出来的数据单位

- MySQL对数据抽象出一个**数据页**的概念，数据页的形式存储在BufferPool里，而不是按行存储在Pool里。
- 数据页  也叫做  缓存页 
- 数据在磁盘上，也是以数据页的形式存在的。

BufferPool中默认下，一个Pool缓存页的大小和磁盘上的一个数据页的大小是一一对应起来的，都是16KB。

Buffer Pool里面分为很多个缓冲页，缓存页里包含 数据和 页的表空间，数据页编号等描述信息。<br />描述信息大概占用缓存页的 5% 左右，大概是800个字节左右。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1635488808063-e2880343-0bf5-42e6-a76e-24bd5874c106.png#clientId=u6b283d15-a697-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=411&id=IB3s4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=602&originWidth=900&originalType=binary&ratio=1&rotation=0&showTitle=false&size=297233&status=done&style=none&taskId=u4156f629-97c0-419f-ab50-48ae1586874&title=&width=614.9943237304688)

<a name="zhgJV"></a>
## 07-磁盘读取到缓存页时 free链表的作用
<a name="He2w0"></a>
### BufferPool是如何初始化的？
启动数据库会先分配一块Pool空间，里面划分好数据页个数，内容都是空的。（大概意思就是先准备空容器） ，且会构建一条 free双向链表，用于存储 空页的描述信息。

<a name="vcCi6"></a>
### 运行时咋知道哪些缓存页时空闲的？
会为Buffer Pool设计一个free链表，他是一个双向链表数据结构，free链表里，每个节点就是一个空闲的缓存页的描述数据块的地址，只要有一个缓存页是空闲的，那么他的描述数据块就会被放入这个free链表中。

<a name="xOgwN"></a>
### free链表：存储缓存页描述信息
存储在BufferPool 空间中，用于串连每个缓存页的描述信息，方便知道缓存页的个数和占用情况。
<a name="LwGbA"></a>
#### 底层数据结构：
双向链表，且有个基础节点，分布指向头结点，和尾节点，用于记录节点个数。
<a name="ToOgg"></a>
#### free链表的实现原理：
free链表，他本身是由Buffer Pool里的描述数据块组成的，你可以认为是每个描述数据块里都有两个指针，一个是free_pre，一个是free_next，分别指向自己的上一个free链表的节点，以及下一个free链表的节点。<br />只有一个基础节点是不属于Buffer Pool的，40字节大小的节点，里面就存放了free链表的头节点的地址，尾节点的地址，还有free链表里当前有多少个节点。

<a name="XCtzt"></a>
### 如何把磁盘数据缓存到BufferPool的缓存页中
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1635491395918-a1bebcae-8b4d-482a-b43a-69070f3e524c.png#clientId=u6b283d15-a697-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=319&id=cv4xr&margin=%5Bobject%20Object%5D&name=image.png&originHeight=353&originWidth=777&originalType=binary&ratio=1&rotation=0&showTitle=false&size=324677&status=done&style=none&taskId=u92f6affd-6293-4e30-9087-c885202b67b&title=&width=701.491455078125)

- Free链表的时间复杂度：O(1)
- Hash表的时间复杂度：O(n) 

<a name="B2Bqq"></a>
#### 如何把磁盘数据存储到内存Pool中？
1.先找在free链表上找到一块描述信息，以及对应的空闲缓存页<br />2.把磁盘数据加载到内存中，但是OS系统层面的空间转换，这里没有说清楚。<br />3.然后free链表上对应的这个描述节点，写入表描述信息，和数据描述信息。<br />4.在链表上删除描述节点。<br />5.同时在Hash表中，对放进去的缓存页号+表空间，作为key存进去。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1635490641007-e4d41b0c-92e8-41c1-b0bb-924d9ccf9c9e.png#clientId=u6b283d15-a697-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=608&id=u0f0a57c6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1216&originWidth=1810&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1459028&status=done&style=none&taskId=ue5152bd3-abbe-4cbd-a01f-3bd4d936488&title=&width=905)
<a name="nPeda"></a>
#### Hash表维护：数据是否重复缓存
数据库中维护了一个**Hash表**，用表空间+数据页号 作为一个Key，缓存页的地址未做Value。<br />需要使用缓存数据页的时候，先去Hash表中查询一下，是否已经存在。

<a name="JQdiw"></a>
#### 思考：

- [ ] 我们写SQL时是对 表+行 进行操作，而数据库内部是对 表+缓存页 操作，那是如何关联上的呢？

<a name="k68TK"></a>
## 08-更新数据:flush链表记录被修改的数据页
补充: Pool也有可能存在 内存碎片，这要取决与你的数据页排列是否紧密。
<a name="HzEOa"></a>
### flush链表的作用？
简单来说就是：记录哪些缓存页被修改过，就加入到flush链表中。<br />以防Pool中的数据刷到磁盘上时，把没有修改的缓存数据也刷上去，减少刷盘的数据。因为有些数据是 查询时，才加入BufferPool 缓存中的，没有修改，就不需要更新到磁盘。此时就可以把记录在flush链表上的 缓存页 过滤掉。
<a name="jsWHv"></a>
#### 底层数据结构：
双向链表，且有个基础节点，分布指向头结点，和尾节点，用于记录节点个数。
<a name="qcjy5"></a>
#### flush链表的实现原理：
(和free 链表是一样的)<br />flush链表，他本身是由Buffer Pool里的描述数据块组成的，你可以认为是每个描述数据块里都有两个指针，一个是free_pre，一个是free_next，分别指向自己的上一个free链表的节点，以及下一个flush链表的节点。<br />只有一个基础节点是不属于Buffer Pool的，40字节大小的节点，里面就存放了flush链表的头节点的地址，尾节点的地址，还有flush链表里当前有多少个节点。

<a name="PFCYb"></a>
## 09-bufferPool缓存页不够：LRU算法淘汰缓存
LRU:(Least Recently Used) 最少使用算法去淘态那些不常用的缓存页，把它刷回磁盘，清空缓存区。
<a name="zpy9v"></a>
### LRU链表实现原理
在Pool中维护一个LRU链表，里面存储每个缓存页号，使用（读，写）到了这个缓冲页，就把它移到 链首，这时链尾的缓存页号，就是不常用的节点，直接清空就行。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1635495048769-ca42c7a1-de54-4bda-9221-18ce154f3f9a.png#clientId=u6b283d15-a697-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=512&id=ud802eeef&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1024&originWidth=2200&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1320165&status=done&style=none&taskId=ub471f453-05f0-4544-996c-9b987d350d3&title=&width=1100)
<a name="dwZ2w"></a>
### 简单LRU算法带来的问题
<a name="ayF6e"></a>
#### 可能出现的问题：

1. 预读机制加载的邻近页，挤掉链尾的 热点缓存页节点。
1. 全表扫描挤进大量缓存页，把之前的热点数据给踢到磁盘。

**解决方案：**<br />下一章节会说到一个 冷热数据分离的方案，优化LRU算法。

<a name="go23g"></a>
#### 什么是预读机制？
当从磁盘上加载一个数据页时，可能会连带着把这个数据页相邻的其他数据页，也加载到缓存里去！

<a name="LiiSu"></a>
#### 触发预读机制有哪些？
1.顺序访问一个区的多个数据页，当超过一个阈值，就会相邻区加载到缓存中。<br />阈值参数设置：innodb_read_ahead_threshold，他的默认值是56

2.Pool里缓存了一个区里的13个连续的数据页，且这些数据页都频繁被访问，此时就会触发预读机制，把这个区里的其他的数据页都加载到缓存里去。<br />参数设置：innodb_random_read_ahead来控制的，他默认是OFF

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1635496592069-6b633f45-aa11-4a8b-bdef-3825f3f1810a.png#clientId=u6b283d15-a697-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=396&id=ua11e4a7f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1028&originWidth=1574&originalType=binary&ratio=1&rotation=0&showTitle=false&size=914490&status=done&style=none&taskId=u26f5cfcd-61f5-472d-8860-f680a0f0c8e&title=&width=606.9971313476562)

- [ ] 上面提到了一个区？ 到底啥叫数据区？和数据页有啥关系呢？

<a name="GlwWL"></a>
## 10-LRU算法的优化：冷热数据分离
对上一章节LRU的优化，解决上面那些热点数据被挤掉，被突然的大数据量涌进踢掉的问题。
<a name="kU4hX"></a>
### 冷热数据分离实现原理

1. 把LRU链表，拆分为两个链表，默认长度为 3：7
1. 缓存页刚加载进来，先进入冷链表的头结点
1. 当数据过1S后 （默认1秒，可以调节innodb_old_blocks_time 参数）
1. 缓存页依然被访问到，则把缓存页放到热链表的头结点上。
1. 然后淘汰的话，先淘汰 冷链表上的  尾结点数据。

<a name="CGdal"></a>
### LRU问题解决
相当与做了一个二级淘汰缓存。把那些预加载进来的缓存页，和全表扫描进来的缓存页，都留在了冷链表，清除的时候不会影响热点数据。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1635497809443-aa5ccbf9-ca24-497f-80b9-b94342f56b8e.png#clientId=u6b283d15-a697-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=259&id=ub2b6f0b5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=372&originWidth=1006&originalType=binary&ratio=1&rotation=0&showTitle=false&size=287305&status=done&style=none&taskId=uce0f641e-f73b-4d6b-a9e8-16a49021550&title=&width=700.9971313476562)
<a name="WeskB"></a>
### 思考和问题

- [x] 这种LRU冷热数据分离的思想，可以用在项目上，或者对Redis上面的热点数据做区分。
- [x] 想想把这种冷热数据分离场景应用在自己的项目上。

- [x] 除了链表满了后会把最冷的数据踢回磁盘，后台线程也会定时去执行冷数据踢回磁盘的操作。

- [ ] 当热链表数据满了之后，是把缓存页数据移到冷链表上还是直接踢到磁盘？

- [ ] 当冷链表尾部上的数据，超过了1S，在次被访问的时候，会被移到冷链表的头部嘛？

<a name="ZzNCi"></a>
## 11-总结LRU，Flush，Free，Hash存储的作用
<a name="tA5OS"></a>
### 复盘一下各链表容器的作用
Free链表：存储空闲缓存页的描述信息，用于记录哪些缓存页可用。<br />Hash表：表空间名+缓存页号 作为Key，Value存储缓存页地址。判断数据是否已经缓存到Pool中。<br />Flush链表：记录被修改的缓存页号，用于刷回磁盘。避免那些未修改的数据被刷回缓存。<br />LRU链表：<br />冷链表：记录刚刚加载进来的缓存，和预读进来的缓存页。<br />热链表：记录1S内被多次访问的热点数据。当被修改后，也会被定时刷回磁盘。

<a name="rH9og"></a>
#### 各个链表之间的互动

- Free链表不足就去清空LRU冷尾节点的缓存页
- 定时刷盘则把增加Free节点的数据，减少Flush节点的数据。
- 数据加载到磁盘就增加LRU链表，HASH表，减少Free节点。

<a name="G8LQv"></a>
#### 被定时刷盘的链表

- 定时把LRU尾部刷入磁盘
- 定时把Flush数据刷入磁盘

<a name="xlsa5"></a>
### 解决热链表数据频繁移动的问题
> 在热数据区域，如果一个缓存页被访问，不一定会立马移动到热数据链表的表头，因为频繁的移动链表也是有性能消耗的，因此，MySQL中设计为，如果热数据链表前25%的缓存页被访问，他们是不会被移动的，只有在后75%中的缓存页被访问，才会移动到表头，这样就能尽可能的减少链表中节点的移动，从而减小性能的损耗


![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1635502347235-565c44eb-9f6c-4dca-a839-d19b3619c164.png#clientId=u6b283d15-a697-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=468&id=ucce20b6e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=936&originWidth=1268&originalType=binary&ratio=1&rotation=0&showTitle=false&size=688509&status=done&style=none&taskId=u6199ef02-db35-4304-adcb-e0fb5d0ece2&title=&width=634)

<a name="rqDim"></a>
## 12-配置多个BufferPool优化数据库性能
<a name="qm8qn"></a>
### 背景
前面介绍过BufferPool里面部门组件，如LRU链表，Flush链表，Free链表，Hash表，的作用和实现。<br />但是当高并发下为了保证数据安全，就必须的上锁，这样就势必导致性能下降，于是数据库在这样的背景下，就诞生了可以设置多个BufferPool缓存的情况。
<a name="JltHq"></a>
### 内存配置
BufferPool一般默认内存设置 1G。<br />所有当是大内存服务器时，可以多设置几个Pool，通常8G内存下，会设置4个Pool。
> innodb_buffer_pool_size = 8589934592
> innodb_buffer_pool_instances = 4

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1635845981247-361e2afe-3ebd-4b29-8c82-b4fd5d577d50.png#clientId=u4e1d484e-92c4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=272&id=ub4966a65&margin=%5Bobject%20Object%5D&name=image.png&originHeight=544&originWidth=1214&originalType=binary&ratio=1&rotation=0&showTitle=false&size=620232&status=done&style=none&taskId=uc9a3384a-907a-4e3c-b5b8-27250d326ec&title=&width=607)<br />假设有32G内存的机器，一般设置BufferPool只会设置20多G，会留几个G给OS系统使用。<br />一般是：32G ，设置20G。  128G内存会设置80G的buffer。<br />常用公式：<br />BufferPool总占用量 = （chunk内存 * bufferPool数量 ）* 的2倍。

<a name="XYi6h"></a>
### MySQL配置命令以及分析
<a name="OrgzU"></a>
#### 查看配置命令：
SHOW ENGINE INNODB STATUS
<a name="Tef1A"></a>
#### 结果分析：
> Total memory allocated xxxx;
> Dictionary memory allocated xxx
> Buffer pool size   xxxx
> Free buffers       xxx
> Database pages     xxx
> Old database pages xxxx
> Modified db pages  xx
> Pending reads 0
> Pending writes: LRU 0, flush list 0, single page 0
> Pages made young xxxx, not young xxx
> xx youngs/s, xx non-youngs/s
> Pages read xxxx, created xxx, written xxx
> xx reads/s, xx creates/s, 1xx writes/s
> Buffer pool hit rate xxx / 1000, young-making rate xxx / 1000 not xx / 1000
> Pages read ahead 0.00/s, evicted without access 0.00/s, Random read ahead 0.00/s
> LRU len: xxxx, unzip_LRU len: xxx
> I/O sum[xxx]:cur[xx], unzip sum[16xx:cur[0]


下面我们给大家解释一下这里的东西，主要讲解这里跟buffer pool相关的一些东西。<br />（1）Total memory allocated，这就是说buffer pool最终的总大小是多少<br />（2）Buffer pool size，这就是说buffer pool一共能容纳多少个缓存页<br />（3）Free buffers，这就是说free链表中一共有多少个空闲的缓存页是可用的<br />（4）Database pages和Old database pages，就是说lru链表中一共有多少个缓存页，以及冷数据区域里的缓存页数量<br />（5）Modified db pages，这就是flush链表中的缓存页数量<br />（6）Pending reads和Pending writes，等待从磁盘上加载进缓存页的数量，还有就是即将从lru链表中刷入磁盘的数量、即将从flush链表中刷入磁盘的数量<br />（7）Pages made young和not young，这就是说已经lru冷数据区域里访问之后转移到热数据区域的缓存页的数<br />量，以及在lru冷数据区域里1s内被访问了没进入热数据区域的缓存页的数量<br />（8）youngs/s和not youngs/s，这就是说每秒从冷数据区域进入热数据区域的缓存页的数量，以及每秒在冷数据区域里被访问了但是不能进入热数据区域的缓存页的数量<br />（9）Pages read xxxx, created xxx, written xxx，xx reads/s, xx creates/s, 1xx writes/s，这里就是说已经读取、创建和写入了多少个缓存页，以及每秒钟读取、创建和写入的缓存页数量<br />（10）Buffer pool hit rate xxx / 1000，就是说每1000次访问，有多少次是直接命中了buffer pool里的缓存的<br />（11）young-making rate xxx / 1000 not xx / 1000，每1000次访问，有多少次访问让缓存页从冷数据区域移动到了热数据区域，以及没移动的缓存页数量<br />（12）LRU len：这就是lru链表里的缓存页的数量<br />（13）I/O sum：最近50s读取磁盘页的总数<br />（14）I/O cur：现在正在读取磁盘页的数量

<a name="A5Agi"></a>
#### Innodb分析

- 关注的是free、lru、flush几个链表的数量的情况，然后就是lru链表的冷热数据转移的情况，然后你的缓存
- 页的读写情况，这些代表了你当前buffer  pool的使用情况。
- 最关键的是两个东西，一个是你的buffer pool的千次访问缓存命中率，这个命中率越高，说明你大量的操作都是直接基于缓存来执行的，性能越高。


<a name="e1zc8"></a>
## 13-chunk支持运行期间BufferPool动态大小调整
在本专栏介绍里，为了考虑MySQL的BufferPool在运行期间，方便动态调整，引入了chunk的概念，其实就是多个数据缓存页，组成了一个chunk，一个BufferPool 里面包含多个chunk快。<br />目的就是把BufferPool拆分成小块，方便调整。
<a name="MMrE5"></a>
### 数据模型
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1635854738244-bbc488d6-678b-4b6a-b1c3-7c6f485a0b64.png#clientId=u4e1d484e-92c4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=445&id=u42da3ff8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=544&originWidth=734&originalType=binary&ratio=1&rotation=0&showTitle=false&size=362078&status=done&style=none&taskId=u1912d5d8-7aed-4928-83a7-788fc11a3e3&title=&width=599.98291015625)

<a name="TqfCI"></a>
## 14-行数据在磁盘上的存储
<a name="xHMMp"></a>
### 对数据页中的每一行数据，在磁盘上是怎么存储的呢？
建表语句：
> CREATE TABLE table_name (columns) ROW_FORMAT=COMPACT
> ALTER TABLE table_name ROW_FORMAT=COMPACT

可以在建表的时候，就指定一个行存储的格式，也可以后续修改行存储的格式。这里指定了一个COMPACT行存储格式，在这种格式下，每一行数据他实际存储的时候，大概格式类似下面这样：<br />变长字段的长度列表，null值列表，数据头，column01的值，column02的值，column0n的值......<br />对于每一行数据，他其实存储的时候都会有一些头字段对这行数据进行一定的描述，然后再放上他这一行数据每一列的具体的值，这就是所谓的行格式。除了COMPACT以外，还有其他几种行存储格式，基本都大同小异。


<a name="POOeY"></a>
### varchar这种变长字段，在磁盘文件中是怎么存储的？
为了解决每个字段的不同长度问题，在磁盘文件中会在每行的 列值前面，增加一个长度描述信息，如下：
> 0x05 null值列表 数据头 hello a a 0x02 null值列表 数据头 hi a a

这个0x05就是一个16进制的数字，代表该所存储的长度为多。

<a name="zc8kf"></a>
### 一行数据中有多个变长字段的时候，如何存放他们的长度？
比如一行数据有VARCHAR(10) VARCHAR(5) VARCHAR(20) CHAR(1) CHAR(1)，一共5个字段，其中三个是变长字段，此时假设一行数据是这样的：hello hi hao a a<br />这时：存储的结构是: 逆序存储？
> 各变长字段数据占用的字节数**按照列的顺序逆序存放**，我们再次强调一遍，是**逆序存放**！

具体原因有点复杂，参考博客：https://blog.csdn.net/jiang18238032891/article/details/108475393


<a name="kJIWZ"></a>
### 为什么行数据里的Null值要放Null值列表统一存储？
因为本来就是 null 值，没必要去按照字符串的方式存放在磁盘上，浪费空间。直接放Null值列表里统一管理。<br />而Null列表值是以二进制bit位存储，可以减少空间。<br />对于允许为NULL的字段都会有一个bit位标识那个字段是否为NULL，也是逆序排列的。


<a name="G1uJD"></a>
### 磁盘上的一行数据如何读取出来？

- 变长度字段：先读取该字段值的长度，然后读出来。
- 不变长度字段：直接按照长度读取出来。
- Null值字段：去null列表里面读出来。

<a name="EmynB"></a>
### 磁盘中一行数据包含那些结构？
主要是这三块信息：<br />**非null行数据，null列表，40bit的数据头**

<a name="SLYfy"></a>
#### 1.非null行数据：

   - 不变长字段值：例如 char 这种类型的值
   - 变长字段值：变长字段，例如varchar这种类型的值
   - 字段长度描述信息块： 跟着变长字段走，用于记录变长字段的实际长度。

<a name="CKLqm"></a>
#### 2.null列表：
bit存储结构，用于存储该列运行为null的值，是否为null，为null则表示1.

<a name="vzvG1"></a>
#### 3.数据头占用40bit： 
用于描述该行数据，里面每个bit都是有意义的，具体如下。   （看看就好）

   - 第一个bit位和第二个bit位，都是预留位，没有意义。
   - 接下一个bit是delete_mask，代表数据是否删除，因为删除后，未必是立马从磁盘上删掉的。
   - 下一个bit位是min_rec_mask，B+树里每一层的非叶子节点里的最小值都有这个标记。
   - 接下来有4个bit位是n_owned，记录数的作用，后面会将。
   - 接着有13个bit位是heap_no，他代表的是当前这行数据在记录堆里的位置。
   - 然后是3个bit位的record_type，这就是说这行数据的类型。  那有那些行数据类型呢？如下：
      - 0代表的是普通类型，1代表的是B+树非叶子节点，2代表的是最小值数据，3代表的是最大值数据
   - 最后是16个bit的next_record，这个是指向他下一条数据的指针。


<a name="ogWpN"></a>
### 行数物理存储，为什么会存在行溢出？
大概意思是：一个数据页大概是16KB，但是有些宽表会直接导致一行数据存储超过16KB，这样一个缓存页存放不下，此时，这行数据里头会有个20Bit大小的指针，指向另一块缓存页用于存储。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636019485513-0293ea4d-6bf8-41b6-8d82-c91f9eb38d70.png#clientId=ucf0d2c60-82d5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=353&id=u796e51d8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=562&originWidth=743&originalType=binary&ratio=1&rotation=0&showTitle=false&size=302427&status=done&style=none&taskId=u8e6b0275-cde5-432e-9181-d4b28348d44&title=&width=466.65623474121094)

<a name="utCX4"></a>
## 15-表空间，数据区，又是什么概念

- 物理层面，表空间就是对应一些磁盘上的数据文件。  当一个表空间里包含的数据页太多的时候，不利于管理，所以在表空间里又引入了一个 “数据区” 的概念。
- 一个数据区 对应 64个数据页 ，每个数据页16kb, 所以一个数据区市1mb, 
- 256个数据区，划分为一组。
- 他的第一组数据区的第一个数据区的前3个数据页，都是固定的，里面存放了一些描述性的数据。

> 平时创建的那些表都是有对应的表空间的，每个表空间就是对应了磁盘上的数据文件，在表空间里有很多组数据区，一组数据区是256个数据区，每个数据区包含了64个数据页，是1mb。
> 表空间的第一组数据区的第一个数据区的头三个数据页，都是存放特殊信息的，属于描述信息。


<a name="tfFOw"></a>
## 16-初步总结Mysql存储模型以及数据读写机制

<a name="oNgri"></a>
### 数据在磁盘文件里是怎么存放的呢？
**数据区分组** （1组256个数据区）<br />**数据区**		（1个数据区256个数据页）<br />**数据页**		（一个数据页 16K的行数据）

- 数据插入的表，表只是一个逻辑概念，其实在物理层面，它对应的是表空间这个概念。
- 表空间就对应着磁盘文件，磁盘文件里存放这数据。
- 磁盘文件里存放的数据，从最基本的角度来看，就是被拆分为一个一个的数据区（extent）分组。
- 每个数据区分组，包含256个extent，每个extent由64个数据页组成。
<a name="wCaCs"></a>
### 结构图：
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636099890600-f801214f-fb7b-4e38-bf9e-33a9a6f33381.png#clientId=uf60492ba-a2b4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=638&id=aq5Zn&margin=%5Bobject%20Object%5D&name=image.png&originHeight=751&originWidth=416&originalType=binary&ratio=1&rotation=0&showTitle=false&size=261178&status=done&style=none&taskId=ub9c85b42-deec-4979-b17f-68cb69c549a&title=&width=353.65625)

<a name="aQ9lU"></a>
## 17-数据文件在操作系统层面的原理
<a name="Pz3Hr"></a>
### 数据库的日志顺序读写，以及数据文件随机读写的原理

- 写RedoLog ，或Binlog日志的时候，就是在一个 日志文件 末尾追加日志，这就是磁盘顺序写。性能高。
- 随机在磁盘上读取数据页，这就是随机读取。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636340457894-3f2e84c6-8446-4ffb-9eac-1a2ff3077ed3.png#clientId=uf60492ba-a2b4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=238&id=u69d2a75d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=337&originWidth=592&originalType=binary&ratio=1&rotation=0&showTitle=false&size=135027&status=done&style=none&taskId=ufc1f39fb-21c9-44b7-9df5-fc079788344&title=&width=417.3295440673828)<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636340346770-981b29ab-ed8c-422c-9abc-286dedc6e9ab.png#clientId=uf60492ba-a2b4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=265&id=u853c2a96&margin=%5Bobject%20Object%5D&name=image.png&originHeight=377&originWidth=629&originalType=binary&ratio=1&rotation=0&showTitle=false&size=189869&status=done&style=none&taskId=u5e6ea0f2-a47b-41b1-b476-ff2bc3a6e8a&title=&width=442.647705078125)


<a name="kPWms"></a>
### Linux文件系统的底层实现
这就是MySQL跟Linux存储系统交互的的一个原理剖析，包括里面的IO调度算法那块的一个优化的点。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636350746460-a507c0af-8f75-45ad-9dc5-4661c847abf1.png#clientId=uf60492ba-a2b4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=470&id=u36ab367a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=703&originWidth=643&originalType=binary&ratio=1&rotation=0&showTitle=false&size=309848&status=done&style=none&taskId=udaa44a73-e041-42b5-9a77-f8b88dd02a8&title=&width=430.3210144042969)

<a name="aKS3U"></a>
### 数据库服务器RAID存储架构初步介绍
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636351355968-8ac492b7-ddad-4af5-8555-ecb9d3394efa.png#clientId=uf60492ba-a2b4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=303&id=ube5b5fe0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=359&originWidth=434&originalType=binary&ratio=1&rotation=0&showTitle=false&size=155823&status=done&style=none&taskId=u2905dc24-3642-400d-9de8-01258f6f8bd&title=&width=366.6505432128906)<br />RAID就是一个磁盘冗余阵列，大概理解为：管理机器多块磁盘的一种 磁盘阵列 技术。<br />方便在多个磁盘时，告诉你读取那个磁盘文件。<br />RAID技术很重要的一个作用：可以实现 **数据冗余机制 **<br />了解一下RAID这种多磁盘冗余阵列技术的基本思想、

<a name="Gthqj"></a>
#### 什么是数据冗余机制？
现在写入一批数据在RAID中的一块磁盘上，这块磁盘坏了，便无法读取。所以RAID机冗余制会在两块数据磁盘里写入相同的数据，放置另一块坏了，就相当于 冗余备份。这是RAID自己管理的，不用我们担心。

<a name="fYbGQ"></a>
### RAID存储架构的电池充放电原理
服务器使用多块磁盘组成的RAID阵列的时候，一般会有一个RAID卡，这个RAID卡是带有一个缓存的，这个缓存不是直接用我们的服务器的主内存的那种模式，他是一种跟内存类似的SDRAM，当然，你大致就认为他也是基于内存来存储的。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636351610398-2a0d6817-1234-440e-8b02-9bb8bd1c65b2.png#clientId=uf60492ba-a2b4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=319&id=uf13b07e6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=313&originWidth=374&originalType=binary&ratio=1&rotation=0&showTitle=false&size=124004&status=done&style=none&taskId=u57ef78d1-6dc1-4f0c-8e6b-b2a3fd25c68&title=&width=381.6505432128906)
<a name="ybAm9"></a>
#### 如果服务器突然断电，SDRAM缓存数据怎么办？
RAID卡里面会自带一个锂电池，或者电容，当服务器没电的时候，会提供电源，把缓存数据写到磁盘上。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636351802413-3bb1360b-f91c-4204-aa02-1c07a5705324.png#clientId=uf60492ba-a2b4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=268&id=u81ea1fc0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=317&originWidth=447&originalType=binary&ratio=1&rotation=0&showTitle=false&size=150040&status=done&style=none&taskId=ud9f918be-2b3d-402d-afdf-5aeeebfdc3f&title=&width=377.9801025390625)
<a name="sMNuq"></a>
## 18-生产事故案例分析
<a name="Zxhlu"></a>
### 数据库30天会又一次性能抖动，性能下降的分析排查
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636351997885-a7c4c1a9-821d-4de6-80db-0797a63710ca.png#clientId=uf60492ba-a2b4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=360&id=u3e711877&margin=%5Bobject%20Object%5D&name=image.png&originHeight=288&originWidth=555&originalType=binary&ratio=1&rotation=0&showTitle=false&size=241466&status=done&style=none&taskId=uce9d472c-4f02-4751-8154-7549ac700ee&title=&width=692.991455078125)
<a name="LUeJa"></a>
### 数据库无法建立连接：Too many connections
背景：在高配置的MySQL服务器下，发现服务器连接不上。
<a name="J0xgg"></a>
#### 排查过程：
查看Java系统SQL连接池的最大数也才400.<br />查看MySQL中my.cnf文件中最大的连接数设置800<br />查看实际连接数，只能到214就无法连接，远远超过设定的数目
> show variables like 'max_connections'

<a name="NPxDd"></a>
#### 发现问题：
最后检查MySQL启动日志，linux操作系统把进程可以打开的文件句柄数限制为了1024了，导致<br />MySQL最大连接数是214！<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636352384563-61fe94d0-effa-4be8-9b6a-f789c3856435.png#clientId=uf60492ba-a2b4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=259&id=u9627c000&margin=%5Bobject%20Object%5D&name=image.png&originHeight=237&originWidth=487&originalType=binary&ratio=1&rotation=0&showTitle=false&size=93461&status=done&style=none&taskId=uaf89e687-e682-4e91-8fc1-3b083c7fc49&title=&width=532.3295440673828)<br />命令：<br />ulimit -HSn 65535<br />然后就可以用如下命令检查最大文件句柄数是否被修改了<br />cat /etc/security/limits.conf<br />cat /etc/rc.local

因为linux操作系统的设计，要尽量避免某个进程一下子耗尽机器上的所有资源，所以默认都会做限制。

简而言之：如果是自己的服务器搭载的MySQL服务，要注意操作系统为MySQL进程分配的文件操作句柄数，有可能操作系统对你操作文件的并发数做了限制，导致连接失败。

<a name="QH2Fx"></a>
## 20-回顾Redo日志对于事物提交后，数据不丢失的意义
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636363257430-32308450-9dee-4f7f-ad22-9adbefc43a66.png#clientId=uf60492ba-a2b4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=335&id=u425c84ab&margin=%5Bobject%20Object%5D&name=image.png&originHeight=308&originWidth=507&originalType=binary&ratio=1&rotation=0&showTitle=false&size=134792&status=done&style=none&taskId=u590e7585-7375-4e09-a50a-4241b0d8f57&title=&width=551.9857788085938)<br />**为什么缓存数据要写到磁盘，Redo数据也同样是写到磁盘，为啥多次一举呢？**<br />因为Redo数据记录的数据量很小，通常只有几十个字节，而一个缓存数据页写到磁盘，是16KB，内存比较大，受影响比较多，所以效率低，不直接存放磁盘。

<a name="t19hf"></a>
### BufferPool执行CRUD之后，写入的RedoLog日志长啥样？
<a name="avQ4Q"></a>
#### redo log里记录的本质：
本质就是，记录的就是在对某个表空间的某个数据页的某个偏移量的地方修改了几个字节的值，具体修改的值是什么，他里面需要记录的就是表空间号+数据页号+偏移量+修改几个字节的值+具体的值。

<a name="UAm1T"></a>
### RedoLog写入的方式
<a name="p9UES"></a>
#### redo log block （RedoPool日志存储结构）
MySQL内有另外一个数据结构，叫做**redo log block**，大概你可以理解为，平时我们的数据不是存放在数据页了的么，用一页一页的数据页来存放数据。<br />那么对于redo log也不是单行单行的写入日志文件的，他是用一个redo log block来存放多个单行日志的。<br />一个redo log block是512字节，这个redo log block的512字节分为3个部分，一个是12字节的header块头，一个是496字节的body块体，一个是4字节的trailer块尾。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636364005331-6270e67b-4cb5-46ab-9da4-59613dc04a77.png#clientId=uf60492ba-a2b4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=288&id=u3ff79db4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=268&originWidth=397&originalType=binary&ratio=1&rotation=0&showTitle=false&size=90667&status=done&style=none&taskId=ua41e28d1-9ccf-45ee-aae3-726e7ec14ff&title=&width=427.32952880859375)<br />大概意思，就是把产生的redo Log日志往 redoLog back 容器里丢，当达到496个字节的时候，就把文件存储到磁盘上。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636364176957-de5f897e-83b6-42e7-a08f-b5cc78c0ec15.png#clientId=uf60492ba-a2b4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=290&id=u6e6cc2f9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=255&originWidth=463&originalType=binary&ratio=1&rotation=0&showTitle=false&size=96674&status=done&style=none&taskId=u9a79180b-33ec-48e0-9e05-d21968fbdf2&title=&width=526.3238525390625)

<a name="skEiH"></a>
#### Redo Log Buffer Pool 日志缓存区

- MySQL在启动的时候，就跟操作系统申请的一块连续内存空间，和BufferPool加载机制差不多。
- redo log buffer也是类似的，他是申请来的一片连续内存，然后里面划分出了N多个空的redo log block
- 如下图所示：
- ![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636364797909-92c98d2c-992e-4351-8b60-f7aef76345c8.png#clientId=uf60492ba-a2b4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=315&id=ufeffd746&margin=%5Bobject%20Object%5D&name=image.png&originHeight=264&originWidth=576&originalType=binary&ratio=1&rotation=0&showTitle=false&size=130149&status=done&style=none&taskId=u8d2a3665-6648-4f4b-905c-7f4f5170932&title=&width=687.9801025390625)
- 其实在我们平时执行一个事务的过程中，每个事务会有多个增删改操作，那么就会有多个redolog，这多个redo log就是一组redo log。
- 其实每次一组redo log都是先在别的地方暂存，然后都执行完了，再把一组redo
- log给写入到redo log buffer的block里去的。如果一组redo log实在是太多了，那么就可能会存放在两个redo log block中

<a name="nXgc4"></a>
### RedoLog buffer 中的缓存日志，什么时候写入磁盘？
写入redo log buﬀer的时候，是写入里面提前划分好的一个一个的redo log block的，选择有空闲空间的redo log block去写入，然后redo log block写满之后，其实会在某个时机刷入到磁盘里去，
<a name="QIr5V"></a>
#### 触发Redo Pool 写入磁盘的四种场景

1. 如果写入redo log buﬀer的日志已经占据了redo log buﬀer总容量的一半了，也就是超过了8MB的redo log在缓冲里了，此时就会把他们刷入到磁盘文件里去。  （容量预判刷盘）

2. 一个事务提交的时候，必须把他的那些redo log所在的redo log block都刷入到磁盘文件里去，只有这样，当事务提交之后，他修改的数据绝对不会丢失，因为redo log里有重做日志，随时可以恢复事务做的修改。

（事务提交刷盘）

3. 后台线程定时刷新，有一个后台线程每隔1秒就会把redo log buﬀer里的redo log block刷到磁盘文件里去。

（后台定时刷盘）

4. MySQL关闭的时候，redo log block都会刷入到磁盘里去。	（MySQL关闭刷盘）

<a name="N5VXS"></a>
## 21-回顾undo log 回滚日志原理
使用场景，是在遇到事务需要回滚的时候，undo日志去是实现复原的日志。比如你的语句是insert操作，而undo记录的就是 delete操作。

<a name="AFCDi"></a>
### undolog 日志长啥样？
<a name="pFQ3X"></a>
#### INSERT语句的undo log

   - undo log的类型是TRX_UNDO_INSERT_REC
   - 这个undo log里包含了以下一些东西：
      - 这条日志的开始位置
      - 主键的各列长度和值
      - 表id
      - undo log日志编号           //  每个undo log日志都是有自己的编号的
      - undo log日志类型		// insert 类似所属的类型
      - 这条日志的结束位置	
- ![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636371114622-01da4cc4-2372-4dff-89da-ffdc4107c7b7.png#clientId=uf60492ba-a2b4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=53&id=u1eb667a4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=48&originWidth=518&originalType=binary&ratio=1&rotation=0&showTitle=false&size=34491&status=done&style=none&taskId=u02113325-4e0f-49bf-8480-d1a709efdbb&title=&width=571.647705078125)

<a name="aSMD6"></a>
# MySQL的事务和锁
<a name="tqJVA"></a>
## 22-MySQL事务相关
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636371241983-cc864823-6c22-4243-a0ef-23cad4852365.png#clientId=uf60492ba-a2b4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=298&id=uc90e2019&margin=%5Bobject%20Object%5D&name=image.png&originHeight=283&originWidth=636&originalType=binary&ratio=1&rotation=0&showTitle=false&size=159823&status=done&style=none&taskId=ufc2ad5ab-f667-4858-a3c1-73632c55a9a&title=&width=669.9772338867188)
<a name="rW74c"></a>
#### 多个事务并发更新查询，为什么会有脏写，脏读问题?
无论是脏写还是脏读，都是因为一个事务去更新或者查询了另一个还没提交的事务更新过的数据。<br />因为另外一个事务还没提交，所以他随时可能会反悔会回滚，导致你更新的数据，或者查询的数据没有了。

<a name="apAG9"></a>
#### 不可重复读和幻读产生的问题？
一个事务多次查询一条数据读到的都是不同的值。这个过程中可能别的事务会修改这条数据的值，而且修改值之后事务都提交了，结果导致人家每次查到的值都不一样，都查到了提交事务修改过的值，这就是所谓的不可重复读。<br />幻读也是，同一条SQL读出了不同的值。

所以这些问题的本质，都是数据库的多事务并发问题，那么为了解决多事务并发问题，数据库才设计了<br />事务隔离机制、MVCC多版本隔离机制、锁机制，用一整套机制来解决多事务并发问题，接下来，我们将要深入讲解这些机制，让大家彻底能够理解数据库内部的执行原理。

<a name="XYbKY"></a>
#### 四种隔离级别：
RU：读未提交：人家没提交事务，你也能读到，会读到脏数据。<br />RC：读已提交：只能读到人家提交事务后的数据，可能会导致幻读，不可重复读等现象。<br />RR：可重复读  ：MVCC 多版本机制，别提交了事务你也读不到，避免幻读。MySQL默认机制<br />SE：serializable 可串行话： 就是不允许多并发进行，只能串行操作。非常严谨的机制，一般不用。

<a name="tiGvZ"></a>
#### 脏读，幻读，脏写，不可重复读 要了解啥意思？
脏写：就是两个事务都更新一个数据，结果有人回滚了把另外一个人更新的数据也回滚没了。<br />脏读：一个事务读到了另外一个事务没提交时修改的数据，结果另外一个事务回滚了，下次读就读不到了。<br />不可重复读：就是多次读一条数据，别的事务老是修改数据值还提交了，多次读到的值不同。<br />幻读：就是范围查询，每次查到的数据不同，有时候别的事务插入了新的值，就会读到更多的数据。

<a name="hk1Q1"></a>
#### Spring注解设置事务隔离级别
在@Transactional注解里是有一个isolation参数的，里面是可以设置事务隔离级别的，具体的设置方式如下：
> @Transactional(isolation=Isolation.DEFAULT)，然后默认的就是DEFAULT值，这个就是MySQL默认支
> 持什么隔离级别就是什么隔离级别。


<a name="WjdK8"></a>
#### 基于undo log多版本链条实现的ReadView机制
注意：！！！<br />大家先不管多个事务并发执行是如何执行的，起码先搞清楚一点，就是多个事务串行执行的时候，每个人修改了一行数据，都会更新隐藏字段txr_id和roll_pointer，同时之前多个数据快照对应的undo log，会通过roll_pinter指针串联起来，形成一个重要的版本链！<br />就是这个多个事务串行更新一行数据的时候，txr_id和roll_pinter两个隐藏字段的概念，包括undo log串联起来的多版本链条的概念！

通过undo log多版本链条，加上你开启事务时候生产的一个ReadView，然后再有一个查询的时候，根<br />据ReadView进行判断的机制，你就知道你应该读取哪个版本的数据。<br />而且他可以保证你只能读到你事务开启前，别的提交事务更新的值，还有就是你自己事务更新的值。假如说是你事务开启之前，就有别的事务正在运行，然后你事务开启之后 ，别的事务更新了值，你是绝对读不到的！或者是你事务开启之后，比你晚开启的事务更新了值，你也是读不到的！

<a name="j8WJa"></a>
#### MVCC机制的底层实现
简单来说，记住一点，Undo Log 的readview 机制 和MVCC 多版本控制的原理差不多，都是用来实现可重复实现某个东西的机制。<br />MySQL实现MVCC机制的时候，是基于**undo log多版本链条**+**ReadView**机制来做的

<a name="D2QG1"></a>
#### 读已提交 和 可重复读 的隔离级别是用MVCC如何基于ReadView实现的？
RC隔离级别如何实现的，他的关键点在于每次查询都生成新的ReadView（可理解为快照），那么如果在你这次查询之前，有事务修改了数据还提交了，你这次查询生成的ReadView里，那个m_ids列表当然不包含这个已经提交的事务了，既然不包含已经提交的事务了，那么当然可以读到人家修改过的值了。


<a name="Kd4AK"></a>
## 23- MySQL锁相关
<a name="RkVn5"></a>
### 多个数据更新同一条数据时，如何避免脏写产生？
加锁！ 实现更新一行数据串行化处理。

<a name="LaxC7"></a>
### 共享锁和独占锁到底是什么？
<a name="pDUod"></a>
#### 共享锁：
实现的语法：<br />select * from table lock in share mode，<br />你在一个查询语句后面加上lock in share mode，意思就是查询的时候对一行数据加共享锁。

<a name="IEchk"></a>
#### 独占锁：
实现的语法：<br />查询操作还能加互斥锁，他的方法是：select * from table for update。<br />注意：如果查询过程中用了索引，就是行锁，没有用索引，就是表锁。

就是更新数据的时候必然加独占锁，独占锁和独占锁是互斥的，此时别人不能更新；但是此时你要查询，默认是不加锁的，走mvcc机制读快照版本，但是你查询是可以手动加共享锁的，共享锁和独占锁是互斥的，但是共享锁和共享锁是不互斥的，规律如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636374750865-c1538448-9a98-48e2-bb34-999b6d58dbfb.png#clientId=uf60492ba-a2b4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=115&id=ueef41597&margin=%5Bobject%20Object%5D&name=image.png&originHeight=116&originWidth=607&originalType=binary&ratio=1&rotation=0&showTitle=false&size=31715&status=done&style=none&taskId=ub32cfd31-fce4-4709-aa3b-f44c9b5edd2&title=&width=601.3295288085938)<br />一般业务开发中，很少去对查询主动加 共享锁，因为本来查询时就遵循MVCC机制，默认是不加锁的。<br />哪怕需要用到这个也是基于Redis， ZK这种分布式锁去实现，这里知道有这么个东西就行了。

<a name="EBUxI"></a>
### DDL语句加的是元数据锁（可理解和表锁差不多）
比如alter table 之类的操作，会默认在表级别加锁，其实不对，虽然增删改操作 会和你的DDL语句发生阻塞。<br />但是内部是通过 元数据锁 metadata locks 来实现的，而不是一个表锁概念，表锁是基于Innodb 引擎层面的。

<a name="ebu4s"></a>
### 如果要尽量避免缓存页ﬂush到磁盘可能带来的性能抖动问题？
<a name="m39ug"></a>
#### 两个核心理论点：
1.减少缓存页flush链表到磁盘的频率。  <br />2.增加flush链表刷到磁盘的速度。
<a name="tjKB9"></a>
#### 两个解决办法：
1.bufferPool扩大内存，导致填满的速度慢一些，刷盘频率降低。<br />2.服务器硬件别用机械，采用固态硬盘，提高磁盘IOPS。

就是MySQL性能随机抖动的问题，最核心的就是把innodb_io_capacity设置为SSD固态硬盘的IOPS，让他刷缓存页尽量快，同时设置innodb_ﬂush_neighbors为0，让他每次别刷临近缓存页，减少要刷缓存页的数量，这样就可以把刷缓存页的性能提升到最高。

<a name="QGvGM"></a>
# MySQL的索引
<a name="Oqq0b"></a>
## 24-索引原理和概念
<a name="VtyDf"></a>
### 磁盘数据页的存储结构
是不是就理解了一个磁盘文件里的多个数据页是如何通过指针组成一个双向链表的！<br />然后一个数据页内部会存储一行一行的数据，也就是平时我们在一个表里插入的一行一行的数据就会存储在数据页里，然后数据页里的每一行数据都会按照主键大小进行排序存储，同时每一行数据都有指针指向下一行数据的位置，组成单向链表，如下图。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636376446182-d46e5142-fb3d-4356-9477-30b02b08ca29.png#clientId=uf60492ba-a2b4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=276&id=u2e4f3403&margin=%5Bobject%20Object%5D&name=image.png&originHeight=271&originWidth=522&originalType=binary&ratio=1&rotation=0&showTitle=false&size=148759&status=done&style=none&taskId=u3fcaf12d-e407-47bf-925c-e45defe8f63&title=&width=531.9914703369141)
<a name="g6PMa"></a>
### 查询没有用索引是如何查询的？
数据页的组成是双向链表，然后挨个遍历链表上的数据页，加载到bufferPool里面。<br />如果你查询的里面有ID，主键，那就基于二分法去查找，如果是其他非索引字段，那就只能全表扫描了。

<a name="BfHro"></a>
### 插入数据，物理存储如何进行 页分裂 ？

- 数据页是用双向链表连接起来的，如果插入数据的时候，可能就要进行链断裂，分裂。
- 就是如果你有数据ID=100了，你突然插入一个 ID=88 的数据，那么就要 页分裂 了。影响效率
- 因为数据是按照ID 从小到大自增排列分布在数据页里面的。    （这也是使用UUID的坏处！！）

<a name="po7ZX"></a>
### 主键索引是如何设计的？如何根据主键索引查询？
所以其实此时就需要针对主键设计一个索引了，针对主键的索引实际上就是主键目录，这个主键目录呢，就是把每个数据页的页号，还有数据页里最小的主键值放在一起，组成一个索引的目录。如下图：<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636377336763-31ece20a-2d70-4235-ae5a-be5415735b91.png#clientId=uf60492ba-a2b4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=313&id=ua6bfa6fd&margin=%5Bobject%20Object%5D&name=image.png&originHeight=313&originWidth=717&originalType=binary&ratio=1&rotation=0&showTitle=false&size=182273&status=done&style=none&taskId=u76facde3-d54a-4404-a29c-b0a8b0c8a19&title=&width=716.9857788085938)

<a name="brOJi"></a>
### 主键索引的页存储物理结构，是如何用B+树来实现的？
大概演变的过程是这样的：<br />数据量多的时候，不可能让你的 主键目录里存放大量的 数据页和最小最大主键。<br />于是，向上在包一层，在索引页面的上一层在套一层，然后套着套着就成了一颗树。<br />最后根据ID去筛选 数据的时候，就用二分法，从上到下开始比较ID大小，知道找到最终数据页。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636378307332-2fb81eea-747c-481f-9ad9-04d6a0c659d9.png#clientId=uf60492ba-a2b4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=344&id=u46ca154f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=210&originWidth=228&originalType=binary&ratio=1&rotation=0&showTitle=false&size=36718&status=done&style=none&taskId=u5a6d6485-cd62-4bc3-b5c8-ae019cdc3f6&title=&width=372.9801025390625)<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636378331740-be0d3fef-80e0-4765-a6ed-e2a05cc4cb4e.png#clientId=uf60492ba-a2b4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=330&id=u69a4520e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=164&originWidth=209&originalType=binary&ratio=1&rotation=0&showTitle=false&size=38378&status=done&style=none&taskId=ua03e0b55-2fa3-4579-8f2a-caac43997fe&title=&width=420.6590881347656)<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636378339513-f82efc2f-b6f2-4a81-97dd-dfd6d7e69e6c.png#clientId=uf60492ba-a2b4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=251&id=u5d916fc1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=153&originWidth=272&originalType=binary&ratio=1&rotation=0&showTitle=false&size=29029&status=done&style=none&taskId=u5a67518f-bbaf-4d30-b663-a641a38e08e&title=&width=445.65340423583984)<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636378623678-d35af88d-3fce-4d85-bf23-73e0f61d9a87.png#clientId=uf60492ba-a2b4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=292&id=u16b83363&margin=%5Bobject%20Object%5D&name=image.png&originHeight=230&originWidth=591&originalType=binary&ratio=1&rotation=0&showTitle=false&size=154019&status=done&style=none&taskId=ubc88c56c-a6d0-4c01-aaad-a99d941aeb7&title=&width=750.9801025390625)
<a name="HRUtb"></a>
### 聚簇索引是啥？更新数据时，咋维护？
 聚簇索引  == 主键索引 

- 大概意思就是：  索引页 +  数据页 组成的B+ 树，就是聚簇索引。  （默认建立的 **主键 + 行数据 **的索引）
- 和普通索引的区别就是，多了一些指向数据页的指针。 里面存的是完整的行数据。
- 注意：区别就是最下的叶子节点，多了一些指针，指向索引页 和 数据页。 （范围查询也和这些指针有关）

如图：<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636379060498-1280048f-58ba-4213-8acf-5ee102f19cd0.png#clientId=uf60492ba-a2b4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=324&id=u43036ee4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=370&originWidth=874&originalType=binary&ratio=1&rotation=0&showTitle=false&size=292645&status=done&style=none&taskId=u92b0da47-8d8e-4bd8-90c4-ec5b9c39f4e&title=&width=764.3210144042969)<br />注意：<br />如果数据量很多，索引页很多，会在搞一个上层索引页，上层索引页就是存放下层索引页号和最小主键值。<br />按照这个顺序，可能就会多出更多的索引页层级来，其实上亿级别的表数据，也就三四层而已，不用担心树的高度。<br />这个聚簇索引是innodb默认按照主键来组织的，所以你在增删改数据的时候，一方面会更新数据页，一方面其实会给你自动维护B+树结构的聚簇索引，给新增和更新索引页，这个聚簇索引是默认就会给你建立的。<br />而且我们表里的数据就是直接放在聚簇索引里的，作为叶子节点的数据。

<a name="Qc6uq"></a>
### 主键之外的二级索引原理
<a name="RUWdU"></a>
#### 主键之外的二级索引，又是如何运作的？
二级索引，会根据 被建索引的name字段，建立一棵单独的B+ 树。如果是联合索引，也是如此。

- [ ] 这里有个问题是，name 字段是根据什么来排序的？

**二级索引结构**<br />和上面聚簇索引的结构差不多，不同的是最底层的叶子结点指向的数据页，数据页里面存放的是行数据的ID 和 当前 索引字段 name。 

**回表聚簇索引的过程**<br />当二级索引检索到最下面的叶子节点，拿到行数据的ID时，然后 回表 去上面那个聚簇索引二分查找，根据ID找到最终的 聚簇索引下的叶子结点，然后拿到指向的数据页，此时就是全行数据了。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636380089259-d71bd475-d75a-4186-8663-4bdc04937636.png#clientId=uf60492ba-a2b4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=283&id=u1ba590c6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=292&originWidth=736&originalType=binary&ratio=1&rotation=0&showTitle=false&size=191117&status=done&style=none&taskId=u587273cd-b6ec-4439-962a-f6bdc3f89d6&title=&width=714.3266906738281)


<a name="u22lt"></a>
## 25-索引问题和优化
<a name="ex6EF"></a>
### insert数据是如何维护好不同索引的B+树？
简单描述就是，开始添加数据的时候就是一个链表，串着很多 数据页，随着 数据页 的增加，然后向上升级一个 索引页 这个索引页存着最大的ID，和最小的ID。当这个索引页ID范围变大的时候，又向上升级裂变 产生一个索引页，然后反复几次，就变成了一棵树。
> 你的数据页越来越多，那么根页指向的索引页也会不停分裂，分裂出更多的索引
> 页，当你下层的索引页数量太多的时候，会导致你的根页指向的索引页太多了，此时根页继续分裂成多
> 个索引页，根页再次往上提上去去一个层级。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636464522493-77b7b817-80b6-4d02-ac39-1a710b6abbcf.png#clientId=udd648b61-6e9a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=279&id=u2e893393&margin=%5Bobject%20Object%5D&name=image.png&originHeight=251&originWidth=586&originalType=binary&ratio=1&rotation=0&showTitle=false&size=131073&status=done&style=none&taskId=ue6303baf-ebda-44cf-8640-4d8d6b0f792&title=&width=650.3266906738281)
<a name="pmI1F"></a>
### 一张表索引是不是越多越好？
不是的！<br />首先，普通索引是在 聚簇索引 的基础上copy出一棵独立的B+树，但是叶子结点指针指向的是 索引字段 （比如，orderNo) . 底层也是双向链表，也是需要耗费内存的。<br />另外，在数据增加，修改的时候，会对树的裂变，和维护，也是性能损耗。<br />其次，在MySQLServer层，有SQL查询优化器，就算建了很多索引也不一定用到，会有成本计算。<br />总之，SQL查询时，尽量避免回表操作。

**数据页内部本身也是一个单向链表。**

<a name="t5SFq"></a>
### 联合索引 查询原理以及全值匹配规则
对于联合索引而言，就是依次按照各个字段来进行二分查找，先定位到第一个字段对应的值在哪个页里，然后如果第一个字段有多条数据值都一样，就根据第二个字段来找，以此类推，一定能定位到某几条数据！ 

<a name="TiXru"></a>
### 最常见的索引使用规则

<a name="D3vFI"></a>
#### 最左侧列匹配
在联合索引中，使用查询条件时，是从左往右开始匹配，中间不能跳过字段去匹配。
<a name="Bctvh"></a>
#### 最左前缀匹配原则
MySQL 建立联合索引的规则是这样的，它会首先根据联合索引中最左边的、也就是第一个字段进行排序，在第一个字段排序的基础上，再对联合索引中后面的第二个字段进行排序，依此类推。<br />综上，第一个字段是绝对有序的，从第二个字段开始是无序的，这就解释了为什么直接使用第二字段进行条件判断用不到索引了（从第二个字段开始，无序，无法走 B+ Tree 索引）！这也是 MySQL 在联合索引中强调最左前缀匹配原则的原因。

<a name="LkQHt"></a>
#### 范围查找规则
就是你的where语句里如果有范围查询，那只有对联合索引里最左侧的列进行范围查询才能用到索引！<br />所以，需要范围查询的字段，在联合索引中一定要放到最后。

<a name="KRgS9"></a>
#### 等值匹配+范围匹配的规则
如果你要是用select * from student_score whereclass_name='1班' and student_name>'' and subject_name<'阿达'<br />那么此时你首先可以用class_name在索引里精准定位到一波数据，接着这波数据里的student_name都是按照顺序排列的，所以student_name>''也会基于索引来查找，但是接下来的subject_name<''是不能用索引的。

<a name="ehX5e"></a>
### order by: 当SQL排序时如何能使用索引？
通常字段通过索引筛选查询出来数据后，会放到内存里面排序返回。 <br />**fileSort排序**：但当数据量很大时，会把数据放 临时磁盘文件 里。然后排序，分页返回，这样速度超级慢。 <br />（内存排序，内存不够时就临时磁盘文件排序）

所以，可以用**联合索引解决问题。**<br />比如 name_age_class 建立联合索引，排序的时候，通过 order by name , age , class 去排序。<br />但是千万不要，在一个联合索引里，字段 有升序 和降序同时存在，必须排序一致。<br />因为：  在联合索引B+树里，本身对字段做了顺序排序，直接按B+树取出来就行，不需要内存中排序了。

<a name="bDxTh"></a>
### group by :当SQL分组聚合时，如何才能使用索引？
其实原理和上面order by 差不多。也是通过内存做分组，然后放不下就去磁盘分组，效率很低。<br />于是乎，也可以用到联合索引，通过联合索引，具有顺序性的原则去实现。

通常而言，对于group by后的字段，最好也是按照联合索引里的最左侧的字段开始，按顺序排列开来，就可完美的运用上索引来直接提取一组一组的数据，然后针对每一组的数据执行聚合函数就可以了。


<a name="GKG3U"></a>
### 回表的损害，覆盖索引是什么？
<a name="GxTEi"></a>
#### 覆盖索引：
索引覆盖查询：不是指一种索引，而是指索引的一种查询方式。<br />比如：你建立了 name_age_cass联合索引，你查询返回就只查这三个字段，那么她就直接扫描普通联合索引树就能拿到结果了，不需要去 聚簇索引 去回表查询 行中其他字段信息了。  这就是 索引覆盖查询。
<a name="ZFfSh"></a>
#### 不用 * 查询，和不分页查询
避免回表次数,能尽量在普通索引中返回最好不过了。

所以写SQL语句，你要注意也许你会用到联合索引，但可能会导致大量的回表到聚簇索引，如果需要回表到聚簇索引的次数太多了，可能就直接给你做成全表扫描不走联合索引了； 因为MySQL中的查询优化器会做成本比较，如果你本来就是要回表查询全量数据，何必去走那个索引树呢？ 还不如全表扫描。

同时在查询时要注意分页，因为走索引的时候，做分页，会告知查询优化器，你只要部分数据，避免MySQL直接不走索引去全表查询了


<a name="XhvoK"></a>
### 设计索引考虑因素
注意索引的长度，不要过长，，长的占用内存。<br />尽量提高索引的区分度，最好不要那种 state =0 或1 的字段。<br />尽量使用数字这种作为索引，不要使用varchar这种，占用内存大，而且不利于排序。<br />对于那种长字符串要作为索引的话，且前面区分度不高，可以选择截取后面部分长度作为索引。

索引在where条件中，不要使用函数。使用函数就失效了。<br />一个表中索引别太多，B+树分裂，会影响性能。<br />主键不要使用UUID之类的，没有顺序性，不方便游标分页，和B+树的顺序递增，会频繁分裂树。


<a name="m0ZCF"></a>
# MySQL实战和执行原理
<a name="FtU4G"></a>
## 26-社交APP的SQL索引实战
<a name="lY6D5"></a>
### 鱼与熊掌不可兼得
假设一个联合索引，age在最左侧，那你的where是可以用上索引来筛选的，但是排序是基于score字段，那就不可以用索引了。那假设你针对age和score分别设计了两个索引，但是在你的SQL里假设基于age索引进行了筛选，是没法利用另外一个score索引进行排序的。<br />所以说，你要明白的第一个难题就是，你的where筛选和order by排序实际上大部分情况下是没法都用到索引的！所谓鱼与熊掌不可兼得，就是这个意思！

<a name="cFz0I"></a>
### 热点数据放联合索引前面
比如那些，城市，省份，这种数据必查的，可以放联合索引的前面。

联合索引中，其中一个字段使用了范围查询，就会导致后面的字段无法走索引。

有些查询最近七天登陆过的用户场景，如果按照时间计算会查询很慢，还不如直接后台搞个定时任务，去判断一下用户是否在最近七天内登录过，然后直接登录就在一个 字段里标记为“1”。

核心重点就是，尽量利用一两个复杂的多字段联合索引，抗下你80%以上的 查询，然后用一两个辅助索<br />引抗下剩余20%的非典型查询，保证你99%以上的查询都能充分利用索引，就能保证你的查询速度和性<br />能！

如果SQL中使用了函数，尽量拿出来在代码里计算，不要耗费Mysql的计算性能。
<a name="oQaya"></a>
### 虚造查询条件
比如，联合索引是province, city, sex, age 四个字段，但是 sex 你不需要筛选，此时如果where条件中没有sex条件的话，那就会导致age索引失效。此时可以把 age in ('男','女') 加上这个条件。<br />虽然不会起到过滤作用，但是能达到age命中索引的效果。<br />简而言之、就是让你的联合索引中间，不要有字段查询缺省，如果有也要虚构出来，比如 age > 0 and <0。

<a name="SMYAj"></a>
## 27-Explain执行计划 和 查询实现原理
<a name="TgLah"></a>
### 执行计划中有个 type字段
一般会显示如下值：const, ref, range , index ,all<br />查询速度从大到小，一次递减。<br />const、ref和range，本质都是基于索引树的二分查找和多层跳转来查询，所以性能都是很高的，然后接下来到index这块，速度就比上面三种要差一些了，因为他是走遍历二级索引树的叶子节点的方式来执行了，那肯定比基于索引树的二分查找要慢多了，但是还是比全表扫描好一些的。

| 索引type级别 | 功能 | 常见出现的地方 | 效率 |
| --- | --- | --- | --- |
| all | 全表扫描，效率低 | 没有索引的时候，全表扫 | 一定要优化 |
| index | 按顺序扫描整个索引，拿到ID ,然后回表去取数据 | 会走索引，而且是扫描全部，排序，这种会用到。 | 尽量优化 |
| range | 在index级别上的升级，只会扫码索引的部分范围内数据，然后拿到数据去回表 | 一般出现在范围查询中，比如 > <<br />betwen, in 这种筛选条件中 | 一般 |
| ref | 在非常小的索引范围内查找，然后回表。 | 索引中小范围搜索，比如命中name值一样的索引块中去查找。 | 高 |
| ref_eq | 在ref的基础上，命中的值唯一，不会出现重复的，然后回表。 | 直接在索引中拿到不会重复的值，这个索引一般是唯一索引。 | 很高 |
| const | 直接是基于主键查询的 | 通常，将一个主键放置到where后面作为条件查询 | 超级高 |

总结就是：<br />你通过普通索引拿出来的数据，一定要尽可能的少，减少去聚簇索引回表的情况。<br />如果回表数据过多，优化器可能会直接去聚簇索引查找，不走二级索引么，不去回表了。

<a name="tKiWP"></a>
### 单表查询中走多个二级索引
因为有可能一条语句中，需要 where name = 'zhangsan' and age = 18;  当为 name 和 age 建立两个单独的索引时，查询顺序是，分别走这两个 二级索引，然后取交集，然后在去聚簇索引中回表查询。 

<a name="nr4LL"></a>
### 多表连接查询原理
例如：<br />select a.* , b.* <br />from a , b <br />where a.name = 'zhangsan' and b.sex = '女' <br />and b.classId = a.classId ;<br />其实内部就是，先查询A表的数据，过滤name = zhangsan 的字段，然后在过滤b表的 sex = '女' ，最后查询出两张临时表，然后通过 临时表中的classID去关联返回出来。

注意：<br />一般前表 是 驱动表，后表示被驱动表。<br />在 A left join B  on A.ID = B.ID  的关连中，尽量让小表驱动大表，这也是一个优化点。

当你的SQL有多表查询，或者关联查询时，explain之后会出现很多行数据，每一行数据代表一张表，或者一个临时表。
<a name="D3KyD"></a>
### explain中的需要特别关注的字段
type： 前面说了，显示利用索引的级别。all，index，ref const之类的。<br />possible_keys： 可能用到的索引<br />rows：扫描行的数量。<br />extra： 这个也要注意一下，有时候会出现如下：<br />using where ：代表全表扫描<br />using index : 代表直接二级索引中扫描<br />using fileSort : 代表需要建立临时磁盘文件，做排序，或者做 order by  group by 。 


<a name="BdjnJ"></a>
## 28-MySQL根据成本优化选择执行计划
说白了，就是SQL里面有个优化器，当你SQL语句会走很多索引时，他会跟据你的表，和索引数据情况，计算各路径查询的成本，选择最优查询路径，减少回表次数。<br />就相当于你有很多条捷径，那到底哪条才是时间最短的捷径呢？

<a name="gwpCa"></a>
### 如何计算查询成本？
<a name="YOtXX"></a>
#### 命令
show table status like '表名';<br />拿到这个表的统计信息，可以看到rows（行数）,data_length, <br /> rows 是一个估算值。<br />data_length  表的聚簇索引的字节数大小   ( data_length / 1024 = kb大小，然后除以16KB，就能计算出🈶多少数据页，此时就能知道数据页数量，和行数，就能计算全表扫描的成本了)。
<a name="ki8Jd"></a>
### 计算公式：
<a name="z107R"></a>
#### IO成本就是：
数据页数量 * 1.0 + 微调值
<a name="OybPm"></a>
#### CPU成本就是：
行记录数 * 0.2 + 微调值<br />二者相加，就是总的成本值。<br />比如你有数据页100个，记录数有2万条，此时总成本值大致就是100 + 4000 =4100，在这个左右。


<a name="MtoLH"></a>
### 子查询优化执行计划
语句：<br />select * from t1 where x1 in (select x2 from t2 where x3=xxx)<br />分析：在下面。
<a name="tWyh3"></a>
#### 临时物化表
通常内部是，先查询出 from t2 where x3=xxx 的数据，形成一张临时表，可能会放在磁盘文件里，也有可能会用memory引擎存储起来，这个过程叫临时 **物化表**  ，也会按照B+树的结构存储起来。<br />然后在挨个对 t1 表中的过滤出来的数据，作比较，会遍历字表中的数据。

注意：在查询过程中，要尽量用小表 驱动大表 去操作。 和 for ( for ())  嵌套优化的道理一样。

在互联网公司里，比较崇尚的是写简单的SQL，复杂的逻辑用Java系统来实现，SQL能单表查询就不要多表关联，能多表关联就尽量别写子查询，能写几十行SQL就别写几百行的SQL，多用代码实现业务逻辑。


<a name="v1yQj"></a>
## 29-案例实战
<a name="gUbfv"></a>
### 千万级运营系统SQL调优
语句：
> SELECT id, name FROM users WHERE id IN (SELECT user_id FROM users_extent_info WHERE
> latest_login_time < xxxxx)

解决方案：<br />说白了，就是是SQL 颗粒化，转化为粒度更小的查询，减少查询范围。<br />或者把子查询拆出来，做两次查询。<br />或者按照分页去查询，反正看你怎么玩了。

<a name="r68YB"></a>
### 亿级数量商品系统的SQL调优
<a name="VpPWj"></a>
#### 背景：
BB半天，根本问题就是，SQL明明对where查询条件做了索引，但是去查看执行计划的时候，发现possible_keys里是有我们的index_category的，所以正常是会用到我们二级索引的。<br />但是extra字段下面显示的是 Using where，表示的是对聚簇索引做了全表扫描，实际没有走我们创建的二级索引。	所以导致的原因就是，我们创建的二级索引没有生效。

**问题语句：**<br />select * from products where category='xx' and sub_category='xx' order by id desc limit xx,xx

<a name="DP2xN"></a>
#### 解决办法：（force index）强制走索引
修改执行计划，强制走二级索引<br />可以在 sql 语句里加上 force index 关键字，强制执行计划，一定走我们的二级索引。
> select * from products force index(index_category) where category='xx' and sub_category='xx'
> order by id desc limit xx,xx


<a name="pNdP7"></a>
#### 为什么在这个语句里会默认对主键的聚簇索引做全表扫描呢？
因为SQL优化器在判断的时候，发现语句后面有order by id desc 的指令，聚簇索引，本来就是按ID顺序排序的，这个时候，他觉得直接去扫描聚簇索引效率更高，避免二级索引扫描后又回表的麻烦。

<a name="fir4p"></a>
#### 为什么没使用index_category这个二级索引进行扫描？
因为优化器判断的时候，为了避免二级索引扫描之后，还依旧查出上万条数据，会对这些数据做，物化处理，持久化成一个临时文件到磁盘，耗费性能，所以不走这个二级索引了。

<a name="eh9gh"></a>
#### 为什么这个SQL以前没有问题，现在突然就有问题了？即使用了聚簇索引
在where查询条件中，这个 category = ‘xxx’ 这个XXX搜索条件，是新创建的，没有对应的记录，所以每次按这个条件去搜的时候都返回为空。每次都对聚餐索引做了全表扫描。

<a name="A6gTH"></a>
### 十亿评论系统的调优
<a name="f96WW"></a>
#### 背景：
SQL语句：
> SELECT * FROM comments WHERE product_id ='xx' and is_good_comment='1' ORDER BY id desc
> LIMIT 100000,20

按照文档上面分析，大概意思就是，每次走完product_id 这个二级索引之后，还有剩下几万的数据，要去聚簇索引中回表，过滤is_good_comment = 1 的值，也就是说这个过程会增加聚簇索引的全表扫描和筛选。<br />于是优化SQL：
> SELECT * from comments a,
> (SELECT id FROM comments WHERE product_id ='xx' and is_good_comment='1' ORDER BY id
> <a name="ZoASc"></a>
#### desc LIMIT 100000,20) b WHERE a.id=b.id

<a name="l3DuN"></a>
#### 优化原理：
就是先执行子查询，子查询反而会使用举措索引，按照ID值倒序方向进行扫描，把符合where product_id ='xx' and is_good_comment='1 的记录筛选出来，不需要存临时排序文件了，减少了IO操作。<br />然后把子查询查出来的ID，再去聚簇索引中 查询数据，直接按照ID去匹配数据，返回。

整个过程都没有走二级索引了。 因为走了二级索引反而筛选出来的值很多，还得去存储临时排序文件，浪费性能。还不如直接走 ID 排好顺序的聚餐索引。

<a name="nOoVV"></a>
### 千万级数据删除慢问题
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1636721403254-9f28eecf-803d-42f8-89be-ffbb78a677ff.png#clientId=u0497018e-93b3-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=517&id=ud6893c5b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=446&originWidth=618&originalType=binary&ratio=1&rotation=0&showTitle=false&size=323895&status=done&style=none&taskId=u258907ec-88c4-4986-808c-a671ec6ea12&title=&width=716.98291015625)<br />废话半天，其实大概意思就是大因为删除的数据量多，而且放在一个事务里，导致新开事务的查询是会读到标记为删除的数据，所以出现千万级的数据扫描，造成了慢查询。<br />而且，长事务下，会对RedoLog UndoLog 造成大量日志，出现性能。这也是一方面的原因吧。
