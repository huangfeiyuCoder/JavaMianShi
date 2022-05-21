<a name="TxIIP"></a>
# 一、基础架构
<a name="Waajh"></a>
## 01-SQL查询如何执行？
<a name="S1rtd"></a>
### Mysql逻辑分层架构图
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1637132691529-08bfcec0-3fd8-4124-ad66-6650a37ad2aa.png#clientId=ua9434c6a-8cfd-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=499&id=u8647b60e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=504&originWidth=632&originalType=binary&ratio=1&rotation=0&showTitle=false&size=275163&status=done&style=none&taskId=ua86f5fb3-4574-480f-9e71-e93f7aafc73&title=&width=625.6420288085938)
<a name="eJPI2"></a>
#### 连接层：
连接监听管理，权限控制，鉴权等。
<a name="GYCxL"></a>
#### 服务层：
**查缓存**： 服务层查询缓存，查询语句作为Key，结果作为Value,生产环境会关掉这种缓存，因为维护成本高，弊大于利，表中有一行数据修改缓存就会失效，不建议用。MySQL8.0后就没有这个功能了。<br />**分析器**：分析SQL的语法，是否正确。<br />**优化器**：对SQL作出执行计划，走成本最低的查询路径，和索引。 Explain会显示执行计划。<br />**执行器**：调度Innodb引擎，去执行SQL语句。可以理解为机器中的调度者。
<a name="vtH9L"></a>
#### 存储引擎：
innodb：实际存储数据的结构，包含redoLog Pool,undoLog ,BufferPool , LRU 等东西。<br />其他：可插拔式的存储引擎，Myciam，Hash,CSV 等等，还有其他存储引擎，但是Mysql 5.5版本之后默认Innodb.
<a name="lzS8O"></a>
### 注意：

- show processlist   命令可以查看Mysql的连接情况，是否有空闲连接等。
- 客户端如果长时间没动静，连接器就会自动将它断开。这个时间是由参数wait的，默认值是8小时。
<a name="nqz7A"></a>
#### 长时间连接导致MySQL宕机？
长时间连接，可能会导致Mysql内存涨的很快，因为执行过程中使用的临时内存是管理在连接对象里面的，这些只有在连接断开后才能释放，如果长时间连接导致OOM，会被系统Kill掉，出现MySQL宕机！！<br />**解决：**

- 定期断开连接，如果内存占用过多就断开。 
- MySQL5.7版后，可执行mysql_reset_connection来重新初始化连接资源，随后不需要做权限校验。 


<a name="v59Hx"></a>
## 02-SQL更新语句如何执行？
分析器，检测SQL语法，判断这是一条SQL语句。
<a name="PAnsG"></a>
### RedoLog
Innodb存储引擎层实现，用来记录本次在数据页上做了什么修改，属于操作日志。空间有限，会循环落到磁盘和删除。
<a name="MLdnE"></a>
### BinLog
MySQLServer服务层实现，属于本次做了什么操作，属于归档日志，从库，备份库一般都是通过这种日志去实现备份。追加日志

<a name="ah9hp"></a>
### RedoLog和BingLog日志的两阶段提交：
会先写Redo日志，后写binglog日志，然后会在redo日志里面标记一个commit标记，然后在保存redo日志，属于两阶段提交。保证两个日志都能最终存到磁盘，避免在写日志的过程中宕机，出现日志不一样，导致主从库不一样的情况。

<a name="P4Bb9"></a>
### RedoLog和BingLog的区别：
1.前者基于存储引擎，后者基于MySQLServer服务层。<br />2.redoLog是物理日志记录哪个数据页修改了什么。 BingLog是逻辑日志，记录了给哪行数据修改成了啥。<br />3.redoLog是循环写，RedoLogPool空间有限，会删除。 BingLog是追加日志，不会删除。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1637135893105-7a0351e4-be4a-4011-9e53-12f812887b79.png#clientId=ua9434c6a-8cfd-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=694&id=u72ea9d12&margin=%5Bobject%20Object%5D&name=image.png&originHeight=794&originWidth=531&originalType=binary&ratio=1&rotation=0&showTitle=false&size=269545&status=done&style=none&taskId=u45b182ed-238b-4e03-938b-f32bff4d4de&title=&width=463.98863220214844)

<a name="uBn1M"></a>
### 拓展：啥是WAL技术？ （_预写式日志_）
WAL的全称是Write-Ahead Logging<br />它的关键点就是先写日志，再写磁盘，也就是先写粉板，等不忙的时候再写账本。<br />参考博客：[https://blog.csdn.net/varyall/article/details/80441457](https://blog.csdn.net/varyall/article/details/80441457)
> _预写式日志_ （WAL） 是一种实现事务日志的标准方法。有关它的详细描述可以在大多数（如果不是全部的话）有关事务处理的书中找到。 简而言之，WAL 的中心思想是对数据文件的修改（它们是表和索引的载体）必须是只能发生在这些修改已经记录了日志之后， 也就是说，在描述这些变化的日志记录冲刷到永久存储器之后。 如果我们遵循这个过程，那么我们就不需要在每次事务提交的时候都把数据页冲刷到磁盘，因为我们知道在出现崩溃的情况下， 我们可以用日志来恢复数据库：任何尚未附加到数据页的记录都将先从日志记录中重做（这叫向前滚动恢复，也叫做 REDO）。

_**预写式日志**：_<br />说白了，可以理解为一种思想，就是把 修改耗时的操作，通过提前记录日志的方式来替代，等有空了再去修改。在Redis，MQ里面的持久化机制中都可以用到。<br />有点像异步操作场景，先创建一个工单，然后后台异步去按照工单去完成任务。


<a name="JROb8"></a>
## 03-事务隔离：为啥改了我还看不见？
ACID：Atomicity、Consistency、Isolation、Durability<br />即：原子性、一致性、隔离性、持久性

<a name="WSbpL"></a>
### SQL标准事务隔离级别：

- 读未提交，读已提交，可重复读，和串行化。
<a name="dQw9j"></a>
#### 读未提交
别人修改，还没提交事务，就被你看到了。
<a name="iJ2JP"></a>
#### 读已提交 （Oracle默认隔离级别）
别人修改，提交了事务，被你看到了。
<a name="QIOJP"></a>
#### 可重复读 （MySQL默认隔离级别）
你启动事务，相当于做了视图快照，不管别人咋查改，你都只能看到你开启事务时的数据情况。直到你的这个事务提交。事务之间互不影响。

- [ ] 这里有必要去了解一下MVCC机制（多版本并发控制）
<a name="fA4Mi"></a>
#### 和串行化
会对同一行记录加 读写锁，所有的请求都是串行操作，效率超级低。

<a name="W9iDW"></a>
### 其他
不要去使用长事务，会导致UndoLog回滚日志变得很多，清理耗费性能。<br />事务的启动方式：手动commit。 或者设置set autocommit=0，SQL中自动提交。

<a name="hUMYn"></a>
# 二、索引和锁
<a name="u6UMe"></a>
## 04-深入浅出索引
<a name="Yf8rL"></a>
### 索引常见数据模型
哈希表，有序数组，搜索树
<a name="ZLGf4"></a>
#### 哈希表

- Key = 索引字段，Value = 聚簇索引的ID值
- 时间复杂度：O1
- 哈希表这种结构适用于只有等值查询的场景，比如Memcached及其他一些NoSQL引擎。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1637137970488-64e6fd10-e0ca-44a4-9ceb-570447d0f530.png#clientId=ua9434c6a-8cfd-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=377&id=u77802eed&margin=%5Bobject%20Object%5D&name=image.png&originHeight=420&originWidth=633&originalType=binary&ratio=1&rotation=0&showTitle=false&size=183203&status=done&style=none&taskId=u39341738-72e5-4957-9346-617187a4bee&title=&width=567.991455078125)

<a name="rW5en"></a>
#### 有序数组

- 按照ID递增顺序存储。底层结构数组。
- 时间复杂度：应该是On
- 适合等值查询，和范围查询的场景中。
- 有序数组索引，只适用于静态存储引擎，不会修改的数据。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1637138148875-3e63b8e1-f28d-430e-8605-32af0648dd8d.png#clientId=ua9434c6a-8cfd-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=177&id=u466ec799&margin=%5Bobject%20Object%5D&name=image.png&originHeight=195&originWidth=645&originalType=binary&ratio=1&rotation=0&showTitle=false&size=118416&status=done&style=none&taskId=ubf2e75eb-0dab-456d-a010-225e9849dec&title=&width=585.991455078125)

<a name="xG1YP"></a>
#### B+索引树

- 每个节点的左儿子小于父节点，父节点又小于右儿子。
- 时间复杂度：O(log(N))
- 最常用的InnoDB索引结构。
- 减少单次查询的磁盘访问次数。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1637138289671-44fd416e-35ce-45fa-81d0-0a1bb9fcb779.png#clientId=ua9434c6a-8cfd-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=340&id=u1323e635&margin=%5Bobject%20Object%5D&name=image.png&originHeight=455&originWidth=642&originalType=binary&ratio=1&rotation=0&showTitle=false&size=222862&status=done&style=none&taskId=ude83ad6a-3459-4b57-b10c-34a2b58928f&title=&width=479.9886169433594)

<a name="eVlxL"></a>
### 什么是聚簇索引，什么是普通索引？
专栏里面提了一嘴，这里自己明白就行，去回顾一下儒猿的MySQL专栏就知道了。
<a name="uSqiX"></a>
### 主键索引和普通索引的查询区别？

- 如果语句是select *fromTwhere ID=500，即主键查询方式，则只需要搜索ID这棵B+树；
- 如果语句是select *fromTwhere k=5，即普通索引查询方式，则需要先搜索k索引树，得到ID的值为500，再到ID索引树搜索一次。这个过程称为回表。

<a name="CQdGi"></a>
### 索引：页分裂，页合并
<a name="k79uD"></a>
#### 页分裂
突然插入ID不是自增的，那么就可能会在B+树上新建一个数据页，那么会重新排序挪动数据，性能很低。
<a name="ElZkI"></a>
#### 页合并
删除中间数据，导致B+树上的数据页的值有改变，可能会合并一些数据页。可以理解为分裂的逆过程。
<a name="uwxvK"></a>
#### 总结

- 尽量使用自增ID，因为自增ID，不涉及其他数据页的挪到，只在后面+1，不会触发分裂。
- 使用自增数字做ID，UUID这种会占用B+树的内存，导致叶子节点占用空间多。

<a name="QvG64"></a>
### KV场景适合业务字段直接做主键的案例
<a name="OGzco"></a>
#### 场景：
有些业务的场景需求是这样的：<br />1. 只有一个索引；	2. 该索引必须是唯一索引。
<a name="H6dUd"></a>
#### 分析：
这就是典型的KV场景。由于没有其他索引，所以也就不用考虑其他索引的叶子节点大小的问题。<br />这时候我们就要优先考虑上一段提到的“尽量使用主键查询”原则，直接将这个索引设置为主键，可以避免每次查询需要搜索两棵树。

<a name="MqeaX"></a>
### SQL回表次数分析案例
回到主键索引（聚簇索引）的过程，就称之为回表。<br />语句：select * from tb where k betwen 3 and 5 ;   K为二级索引
<a name="Sw7Oa"></a>
#### 分析执行流程：
> 1. 在k索引树上找到k=3的记录，取得 ID = 300；
> 2. 再到ID索引树查到ID=300对应的R3；
> 3. 在k索引树取下一个值k=5，取得ID=500；
> 4. 再回到ID索引树查到ID=500对应的R4；
> 5. 在k索引树取下一个值k=6，不满足条件，循环结束。

<a name="PlKpN"></a>
#### 疑问：
 二级索引 驱动 聚簇索引 去筛选数据，相当于两个嵌套for循环，时间复杂度为 O(m*n) ，为啥不直接先 二级索引查出来用Map存储，然后在去聚簇索引中遍历，筛选呢。这样就 Ologn + O1 + Ologm 效率不是更高点？

<a name="LgCuQ"></a>
### 什么是覆盖索引？
就是可以理解为，你要查询返回的字段，都在二级索引里面有了，不需要回表遍历主键索引，这个过程就叫覆盖索引。一般会根据查询，建立一些联合索引，把需要常查询的字段都放在二级索引上面。<br />可以减少主键树的搜索次数，提升性能。

<a name="H9Szt"></a>
### 最左前缀原则
就是建立联合索引的时候，会从左到右开始检索索引字段，如果中间有索引缺失，就会导致后面的字段无法命中索引。所有，要按照查询的顺序去设计联合索引的顺序。

<a name="uOWL7"></a>
### 索引下推 （违背最左前缀原则的设计）

- 在MySQL 5.6之前，只能从ID3开始一个个回表。到主键索引上找出数据行，再对比字段值。
- 而MySQL 5.6 引入的索引下推优化（index condition pushdown)， 可以在索引遍历过程中，对索引中包含的字段先做判断，直接过滤掉不满足条件的记录，减少回表次数。

说白了，和最左匹配原则的设计有点相反，会把联合索引后面那些不走索引的字段，也去做筛选，减少扫描主键索引的麻烦。<br />图4跟图3的区别是，InnoDB在(name,age)索引内部就判断了age是否等于10，对于不等于10的<br />记录，直接判断并跳过。
<a name="Shbrb"></a>
#### 5.6之前，无索引下推过程：
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1637142232510-01a1bdaa-d7cc-4e8c-922b-1ae5d9ce5082.png#clientId=ua9434c6a-8cfd-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=186&id=ub8bbbed7&margin=%5Bobject%20Object%5D&name=image.png&originHeight=239&originWidth=614&originalType=binary&ratio=1&rotation=0&showTitle=false&size=153485&status=done&style=none&taskId=ueb16daf9-28d2-419d-87c0-4bf85370b7c&title=&width=477.6590881347656)
<a name="DHy55"></a>
#### 5.6之后，有索引下推过程：
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1637142252576-6dbbf63a-75b6-459d-87b1-c1690829a0cb.png#clientId=ua9434c6a-8cfd-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=208&id=u92871b79&margin=%5Bobject%20Object%5D&name=image.png&originHeight=264&originWidth=622&originalType=binary&ratio=1&rotation=0&showTitle=false&size=157563&status=done&style=none&taskId=u6ebd72e7-3ae9-42f1-bb86-9bb3318a79c&title=&width=489.3295440673828)
<a name="wRauH"></a>
## 05-全局锁和表锁
大致分为三类锁：全局锁，表锁，行锁。
<a name="uuzYO"></a>
### 全局锁
在用数据库备份的时候，会对库加全局锁，保证数据的安全性。
<a name="uXX6H"></a>
### 表级别锁
表级别的锁又分为：元数据锁，和表锁
<a name="xPHqK"></a>
#### 元数据锁
    元数据锁（MDL）MySQL5.5之后对所有的增删改查都默认加MDL锁，防止在查询修改数据时，字段有改变。
<a name="fGtHb"></a>
#### 表锁：
    执行Lock tables 语法的时候对全表加的锁，系统不默认加。

<a name="WRfgo"></a>
### 行级别锁
并不是所有的存储引擎都支持行级锁，这个由特定的存储引擎去实现。<br />注意：<br />Innodb行级锁的特点是，只有在用到这行数据的时候，才对这行加锁。不是开启事务就对这行数据加了锁，而是在事务中用到了才对这行加锁。<br />小技巧：<br />**如果你的事务中需要锁多个行，要把最可能造成锁冲突、最可能影响并发度的锁尽量往后放。**
<a name="WW6Az"></a>
### 两阶段锁协议
在innodb事务中，行锁是在需要的时候才加上的，并不是不需要就立刻释放，而是要等到事务结束后才释放。 这就是** 两阶段锁协议**<br />**简而言之：就是行锁不是在开启事务的时候加上的，而是在用到的时候加上的，但却是最后一起释放的，所以小技巧是：把容易出现并发的数据写最后。**<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1637424291906-4b2f5858-b0db-4843-b55f-5493045d6c79.png#clientId=u34652b79-808b-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=782&id=ua5861142&margin=%5Bobject%20Object%5D&name=image.png&originHeight=917&originWidth=685&originalType=binary&ratio=1&rotation=0&showTitle=false&size=549601&status=done&style=none&taskId=uc532cfa5-4985-4972-a2b1-04d0484c93f&title=&width=584.5)
<a name="H9hMn"></a>
### InnoDB中死锁问题
当存储引擎中出现了两个事务相互占用了对方释放条件资源，出现了死锁情况，这个时候InnoDB中有两种解决策略。
<a name="XXSzd"></a>
#### 超时检测策略：
设置参数innodb lock wait timeout ，默认60S，但是一般只会设置1S。业务中一般不会设置太长的时间等待。
<a name="WYWvw"></a>
#### 死锁自检策略：
设置innodb_deadlock_detect，开启死锁自检<br />自检发现死锁后，主动回滚其中一个事务，让另一个事务继续执行。<br />但是在自检过程中很耗费CPU资源，会有额外负担。特别是在高并发下，自检涉及的时间复杂度非常高，所以在热点行更新上会有性能问题。
<a name="VkWJd"></a>
### 解决热点行死锁更新的性能问题

1. 从并发上面入手，减少数据库的并发数，降低自检过程中的时间复杂度。
1. 直接关闭 死锁检测，让死锁后直接回滚，让业务代码重试。
1. 从热点行上入手，增加热点行的数量，比如业务是修改一行汇总数据的，可以改成修改成随机十条数据，然后后面统一汇总，分而治之的思想。

<a name="ldNpm"></a>
## 06-事务到底隔离还是不隔离？
额。。。MVCC机制，看的有点迷糊，直接上结论吧。。记住几个关键字<br />InnoDB的行数据有多个版本，每个数据版本有自己的rowtrx_id，每个事务或者语句有自己的一致性视图。普通查询语句是一致性读，一致性读会根据rowtrx_ID和一致性视图确定数据版本的可见性。
<a name="XJwzu"></a>
### MVCC实现的几个关键点：
每个事物对应一个transactionID，在事务开始时申请。<br />每行数据也会有多个版本，叫rowtrx_id，和transactionID相关联，实现多版本数据快照。如下图：<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1637426502042-d9741870-a86b-4a17-b207-1b0a9168a568.png#clientId=u34652b79-808b-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=154&id=u771a4f08&margin=%5Bobject%20Object%5D&name=image.png&originHeight=267&originWidth=628&originalType=binary&ratio=1&rotation=0&showTitle=false&size=164773&status=done&style=none&taskId=u221d03d0-044c-4035-a85b-1b028ac5b9c&title=&width=362.9772644042969)
<a name="o7mT8"></a>
### MVCC的视图和快照是不是很占内存？
注意：**MVCC数据快照不是物理存在，而是通过Undo日志的逻辑来实现的**<br />语句更新会生成undoLog日志，那他在这里怎么实现呢？上面图的三个虚拟箭头就代表的undoLog，可以理解为是UDOLog依次记录了多个数据版本的关系。

<a name="surTQ"></a>
### 事务的可重复读能力是怎么实现的？
    可重复读的核心就是一致性读（consistent read）；而事务更新数据的时候，只能用当前读。如果当前的记录的行锁被其他事务占用的话，就需要进入锁等待。
<a name="SHOb5"></a>
### 而读提交的逻辑 和 可重复读的逻辑类似
    它们最主要的区别是：在可重复读隔离级别下，只需要在事务开始的时候创建一致性视图，之后事务里的其他查询都共用这个一致性视图；在读提交隔离级别下，每一个语句执行前都会重新算出一个新的视图。

<a name="CP85C"></a>
## 07-普通索引和唯一索引怎么选择？
从普通索引和唯一索引的选择中，后面会引出ChangeBuffer的更新原理。

- [ ] 至于这两个索引之间怎么选择呢？
- 读场景，唯一索引好一点点，找到符合的后不需要继续查找。
- 单纯的更新场景，普通索引好点，会利用ChangeBuffer减少磁盘IO。
- 更新后读场景，普通索引的ChangeBuffer会带来和数据页的Merge消耗，影响性能，可以在设置中关闭CBuffer缓存。

<a name="yL315"></a>
### 读场景分析：
**唯一索引**，遍历B+树后，找到叶子节点上的数据页后，用二分法查找到数据后就停止查找了。<br />**普通索引**，遍历B+树后，找到叶子节点上的数据页后，在数据页找到符合的数据后，还有继续查找下一个数据，直到这个数据页或下一个数据页没有符合的数据为止。<br />**所以**：区别，仅仅在一个数据页上多个到不符合的关系，但是一个数据页才16K，一般也就近1000个K，所以**性能区别微乎其微**。

<a name="kiXS3"></a>
### 更新场景分析：
<a name="AfhQt"></a>
#### 唯一索引：
不使用ChangeBuffer，直接保存到磁盘，因为要判断最终的磁盘数据是否有重复数据。
<a name="qeCoh"></a>
#### 普通索引：
使用ChangeBuffer，直接保存之后就完事了。
<a name="S1Gj0"></a>
#### 总结：
所以：在单纯的修改的场景中，使用普通索引更为合适，因为ChangeBuffer是行数据修改缓存，相当于直接记录了修改日志，类似与（前面说的WAL机制），可以提高修改的速度。<br />但是！！ChangeBuffer也是位于BufferPool内存中，如果被修改的这个数据修改之后要直接查寻，那么这行数据对应磁盘上的的数据页，以及BufferPool中的缓存页，需要先和ChangeBuffer做一个Merge操作，合并之后才算是最终数据。<br />所以：在修改后立马要查询的场景中，不太适合普通索引，因为反而会增加ChangeBuffer数据的频繁Merge的烦恼。

<a name="cDGiH"></a>
### 什么是ChangeBuffer修改缓存？
简说：相当于是在普通索引修改时，记录的修改日志，和RedoLog差不多，用来避免行数据修改后操作磁盘的烦恼。位于BufferPool内存中，最多占比50%。

- 和儒猿专栏不同中讲的 数据页缓存列表，Flush链表不同，那个记录的是修改后数据页的缓存。
- 和RedoLog不同，但是逻辑上有点相似，redoLog主要节省的是随机写磁盘的IO消耗，而ChangeBuffer主要节省的是 随机读磁盘的IO操作。！！！
- 而且ChangeBuffer对唯一索引没有作用，只有普通索引才有使用。

<a name="EjPjU"></a>
### RedoLog和Change有啥不同？又和Flush链表有啥关系？

- 和RedoLog不同，但是逻辑上有点相似，redoLog主要节省的是随机写磁盘的IO消耗，而ChangeBuffer主要节省的是 随机读磁盘的IO操作。！！！
- 而且ChangeBuffer对唯一索引没有作用，只有普通索引才有使用。
- [ ] 说真的我也不理解**Change和Redo为啥是这个区别**，后面再百度了解。

<a name="ZBOkV"></a>
## 08-MySQL为啥会选错索引
因为一张表中可以有多个索引，可以选择性的去走，选择的依据是成本最小，但是MySQL Server中的SQL优化器，也会有分析错误的时候，用EXplain分析的时候，法没有按你当初的的想法走索引，可能是优化器做了错误的选择。

<a name="ESdQ4"></a>
### MySQL优化器选择索引考虑的几个因素？
扫描行数，回表次数数量，是否要排序等等。<br />总之：<br />最终目的以，花最小代价为核心，去完成SQL任务。<br />扫描行数越少越好，回表越少越好，访问磁盘IO越少越好，CPU计算越少越好。

<a name="C2xMg"></a>
### 扫描行数是怎么判断的？
MySQL是根据索引的区分度来估算扫描行数的，索引中不同值称之为基数，基数越大，索引效果越好。<br />因为把整张表取出来一行行统计，虽然可以得到精确的结果，但是代价太高了，所以只能选择“采样统计”。

**使用采样统计**：随机抽取N个数据页，得到索引的平均基数，然后乘以总页数，就能得到这个索引的基数。<br />而数据表是会持续更新的，索引统计信息也不会固定不变。所以，当变更的数据行数超过1/M的时候，会自动触发重新做一次索引统计。<br />所以，有时候统计行数也是不准的。

<a name="DbHab"></a>
### analyze table t 强制重新统计索引信息
> analyze table t 命令，可以用来重新统计索引信息。

<a name="Q0wK1"></a>
### 
<a name="rN6h4"></a>
### 如果优化器错误的选择了索引，三种解决方案
1.采用force index 强行指定走那个索引，但是索引改名字就会报错。<br />2.删除误走的那个索引，在不影响业务的情况下。<br />3.在order by 后面增加你要使用的索引字段，让优化器偏重考虑你想走的那个索引。

<a name="XtM4b"></a>
## 09-怎么给字符串加索引？（前缀索引的使用）
<a name="RNSqX"></a>
### 前缀索引
其实**内部结构就是普通索引**，只是使用方式不同而称之为前缀索引。当一个字符串很长，担心占用索引树的内存，而去对索引字段做一个指定长度截取。<br />但是前缀索引和前面说到的覆盖索引，是有点相反的，覆盖索引是你查询返回的字段，全部在索引树上，不需要回表聚合索引去查询就能返回。而这个前缀索引存储的不是完整的字段内容，需要回表。<br />所以，如果截取前缀位数，如果不好好设计，也会增加回表的次数，影响效率。
<a name="qUF89"></a>
#### 如何分析前缀索引截取多少长度效率最高？（计算索引的区分度）
比如，对一个email字段做前缀索引设计，需要计算索引不同长度的区分度，看哪个长度区分度最高就最好。
```java
先查全部行数
mysql> select count(distinct email) as L from SUser;
// 然后查询不同长度的行数
mysql> select
count(distinct left(email,4)）as L4,
count(distinct left(email,5)）as L5,
count(distinct left(email,6)）as L6,
count(distinct left(email,7)）as L7,
from SUser;
// 然后看哪个长度
      使用前缀索引很可能会损失区分度，所以你需要预先设定一个可以接受的损失比例，比如5%。然后，在返回的L4~L7中，找出不小于 L * 95%的值，假设这里L6、L7都满足，你就可以选择前缀长度为6。
```
<a name="zCjUq"></a>
### 引出实践问题
如果给你一个省份的所有身份证ID，让你按照这个设计一个索引。这时，身份证长度很长，且字符串前面有9位数字接近相同，且区分度不高，占用B+索引树的内存，严重影响索引的效率。此时你咋办？
<a name="xQNPL"></a>
### 解决方案
**1.把身份证倒叙存储，然后截取前6位，作为索引。  （前缀索引）**
```java
mysql> select field_list from t where id_card = reverse('input_id_card_string');
```
**2.截取身份证号码后面6位，存储一个新字段，然后作为索引。**<br />**3.使用hash字段，把身份证号码先哈希计算，然后存储到一个新字段里，可以减少内存。**
```java
mysql> alter table t add id_card_crc int unsigned, add index(id_card_crc);
mysql> select field_list from t where id_card_crc=crc32('input_id_card_string') and id_card='input_id_card_string';
```
<a name="mdMUB"></a>
#### 那么这三张方案的由优缺点如何呢？
从时间，内存，CPU三个角度去回答。

- 都不能排序，不能查询索引范围。
- 前缀索引，影响区分度，可能增加回表次数。还有SQL中的函数计算
- 截取长度作为新字段，占用内存
- hash计算，额外调用一次reverse函数消耗CPU计算，占用额外空间。
<a name="MwteV"></a>
### 总结：
1. 直接创建完整索引，这样可能比较占用空间；<br />2. 创建前缀索引，节省空间，但会增加查询扫描次数，并且不能使用覆盖索引；<br />3. 倒序存储，再创建前缀索引，用于绕过字符串本身前缀的区分度不够的问题；<br />4. 创建hash字段索引，查询性能稳定，有额外的存储和计算消耗，跟第三种方式一样，都不支持范围扫描。


<a name="iLQPH"></a>
# 三、生产实践案例
<a name="Sk6Nn"></a>
## 10-MySQL会抖动的案例
<a name="GzOYA"></a>
### SQL语句为什么会突然变慢（抖动）？
平时很快的更新操作，其实就在写内存和日志，而MySQL偶尔“抖”一下的瞬间，可能是在刷脏页（flush）。
<a name="FuVwb"></a>
### 什么情况会引发数据库的flush过程呢？
**1.Innodb的redoLog写满了，系统停止所有更新操作，checkpoint往前推进，redo log留出空间继续写。**
> checkpoint可不是随便往前修改一下位置就可以的。比如图2中，把checkpoint位置从CP推进到CP’，就需要将两个点之间的日志（浅绿色部分），对应的所有脏页都flush到磁盘上。之后，图中从write pos到CP’之间就是可以再写入的redo log的区域。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1637652197500-14931a29-fb1a-4694-9a82-8f9f1eb9a9ef.png#clientId=u2a1fec49-4e15-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=390&id=u270de748&margin=%5Bobject%20Object%5D&name=image.png&originHeight=392&originWidth=385&originalType=binary&ratio=1&rotation=0&showTitle=false&size=105819&status=done&style=none&taskId=u3e02a122-9135-4ae6-9c52-5ee84d9eaef&title=&width=383.3125)<br />**2.系统BufferPool内存不足，需淘汰数据（缓存）页，增加新的缓存页进来。**<br />**3.空闲时守护线程，后台刷脏缓存页到磁盘。**<br />**4.正常关闭MySQL时，会把内存的flush链表上的脏页都flush到磁盘上。**

<a name="XcIiD"></a>
#### 上面四种场景对性能的影响：
前面两种会造成用户SQL停顿，后面两种级别上可以忽略不计。

<a name="xAd04"></a>
### Innodb刷脏页的控制策略-(记住几个关键点)

- 要用到innodb_io_capacity这个参数了，它会告诉InnoDB你的磁盘能力。
- InnoDB的刷盘速度就是要参考这两个因素：一个是脏页比例，一个是redo log写盘速度。
- 参数innodb_max_dirty_pages_pct是**脏页比例上限，默认值是75%。**
- **有趣策略：**innodb_flush_neighbors 参数设置是否把旁边的脏缓存页，也刷入磁盘。

<a name="tyJqT"></a>
## 11-删除数据，为啥表文件大小没有改变
把表数据都删除之后发现文件大小依旧没改变，可能原因是：B+树上的数据页依旧存在，只是标记了“删除”状态，不会立马删除，等待后面的数据复用。这个也叫**数据页空洞**。<br />还有个原因虽然专栏没讲，但是也可以说道说道。那就是BufferPool会对修改的脏页做缓存，不会修改后立马刷入磁盘当中，这或许也是一个微不足道的原因。<br />**例如下图：**<br />把R4这行删了，PageA这个数据页依旧存在，占用这么多空间。这是防止B+树分裂，聚合手段。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1637654608885-f03c6bd8-0626-4891-b472-908b2091d5ab.png#clientId=u2a1fec49-4e15-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=313&id=uc3c07bbc&margin=%5Bobject%20Object%5D&name=image.png&originHeight=297&originWidth=387&originalType=binary&ratio=1&rotation=0&showTitle=false&size=84587&status=done&style=none&taskId=udc8e3c64-46a6-43d5-b491-34e1e3cd82c&title=&width=407.9829406738281)
<a name="YSYyI"></a>
### 删除，插入数据都可能造成数据页空洞
当突然删除自增ID中间行的数据时，或者插入时，就会引发B+树的撕裂和聚合。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1637654701150-acdc7f28-1d6f-4997-82b3-d460dacfea83.png#clientId=u2a1fec49-4e15-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=518&id=u1ea1426b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=794&originWidth=630&originalType=binary&ratio=1&rotation=0&showTitle=false&size=308517&status=done&style=none&taskId=u5533b90a-d8fb-430d-9798-6860b0a131a&title=&width=411)
<a name="GAB7x"></a>
### 怎么回收表空间呢？
1.对表执行：alter table t engine=InnoDB；里面会建立临时表B，会自动完成转存数据、交换表名、删除旧表的操作。<br />2.MySQL5.6之后的**OnlineDDL**。还有**inplace** ，好像也是和这有关。

- [ ] 没怎么看懂，这两个是啥东西，回头百度查查吧。

3.直接物理删除表，重新建一张表。
> 现在你已经知道了，如果要收缩一个表，只是delete掉表里面不用的数据的话，表文件的大小是不会变的，你还要通过alter table命令重建表，才能达到表文件变小的目的。我跟你介绍了重建表的两种实现方式，Online DDL的方式是可以考虑在业务低峰期使用的，而MySQL 5.5及之前的版本，这个命令是会阻塞DML的，这个你需要特别小心。


<a name="hqc1J"></a>
## 12-count(X) 统计行数分析
统计一张表的行数，这个函数，不同存储引擎实现的方式是不同的。innodb和MyISAM就不一样。

- MyISAM引擎：内部直接保存了表中所有的行数，count查询非常快。
- Innodb引擎：虽然内部也存储了估算行数，但不是精确的，前面章节说过，是在B+索引上抽样算出平均值，然后乘以总页数得到的行数。
<a name="J5Sec"></a>
### 为什么Innodb引擎不能把总数也存起来呢？
因为Innodb引擎是通过MVCC支持事务的，在并发情况下，不同事务拥有不同的快照数据，无法统计最精确的行数。<br />show Tables status 显示的行数也不能直接使用，是通过平均值估算出来的。

<a name="angpX"></a>
#### 优化解决方案：
1.自己把行数统计起来，存到Redis里面，实时根据业务变化更新行数缓存。（不建议，并发受限，数据也需要和业务强耦合。）
> 把计数放在Redis里面，不能够保证计数和MySQL表里的数据精确一致的原因，是这两个不同的存储构成的系统，不支持分布式事务，无法拿到精确一致的视图。而把计数值也放在MySQL中，就解决了一致性视图的问题。

2. 缩短查询范围，分块统计。（会涉及多次查询，效率并不高，而且可能存在并发问题）
2. 直接count(*), 因为Innodb对这个函数使用做了专门的优化。 
- [ ] 具体怎么优化的我也不知道，百度查查吧？

<a name="myzxo"></a>
### 不同count(*),count(id),count(1),count(字段)的查询效率？
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1637656990534-a7173845-b296-4a76-91aa-2b63c5ee5c87.png#clientId=u2a1fec49-4e15-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=912&id=u6ab540de&margin=%5Bobject%20Object%5D&name=image.png&originHeight=786&originWidth=516&originalType=binary&ratio=1&rotation=0&showTitle=false&size=354127&status=done&style=none&taskId=u9cc37456-964d-4e05-a2c7-23727e8dc1b&title=&width=598.9772644042969)
<a name="z3q10"></a>
#### 所以结论是：
**按照效率排序的话，count(字段)<count(主键id)<count(1)≈count(*)，所以我建议**<br />**你，尽量使用count(*)。**

<a name="M3PzF"></a>
## 13-日志和索引问题总结
<a name="LJWm1"></a>
### 1、不要混淆RedoLog中的commit。
是写在redoLog文件里面的commit标记，只是为了解决在事务提交时，保证BinLog已经完完全全的写入到磁盘的标记，防止RedoLog和BinLog都没有写成功的状态。有点像两阶段提交时的commit。只有当两个日志都说OK时，才提交事物。
<a name="aJ2SA"></a>
### 2、MySQL怎么知道BinLog是完整的？（看看就行）
一个事务的BinLog是有完整格式的，MySQL通过校验CheckSum的结果来发现。
<a name="XeuKk"></a>
### 3、RedoLog和BinLog是怎么关联起来的？
他们有一个共同的字段，叫**XID**，会按顺序扫描redo log。

<a name="Nma7p"></a>
### 4、RedoLog一般设置多大？
 不能太小，太小会导致很快写满，WAL机制的能力就发挥不出来。<br />一般常见几个TB磁盘的话，会设置4个文件，每个文件1GB。

<a name="K8sIp"></a>
## 14-Order by 排序的工作原理
<a name="rrSJj"></a>
### 排序案例，引出问题和概念
<a name="SW54I"></a>
#### 查询SQL：
select city,name,age from t where city='杭州' order by name limit 1000 ;<br />设置  city 为普通索引。
<a name="tczdO"></a>
#### 分析：
用Explain命令查看，发现Extra字段，有个Using fileSort，这代表要排序。MySQL会给每个请求线程分配一块内存区域用来排序，称之为 **sort_buffer，**<br />注意要和Sort_file 区分开。前者是内存，后者是磁盘文件
<a name="Fmu8a"></a>
#### 分析语句执行流程如下： 
1. 初始化sort_buffer，确定放入name、city、age这三个字段；<br />2. 从索引city找到第一个满足city='杭州’条件的主键id，也就是图中的ID_X；<br />3. 到主键id索引取出整行，取name、city、age三个字段的值，存入sort_buffer中；<br />4. 从索引city取下一个记录的主键id；<br />5. 重复步骤3、4直到city的值不满足查询条件为止，对应的主键id也就是图中的ID_Y；<br />6. 对**sort_buffer**中的数据按照字段name做快速排序；<br />7. 按照排序结果取前1000行返回给客户端。
<a name="wUYU5"></a>
### 全字段排序
简单来说，就是从city索引树上拿到数据ID之后，去主键索引上拿到需要返回的字段数据，然后把需要返回的字段一起在内存里面做排序，这就是**全字段排序。  **上面那个例子就是全字段排序<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1637661276209-662ee6ac-4585-4037-8140-d6b3d03e3096.png#clientId=u2a1fec49-4e15-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=411&id=uba71855b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=484&originWidth=644&originalType=binary&ratio=1&rotation=0&showTitle=false&size=212475&status=done&style=none&taskId=ucb1887f9-32ef-4390-91c6-a0ccf9051dc&title=&width=546.6505432128906)
<a name="SAf5g"></a>
### 引出三个问题概念：
<a name="UPx6j"></a>
#### 磁盘临时文件排序：Sort_file
直接在内存排序，内存有限，如放不下，就会**生成临时文件放到IO磁盘上排序**。这个过程称之为soft_file，也称为**外部排序**。

<a name="h2paI"></a>
#### 磁盘外部排序 的算法：并归排序算法
MySQL将要排序的数据，分成12份，每份单独排序后存在临时文件中，然后再把12个有序文件合并成一个有序大文件，这个过程叫 **并归排序算法 。 **分而治之，化整为零，有点像Hadoop中Mapreduce的思想。

<a name="BwsT5"></a>
#### rowId排序 ： （MySQL认为排序的字段长度太大的优化）

- max_length_for_sort_data，是MySQL中专门控制用于排序的行数据的长度的一个参数。
- 如果超过了这长度，就会优化查询的步骤，以上面那个例子就会改成如下：

1. 初始化sort_buffer，确定放入两个字段，即name和id；<br />2. 从索引city找到第一个满足city='杭州’条件的主键id，也就是图中的ID_X；<br />3. 到主键id索引取出整行，取name、id这两个字段，存入sort_buffer中；<br />4. 从索引city取下一个记录的主键id；<br />5. 重复步骤3、4直到不满足city='杭州’条件为止，也就是图中的ID_Y；<br />6. 对sort_buffer中的数据按照字段name进行排序；<br />7. 遍历排序结果，取前1000行，并按照id的值回到原表中取出city、name和age三个字段返回给客户端。

简单来说：

- 看上去和**全字段排序**步骤差不多，就一点区别，rowid排序多访问了一次表t的主键索引，就是步骤7。
- 在排序时，先对字段排序，然后按排好序的数据，用主键ID，去聚合索引中拿到你需要返回的字段内容。
- 就是多了一次回表，把不排序的字段拿到，减少排序时的内存占用

<a name="yKFiY"></a>
### 优化

1. 可和查询条件，及要排序的字段，建立**联合索引**，通过索引树自带的排序来返回数据。
1. 使用**覆盖索引**，直接返回排好序的内容，其实和第一个方案差不多。
1. 之前看到过一个案例，这里没讲，好像是直接在二级索引上查询出主键ID，做一个子查询，然后再去分页排序。好像和这个场景有点点关系。。

<a name="yeudS"></a>
## 15-查询随机数据：order by rand() 的弊端
<a name="SN7jK"></a>
### 关键点
听专栏BB了一大堆数据，难以理解，就记下几个关键就行了。<br />MySQL 5.6版本引入的一个新的排序算法，即：优先队列排序算法。和之前的并归排序有点区别。

- [ ] 具体啥区别，后面百度去吧。

对于内存表，回表只是去内存上的主键索引上访问而已，不存在访问磁盘。优化器没有这一层顾虑，那么它会优先考虑的，就是用于排序的行越少越好了，所以，MySQL这时就会选择rowid排序。
<a name="Rrd8t"></a>
### 排序流程图
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1637667439257-67c70259-06c3-410a-a132-6255e720a2cc.png#clientId=u931ac670-1747-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=537&id=uead46bfc&margin=%5Bobject%20Object%5D&name=image.png&originHeight=506&originWidth=655&originalType=binary&ratio=1&rotation=0&showTitle=false&size=201679&status=done&style=none&taskId=u8a7f502f-3326-4666-bb7a-8e896e77cea&title=&width=695.3238525390625)<br />**order by rand() 使用了内存临时表，排序的时候使用了rowID排序方法。**
<a name="E54Vi"></a>
### 
<a name="l4ABR"></a>
### 随机取值的优化方案
1.直接取ID的最大，和最小值，然后在代码中计算，随机生成几个ID，拿ID直接去精确查找，不香嘛！！！<br />2.没有ID你拿个其他字段的随机取出来几条数据，然后在代码处理，不香嘛！！！  
> 用随机函数生成一个最大值到最小值之间的数，作为ID去查询： X = (M-N)*rand() +N;

<a name="m23pK"></a>
### 
<a name="JFhuQ"></a>
### Explain分析时出现Using temporary和 Using filesort的意思
Explain分析SQL，其中Extra字段显示

- Using temporary，表示的是需要使用临时表；
- Using filesort，表示的是需要执行排序操作。

<a name="di43k"></a>
## 16-使用索引错误案例
其实下面三个案例，归根结底的错误原因，就是用了函数。<br />对索引字段做函数操作，可能会破坏索引值的有序性，因此优化器就决定放弃走树搜索功能。
<a name="QJpK0"></a>
### 案例一：索引上用函数

- 在where后面对索引字段运用了函数，导致索引失效，或者全索引扫描。
- 当优化器判断，走二级索引的行数依旧比主键索引要少时，可能也会用到二级索引，只是无法再使用索引快速定位功能，而只能使用**全索引扫描**。 如果要多就会直接 不用二级索引，也就会导致索引失效。

<a name="DgRjD"></a>
### 案例二：索引隐式类型转换
查询的字段类型是字符串，你传进去的参数是int类型，结果在查询时做了隐式类型转换，相当于对字段用了函数，这个时候也会导致索引失效。   （工作中我就才过坑！！）

<a name="cKQAD"></a>
### 案例三：隐式字符编码转换
**字段的编码规则不一样，或者不同表的编码规则不一样，在做联合查询时，导致索引失效。**<br />（题外话：面试可以聊聊：19年10月，我在财务做确收时，生产上就犯了这样的错误！！！）<br />当时，数据库编码规则没有统一，我建设了一张新表，和老表做连接查询，结果没注意到两张表的编码规则不一样，结果直接把线上数据库实例CPU跑高，被DBA叫过去了。<br />原因就是因为SQL语句连接查询时，编码规则不一样，导致索引失效，CPU飙高。

<a name="aEMex"></a>
## 17-查一行数据也执行这么慢
在数据量和服务正常的情况下，查询一行数据也变得很慢的原因有，锁表，被其他事务上锁了，等等。
<a name="ktkjT"></a>
### 第一类：长时间查询不返回
showprocesslist命令，看看语句处于什么状态，是否锁表。
<a name="DnzhJ"></a>
#### 1.等MDL锁：
showprocesslist命令查看出现Waiting for table metadata lock，有可能是在**修改表结构。**
<a name="kWhMB"></a>
#### 2.等flush
flush tables t with read lock; 就是对表做了，FTWRL全局读锁，一般出现在 **数据库备份**，或者**清除数据库表缓存** 的场景中 。
<a name="RLU8M"></a>
#### 3.等行锁
  select * from t where id=1 lock in share mode;  被其他事务，对该行加了读锁。

<a name="fuQhC"></a>
### 问题：下面这行，是加行锁，还是表锁？
> begin;
> select * from t where c=5 for update;
> commit;

答案:如果这个C是有索引的那就是行锁，如果没有，那就是表锁。


<a name="QiGZW"></a>
## 18-幻读是什么？且产生的问题？（间隙锁诞生）
<a name="WiV4n"></a>
### 引出问题
幻读指的是一个事务在前后两次查询同一个范围的时候，后一次查询看到了前一次查询没有看到的行。<br />这里举个案例： A，B代表不同事务，B执行一次Update之后就自动提交事务。<br />A：开启事物<br />A： select * from t where d=5 for update;    查询返回，1，2，3 这三行数据<br />B：update t d = 5 where id = 8 ;  把ID=8的数据d ,修改成5。 <br />A：select * from t where d=5 for update;   此时查询，返回。1，2，3，8 这四行数据。
<a name="lqBZk"></a>
#### 分析：
事务A明明上了写锁，但还是会有并发问题，事务前后读到不一样的数据。出现了 “幻读”。<br />从上面看来，及时吧所有的记录都加上了锁，也组织不了新插入的数据，这也是“幻读”要单独拿出来解决的原因之一。
<a name="Kfpca"></a>
### 可重读级别下，如何解决幻读？（上间隙锁）

- 解决幻读问题，InnoDB只好引入新的锁，也就是间隙锁(GapLock)。
   - 也就是说，不仅仅是锁住了行数据，把行两边的数据也上锁了。  
   - （大概理解就是把那一个数据页给锁了，可能这种理解也不太对。）
- **间隙锁是在可重复读级别下才会生效**的。所以，如果把隔离级别设置为读提交的话，就没有间隙锁了。
<a name="cSXW2"></a>
#### 跟间隙锁存在冲突关系的情况
往这个加了间隙锁的行数据附近，插入一条记录，这个操作就会冲突。

<a name="ND8r8"></a>
### next-key lock的概念
next-key lock和SQL语句加锁规则，以及语句的加锁范围，有关系。 

- [ ] 没咋看明白，后面百度查查是什么吧？

<a name="iSiOD"></a>
### 小技巧：线上提交Delete脚本刷数据时，尽量加 limit 
！！！但是请注意<br />在读写分离，主从数据库的架构时，这样些可能会导致主从不一致的情况<br />专栏后续章节“保持主备一致”时提到了这个问题。<br />delete from t where c =10 limit 2;<br />这样不仅可以控制删除数据的条数，让操作更安全，还可以减小加锁的范围。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1637673472254-a5a33012-7542-4109-a2a5-75d02a44c728.png#clientId=u931ac670-1747-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=502&id=u19f9a0f8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=700&originWidth=702&originalType=binary&ratio=1&rotation=0&showTitle=false&size=272021&status=done&style=none&taskId=u44f0c07b-6586-45d2-b254-55f1490064c&title=&width=502.99147033691406)

<a name="TX31Q"></a>
## 19-MySQL有哪些“紧急”提高性能的方法？
现实有很多业务场景，需要临时短期内，提高MySQL性能的一些办法。先让业务跑起来，但存在一定风险。
<a name="pmoa4"></a>
### 案例一：短连接风暴
业务高峰期，数据库连接增长，需要提高连接数，提高并发。<br />修改max_connections参数，提高连接数量。<br />导致，新连接的线程，无法获得CPU，或内存飙高，导致适得其反。
<a name="yG9qC"></a>
#### 解决连接数过多的方案

- **踢掉无用连接**

设置wait_timeout参数, 踢掉没有实际工作的连接。

- **减少连接时的消耗**

设置–skip-grant-tables参数，重新启动，减少连接时的权限校验等。

<a name="XW4oV"></a>
### 案例二：慢查询性能问题
<a name="U3qt3"></a>
#### 索引没设计好
分析SQL执行计划，重新建索引
<a name="sYm75"></a>
#### SQL语句有问题
结合业务，修改SQL语句，重新Hotfix、
<a name="pwC78"></a>
#### MySQL优化器错误的选择了索引
分析执行计划，是不是自己设的索引，SQL执行器没有用，需要重写修改sql或者加索引。<br />（前面有章节讲了怎么解决 错误使用了索引问题）

<a name="VOWz0"></a>
# 四、分库分表相关
<a name="iqGpc"></a>
## 20-MySQL是怎么保证数据不丢失的？
通过WAL机制，记录binLog日志，和redoLog 日志。
<a name="P1i82"></a>
### BinLog日志
事务执行过程中，先把日志写到binlog cache，事务提交的时候，再把binlog cache写到binlog文件中。<br />一个事务的binlog是不能被拆开的，因此不论这个事务多大，也要确保一次性写入。这就涉及到了binlog cache的保存问题。如果这个线程分配的内存放不下，就会暂存到磁盘。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1637675454950-386dc778-a77c-4f80-8b2c-55def3f3110c.png#clientId=u896efb9d-4fa5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=569&id=ua11d7866&margin=%5Bobject%20Object%5D&name=image.png&originHeight=677&originWidth=815&originalType=binary&ratio=1&rotation=0&showTitle=false&size=351336&status=done&style=none&taskId=u1c8ca097-ee12-432a-bffc-b163f28f76f&title=&width=685.5)
<a name="mh3YK"></a>
### redoLog日志
   这里不多讲了，直接上儒猿专栏上的笔记，更加清晰。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1637675239392-7d914992-e8e5-41c5-8aeb-863d7666c057.png#clientId=u896efb9d-4fa5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=696&id=ua4d76011&margin=%5Bobject%20Object%5D&name=image.png&originHeight=777&originWidth=802&originalType=binary&ratio=1&rotation=0&showTitle=false&size=374604&status=done&style=none&taskId=u2b356770-545e-45af-9237-dcd68f01a88&title=&width=718)

<a name="hOQWr"></a>
## 21-MySQL怎么保证主备一致性？
<a name="lE7eA"></a>
### ![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638271257499-63be10c7-d684-4fb0-80c7-837fd1b7b304.png#clientId=ud853a6e3-8ad9-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=442&id=k0uuz&margin=%5Bobject%20Object%5D&name=image.png&originHeight=434&originWidth=670&originalType=binary&ratio=1&rotation=0&showTitle=false&size=244450&status=done&style=none&taskId=u30ba0ae3-7f7c-4c71-9040-1b7e83609e2&title=&width=683)

- 主库和从库之间维持一个长连接。
- DumpThread发送出binglog日志，从库上的IOThread 接收日志，然后生成relayLog，最终同步成功。

<a name="R2BAE"></a>
### 一、建议把备份库设置只读(Readonly)模式

1. 可能有运营人员在上面执行查询SQL，防止误操作。
1. 避免切换时出现bug,可能造成主备不一致。
1. 可以用readOnly状态，判断节点的角色。

<a name="X5Vr1"></a>
### 二、binLog日志的三种格式
一种是statement，一种是row。第三种格式，叫作mixed，其实它就是前两种格式的混合。
<a name="br5xT"></a>
#### statement格式
当binlog_format=statement时，binlog里面记录的就是SQL语句的原文。
<a name="qv6XQ"></a>
#### statement存在的问题：

- 当执行delete from tb where id > 123 limit; 时，删除时加分页语句，可能会造成主从不一致。
- 前面章节的“小技巧” 在线上环境delete 后面加limit，可以防止索引后续多余的扫描。和这点有冲突！！
<a name="QdTRE"></a>
#### row格式
row格式的binlog里没有了SQL语句的原文，是替换成了两个event，还需要借助mysqlbinlog工具才能看到日志实际的内容。
<a name="RVhAi"></a>
#### row存在的问题和好处：

- row格式的缺点是，很占空间。因为row格式记录的不是原语句，而是记录在主库上对应的行表信息变化。
- 比如delete语句删掉10万行，用statement的话就是一个SQL语句被记录到binlog中，占用几十个字节。但用row格式的binlog，要把10万条记录写到binlog中。很占空间，同时写binlog也耗费IO资源，影响执行速度。

**好处：**<br />方便精确恢复数据。
<a name="nocYV"></a>
#### mixed格式
一种折中BinLog方式，会自动判断你的SQL语句是否能引起主备不一致。<br />有存在不一致可能就用row的方式传输，主备一致的话就用statement格式传输。<br />**问题**<br />最合理的方式，但是考虑到数据恢复的场景，线上一般还是用row格式传输。
<a name="old63"></a>
### 
<a name="odXri"></a>
### 三、两台MySQL主节点循环复制同步数据问题怎么解决 ？
<a name="xLdRd"></a>
#### 双M模式图
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638272610497-1821b61a-aa55-4c5e-bf4f-463e06bdec6b.png#clientId=ud853a6e3-8ad9-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=253&id=uc66cb733&margin=%5Bobject%20Object%5D&name=image.png&originHeight=278&originWidth=630&originalType=binary&ratio=1&rotation=0&showTitle=false&size=126978&status=done&style=none&taskId=u84ba8cf9-b0d3-4764-88f7-729b31f1efa&title=&width=574)<br />简单来说：<br />大概意思就是两台MySQL节点，都是主节点，同时提JAVA客户端使用，但是互为主从。那么就会出现两台节点循环复制数据问题，那么就需要设置每个节点的 ServerID不相同， **本机只同步非自身ServerID的binLog日志**，就能解决循环复制问题了。


<a name="B5gT8"></a>
## 22-MySQL的高可用和读写分离
<a name="iJseN"></a>
### MySQL主从延迟问题
<a name="Cwa0p"></a>
#### 同步延迟：
就是主库执行完事务，至 从库执行完事务，之间的这个时间，就叫同步延迟时间。
<a name="SmSHg"></a>
#### HA系统主备切换策略
可靠性优先策略，可用性优先策略

- [ ] 策略算法比较繁琐，有兴趣百度查查吧。

<a name="TXOQK"></a>
### 主备延迟的来源有哪些？

1. 主从网络延迟
1. 从库机器配置比较差
1. 从库压力大，例如运营直接在数据库查询统计数据等，导致从库短暂CPU飙高。
1. 从库的IoThread线程配置较少，同步速度比较慢。
1. 主库执行了“大事务”操作。比如delete大量数据或者执行DDL操作，导致主从延迟。
<a name="uBdMx"></a>
### 如何避免主从延迟
1.避免大事务操作 & delete全量数据会造成大事务 = **都会产生主从延迟**<br />2.配置一主多从，提高从库同步能力<br />3.把BinLog同步到Kafka，利用外部系统做专门的统计操作。

<a name="ONz1x"></a>
### MySQL各版本从库并行复制策略
<a name="dMpZe"></a>
#### 几个关键点：
各个MySQL版本都有不同的并行复制策略，这里简单记住几个关键字。

- MySQL5.6版本之前，MySQL只支持单线程复制，所以主从延迟严重。
- MySQL5.6版本，支持并行复制，只支持的粒度是**按库并行**。
- MySQL5.7版本，做了优化，**可以同步处于“prepare阶段的事务”**（相当于发生在RedoLog生成后）。不用和之前一样非得处于redoLog的“commit”阶段才能执行BinLog同步。把同步动作提前了，就效率更高了。

<a name="RwaFP"></a>
#### 主备流程图
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638275231489-f9702823-bcb9-45b5-9f3f-d73513658a95.png#clientId=ud853a6e3-8ad9-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=321&id=u71276bb2&margin=%5Bobject%20Object%5D&name=image.png&originHeight=441&originWidth=715&originalType=binary&ratio=1&rotation=0&showTitle=false&size=239658&status=done&style=none&taskId=u5644777b-7244-4b6a-b217-1eea69093cc&title=&width=520.5)
<a name="Wat3h"></a>
#### 多线程模型图
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638275425108-9e7657ef-23cd-4f9b-b837-6e732806373a.png#clientId=ud853a6e3-8ad9-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=374&id=u65df0f27&margin=%5Bobject%20Object%5D&name=image.png&originHeight=418&originWidth=582&originalType=binary&ratio=1&rotation=0&showTitle=false&size=188876&status=done&style=none&taskId=ue6021bac-f626-48f7-a416-0d9bf58054a&title=&width=521)<br />**coordinator**：负责读取realyLog(中转日志)和分发事务。<br />**work_n**：表示同步数据到磁盘的工作线程。

<a name="fj5m0"></a>
#### 问题：coordinator按什么规则吧Realy日志分配给Work去执行，为啥要这么按顺序？
因为考虑调CPU时间分片停顿原因，无法保证有顺序的执行不同事务，所以需要把同一个事务下的relay日志分配给相同的Work执行，避免顺序错乱，发生主从不一致。<br />**所以两个基本要求：**<br />1. 不能造成更新覆盖。要求更新同一行的两个事务，必须被分发到同一个worker中。<br />2. 同一个事务不能被拆开，必须放到同一个worker中。

<a name="pZOGc"></a>
### GTID模式实现主从切换策略
<a name="V5Kmf"></a>
#### 如何保证MySQL一主多从的切换正确性？

- 介绍了一主多从的主备切换流程。在这个过程中，从库找新主库的位点是一个痛点。由此，我们引出了MySQL 5.6版本引入的GTID模式，介绍了GTID的基本概念和用法。
- 基于位点切换，但是从库找新主库，很麻烦，所有后面不这么玩。

<a name="Qm2bB"></a>
### 读写分离有哪些坑
<a name="W8Q0g"></a>
#### 客户端读写分离和服务端读写分离有哪些特点？
1. 客户端直连方案: 不需要进过proxy层转发，性能稍好，整体架构简单，但是需要依赖项目，耦合度高，不方便排查问题，出现主备切换，数据库迁移是，需要调整数据库连接信息。<br />2. 带proxy的架构：对客户端透明，连接时不需要关注后端细节，但是对后端运维团队要求更高，需要有高可用的架构，相对比较复杂。

<a name="s5Jqr"></a>
#### 修改后立马查询，读出修改前的数据
这是因为有主备延迟，需要强制读主库，或者实现别的解决办法。

- 强制走主库：强制走主库，例如阿里云ADS有个标记：/*FORCE_MASTER*/ ，放在SQL前就行了
- sleep方案：从业务角度上避免，先由前端显示假数据，然后在查询。
- 判断主备有无延迟：依赖数据库的配置了，知道就好
- 等主库位点方案
- GTID方案：

<a name="d1eEZ"></a>
### 如何判断一个数据库实例是否出来问题？
<a name="sSDBe"></a>
### 并发查询不等于并发连接
并发连接，是MySQLServer层设置的客户端最大连接数<br />并发查询，是InnoDB层设置的最大查询时的并发数。

<a name="lzg0j"></a>
#### 判断方案一：select(1) 不能判断数据库是否正常

- 所以select (1) 这种方式只能判断数据库是否达到了最大连接数，无法判断引擎并发查询数是否以满。
- 一般innodb引擎的并发查询数为20
- 可通过innodb_thread_concurrency 参数去设置

<a name="d92f5"></a>
#### 判断方案二：优先考虑update系统表

- 在系统库（mysql库）里创建一个表，比如命名为health_check，里面只放一行数据，然后定期执行：
- 并且检查这行数据的时间戳是否发生变化，从而判断是否可用。

<a name="GFnLC"></a>
#### 判断方案三：内部统计磁盘利用率

- MySQL可以告诉我们，内部每一次IO请求的时间，MySQL 5.6版本以后提供的performance_schema库，就在file_summary_by_event_name表里统计了每次IO请求的时间。
- 可以通过IO的请求时间判断数据库是否有效。

<a name="B6bQf"></a>
# 五、面试问题记录
<a name="kIgzV"></a>
## 23-问题答疑
<a name="qpJBp"></a>
### 怎么看是否死锁？
执行下面语句，会有部分输出，有一节LATESTDETECTED DEADLOCK，就是记录的最后一次死锁信息。<br />还会输出部分事务信息。
> 查看是否死锁：showengine innodb status


什么是GTID模式? 简单了解一下

<a name="W66Rh"></a>
### 误删数据怎么办？
<a name="bAahR"></a>
#### 误删数据的几种命令

- delete误删数据行
- drop table ,truncate table 语句误删数据表
- drop database 语句误删数据库
- rm 命令误删整个实例

<a name="pBaqu"></a>
#### delete误删数据行
可以用Flashback工具通过闪回把数据恢复回来。

- 原理是: 修改BinLog内容。
- 前提是，配置了binlog_format=row和 binlog_row_image=FULL。

<a name="VNkSY"></a>
#### 误删数据库/表
这种情况下，要想恢复数据，就需要使用全量备份，加增量日志的方式了。<br />前提是：有定期的全量备份，并且实时备份binlog。<br />比如每天半夜去几点去全量备份一次。

<a name="JUZzR"></a>
#### 预防误删库/表的方法

- 账号权限分离
- 规范操作，代码层面去review，把控审批SQL风险。
> show grants 命令查看账户的权限
> 

<a name="Oe9yX"></a>
#### rm删除数据
如果实例被删除了，基本上可以考虑打包滚蛋了。<br />一般不会出现，如果一主多从的话，HA机制会立马选举一个新的写库出来。

<a name="bmZgR"></a>
### 为什么Kill掉SQL进程，依旧存在？
简而言之就是：因为MySQL为了避免MDL读锁，无法释放，不能立马终止，需要让他查询线程返回后才能停止。<br />并不是Kill之后就立马停止，而是通知查询引擎让SQL进入终止状态。<br />有点像暂停Java中的线程，不会立马停止，会让他进入安全节点，和安全区，或者阻塞状态，让线程处于安全状态时，才会去做GC垃圾回收。

<a name="TLxdi"></a>
### 大查询会把MySQL内存用光？
背景是：10G的服务器，全表查询20G的表，会用光所有内存？<br />不会的，兄弟！

- MySQL采用的是边算边发的逻辑，不会在Server端保留完整的结果集，查询一部分就丢到 net_Buffer区域，发送给客户端。
- 如果客户端没有及时接收，就会等待之后在查询。
- 如果涉及到排序，也会触发SortFile机制，把结果集分别存储到多个临时文件中，然后最后并归排序。

<a name="Z7s30"></a>
### InnoDB中的缓存链表也做了LRU冷热算法优化
不多BB了，儒猿专栏讲的更清楚。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638452886620-525dcf66-6dde-4822-aea3-1b9e5e154c86.png#clientId=u2ced8130-e993-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=685&id=u5efc6fdc&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1370&originWidth=1580&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1085710&status=done&style=none&taskId=uf22d5cfc-7614-41f0-82ae-c3087dc906b&title=&width=790)


<a name="S5lqY"></a>
## 24-可不可以使用Join连接查询？
主要是围绕以下两个问题分析

- 不让使用join，使用join有什么问题呢？
   - 在走NLJ连接策略，使用了索引时，可以使用join，但是连表数量不能超过3张。
   - 在走BNL连接策略，没有用索引，如果数据量大，还是建议分开为单表查询。
- 两个大小不同的表做join，应该用哪个表做驱动表呢？
   - 小表驱动大表，就和两个嵌套的for 循环一样。

<a name="DayfP"></a>
### 引出背景
语句：select * from t1 straight_join t2 on (t1.a=t2.a);<br />先假设： t2.a 表走了索引<br />t1 = 驱动表， t2 = 被驱动表 
<a name="Lxo9m"></a>
#### 执行流程
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638453993798-f0da9a0a-9f9f-4cda-a92a-345c212062c5.png#clientId=u2ced8130-e993-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=88&id=u28d33f29&margin=%5Bobject%20Object%5D&name=image.png&originHeight=80&originWidth=670&originalType=binary&ratio=1&rotation=0&showTitle=false&size=95152&status=done&style=none&taskId=ua1fb8ca5-f27a-4b2f-8143-34e19341d1c&title=&width=741)<br />在这条语句里，被驱动表t2的字段a上有索引，join过程用上了这个索引，因此这个语句的执行流程是这样的：<br />1. 从表t1中读入一行数据 R；<br />2. 从数据行R中，取出a字段到表t2里去查找；<br />3. 取出表t2中满足条件的行，跟R组成一行，作为结果集的一部分；<br />4. 重复执行步骤1到3，直到表t1的末尾循环结束。<br />（说白了，就是两个for嵌套循环，只是两个都是走的索引）
<a name="HTv5I"></a>
### NLJ 连表策略 (被驱动表用了索引)
因为被驱动表，用上了a,索引，没有走全表扫描，所以称之为“IndexNested-Loop Join”，简称NLJ。

BNL连表策略（被驱动表没有索引）<br />当被驱动表没有走索引时，这时会触发BlockNested-Loop Join 策略，会把驱动表的值全部查询出来放内存中，然后挨个去遍历 被驱动表，这时就避免了 驱动表每次去走磁盘IO。

<a name="Jxt4y"></a>
### 问题：如果驱动表所有数据，内存放不下，咋弄？
Innodb中采用的是分段机制，把 驱动表 拆分成多段数据去挨个遍历 被驱动表。<br />就是每次查询一批数据放到 join buffer 内存中，去遍历被驱动表。所以join buffer 设置的空间越大，遍历被驱动表的次数越少。<br />join buffer 设置参数：join_buffer_size：1200， 默认是256K

<a name="nXsS7"></a>
### BNL 连表策略
说白了就是被驱动表没有用上索引，只能把驱动表先查询到内存上来。 
<a name="LDLbt"></a>
#### BNL算法流程（joinBuffer分批遍历被驱动表）
被驱动表上没有可用的索引，算法的流程是这样的：<br />1. 把表t1数据读入线程内存joinbuffer中，由于语句中写的是select *，因此是把整个表t1放入了内存；放不下就分批进入joinbuffer，<br />2. 扫描表t2，把表t2中的每一行取出来，跟join_buffer中的数据做对比，满足join条件的，作为结果集的一部分返回。

<a name="Xjbof"></a>
### Join语句怎么优化？
<a name="M6Oku"></a>
#### SQL层面

- 尽量给被驱动表的关联字段加上索引，转成NLJ算法。
- 缩小扫表范围，如缩小查询时间范围，分批查询。减少join_buffer的占用量，减少磁盘IO的使用。
- 采用 straoght_join 来指定连接顺序，让小表驱动大表去扫描。
- 可以先单表查询出来，然后groupby后，转成Hash结构，然后在查出另一个表数据，然后去和Hash表匹配。
<a name="T65Xf"></a>
#### 服务端层面

- 配置join_buffer_size大小，让其减少IO的可能。
- 通过业务手动，设置一张临时宽表，转成单表查询。
- 或者生成一张临时表，然后查询。
<a name="SUAOs"></a>
#### 采用临时表替换成join查询
> create temporary table temp_t like t1;
> alter table temp_t add index(b);
> insert into temp_t select * from t2 where b>=1 and b<=2000;
> select * from t1 join temp_t on (t1.b=temp_t.b);



<a name="yE7V9"></a>
## 25-临时表相关问题汇总
<a name="HxYIG"></a>
### 为什么临时表可以重名？
临时表的特征

- 临时表都是默认为Memory引擎实现。
- 不同session的临时表可以重名的，不同session执行SQL时join优化，不需担心表名重复建表失败问题。
- 不需要担心数据删除问题，客户端连接异常断开，或者数据库重启，都会自动清理数据。

<a name="JElvS"></a>
### 临时表的应用

- 用作复杂查询时，用临时表来优化
- 分库分表系统的跨库查询就是一个典型的使用场景，proxy层从不同库中拿到数据，生成临时表数据返回。
- 用union时，因为要去重，会生成临时表。但是Union All 语法不使用，因为不需要去重，可直接返回。
- 有时候sort_buffer排序，也会生成临时表排序。

<a name="vKbdG"></a>
### 临时表只在线程内自己可以访问，为什么需要写到binlog里面？
在binlog_format='row’的时候，临时表的操作不记录到binlog中！选择binlog_format时会写到BinLog中。

<a name="rN2mH"></a>
### 主备复制时，临时表传过去会被其他同步线程访问到？

- 不会的！如果设置statment格式的binlog日志，哪怕传过去了创建临时表的SQL语句，也只是多此一举，不会被其他Seesion查询，因为这个表示和SeesionID 直接关联的，其他无法访问。

<a name="P7hac"></a>
### 如何优化union语句？
因为这个需要去两张表的并集，且去重，会在系统中生成临时表，如果用union all 就不会有临时表生成，会使用索引，就能提高效率。

<a name="zdu4W"></a>
### 如何优化group by 语句？
例如SQL：select id%10 as m, count(*) as c from t1 group by m;
<a name="E7eXh"></a>
#### 优化方案
1. 如果对group by语句的结果没有排序要求，要在语句后面加 order bynull；<br />2. 尽量让group by过程用上表的索引，确认方法是explain结果里没有Using temporary和 Usingfilesort；<br />3. 如果group by需要统计的数据量不大，尽量只使用内存临时表；也可以通过适当调大tmp_table_size参数，来避免用到磁盘临时表；<br />4. 如数据量太大，使用SQL_BIG_RESULT提示，来告诉优化器直接使用排序算法得到group by的结果。
<a name="pv7Vw"></a>
#### 
<a name="xNuo4"></a>
#### 原理分析：
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638513419852-a2e03fc7-089b-4ddc-956a-fb43f0b8d808.png#clientId=ub974baa3-d90f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=70&id=ud5791d80&margin=%5Bobject%20Object%5D&name=image.png&originHeight=80&originWidth=1009&originalType=binary&ratio=1&rotation=0&showTitle=false&size=133140&status=done&style=none&taskId=u8e89559c-cfe9-4a7b-a819-0caeb03d651&title=&width=886)<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638513612025-5678a6dd-86d8-4e03-9ee0-a18e586c0b7e.png#clientId=ub974baa3-d90f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=629&id=u888745ca&margin=%5Bobject%20Object%5D&name=image.png&originHeight=556&originWidth=710&originalType=binary&ratio=1&rotation=0&showTitle=false&size=205196&status=done&style=none&taskId=uf69e418b-641e-4ebb-b78f-e3d8b9702ac&title=&width=803)<br />指定Order by null :<br />对SQL进行group by操作，会默认对你的字段做排序操作，可能会涉及到using file_sort,会有磁盘IO浪费，可以指定 在SQL后面加 order by null . 

<a name="eo0JJ"></a>
#### 设计独立统计列，利用索引，优化group by
group by的语义逻辑，是统计不同的值出现的个数。但每一行的id%100的结果是无序的，所以就需要有一个临时表，来记录并统计结果。那么如果扫描过程中可以保证出现的数据是有序的，是不是就简单了呢？

- 语句：alter table t1 add column z int generated always as(id % 100), add index(z);
- 查询就变成了：select z ，count(*) as c from t1 group by z;

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638515572525-42049a82-79e4-42d4-b83d-1afe39780315.png#clientId=ub974baa3-d90f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=140&id=u3854ccea&margin=%5Bobject%20Object%5D&name=image.png&originHeight=280&originWidth=1698&originalType=binary&ratio=1&rotation=0&showTitle=false&size=403117&status=done&style=none&taskId=u581a9fd6-3d8e-4d0f-8529-f6a663c293f&title=&width=849)

<a name="ZDTlV"></a>
#### group by优化方法-直接排序
如遇到不能加索引的场景，那要告诉优化器，不要创建临时表，直接用磁盘临时表排序，减少临时表的创建。<br />在group by语句中加入SQL_BIG_RESULT这个提示（hint），就可以告诉优化器：这个语句涉及的数据量很大，请直接用磁盘临时表。<br />语句：select SQL_BIG_RESULT id%100 as m, count(*) as c from t1 group by m;<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638515778608-38882a07-233d-4f69-b839-27c89572d34a.png#clientId=ub974baa3-d90f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=133&id=u22bcc1b3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=266&originWidth=1714&originalType=binary&ratio=1&rotation=0&showTitle=false&size=426579&status=done&style=none&taskId=ufeda3db8-5456-4d89-b43d-bef22564cc5&title=&width=857)


<a name="kFqla"></a>
## 26-都说Innodb好，那还要不要使用Memory引擎？
<a name="jo2bj"></a>
### 使用场景：

- Memory常用在**内存临时表**
- Innodb常用在普通建表的时候、
<a name="dEIXt"></a>
### 有序性：

- Memory是按照插入顺序存储
- Innodb是按照主键ID的顺序存储
<a name="d4s0x"></a>
### 是否B+树分裂

- 数据文件有空洞时（理解为ID有间隙行数据），Innodb插入时，为保证数据有序性，只能在固定位置写入。
- 而Memory表找到空位就可以插入。
<a name="IYoUS"></a>
### 表内存存储结构

- Innodb是数据存储在聚簇索引树上，然后ID值存储在普通索引树上。
> InnoDB 引擎把数据放在主键索引上，其他索引上保存的是主键 id。这种方式，我们称<br />之为索引组织表(Index Organizied Table)。

- Memory是数据和索引分开存储，所以所有索引的“地位”一样。
> Memory 引擎采用的是把数据单独存放，索引上保存数据位置的数据组织形式，我们<br />称之为**堆组织表**(Heap Organizied Table)。

<a name="bbMci"></a>
### 是否有聚簇索引

- Innodb有聚簇索引和普通索引，用普通索引查询时需要走普通索引，然后走聚簇索引，要回表。
- Memory只有一个索引，所有的索引级别都一样，不需要回表。

<a name="NCpZi"></a>
### 是否支持长数据结构

- InnoDB 支持变长数据类型，不同记录的长度可能不同;
- 内存表不支持 Blob 和 Text 字段，并且即使定义了 varchar(N)，实际也当作 char(N)，也就是固定长度字符串来存 储，因此内存表的每行数据长度相同。由于内存表的这些特性，每个数据行被删除以后，空出的这个位置都可以被接下来要插入的 数据复用。

[

](https://blog.csdn.net/qq_20009015/article/details/104957662)
<a name="oAj5b"></a>
### 锁粒度不同
内存表不支持行锁，只支持表锁。因此，一张表只要有更新，就会堵住其他所有在这个表上<br />的读写操作。
<a name="Poobd"></a>
### 其他
数据放在内存中，是内存表的优势，但也是一个劣势。因为，数据库重启的时候，所有的内<br />存表都会被清空。

<a name="vtJxG"></a>
## 27-问题汇总和补充

<a name="RIRmR"></a>
### insert ....select 语句带来的问题
> 语句：insert into t2(c,d) select c,d from t;

- 在可重复读隔离级别下，binlog_format=statement时执行，会给select的表里扫描到的记录和间隙加读锁。
- 如果insert和select的对象是同一个表，则可能造成循环写入。这种情况，需要引入用户临时表来做优化。


<a name="BUZbV"></a>
### 怎么快速复制一张表？
<a name="bFIM4"></a>
#### mysqlDump方法

- 使用mysqldump命令将数据导出成一组INSERT语句。
<a name="fo7BL"></a>
#### 导出CSV文件

- 另一种方法是直接将结果导出成.csv文件。
- 可以用客户端工具，也可以用SQL 语句:
> select * from db1.t where a>900 into outfile '/servertmp/t.csv';

<a name="sLq8J"></a>
#### 物理拷贝磁盘存储文件

- 不能通过直接复制frm 和idb文件的方式去复制，但可以通过：**导出+导入表空间 的方式导出数据**。
- 在**MySQL 5.6版本引入了可传输表空间**(transportable tablespace)的方法，
- 可以通过 导出+导入表空间 的方式，实现物理拷贝表的功能。


<a name="QtiHy"></a>
### grant之后要跟着flush privileges吗？
（看看就好，没必要太关注这个点，又不是DBA，直接看总结。）<br />grant语句会同时修改数据表和内存，判断权限的时候使用的是内存数据。因此，规范地使用grant和revoke语句，是不需要随后加上flush privileges语句的。flush privileges语句本身会用数据表的数据重建一份内存权限数据，所以在权限数据可能存在不一致的情况下再使用。<br />而这种不一致往往是由于直接用DML语句操作系统权限表导致的，所以我们尽量不要使用这类语句。另外，在使用grant语句赋权时，你可能还会看到这样的写法：这条命令加了identified by ‘密码’， 语句的逻辑里面除了赋权外，还包含了：<br />1. 如果用户’ua’@’%'不存在，就创建这个用户，密码是pa；<br />2. 如果用户ua已经存在，就将密码修改成pa。grant super on *.* to 'ua'@'%' identified by 'pa';<br />这也是一种不建议的写法，因为这种写法很容易就会不慎把密码给改了。

<a name="kziT9"></a>
### 为什么公司规范禁止使用分区表？
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638520411131-d90c5b82-a74a-4d3a-9596-c8596ffe9aa6.png#clientId=ub974baa3-d90f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=245&id=uc0102a95&margin=%5Bobject%20Object%5D&name=image.png&originHeight=283&originWidth=702&originalType=binary&ratio=1&rotation=0&showTitle=false&size=190024&status=done&style=none&taskId=u512be9fe-8010-4210-85ab-4a1ca646d70&title=&width=607)<br />建议（了解一下就好了）<br />如果要使用分区表，就不要创建太多的分区。我见过一个用户做了按天分区策略，然后预先创建了10年的分区。这种情况下，访问分区表的性能自然是不好的。有两个问题需要注意：

1. 分区并不是越细越好。实际上，单表或者单分区的数据一千万行，只要没有特别大的索引，对于现在的硬件能力来说都已经是小表了。
1. 分区也不要提前预留太多，在使用之前预先创建即可。比如，如果是按月分区，每年年底时再把下一年度的12个新分区创建上即可。对于没有数据的历史分区，要及时的drop掉。
1. 至于分区表的其他问题，比如查询需要跨多个分区取数据，查询性能就会比较慢，基本上就不是分区表本身的问题，而是数据量的问题或者说是使用方式的问题了。

<a name="r6WDN"></a>
### <br />
<a name="wBHyl"></a>
### 如果用left join的话，左边的表一定是驱动表吗？

- 使用left join时，左边的表不一定是驱动表。
- 在用Join 的时候可以用 straoght_join 指定用哪张表做驱动表。

<a name="aTycE"></a>
### 驱动表判断条件可以写在where后面嘛？
问题：如果两个表的join包含多个条件的等值匹配，是都要写到on里面呢，还是只把一个条件写到on里面，其他条件写到where部分？

- 不能，如果是left join 查询，可能会导致驱动表为null的数据查询不出来。
- 如果是 join  内关联，那就可以写到where后面。

小技巧：尽可能的把筛选被驱动表的数据，减少查出来判断放临时表中的数据。

<a name="JtujP"></a>
### 使用distinct 和 group by 去重复效率谁高？
效率相同！！ 和 count(*) ，count(id) <br />不需要执行聚合函数时，distinct 和group by这两条语句的语义和执行流程是相同的，因此执行性能也相同。
<a name="u7wiP"></a>
#### 执行流程：
这两条语句的执行流程是下面这样的。<br />1. 创建一个临时表，临时表有一个字段a，并且在这个字段a上创建一个唯一索引；<br />2. 遍历表t，依次取数据插入临时表中：如果发现唯一键冲突，就跳过；否则插入成功；<br />3. 遍历完成后，将临时表作为结果集返回给客户端。

<a name="vNsGb"></a>
### 备库自增主键可能会和主库不一致？

- 不会的，即使两个INSERT语句在主备库的执行顺序不同，自增主键字段的值也不会不一致。
- 因为在binLog日志记录时：在insert语句之前，还有一句SETINSERT_ID=1, 指定主键的值。

<a name="zFA2L"></a>
### MySQL中内部的自增ID用完了怎么办?
<a name="QHAN2"></a>
#### 各个ID介绍：

- row_id：如果你没指定主键，则innodb会自动生成row_id作为主键，长度为6个字节。
- Xid：redo log和binlog相配合使用时，它们有一个共同的字段叫作Xid。它在MySQL中是用来对应事务的。
   - Xid是由server层维护的。InnoDB内部使用**Xid**，就是为了能够在InnoDB事务和server之间做关联。
- 但是，InnoDB自己的**trx_id**，是另外维护的。

<a name="TtWJ3"></a>
#### 上限分析：
每种自增id有各自的应用场景，在达到上限后的表现也不同：<br />1. 表自定义的自增id达到上限后，再申请时它的值就不会改变，继续插入报主键冲突的错。<br />2. row_id达到上限后，则会归0再重新递增，如果出现相同的row_id，后写的数据会**覆盖之前的数据**。<br />3. Xid只需要不在同一个binlog文件中出现重复值即可。理论上会出现重复值，但是概率极小，可忽略不计。<br />4. InnoDB的max_trx_id 递增值每次MySQL重启都会被保存起来。<br />5. thread_id是我们使用中最常见的，而且也是处理得最好的一个自增id逻辑了。

- [ ] 当然，在MySQL里还有别的自增id，比如table_id、binlog文件序号等，自己感兴趣在了解

<a name="NgVSK"></a>
### 如果自己设置的自增主键快用完了怎么办？
1.如果用的int类型，那么修改字段类型为bigint，他的存储量远远大于int类型。<br />2.再不济就分表操作，把历史数据按时间段拆分出来。<br />其实：主要还是看你是ID生成的区间快用完了，还是列存储的空间快用完了，还得分情况思考。

<a name="P5SEL"></a>
### 大表怎么修改字段？或者增加字段呢？
因为在大表要加减字段，或者修改类型，是会导致长时间锁表的，所以最好不要直接在线上提交DDL语句去修改，而是通过第三方改表工具（gh-ost），内部原理生成临时表，然后rename改名字切换替换老表的方式去实现的。


<a name="npip4"></a>
# 六、总结回顾

- [ ] 总结章节中重点内容，用特殊颜色标记出来
- [ ] 1个月后，复习一次
- [ ] 自己搜罗常见的20个MYSQL面试题
- [ ] 把Using where ,Using index ,Using template, Using sort_file,这些玩意总结一下。
- [ ] 把那些sort_buffer , join_buffer , chanageBuffer, BufferPool，RedoBuffer, undoBuffer 总结一下

