<a name="O9FiC"></a>
## 00-开篇
<a name="lDIy1"></a>
### 前言
同样是使用 Redis，但是不同公司的“玩法”却不太一样，比如说，有做缓存的，有做数据库的，也有用做分布式锁的。不过，他们遇见的“坑”，总体来说集中在四个方面：

- CPU 使用上的“坑”，例如数据结构的复杂度、跨 CPU 核的访问；
- 内存使用上的“坑”，例如主从同步和 AOF 的内存竞争；
- 存储持久化上的“坑”，例如在 SSD 上做快照的性能抖动；
- 网络通信上的“坑”，例如多实例时的异常网络丢包。

随着这些深入的研究、实战操作、案例积累，我拥有了一套从原理到实战的 Redis 知识总结。这一次，我想把我多年积累的经验分享给你。<br />只关注零散的技术点，没有建立起一套完整的知识框架，缺乏系统观，但是，系统观其实是至关重要的。
<a name="N4rG3"></a>
### 学习方向
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638781441533-4cc6a297-7037-4929-92a1-732e947a7752.png#clientId=u7f727166-ae90-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=447&id=uc0785ef8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=335&originWidth=461&originalType=binary&ratio=1&rotation=0&showTitle=false&size=152313&status=done&style=none&taskId=ue509ae42-62bb-4a29-8c94-3ebddec0cfb&title=&width=615.5)

- “两大维度”就是指系统维度和应用维度。
- “三大主线”也就是指高性能、高可靠和高可扩展（可以简称为“三高”）。[<br />](https://time.geekbang.org/column/article/268247)

首先Redis的各项关键技术的设计原理，能够为你判断和推理问题打下坚实的基础，而且，能从中掌握一些优雅的系统设计规范，例如：run-to-complete 模型，epoll 网络模型，可以应用到你后续你的开发实践中。
<a name="CX2MJ"></a>
### 如何快速的知道该学那些东西？
<a name="ojBMa"></a>
#### 三大主线
接下来就要看“三大主线”的魔力了。别看技术点是零碎的，其实你完全可以按照这三大主线，给它们分下类，就像图片中展示的那样，具体如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638781767826-71ce934a-2861-40cf-af53-e1fe4fefc577.png#clientId=u7f727166-ae90-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=132&id=ue1558bf9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=121&originWidth=453&originalType=binary&ratio=1&rotation=0&showTitle=false&size=66364&status=done&style=none&taskId=u7f2dae63-9282-4e6d-aad2-38b52a45130&title=&width=493.5)<br />这样就有了一个结构化的知识体系。遇到问题时可快速找到影响这些问题的关键因素。<br />其次，在应用维度上，我建议你按照两种方式学习: “应用场景驱动”和“典型案例驱动”，一个是“面”的梳理，一个是“点”的掌握。

<a name="hfD7g"></a>
### Redis对应问题画像图
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638781919109-2c37d0cb-29f3-4ed3-a666-6669c2c1caae.png#clientId=u7f727166-ae90-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=493&id=u0f5ec4ca&margin=%5Bobject%20Object%5D&name=image.png&originHeight=452&originWidth=641&originalType=binary&ratio=1&rotation=0&showTitle=false&size=319990&status=done&style=none&taskId=u6cb9cdfa-0143-45bc-8bb3-b0b8f22a4af&title=&width=699.5)
<a name="GMtYv"></a>
### 课程设计三部曲

1. 基础篇：简历网状只是结构
1. 实践篇：场景和案例驱动
1. 未来篇：解锁新特性

<a name="Hcn88"></a>
## 01-一个基本键值数据库包含什么？
<a name="Qfip7"></a>
### 基本结构
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638783138306-aad747ab-3844-44dc-b93f-919395ab3a82.png#clientId=u7f727166-ae90-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=638&id=u9662f5f7&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1276&originWidth=2066&originalType=binary&ratio=1&rotation=0&showTitle=false&size=925326&status=done&style=none&taskId=uc41175bc-01a3-4ec2-985d-53702b23874&title=&width=1033)<br />Redis存储类型支持：<br /> Redis支持的 value 类型包括了 String、哈希表、列表、集合等。<br />PUT/GET/DELETE/SCAN 是一个键值数据库的基本操作集合。<br />演进变化：<br />调用方式有所变化，数据模型有变化，持久化有变化。
<a name="yM4uA"></a>
### 通常访问模式有两种
和MySQL，和其他独立系统，一样，都是这么玩。
<a name="ATeEW"></a>
#### 1.函数库调用：
一种是通过函数库调用的方式供外部应用使用，比如，上图中libsimplekv.so，就是以动态链接库的形式链接到我们自己的程序中，提供键值存储功能；
<a name="l3aaA"></a>
#### 2.网络调用：
这种形式提供广泛的键值存储服务。在上图中，我们可以看到，网络框架中包括 Socket Server 和协议解析。

- [ ]  Redis 现有的客户端和通信协议，有哪些？

<a name="p1xJX"></a>
## 02-数据结构：Redis有哪些慢操作？
<a name="VbLPR"></a>
### 不简单的Value存储类型之 底层数据结构
 String（字符串）、List（列表）、Hash（哈希）、Set（集合）和 Sorted Set（有序集合）吗？”其实，这些只是 Redis 键值对中值的数据类型，也就是数据的保存形式。但实际底层存储的数据结构就没有那么简单了。
<a name="NgGJl"></a>
#### Redis数据类型对应的底层数据结构
动态字符串，双向链表，压缩列表，哈希表，跳表，整体数组。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638784519472-309444bc-e932-45fb-ad31-c66e9d916920.png#clientId=u7f727166-ae90-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=198&id=u0ea3e8d1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=378&originWidth=1292&originalType=binary&ratio=1&rotation=0&showTitle=false&size=297122&status=done&style=none&taskId=u53235eb6-0ad6-479e-b2fc-812e6c228e7&title=&width=678)

- String 类型的底层实现只有一种数据结构，也就是简单动态字符串。
- 而 List、Hash、Set 和 Sorted Set 这四种数据类型，都有两种底层实现结构。
- 通常把这四种类型称为集合类型，它们的特点是一个键对应了一个集合的数据。
- ![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638795920920-8a1cb318-e61e-49c3-b417-b41617e94094.png#clientId=uc69b1aff-0693-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=325&id=ubb89d5f1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=558&originWidth=864&originalType=binary&ratio=1&rotation=0&showTitle=false&size=259719&status=done&style=none&taskId=u18cbb048-3bba-4118-9591-3aaee2c1b3e&title=&width=503)
<a name="XHJPi"></a>
#### 衍生出的问题:

- [ ] 这些数据结构都是值的底层实现，键和值本身之间用什么结构组织？

- [ ] 为什么集合类型有那么多的底层结构，它们都是怎么组织数据的，都很快吗？

- [ ] 什么是简单动态字符串，和常用的字符串是一回事吗？

<a name="tRyLM"></a>
### Key和Value用什么结构组织去实现
为了实现从键到值的快速访问，Redis 使用了一个**全局哈希表**来保存所有键值对。<br />哈希表底层其实是个数组，数组的每个元素称为：“哈希桶”。哈希桶中保存了键值对数据 entry指针。

哈希桶中的元素保存的并不是值本身，而是指向具体值的指针。不管值是 String，还是集合类型，哈希桶中的元素都是指向它们的指针。<br />哈希桶中的 entry 元素中保存了*key和*value指针，分别指向了实际的键和值，即使值是一个集合，也可以通过*value指针被查找到。	（有点像ConcurrentHashMap的结构，数组+链表）<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638792955179-bbdb0654-7c29-4c7c-9800-2068fcf95df9.png#clientId=uc69b1aff-0693-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=678&id=u1204cf05&margin=%5Bobject%20Object%5D&name=image.png&originHeight=678&originWidth=1324&originalType=binary&ratio=1&rotation=0&showTitle=false&size=295674&status=done&style=none&taskId=ud802c6fb-f105-4bfe-b8ea-88828a712d1&title=&width=1324)<br />寻址时间复杂度为 O(1),  所有耗时操作主要在Hash计算上面。
<a name="M0MQ2"></a>
### 全局哈希表冲突问题 和 reHash 带来的阻塞问题
当数据量大，Redis操作可能变慢，那就是哈希表的冲突问题和 rehash 可能带来的操作阻塞。<br />**总结解决方案：**<br />1.哈希桶下挂冲突数据链表。 <br />2.哈希表扩容，分散哈希取模范围，减少冲突。

<a name="vY7tq"></a>
#### Redis 解决哈希冲突的方式
Redis 解决哈希冲突的方式，就是链式哈希。链式哈希也很容易理解，就是指同一个哈希桶中的多个元素用一个链表来保存，它们之间依次用指针连接。<br />说白了，和HashMap一样，一旦Hash值冲突了，就放数组节点下挂一个链表存储数据。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638793392191-6df2710a-8048-4133-8bc3-621a100f7d40.png#clientId=uc69b1aff-0693-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=1178&id=u290e63f6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1178&originWidth=1308&originalType=binary&ratio=1&rotation=0&showTitle=false&size=670511&status=done&style=none&taskId=u80e0c932-e274-4cac-b64b-557ee6601e2&title=&width=1308)<br />同时衍生出一个问题， 如果冲突数过多，链表长度过长，会带来查询性能问题，怎么办呢？？<br />答案就是，全局哈希表扩容
<a name="Klvm2"></a>
### 
<a name="z30Ci"></a>
### Redis全局哈希表的扩容问题（ReHash操作）
当哈希冲突频繁的时候，这时 Redis的全局哈希表 就会自动扩容，分散哈希值存储于不同的桶上，也就是ReHash。  （这点和HashMap不一样，Map是转成链表转红黑二叉树。超过负载因子才扩容）<br />Redis 会对哈希表做 rehash 操作。rehash 也就是增加现有的哈希桶数量，让逐渐增多的 entry 元素能在更多的桶之间分散保存，减少单个桶中的元素数量，从而减少单个桶中的冲突。**那具体怎么做呢？**
<a name="JAF89"></a>
#### ReHash的具体实现 （全局哈希表扩容）

- Redis默认使用两个全局哈希表：哈希表 1 和哈希表 2。当一个需要扩容时，另一个容量*2倍，把另一个哈希表的数据复制到扩容后的哈希表中。
- 一开始，当你刚插入数据时，默认使用哈希表 1，此时哈希表 2 并没有被分配空间。随着数据逐步增多，Redis 开始执行 rehash，这个过程分为三步： ( 有点像复制清除算法)
      1. 给哈希表 2 分配更大的空间，例如是当前哈希表 1 大小的两倍；	
      1. **把哈希表 1 中的数据重新映射并拷贝到哈希表 2 中； （耗时阻塞）**
      1. 释放哈希表 1 的空间。
- 到此，我们就可以从哈希表 1 切换到哈希表 2，用增大的哈希表 2 保存更多数据，而原来的哈希表 1 留作下一次 rehash 扩容备用。

<a name="TQ1SS"></a>
#### 渐进式 rehash策略（优化Hash表的复制阻塞过程）
上面第二步，复制Hash表是一个很耗时的操作，为了避免线程阻塞，于是采用了渐进式Rehash策略。<br />简单来说：在第二步拷贝数据时，Redis 仍然正常处理客户端请求，每处理一个请求时，从哈希表 1 中的第一个索引位置开始，顺带着将这个索引位置上的所有 entries 拷贝到哈希表 2 中；等处理下一个请求时，再顺带拷贝哈希表 1 中的下一个索引位置的entries。如下图所示：<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638794910160-9df918f8-fc08-431e-8883-3a6ac2a9b4dc.png#clientId=uc69b1aff-0693-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=528&id=u1123e0c1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=890&originWidth=1346&originalType=binary&ratio=1&rotation=0&showTitle=false&size=590203&status=done&style=none&taskId=u55755129-ef05-464d-8cc1-0352c13c595&title=&width=799)<br />这样就巧妙地把一次性大量拷贝的开销，分摊到了多次处理请求的过程中，避免了耗时操作，保证了数据的快速访问。

<a name="Dto4P"></a>
### 集合数据操作效率（Set,Hash,SortSet,List的效率）
分别聊聊 集合类型 的 底层数据结构 和 操作复杂度。

- 集合类型的底层数据结构主要有 5 种：整数数组、双向链表、哈希表、压缩列表 和 跳表。
- 哈希表上面以经说了，数组和双向链表比较常见，就不多BB了，他们操作特点都是：顺序读写，时间复杂度基本都为：O(n)。  重点记一下压缩列表和跳表。

<a name="kqFIZ"></a>
#### 压缩列表：
（说白了，就是底层是数组，但是数组前面有长度，偏移量等统计数据，方便快速找 表头和表尾）<br />**但不同点是**，压缩列表表头有三个字段： zlbytes、zltail 和 zllen，分别表示列表长度、列表尾的<br />偏移量和列表中的 entry 个数； 压缩列表在表尾还有一个 zlend，表示列表结束。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638795626967-82f03079-59f1-4e0a-971a-95773c782724.png#clientId=uc69b1aff-0693-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=254&id=u6af04785&margin=%5Bobject%20Object%5D&name=image.png&originHeight=254&originWidth=1360&originalType=binary&ratio=1&rotation=0&showTitle=false&size=143562&status=done&style=none&taskId=ud6857a73-9ce2-4ca5-850a-63950ba593e&title=&width=1360)<br />**优点是**：<br />如果要查找定位第一个元素和最后一个元素，可以通过表头三个字段的长度直接定位，复杂度是 O(1)。而查找其他元素时，就没有这么高效了，只能逐个查找，此时的复杂度就是 O(N) 了。

<a name="Wlqx0"></a>
#### 跳表：
首先底层是 有序的链表 。查找起来非常慢，所以就在链表的基础上，增加了多级索引，通过索引位置的跳转，定位到大概位置。	有点像 二分法，又有点像二叉树。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638795866279-504ac531-ce65-4444-93bb-e030897dc725.png#clientId=uc69b1aff-0693-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=870&id=u14cfe162&margin=%5Bobject%20Object%5D&name=image.png&originHeight=870&originWidth=1354&originalType=binary&ratio=1&rotation=0&showTitle=false&size=551722&status=done&style=none&taskId=u2c8bec11-9fb5-4501-9503-e16f75c2ebf&title=&width=1354)
<a name="Vr5Qm"></a>
#### 四句口诀 避免高复杂度操作

- **单元素操作是基础；**
   - 指每种集合对单个数据实现增删改查操作。时间复杂度一般为O(1)
- **范围操作非常耗时；**
   - 指集合类型的遍历操作，可以返回集合中所有数据，时间复杂度一般为O(n)
   - 2.8版本之后，引入了Scan游标，渐进式遍历，每次只返回部分数据，避免阻塞。
- **统计操作通常高效；**
   - 指统计集合个数，例如LLEN，SCARD这类操作，时间复杂度为O(1)。 （跳表实现）
- **例外情况只有几个；**
   - 某些数据结构的特殊记录，例如压缩列表和双向链表都会记录表头 和 表位偏移量。对于List类型的LPOP，RPOP，RPUSH等是在表头，表尾做操作的指令，可直接用偏移量定位，时间复杂度O(1)，从而可以快速实现。
<a name="XyvHH"></a>
### 底层结构总结和复习：
学习了 Redis 的底层数据结构，这既包括了 Redis 中用来保存每个键和值的全局哈希表结构，也包括了支持集合类型实现的双向链表、压缩列表、整数数组、哈希表和跳表这五大底层结构。<br />Redis 之所以能快速操作键值对，一方面是因为 O(1) 复杂度的哈希表被广泛使用，包括String、Hash 和Set，它们的操作复杂度基本由哈希表决定，另一方面，Sorted Set 也采用了 O(logN) 复杂度的跳表。不过，集合类型的范围操作，因为要遍历底层数据结构，复杂度通常是 O(N)。这里，我的建议是：用其他命令来替代，**例如可以用 SCAN 来代替，避免在 Redis 内部产生费时的全集合遍历操作。**<br />当然，我们不能忘了复杂度较高的 List 类型，它的两种底层实现结构：双向链表和压缩列表的操作复杂度都是 O(N)。因此，我的建议是：因地制宜地使用 List 类型。例如，既然它的 POP/PUSH 效率很高，那么就将它主要用于 FIFO 队列场景，而不是作为一个可以随机读写的集合。

<a name="b8hjN"></a>
## 03-高性能IO模型：为什么单线程能那么快？

- Redis是基于内存型的KV数据库。底层有链表，全局哈希表，跳表等高效存储数据结构。
- Redis单线程操作，减少上下文的复用，这里指的是 Redis的网络IO和键值对读写是由一个线程完成，咱们把这个称之为 **主线程**。
- 底层采用IO多路复用机制，使其在网络IO操作中能处理大量并发请求，避免IO阻塞，实现高吞吐。
<a name="cBMPF"></a>
### Redis为什么用单线程？

1. 避免上下文的切换，里面用IO多路复用来提升效率。
1. 多线程的的创建过程，开销大。开启多线程数达到一定阈值，吞吐率就睡适得其反。（为啥不用线程池？）
1. 多线程模式面临共享资源的并发访问控制问题，可能带来会加锁串行，影响效率。

<a name="OjOUw"></a>
### 基于IO模型与阻塞点
说实话，有点复杂，没有那么简单，记住几个关键字就好了，阻塞模型转为 非阻塞模型，依赖Linux系统的select和epoll机制。去实现IO多路复用。

- [ ] 去搜搜select 和 epoll ，看是个什么玩意？
<a name="rBadW"></a>
#### 阻塞型模式
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638866089847-330f7d49-f9f2-4a63-a020-0586b65766b1.png#clientId=uf520692b-d1f1-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=772&id=ua06b03f0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=772&originWidth=1260&originalType=binary&ratio=1&rotation=0&showTitle=false&size=404351&status=done&style=none&taskId=u9f66a491-caba-4f14-946c-dfe85367945&title=&width=1260)<br />在这里的网络 IO 操作中，有潜在的阻塞点，分别是 accept() 和 recv()。当 Redis监听到一个客户端有连接请求，但一直未能成功建立起连接时，会阻塞在 accept() 函数这里，导致其他客户端无法和 Redis 建立连接。
<a name="KAyB4"></a>
#### 非阻塞模式
专栏没说太多，简单概括就是，Redis中设置了非阻塞式的IO模型，采用多路复用机制。<br />Socket 网络模型的非阻塞模式设置，依赖三个函数。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638866254537-f9038708-e542-45a5-83b1-9eb1699c5c41.png#clientId=uf520692b-d1f1-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=410&id=udf1eef26&margin=%5Bobject%20Object%5D&name=image.png&originHeight=410&originWidth=1268&originalType=binary&ratio=1&rotation=0&showTitle=false&size=349304&status=done&style=none&taskId=ue6d9f7ad-18fb-41e1-a463-17d2b72d326&title=&width=1268)
<a name="rZ30U"></a>
### 基于多路复用的高性能IO模型：select/epoll机制
Linux 中的 IO 多路复用机制是指一个线程处理多个 IO 流，就是我们经常听到的select/epoll 机制。<br />简单来说，在 Redis 只运行单线程的情况下，该机制允许内核中，同时存在多个监听套接字和已连接套接字。内核会一直监听这些套接字上的连接请求或数据请求。一旦有请求到达，就会交给 Redis 线程处理，这就实现了一个 Redis 线程处理多个IO 流的效果。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638867238291-a43032a9-9cee-499f-b0ce-09a9442139c7.png#clientId=uf520692b-d1f1-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=916&id=u9e49edee&margin=%5Bobject%20Object%5D&name=image.png&originHeight=916&originWidth=1226&originalType=binary&ratio=1&rotation=0&showTitle=false&size=568286&status=done&style=none&taskId=u6ce09903-021a-4128-8240-5bb34d5f96a&title=&width=1226)<br />为了在请求到达时能通知到 Redis 线程，select/epoll 提供了基于事件的回调机制，即针对不同事件的发生，调用相应的处理函数。

<a name="C1NYS"></a>
## 04-AOF日志：如何避免Redis数据丢失？
Redis的两大持久化机制：AOF日志 和 RDB**快照**。
<a name="WHzoL"></a>
### AOF日志是如何实现的？

- AOF日志是先做执行操作，后记录AOF日志的 和 MySQL那种WAL(预写式日志)相反。
- 所以如果服务器宕机，有可能丢失当前操作的日志，也就是上一次的命令日志。
<a name="PSSDR"></a>
#### 后写式AOF日志的好处？

1. 可以减少错误命令和日志的产生，因为先操作，后记录日志，就不需要去检查操作语法是否正确了。
1. 不会阻塞当前主线程的操作，避免了当前命令的阻塞，但是也有可能会会给下一个命令带来阻塞风险。

<a name="d09VT"></a>
### AOF的日志三种持久化策略

- **Always**，同步写回：每个写命令执行完，立马同步地将日志写回磁盘；
- **Everysec**，每秒写回：每个写命令执行完，只是先把日志写到 AOF 文件的内存缓冲区，每隔一秒把缓冲区中的内容写入磁盘；
- **No**，操作系统控制的写回：每个写命令执行完，只是先把日志写到 AOF 文件的内存缓冲区，由操作系统决定何时将缓冲区内容写回磁盘。 写到OS Cache，让操作系统自己存储，这种情况Redis宕机不会丢失，操作系统宕机就会丢失日志。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638877861005-167d1d14-4f18-4eab-9bc7-408613aa04e5.png#clientId=udedb0e05-f0cd-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=340&id=u0d6383dd&margin=%5Bobject%20Object%5D&name=image.png&originHeight=340&originWidth=1248&originalType=binary&ratio=1&rotation=0&showTitle=false&size=371487&status=done&style=none&taskId=ud291d2a2-ff5c-48a8-a2b0-60331e050b7&title=&width=1248)

<a name="ksc88"></a>
### AOF的重写机制：解决日志增量文件过大问题
因为AOF是追加记录操作日志，可能会出现日志文件过大问题。所以AOF重写机制简单来说，就是把对相同列表的操作，都合并了，只记录最后一次修改的日志。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638877946556-277304e9-7aee-43a5-85e9-a21280133616.png#clientId=udedb0e05-f0cd-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=420&id=u04c5f756&margin=%5Bobject%20Object%5D&name=image.png&originHeight=420&originWidth=1300&originalType=binary&ratio=1&rotation=0&showTitle=false&size=332903&status=done&style=none&taskId=uc4d9cd91-1433-4ffd-8556-549eb2bd132&title=&width=1300)
<a name="fNeuV"></a>
#### 
<a name="dxg8l"></a>
#### AOF异步拷贝持久化：解决日志重写阻塞问题
上面说到AOF重写后，日志文件也必定不小，所以重写过程是有 主线程 fork出来的一个子线程去操作的，相当与异步操作，减少主线程的IO阻塞。<br />重写的过程总结为“一个拷贝，两处日志”。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638878193700-e4d00710-4a52-4f99-ba1a-65bf4a223729.png#clientId=udedb0e05-f0cd-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=662&id=u6584fca4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=662&originWidth=1296&originalType=binary&ratio=1&rotation=0&showTitle=false&size=372682&status=done&style=none&taskId=ub4806631-e9d5-44f8-b667-bb81cc0bb7b&title=&width=1296)<br />总结来说，每次 AOF 重写时，Redis 会先执行一个内存拷贝，用于重写；然后，使用两个日志保证在重写过程中，新写入的数据不会丢失。而且，因为 Redis 采用额外的线程进行数据重写，所以，这个过程并不会阻塞主线程。

<a name="W7BVs"></a>
## 05-内存快照：Redis如何利用RDB快照恢复？
因为AOF日志记录的只是操作命令日志，当Redis宕机恢复时，需要一个个执行命令太慢，所以Redis提供RDB快照日志，也可以理解为在固定时刻内的快照，能够快速恢复到某一时间点。
<a name="KCJfm"></a>
### RDB文件两种持久化策略
Redis提供了两个命令来生成RDB文件分别是：save，bgsave。<br />save：在主线程中执行，会导致阻塞；<br />bgsave：**创建一个子进程**，专门用于写入 RDB 文件，避免主线程的阻塞。（默认配置）

<a name="phPbU"></a>
### RDB文件做快照时，文件能修改嘛？
理论上，主线程能够正确接收请求，但是为了包装快照完整性，只能处理 读请求，不能修改正在执行快照的数据。但是为了提高Redis的效率，就会借助操作系统提供的** 写时复制技术**（Copy-On-Write, COW），在执行快照的同时，正常处理写操作。   （有点像多版本快照机制）<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638878948173-e1ec693a-16e2-44c1-b42d-be42eb631824.png#clientId=udedb0e05-f0cd-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=774&id=uf0bb384c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=774&originWidth=1220&originalType=binary&ratio=1&rotation=0&showTitle=false&size=367232&status=done&style=none&taskId=uc2eed834-c1f8-4b94-990b-4eb1a399233&title=&width=1220)

<a name="IecAl"></a>
### RDB记录增量快照：只记录修改的数据
为了解决RDB文件过大，fork子线程过度消耗问题，于是采用，只记录修改后的数据。减少性能消耗。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638879163987-49bb0475-e47a-4a67-8a03-b8ace1aa2b06.png#clientId=udedb0e05-f0cd-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=592&id=ubf3042c5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=592&originWidth=1262&originalType=binary&ratio=1&rotation=0&showTitle=false&size=474258&status=done&style=none&taskId=uc4b12011-7b6a-4c9a-a0cc-f92692b8cae&title=&width=1262)
<a name="rvlOB"></a>
### RDB 和 AOF 混合日志：保证数据完整性
AOF会丢失上一条操作日志，RDB会丢掉最近还未快照的数据，所以Redis 4.0版本中提出来 混合使用AOF日志 和 RDB内存快照 的方法。<br />快照不用很频繁地执行，这就避免了频繁 fork 对主线程的影响。而且，AOF日志也只用记录两次快照间的操作，也就是说，不需要记录所有操作了，因此，就不会出现文件过大的情况了，也可以避免重写开销。<br />又能快速恢复，又能记录每次修改的操作日记，完美！但是最后一次操作还是有可能丢失。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638879308046-6492b358-9365-4ddc-b34e-65a38f385f68.png#clientId=udedb0e05-f0cd-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=888&id=ub57dc881&margin=%5Bobject%20Object%5D&name=image.png&originHeight=888&originWidth=1258&originalType=binary&ratio=1&rotation=0&showTitle=false&size=590686&status=done&style=none&taskId=ua9e1ca84-0959-4f3f-bc5a-f02c91fccf4&title=&width=1258)

<a name="k5v8K"></a>
## 06-数据同步：主从库如何实现数据一致性
<a name="Rio7R"></a>
### 主从库的职责
Redis提供了主从库模式，保证数据一致性。主从库之间采用读写分离的模式。

   - 读操作：主库、从库都可以接收；
   - 写操作：首先到主库执行，然后，主库将写操作同步给从库。** 从源头上规定只能主库修改**

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638879724919-2532e38e-77c1-4868-8435-a971e1b9dea0.png#clientId=udedb0e05-f0cd-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=886&id=uc8638e86&margin=%5Bobject%20Object%5D&name=image.png&originHeight=886&originWidth=1254&originalType=binary&ratio=1&rotation=0&showTitle=false&size=435587&status=done&style=none&taskId=ub819af20-849b-42ab-8cb7-4d812b470e4&title=&width=1254)

<a name="VUU9r"></a>
### Redis主从库第一次同步的三个阶段
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638879918088-64532342-3245-429e-8995-5f6db23b1bb9.png#clientId=udedb0e05-f0cd-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=648&id=u698689cd&margin=%5Bobject%20Object%5D&name=image.png&originHeight=648&originWidth=1274&originalType=binary&ratio=1&rotation=0&showTitle=false&size=416172&status=done&style=none&taskId=ucd27015c-5d6b-4e6e-881e-7d6c2c5eeb9&title=&width=1274)<br />简单来说：就是主库建立连接后，发送RDB文件过去，然后从库接受RDB文件，然后清空先有数据。然后主库把新产生的AOF日志记录通过repl buffer 发送给从库同步。

<a name="OiO6J"></a>
### 主-从-从 级联模式分担从主库的压力
**主从从模式 和 长连接命令传输  减少主库的压力。**

- 简单来说：如果一主多从，主库先同步了一份RDB文件给从库，然后从库就可以那RDB文件去同步给另外一个从库了，这样就减少了主库 fork子线程 传输RDB文件的压力的。
- 基于长连接的命令传播，可以避免频繁建立连接的开销。 （网络阻塞会出现数据不一致的情况）

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638880340297-7dc3a59b-80d4-4156-bddd-9e8d8d32b623.png#clientId=udedb0e05-f0cd-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=718&id=u4ec6415e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=718&originWidth=1308&originalType=binary&ratio=1&rotation=0&showTitle=false&size=297205&status=done&style=none&taskId=u35fbee04-39e6-433e-96f5-c479d4d4b05&title=&width=1308)

<a name="n0eG5"></a>
### 主从库之间网络断开怎么办？

- 在Redis2.8之前，只能完整的全量复制。
- 在Redis2.8之后，就可以通过只同步 repl_log_buffer同步新修改的数据的方式，去同步数据。

  <br />当主从库断连后，主库会把断连期间收到的写操作命令，写入**repl_log_buffer**当中，repl_backlog_buffer 是一个环形缓冲区，主库会记录自己写到的位置，从库则会记录自己已经读到的位置。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638880734296-25034789-e7cd-4930-98bc-cd723d817360.png#clientId=udedb0e05-f0cd-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=424&id=u88c7aeaa&margin=%5Bobject%20Object%5D&name=image.png&originHeight=424&originWidth=1284&originalType=binary&ratio=1&rotation=0&showTitle=false&size=350104&status=done&style=none&taskId=u1781047a-25c8-4442-bced-3f3181f3ff8&title=&width=1284)
<a name="yyPef"></a>
### Redis 的主从库同步的三种基本原理
全量复制、基于长连接的命令传播，以及增量复制

<a name="XDTC9"></a>
### 总结建议
简单来说：单机实例不要太大，主要网络同步时的配置参数。

-     全量复制虽然耗时，但是对于从库来说，如果是第一次同步，全量复制是无法避免的，所以，我给你一个小建议：一个 Redis 实例的数据库不要太大，一个实例大小在几 GB 级别比较合适，这样可以减少 RDB 文件生成、传输和重新加载的开销。另外，为了避免多个从库同时和主库进行全量复制，给主库过大的同步压力，我们也可以采用“主 - 从 - 从”这一级联模式，来缓解主库的压力。
-     我特别建议你留意一下 repl_backlog_size 这个配置参数。如果它配置得过小，在增量复制阶段，可能会导致从库的复制进度赶不上主库，进而导致从库重新进行全量复制。所以，通过调大这个参数，可以减少从库在网络断连时全量复制的风险。

<a name="mjjY2"></a>
## 07-哨兵机制：主库挂了，如何不间断服务？
主库挂了后，涉及了三个问题：<br />1.主库真的挂了吗？<br />2.该选择哪个从库作为主库？<br />3.怎么把新主库的相关信息通知给从库和客户端呢？<br />从上面三个问题引出哨兵机制

<a name="IDCzl"></a>
### 哨兵机制的基本流程
哨兵是运行在特殊模式下的 Redis 进程，主从库实例运行的同时，它也在运行。（哨兵也是Redis实例）<br />哨兵主要负责的就是三个任务：监控、选主（选择主库）和通知。<br />哨兵会周期性的向Redis主从实例发送ping命令，如果没有反应，则判断为下线。
<a name="WykWv"></a>
#### 主节点宕机后流程：
主库下线则要自动切换主库，新主节点选举出后，在通知各个从节点，让他们执行repl命令，和新主库建立连接，并进行数据复制。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638882027425-aa394fed-237b-49c6-a18d-92d05d24d90e.png#clientId=udedb0e05-f0cd-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=432&id=uf9d04e3e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=432&originWidth=1246&originalType=binary&ratio=1&rotation=0&showTitle=false&size=216289&status=done&style=none&taskId=ud24b7767-b4f6-4af5-88fd-a79a4269ddb&title=&width=1246)

<a name="TuGh2"></a>
### 如何解决 哨兵误判主节点下线？
如何减少误判呢？那就是哨兵节点通常采用集群模式，让多个哨兵节点对主库发起ping判断，按照少数服从多数的原则去判断，主机是否下线。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638882269594-d755926f-cf4a-40e3-b5fd-cbd0648135e9.png#clientId=udedb0e05-f0cd-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=536&id=ub666ee19&margin=%5Bobject%20Object%5D&name=image.png&originHeight=536&originWidth=1270&originalType=binary&ratio=1&rotation=0&showTitle=false&size=288731&status=done&style=none&taskId=u9d1b4ca1-0369-4b50-ad51-d70bacd5fdb&title=&width=1270)
<a name="lHgw5"></a>
### 如何选举主节点，选举规则是什么？
我把哨兵选择新主库的过程称为“筛选 + 打分”规则。 反正是围绕着 效率最好的方式来选举。<br />通过：**网络**，**同步速度**，旧主库的**同步程度**来打分。 ![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638882471082-9406bc77-f217-4ba9-afc7-8c130df5bc14.png#clientId=udedb0e05-f0cd-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=602&id=u3a56bba4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=602&originWidth=1308&originalType=binary&ratio=1&rotation=0&showTitle=false&size=275680&status=done&style=none&taskId=ud44aaf62-9d87-420d-9923-ac109f28089&title=&width=1308)

<a name="CdM8E"></a>
## 08-哨兵挂了，主库还能切换吗？
<a name="ms7kw"></a>
### 哨兵集群组成机制：pub/sub

- 配置哨兵机制集群的时候，通常只要设置 **主库的IP** 和 **端口**，没配置其他哨兵连接信息。
<a name="cjkWR"></a>
#### 各个哨兵之间的信息传递
哨兵实例之间可以相互发现，要归功于 Redis 提供的 pub/sub 机制，也就是发布 / 订阅 机制。<br />在主库上订阅消息，获得其他哨兵的连接信息，当多个哨兵实例都在主库上做了发布订阅操作之后，他们就能彼此知道各个哨兵的信息了。（发布在同一个频道上，和Java客户端的发布订阅消息区分开来。）<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638883443627-1e88f9ff-ba1d-44be-9c58-fd32a8604079.png#clientId=udedb0e05-f0cd-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=690&id=u6b417118&margin=%5Bobject%20Object%5D&name=image.png&originHeight=690&originWidth=1156&originalType=binary&ratio=1&rotation=0&showTitle=false&size=339612&status=done&style=none&taskId=u0dd4be1a-25b6-44c9-851b-d3919df62f5&title=&width=1156)
<a name="UqhwL"></a>
#### 哨兵如何知道从库的信息呢？INFO命令
这是由哨兵 **向主库发送 INFO 命令** 来完成的，主库返回从库信息给哨兵。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638883430845-a7a8ac39-fd49-4564-bebd-aa760f4d46ac.png#clientId=udedb0e05-f0cd-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=734&id=uc8306c93&margin=%5Bobject%20Object%5D&name=image.png&originHeight=734&originWidth=1104&originalType=binary&ratio=1&rotation=0&showTitle=false&size=286582&status=done&style=none&taskId=u56601732-1c78-443d-ba10-18efcc03b07&title=&width=1104)

<a name="IML0A"></a>
#### 客户端连接哨兵，实现事件通知？
哨兵就是一个运行在特定模式下的 Redis 实例，所以对客户端也提供pub/sub 机制，客户端可以从哨兵订阅消息和发布消息。  （知道就好了！！不会问的）<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638883718160-2498d099-a829-4c78-82dd-1a254eed9773.png#clientId=udedb0e05-f0cd-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=800&id=u65c93cb0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=800&originWidth=1316&originalType=binary&ratio=1&rotation=0&showTitle=false&size=724723&status=done&style=none&taskId=u1ae9d4f2-4977-4c3d-9e0e-83a882895dd&title=&width=1316)
<a name="nw4fu"></a>
#### Leader选举机制：由那个哨兵执行主从切换呢？
确定由那个哨兵执行主从切换的过程，和主库判断下线的过程类似，也是 投票仲裁 的过程，具体的细节咱们不多逼逼了，反正称之为：**Leader 选举机制**<br />任何一个想成为 Leader 的哨兵，要满足两个条件：<br />第一，拿到半数以上的赞成票；<br />第二，拿到的票数同时还需要大于等于哨兵**配置文件中的 quorum 值**。

<a name="W2TdN"></a>
### 总结与建议：

- 这节课上，我就向你介绍了支持哨兵集群的这些关键机制，包括：
   - 基于 pub/sub 机制的哨兵集群组成过程；
   - 基于 INFO 命令的从库列表，这可以帮助哨兵和从库建立连接；
   - 基于哨兵自身的 pub/sub 功能，这实现了客户端和哨兵之间的事件通知。
- 分享一个经验：要保证所有哨兵实例的配置是一致的，尤其是主观下线的判断值 down-after-milliseconds。我们曾经就踩过一个“坑”。当时，在我们的项目中，因为这个值在不同的哨兵实例上配置不一致，导致哨兵集群一直没有对有故障的主库形成共识，也就没有及时切换主库，最终的结果就是集群服务不稳定。所以，你一定不要忽略这条看似简单的经验。

<a name="ZhlZs"></a>
## 09-切片集群：数据增多，是加内存还是加实例？

- 切片集群，说白了就是 Redis Cluster 模式，分片存储，相当于多主多从。

Redis Cluster相比哨兵机制的好处：<br />减少大内存Redis实例的RDB同步痛苦。<br />扩容方便，但实例发生变化，对应的哈希槽会随着变化，这个过程耗时，可能对客户端有报错影响。
<a name="YaDG0"></a>
### Redis Cluster分片集群模式的好处
当你一台Redis实例的内存超过16G+ 的时候，就会RDB持久化困难，造成IO阻塞，所以一条Redis主节点是远远不够的，只能上集群分片模式，才能保存更多的数据。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638884672217-30bad307-7533-42cb-8043-56c3d28a83db.png#clientId=uc371c85d-5147-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=766&id=u402792dd&margin=%5Bobject%20Object%5D&name=image.png&originHeight=766&originWidth=1284&originalType=binary&ratio=1&rotation=0&showTitle=false&size=429271&status=done&style=none&taskId=ube50e75a-dc0e-4c6e-a38f-afb4f9adc89&title=&width=1284)<br />分片集群衍生出的两个问题：  （看完本章节后面的内容就知道答案了）

- [ ] 数据切片后，在多个实例之间如何分布？
- [ ] 客户端怎么确定想要访问的数据在哪个实例上？
- [ ] Redis的一致性哈希算法，这里没有提到，自己下去了解一下？
- [ ] Redis不同哈希槽实例出现 数据倾斜 问题也没有提到？

<a name="MgObt"></a>
### 数据如何分片在实例上?
Redis Cluster 方案采用**哈希槽**，来处理数据和实例之间的映射关系。一个切片集群共有 16384个哈希槽，这些哈希槽类似于数据分区，每个键值对都会根据它的 key，被映射到一个哈希槽中。
<a name="ADUCE"></a>
#### 数据，哈希槽，实例 这三者的映射分布情况图：
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638885261710-79ec35d9-f738-4489-bd1f-0e9cb037fe11.png#clientId=uc371c85d-5147-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=652&id=u451dd800&margin=%5Bobject%20Object%5D&name=image.png&originHeight=652&originWidth=1244&originalType=binary&ratio=1&rotation=0&showTitle=false&size=376852&status=done&style=none&taskId=u56ced66a-16d4-4529-8f26-8f6963f158a&title=&width=1244)

- Redis实例会把 哈希槽 实例信息发给其他节点，这样在一个节点上就能访问到所有信息。
- 在集群中，实例有新增或删除，Redis 需要重新分配哈希槽；
- 为了负载均衡，Redis 需要把哈希槽在所有实例上重新分布一遍。
<a name="RXHxO"></a>
#### 客户端访问实例 重定向机制
为了提高效率：客户端把哈希槽信息缓存在本地，请求键值对时，在本地计算出哈希值所在的实例，然后直接发送给相应的实例就行了。如果这个实例没有找到对应的哈希槽，那么会返回一个准确的实例地址给客户端重新请求，这种称为 **重定向机制**。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638885730231-19a77891-b24b-4229-97d8-7f1ff589e955.png#clientId=uc371c85d-5147-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=1064&id=u69427421&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1064&originWidth=1210&originalType=binary&ratio=1&rotation=0&showTitle=false&size=783716&status=done&style=none&taskId=u2e45d495-c7ac-4f18-a1ac-f3af08db18a&title=&width=1210)

<a name="nhiC4"></a>
## 10-前面章节问题答疑和面试题
<a name="QRJuM"></a>
### 
<a name="Og7fC"></a>
### 1. Redis和SimpleKV数据相比，还缺少什么？
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638886461516-0239e38d-98ff-4d8b-99e8-e70995350230.png#clientId=uc371c85d-5147-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=898&id=ud0de7c3b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=898&originWidth=1306&originalType=binary&ratio=1&rotation=0&showTitle=false&size=942620&status=done&style=none&taskId=u74b7bf1a-af68-47df-a501-85ff2353071&title=&width=1306)

<a name="IIoZe"></a>
### 2. 整数数组和压缩列表的优势是什么?
省空间，元素一个一个紧密相连，避免指针的使用开销，非常节省空间。

<a name="DJ8Xr"></a>
### 3. Redis 基本 IO 模型中还有哪些潜在的性能瓶颈？
例如 bigkey、全量返回等操作，都是潜在的性能瓶颈。比较Redis是单线程操作。

<a name="HZ4LL"></a>
### 4. AOF 重写过程中有没有其他潜在的阻塞风险？
两个风险：<br />风险一：fork出子线程时，需要拷贝主线程的内存，如果内存过大，就会拷贝慢，影响主线程的运行。<br />风险二：fork出的子线程会共享主线程内存，如果遇到BigKey（数据量大的集合类型），主线程会因为申请大空间而面临阻塞风险，因为操作系统分配内存时，有查找和锁的开销。这就会导致阻塞。

<a name="K1wem"></a>
### 5. AOF重写为啥要拷贝？不共享使用日志呢？
如果都用 AOF 日志的话，主线程要写，bgrewriteaof 子进程也要写，这两者会竞争文件系统的锁，这就会对 Redis 主线程的性能造成影响。

<a name="gmW28"></a>
### 6. 为什么主从库间的复制不使用 AOF？
1.**RDB是二进制文件**，写入效率比AOF高。<br />2.从库进行恢复时，RDB是快照，比AOF一个个命令执行起来快咯。

<a name="zjFlY"></a>
### 7. 主从切换时，客户端能否正常请求操作？
 	主从一般是读写分离模式，所以从库依然可以提供读操作，但是写操作无法执行。

<a name="HUVsw"></a>
### 8. 如何让客户端不感知服务中断，哨兵和客户端该怎么做？
一方面，客户端能缓存应用发送的请求，异步去给服务端发送操作，只需要给客户端返回ACK就行。<br />另一方面，客户端能重新和新主库建立连接，哨兵需要提供订阅频道，通知客户端新主节点。

<a name="PAPXV"></a>
### 9. 哨兵越多越好？
哨兵不是越多越好，虽然越多能够减少误判率，但是需要每一个节点投票，这样会导致投票效率低。

<a name="YycJW"></a>
### 10. 为什么Redis不用一个表记录哈希槽键值对和实例关系？
用INFO命令感知不同节点的哈希槽分布情况，方便实例数量发生变化，效率更高。如果用表记录键值对和实例的关系，就需要修改表，增加主线程的开销。

<a name="vSD0z"></a>
### 11. Redis的ReHash触发时机 和 渐进式执行机制？

- Redis什么时候对全局哈希表做ReHash？（重新分配哈希桶）
<a name="CR7Gq"></a>
#### Redis的装载因子
Redis使用装载因子，来判断是否需要做 rehash。装载因子的计算方式是：<br />哈希表中所有 entry数 / 哈希表的哈希桶个数 = 装载因子
<a name="UcjgC"></a>
#### 定时触发ReHash，重新计算Hash桶
Redis 会根据装载因子的两种情况，来**定时触发 rehash 操作**：

- 装载因子 >= 1时，在没有生成RDB，和重写AOF时（避免影响效率），可以允许执行ReHash。
- 装载因子 >= 5，此时性能已经受到影响，哪怕再做RDB和AOF的重写，也需要紧急ReHash，此时就要采用渐进式的方式执行ReHash，每次执行时长不超过 1ms，以免对业务产生影响。

<a name="LBnDM"></a>
### 12. 主线程，子线程，和后台线程的联系与区别？
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638946604361-e2617b5d-a558-4872-9d1a-796e477aefb1.png#clientId=u01f487ba-28d9-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=736&id=uc07a27ef&margin=%5Bobject%20Object%5D&name=image.png&originHeight=736&originWidth=1156&originalType=binary&ratio=1&rotation=0&showTitle=false&size=297434&status=done&style=none&taskId=u72bf2e50-c686-430b-aed4-b312d16514c&title=&width=1156)

<a name="Lj15u"></a>
### 13. AOF日志写时复制的底层原理？
> RDB 方式进行持久化时，会用到写时复制机制。bgsave 子进程相当于复制了原始数据，而主线程仍然
> 可以修改原来的数据。
> 对 Redis 来说，主线程 fork 出 bgsave 子进程后，bgsave 子进程实际是复制了主线程的页表。这些页表中，就保存了在执行 bgsave 命令时，主线程的所有数据块在内存中的物理地址。这样一来，bgsave 子进程生成 RDB 时，就可以根据页表读取这些数据，再写入磁盘中。如果此时，主线程接收到了新写或修改操作，那么，主线程会使用写时复制机制。具体来说，写时复制就是指，主线程在有写操作时，才会把这个新写或修改后的数据写入到一个新的物理地址中，并修改自己的页表映射。

简单来说：有点像浅拷贝，fork出的子线程，只是拷贝了页表，里面存储的是物理地址的，而不是真正数据地址，如果主线程发生了改变，就会对页表内的物理地址修改。  <br />联想到了，JVM栈中的reference句柄，指向堆对象中的地址。![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638946842409-abc21bb3-a019-4d65-a7ea-8e01af3eb0b9.png#clientId=u01f487ba-28d9-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=944&id=ueabf87c6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=944&originWidth=1300&originalType=binary&ratio=1&rotation=0&showTitle=false&size=709892&status=done&style=none&taskId=ub8178e67-ad14-4a41-860d-ba0cd6c1089&title=&width=1300)
<a name="AbnQS"></a>
### 14. replication buffer 和 repl_backlog_buffer 的区别
总的来说，replication buffer 是主从库在进行全量复制时，主库上用于和从库连接的客户端的 buffer，而repl_backlog_buffer 是为了支持从库增量复制，主库上用于持续保存写操作的一块专用 buffer。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638947069341-5875aa69-077c-4b69-a7fd-05ed68a1671f.png#clientId=u01f487ba-28d9-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=964&id=uaf9107e4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=964&originWidth=1252&originalType=binary&ratio=1&rotation=0&showTitle=false&size=456483&status=done&style=none&taskId=u7c94ac10-f59c-4b68-bce9-dfd934362c9&title=&width=1252)

<a name="TVBjN"></a>
## 11-String类型存储结构：为啥不好用了？
背景：<br />存储1亿个KV键值对，结果大约占用了6.4G的内存，而且还导致生成RDB文件响应变慢的问题。<br />分析如下：
<a name="fDzcT"></a>
### 为什么String类型开销大？
因为之前第二章说到，Redis 会使用一个全局哈希表保存所有键值对，哈希表的每一项是个dictEntry的结构体，用来指向一个键值对。dictEntry 结构中有三个 8 字节的指针，分别指向 key、value 以及下一个 dictEntry，三个指针共 24 字节，如下图所示：  <br />（**dictEntry 三个就占用了 24 字节，所以每个KV都会浪费24个字节，开销占比超过Value的内存了**）<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638947447877-459664d9-18f2-4d58-b87f-880c048da41b.png#clientId=u01f487ba-28d9-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=664&id=u589d5822&margin=%5Bobject%20Object%5D&name=image.png&originHeight=664&originWidth=1144&originalType=binary&ratio=1&rotation=0&showTitle=false&size=303416&status=done&style=none&taskId=ua0ef4b80-c308-4a5b-926f-35bc44e3acf&title=&width=1144)
<a name="LvA8b"></a>
### 为什么用压缩列表替代可以节省内存？
压缩列表（ziplist），这是一种非常节省内存的结构，表头有三个字段 zlbytes、zltail 和 zllen，分别表示列表长度、列表尾的偏移量，以及列表中的 entry 个数。压缩列表尾还有一个 zlend，表示列表结束。<br />**Redis 基于压缩列表实现了List、Hash 和 Sorted Set 这样的集合类型**，这样最大好处就是节省了dictEntry的开销。当你用String类型时，一个键值对就有一个 dictEntry，要用 32 字节空间。但采用集合类型时，一个key就对应一个集合的数据，能保存的数据多了很多，但也只用了一个dictEntry，这样就节省了内存。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638947642083-6bb6057f-64e3-4c92-ad18-f8faaec2a517.png#clientId=u01f487ba-28d9-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=366&id=ua2c353a5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=366&originWidth=1234&originalType=binary&ratio=1&rotation=0&showTitle=false&size=137297&status=done&style=none&taskId=u668b3688-21ac-460c-a0f7-21095012e82&title=&width=1234)

<a name="n1T6R"></a>
### 如何用集合类型保存单值的键值对？
在保存单值的键值对时，可以采用基于 Hash 类型的二级编码方法。这里说的二级编码，就是把一个单值的数据拆分成两部分，前一部分作为 Hash 集合的 key，后一部分作为Hash 集合的 value，这样一来，我们就可以把单值数据保存到 Hash 集合中了。
> 以图片 ID 1101000060和图片存储对象 ID 3302000080 为例，我们可以把图片 ID 的前7 位（1101000）作为 Hash 类型的键，把图片 ID 的最后 3 位（060）和图片存储对象ID 分别作为 Hash 类型值中的 key 和 
> value。
> 注意：！！！这里设置ID最后3位是有讲究的，因为Hash的Key个数超过1000，可能会转成 哈希表存储，无法使用 压缩链表存储。

代码：<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1638950539229-1ba2de5a-5ec6-4c25-ba94-94d0031bf651.png#clientId=u01f487ba-28d9-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=380&id=ucd8d247f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=380&originWidth=740&originalType=binary&ratio=1&rotation=0&showTitle=false&size=170252&status=done&style=none&taskId=uf8608825-8e3d-423f-af70-54af8ed51e9&title=&width=740)
<a name="RyVe0"></a>
### Redis的Hash类型两种底层实现，压缩列表和哈希表，如何抉择呢？
Hash类型底层结构什么时候使用压缩列表，什么时候使用哈希表呢？其实，**Hash类型设置了用压缩列表保存数据时的两个阈值**，一旦超过了阈值，Hash 类型就会用哈希表来保存数据了。分别对应以下两个配置项：

- hash-max-ziplist-entries：表示用压缩列表保存时哈希集合中的最大元素个数。
- hash-max-ziplist-value：表示用压缩列表保存时哈希集合中单个元素的最大长度。

如果集合数量超过了阈值，会自动把 Hash 类型的实现结构，由压缩列表转为哈希表。而且这种转向是不可逆的，就是不能从哈希表转成 压缩列表。<br />所以Redis中的Hash集合对应的元素不能过多，过多就转成哈希表了，无法使用压缩列表，所以需要上面的 二级编码方法 去保证Hash集合中的Value最多3位数，不超过1000.

<a name="UpRSE"></a>
## 12-巧用Redis集合实现统计场景
罗列几个统计场景，例如：

1. 在移动应用中，需要统计每天的新增用户数和第二天的留存用户数；
1. 在电商网站的商品评论中，需要统计评论列表中的最新评论；
1. 在签到打卡中，需要统计一个月内连续打卡的用户数；
1. 在网页访问记录中，需要统计独立访客（Unique Visitor，UV）量。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639031527782-f82b3327-bb7f-46e0-bf04-9c983b03b9be.png#clientId=u75ee1d21-304a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=800&id=u3025bcf2&margin=%5Bobject%20Object%5D&name=image.png&originHeight=800&originWidth=1306&originalType=binary&ratio=1&rotation=0&showTitle=false&size=810196&status=done&style=none&taskId=ubb968cb8-fd82-45ec-9735-e367df4f9d1&title=&width=1306)<br />还有GEO，和自定义新数据类型
<a name="qezea"></a>
### 聚合统计
聚合统计：就是指统计多个集合元素的聚合结果，包括：

- 统计多个集合的共有元素（**交集统计**）；
- 把两个集合相比，统计其中一个集合独有的元素（**差集统计**）；
- 统计多个集合的所有元素（**并集统计**）；
<a name="yWGnx"></a>
#### 用Set实现：求每日新增用户方案
上面提到，统计每天的新增用户数和第二天的留存用户数，就可以用Redis的聚合统计实现。<br />实现方式：可用一个集合记录所有登录过App的用户 ID，同时，用另一个集合记录每天登录过App的用户 ID。对这个两个集合做聚合统计，就能查出每天新增的用户，和连续在线的用户。

1. 可以使用 Set 类型，把key设置为 user:id，表示记录的是用户ID，value就是Set 集合，里面是所有登录过 App的用户ID，称之为：累计用户Set。 （需要每天把新注册用户加进来）
1. 每日新增一个Set，记录当天登录的UserID。例如：“user:id:20200803” 这个Set中的用户就是当天的新增用户。
1. 此时：**累计用户set - 当日登录用set = 当日新增userID**.
1. 这就完成了每日新增用户统计方案。

注意！！ **Redis，对数据量大的Set做集合运算，可能会产生IO阻塞。**

<a name="eWttS"></a>
### 排序统计
<a name="UQ8oq"></a>
#### 用List实现：最新评论留言排序实现
商品评论中，需要展示最新排序的留言，Redis中List 是按照元素进入 List 的顺序进行排序的，而 Sorted Set 可以根据元素的权重来排序。<br />**实现步骤**：

1. 一个商品对应一个List，依次把留言插入排序，但是随着新留言的进来，不好排序。
1. 所以，采用SortSet排序，按照先后评论时间设置一个权重，保存到Set中，即使有新留言进来也不会出现排序错乱问题。
1. Sorted Set 的 ZRANGEBYSCORE 命令就可以按权重排序后返回元素。

所以，在面对需要展示最新列表、排行榜等场景时，如果数据更新频繁或者需要分页显<br />示，建议你优先考虑使用 Sorted Set。

<a name="fmHIg"></a>
### 二值状态统计
在签到打卡的场景中，我们只用记录签到（1）或未签到（0），所以它就是非常典型的二值状态，在签到统计时，每个用户一天的签到用 1 个 bit 位就能表示，内存占用很低，我们就可以选择 Bitmap。这是 Redis 提供的扩展数据类型。

- **Bitmap 本身是用 String 类型作为底层数据结构实现的一种统计二值状态的数据类型**
- （so，最终的基本数据类型还是5种，GEO，BitMap，HLogLog那些只是拓展数据类型）
- Bitmap 也能做多个 Bitmap 间的聚合计算，包括与、或和异或操作。
<a name="JIQyP"></a>
#### 用BitMap类型实现：千万用户签到功能？
（其实也可以做订单去重功能，也就是布隆过滤器原理，但是有误判率）<br />**实现用户每日签到功能的步骤：**<br />第一步：执行该用户8月3日签到命令，如：SETBIT uid:sign:3000:202008 2 1<br />第二步：检查该用户8月3日签到命令，如：GETBIT uid:sign:3000:202008 2<br />第三步：统计该用户8月总签到次数，如：BITCOUNT uid:sign:3000:202008

<a name="CEbpx"></a>
#### BitMap的统计功能实现：1亿用户签到近10天登录情况？
如记录了 1 亿个用户10天的签到情况，你有办法统计出这 10 天连续签到的用户总数吗？<br />**实现原理**：<br />Bitmap 支持用 BITOP 命令对多个 Bitmap 按位做“与”“或”“异或”的操作，操作的结果会保存到一个新的 Bitmap 中。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639030253420-d3ad2412-65da-4b38-9533-63e5f27beec3.png#clientId=u75ee1d21-304a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=392&id=udfbaf065&margin=%5Bobject%20Object%5D&name=image.png&originHeight=392&originWidth=1252&originalType=binary&ratio=1&rotation=0&showTitle=false&size=170896&status=done&style=none&taskId=ua39f2ea8-5f14-4180-922f-c059261d846&title=&width=1252)<br />**分析问题**：<br />回到刚刚那个问题，在统计 1 亿个用户连续 10 天的签到情况时，你可以把每天的日期作为<br />key，每个 key 对应一个 1 亿位的 Bitmap，每一个 bit 对应一个用户当天的签到情况。<br />**实现方案：对BitMap 做 & 与操作**<br />我们对 10 个 Bitmap 做“与”操作，得到的结果也是一个 Bitmap。在这个Bitmap 中，只有 10 天都签到的用户对应的 bit 位上的值才会是 1。最后，我们可以用BITCOUNT 统计下 Bitmap 中的 1 的个数，这就是连续签到 10 天的用户总数了。

<a name="RiNZR"></a>
### 基数统计
再来看一个统计场景：基数统计。**基数统计就是指统计一个集合中不重复的元素个数**。对应到我们刚才介绍的场景中，就是统计网页的 UV。（说白了，每个页面，对应每个用户一次访问，多次访问时就要去重统计）<br />**分析：**

- Redis中Hash,Set都可以实现去重，但是占用内存过大，数据量大的话不合适。
- HyperLogLog 是一种用于统计基数的数据集合类型。适合

<a name="P33Wo"></a>
#### HyperLogLog实现去重功能实现：亿万数据去重
使用HyperLogLog，优势在于，当集合元素数量非常多时，它计算基数所需的空间总是固定的，而且还很小。<br />**实现步骤：**

- 以一个页面作为一个HLogLog集合，里面把访问的用户添加进去，一旦添加进去重复添加就只算一次。
- 添加命令：PFADD page1:uv user1 user2 user3 user4 user5
- 统计总数命令：PFCOUNT page1:uv 

注意！！**HyperLogLog 的统计规则是基于概率完成的，所以它的统计结果是有一定误差的，标准误算率是 0.81%。**
<a name="IeZnw"></a>
### <br />
<a name="ldFu2"></a>
## 13-GEO是什么数据类型？可以自定义新数据类型？

- Redis 的 5 大基本数据类型：String、List、Hash、Set 和Sorted Set
- Redis的 3 大拓展数据类型： Bitmap、HyperLogLog 和 GEO。
<a name="TTDkz"></a>
### 面向 LBS（地图位点存储） 应用的 GEO 数据类型
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639031700623-c63fa4c9-4079-49bb-8ae5-8d8c8665ab75.png#clientId=u75ee1d21-304a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=724&id=dB5Q3&margin=%5Bobject%20Object%5D&name=image.png&originHeight=724&originWidth=1172&originalType=binary&ratio=1&rotation=0&showTitle=false&size=386626&status=done&style=none&taskId=u0e5f4694-ec12-412c-b60f-e200f702130&title=&width=1172)
<a name="nz7gS"></a>
### GEO 的底层结构

- 可以使用Hash作为底层结构，但是无法排序，所以不合适。使用SortSet类型更为合适。
- Sorted Set 类型也支持一个 key 对应一个 value 的记录模式，其中，key 就是 SortedSet 中的元素，而 value 则是元素的权重分数。更重要的是，Sorted Set 可以根据元素的权重分数排序，支持范围查询。这就能满足 LBS 服务中查找相邻位置的需求了。

<a name="JYhGk"></a>
#### GeoHash的编码方法
Sorted Set 元素的权重分数是一个浮点数（float 类型），而一组经纬度包含的是经度和纬度两个值，是没法直接保存为一个浮点数的，那该怎么保存呢？（GeoHash的编码方法解决）<br />具体不多介绍了，后面去查资料吧

<a name="tRisr"></a>
#### Redis的基本对象结构
RedisObject的内部组成 type,、encoding,、lru 和 refcount 4 个元数据，以及 1个*ptr指针。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639036148040-b34517c4-45a6-45b7-a652-0f7358a6316a.png#clientId=u75ee1d21-304a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=322&id=u2eda43e5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=642&originWidth=446&originalType=binary&ratio=1&rotation=0&showTitle=false&size=94489&status=done&style=none&taskId=u3c3e41e4-7dce-49d5-bd5c-31182e6c50a&title=&width=224)

- type：表示值的类型，涵盖了我们前面学习的五大基本类型；
- encoding：是值的编码方式，用来表示 Redis 中实现各个基本类型的底层数据结构，
   - 例如 SDS、压缩列表、哈希表、跳表等；
- lru：记录了这个对象最后一次被访问的时间，用于淘汰过期的键值对；
- refcount：记录了对象的引用计数；
- *ptr：是指向数据的指针。

<a name="blr8H"></a>
## 14-如何在Redis中保存时间序列数据?
需求背景：<br />记录用户在网站或者 App 上的点击行为数据，来分析用户行为。这里的数据一般包括用户 ID、行为类型（例如浏览、登录、下单等）、行为发生的时间戳等。<br />这些数据的特点是没有严格的关系模型，记录的信息可以表示成键和值的关系
<a name="r6iTp"></a>
### 时间序列数据的读写特点
按照时间节点，持续高并发的写入，修改少，需要插入速度快，要求尽量不阻塞。而且查询需要快，可能设计范围查询，聚合计算，等等。<br />Redis 提供了保存时间序列数据的两种方案，分别可以基于 Hash 和 Sorted Set 实现，以及基于RedisTimeSeries 模块实现。

<a name="PuZ8e"></a>
### 方案一：基于 Hash 和 Sorted Set 保存时间序列数据

- Hash类型，时间复杂度O(1),可以实现对单键的快速查询。满足了时间序列数据的单键查询需求。我们可以把时间戳作为 Hash 集合的 key，把记录状态值作为 Hash 集合的 value。
- Hash缺点：不支持范围查询。
- 用Sorted Set来保存时间序列数据，因为它能根据元素的权重分数排序。把时间戳作为SortedSet集合的元素分数，把时间点上记录的数据作为元素。就能实现范围查询。
<a name="BSGJp"></a>
#### 问题：如何保证Hash和SortSet是一个原子性操作？
上面提到，要同时写入两个操作，那么就要保证原子性。<br />那 **Redis 是怎么保证原子性操作的呢？**<br />这里就涉及到了 Redis 用来实现简单的事务的MULTI 和 EXEC 命令。当多个命令及其参数本身无误时，MULTI 和 EXEC 命令可以保证执行这些命令时的原子性。

- MULTI 命令：表示一系列原子性操作的开始。收到这个命令后，Redis 就知道，接下来再收到的命令需要放到一个内部队列中，后续一起执行，保证原子性。
- EXEC 命令：表示一系列原子性操作的结束。一旦 Redis 收到了这个命令，就表示所有要保证原子性的命令操作都已经发送完成了。此时，Redis 开始执行刚才放到内部队列中的所有命令操作。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639038154218-2b7abfd1-72a4-4f5e-9df1-698579e2c602.png#clientId=u75ee1d21-304a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=684&id=ud1110c45&margin=%5Bobject%20Object%5D&name=image.png&originHeight=684&originWidth=1176&originalType=binary&ratio=1&rotation=0&showTitle=false&size=323415&status=done&style=none&taskId=u73f6ca35-9ca9-45a2-9128-6bcff039832&title=&width=1176)

<a name="HfzEc"></a>
### 方案二：基于 RedisTimeSeries 模块保存时间序列数据

- 用SortSet存储时间戳，不方便做汇总数据，于是可以使用一个新数据结构RedisTimeSeries
- 当用于时间序列数据存取时，RedisTimeSeries 的操作主要有 5 个：
1. 用 TS.CREATE 命令创建时间序列数据集合；
1. 用 TS.ADD 命令插入数据；
1. 用 TS.GET 命令读取最新数据；
1. 用 TS.MGET 命令按标签过滤查询数据集合；
1. 用 TS.RANGE 支持聚合计算的范围查询。


<a name="pGcIK"></a>
## 15-消息队列的考验：Redis适合做消息队列吗？
使用消息队列时的三大需求：消息保序、重复消息处理和消息可靠性保证，这三大需求可以进一步转换为对消息队列的三大要求：

   - 消息数据有序存取	
   - 消息数据具有全局唯一编号
   - 消息数据在消费完成后被删除。
<a name="ZB46H"></a>
### 用 List 和 Streams 实现消息队列的特点和区别
List的读取命令：<br />BRPOPLPUSH <br />LPUSH mq<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639039647929-5acc9031-83e6-46d3-a2fa-3f3c2c46c403.png#clientId=u75ee1d21-304a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=416&id=u883dc519&margin=%5Bobject%20Object%5D&name=image.png&originHeight=416&originWidth=1288&originalType=binary&ratio=1&rotation=0&showTitle=false&size=585037&status=done&style=none&taskId=uce927161-f1a0-4d22-876d-a2f5d4b40ef&title=&width=1288)
<a name="khK3d"></a>
## 后面会介绍影响Redis性能的5大潜在因素？

1. **Redis内部的阻塞式操作**
1. **CPU核和NUMA架构的影响**
1. **Redis关键系统配置**
1. **Redis内存碎片**
1. **Redis缓冲区**
<a name="HYry4"></a>
## 16-异步机制：如何避免单线程模型的阻塞？
<a name="iMNIk"></a>
### Redis实例有哪些阻塞点？
从如下分析：总结五个阻塞点：5个方向分析；
<a name="Daiy1"></a>
### 总结的5个阻塞点：

1. 集合全量查询和聚合操作；
1. bigkey 删除；
1. 清空数据库；
1. AOF 日志同步写；
1. 从库加载 RDB 文件。
<a name="ulaW0"></a>
### 5方向分析阻塞原因：
**客户端**：网络 IO，键值对增删改查操作，数据库操作；<br />**磁盘**：生成 RDB 快照，记录 AOF 日志，AOF 日志重写；<br />**主从节点：**主库生成、传输 RDB 文件，从库接收 RDB 文件、清空数据库、加载 RDB文件；<br />**切片集群实例**：向其他实例传输哈希槽信息，数据迁移。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639040154273-b091f1cf-9cc3-41fb-b854-1e0cf120d6a3.png#clientId=u75ee1d21-304a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=433&id=uc27093fc&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1020&originWidth=1198&originalType=binary&ratio=1&rotation=0&showTitle=false&size=483876&status=done&style=none&taskId=u38804bf7-c2f7-437d-b5dc-2bbeb3b2ef9&title=&width=508)
<a name="aG2uK"></a>
#### 客户端-阻塞原因
客户端中网络IO采用多路复用，虽然不会导致阻塞，但是对键值对的CRUD，是会影响主线程的阻塞的。<br />阻塞点分析：

1. 求交、并和差集。这些操作就会成为第一个阻塞点：集合全量查询和聚合操作。
1. 集合自身的删除同样也有潜在的阻塞风险。因为释放空间需要耗时。bigkey 删除操作就是 Redis 的第二个阻塞点。
1. 清空数据库（例如 FLUSHDB 和 FLUSHALL 操作），是第三个阻塞点。
<a name="vn0Rh"></a>
#### 磁盘-阻塞原因

4. Redis 直接记录 AOF 日志时，会根据不同的写回策略对数据做落盘保存。会阻塞主线程，这就出现了第四个阻塞点：AOF日志同步写。
<a name="Vr6If"></a>
#### 主从节点-阻塞原因

5. 主库生成的RDB文件，是由子线程传输，固然不会阻塞，但是从库接受到文件之后，需要FLUSHDB 命令清空从库，就遇上了第三个阻塞点。

加载RDB文件，如果文件很大，就会称为第五个阻塞点。
<a name="WPsW3"></a>
#### Redis分片集群-阻塞原因

6. 使用RedisCluster方案，遇上BigKey迁移的话，可能就造成主线程阻塞。


<a name="PHA9x"></a>
## 17-为什么CPU结构会影响Redis性能？
建议：CPU 多核的场景下，用 taskset 命令把 Redis 实例和一个核绑定，可以减少 Redis 实例在不同核上被来回调度执行的开销，避免较高的尾延迟；在多 CPU 的 NUMA 架构下，如果你对网络中断程序做了绑核操作，建议同时把Redis 实例和网络中断程序绑在同一个 CPU Socket 的不同核上，这样可以避免 Redis 跨Socket 访问内存中的网络数据的时间开销。
<a name="xXAqy"></a>
### 绑核的风险和解决方案
方案一：一个 Redis 实例对应绑一个物理核<br />方案二：优化 Redis 源码
<a name="B7hX8"></a>
#### 小结： （了解就好了）
在多核 CPU 架构下，Redis 如果在不同的核上运行，就需要频繁地进行上下文切换，这个过程会增加 Redis 的执行时间，客户端也会观察到较高的尾延迟了。所以，建议你在Redis 运行时，把实例和某个核绑定，这样，就能重复利用核上的 L1、L2 缓存，可以降低响应延迟。<br />为了提升 Redis 的网络性能，我们有时还会把网络中断处理程序和 CPU 核绑定。在这种情<br />况下，如果服务器使用的是 NUMA 架构，Redis 实例一旦被调度到和中断处理程序不在同一个 CPU Socket，就要跨 CPU Socket 访问网络数据，这就会降低 Redis 的性能。所以，我建议你把 Redis 实例和网络中断处理程序绑在同一个 CPU Socket 下的不同核上，这样可以提升 Redis 的运行性能。

- [ ] **面试题：Redis既然是单线程的，是不是用1核处理器就行了？**

<a name="StYeG"></a>
## 18-波动的响应延迟：Redis突然变慢你怎么处理？
<a name="BdvwR"></a>
### 线上 Redis 突然变慢了，你怎么处理？
1.查看Redis的响应延迟<br />2.当前环境下的Redis基线性能判断<br />从 2.8.7 版本开始，redis-cli 命令提供了–intrinsic-latency 选项，可以用来监测和统计测试期间内的最大延迟，这个延迟可以作为 Redis 的基线性能。其中，测试时长可以用–intrinsic-latency 选项的参数来指定。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639042914491-6f50ff26-9b39-4c15-902e-639b8f0e0cdd.png#clientId=u75ee1d21-304a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=1094&id=u2143d9c6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1094&originWidth=1570&originalType=binary&ratio=1&rotation=0&showTitle=false&size=618841&status=done&style=none&taskId=uc2aaedee-ad38-4db8-bc9c-ff4cd5902c1&title=&width=1570)
<a name="cZFIR"></a>
### Redis 自身操作变慢的影响
<a name="fIdEZ"></a>
#### 1. 慢查询命令
可以查看Redis日志，或者使用 latency monitor 工具，查询变慢的请求。<br />** 如有变慢的命令，可以使用其他高效命令代替**。<br />比如：返回一个Set中的集合，不要使用SMEMBERS命令而是使用SSCAN多次分页返回。<br />不要使用Keys，会遍历所有的Key，造成主线程阻塞。<br />**执行排序、交集、并集操作时变慢**<br />执行排序、交集、并集操作时变慢，可以在客户端完成，而不要用 SORT、SUNION、SINTER 这些命令，以免拖慢 Redis 实例。
<a name="Z39lY"></a>
#### 2.过期Key操作
过期 key自动删除机制。它是 Redis 用来回收内存空间的常用机制，会造成主线程阻塞。<br />（Redis 4.0 后可以用异步线程机制来减少阻塞影响）<br />如果同一时间内，有大量的Key失效，也会造成 主线程阻塞。

<a name="TXvle"></a>
### Redis每100毫秒定期删除失效的Key，此时有两种判断机制

1. 采样 配置的 个数的 key，并将其中过期的key 全部删除；
1. 如果失效的Key超过了25%，就会反复执行分批删除动作，会导致主线程阻塞，所以尽量不要让同一时间点，有大量的Key失效。

<a name="dFGZo"></a>
## 19-波动响应延迟：Redis变慢原因
文件系统和操作系统两个维度分析，具体就不多BB了，了解一下就行
<a name="HI62G"></a>
### 梳理9 个检查点的 Checklist
遇到 Redis性能变慢时，按照这些步骤逐一检查，高效地解决问题：

1. 获取 Redis 实例在当前环境下的基线性能。

2. 是否用了慢查询命令？如果是的话，就使用其他命令替代慢查询命令，或者把聚合计算命令放在客户端做。

3. 是否对过期 key 设置了相同的过期时间？对于批量删除的 key，可以在每个 key 的过期时间上加一个随机数，避免同时删除。

4. 是否存在 bigkey？ 对于 bigkey 的删除操作，如果你的 Redis 是 4.0 及以上的版本，可以直接利用异步线程机制减少主线程阻塞；如果是 Redis 4.0 以前的版本，可以使用SCAN 命令迭代删除；对于 bigkey 的集合查询和聚合操作，可以使用 SCAN 命令在客户端完成。

5. Redis AOF 配置级别是什么？业务层面是否的确需要这一可靠性级别？如果我们需要高性能，同时也允许数据丢失，可以将配置项 no-appendfsync-on-rewrite 设置为yes，避免 AOF 重写和 fsync 竞争磁盘 IO 资源，导致 Redis 延迟增加。当然， 如果既需要高性能又需要高可靠性，最好使用高速固态盘作为 AOF 日志的写入盘。

6. Redis 实例的内存使用是否过大？发生 swap 了吗？如果是的话，就增加机器内存，或者是使用 Redis 集群，分摊单机 Redis 的键值对数量和内存压力。同时，要避免出现Redis 和其他内存需求大的应用共享机器的情况。

7. 在 Redis 实例的运行环境中，是否启用了透明大页机制？如果是的话，直接关闭内存大页机制就行了。

8. 是否运行了 Redis 主从集群？如果是的话，把主库实例的数据量大小控制在 2~4GB，以免主从复制时，从库因加载大的 RDB 文件而阻塞。

9. 是否使用了多核 CPU 或 NUMA 架构的机器运行 Redis 实例？使用多核 CPU 时，可以给 Redis 实例绑定物理核；使用 NUMA 架构时，注意把 Redis 实例和网络中断处理程序运行在同一个 CPU Socket 上。

<a name="vDYVn"></a>
## 20-删除数据后：Redis内存占用依旧很高？
Redis 释放的内存空间会由内存分配器管理，并不会立即返回给操作系统。所以，操作系统仍然会记录着给 Redis 分配了大量内存。
<a name="AQuf7"></a>
### 什么是内存碎片？
应用申请的是一块连续地址空间的 N 字节，但在剩余的内存空间中，没有大小为 N 字节的连续空间了，那么，这些剩余空间就是内存碎片。
<a name="Xeq2V"></a>
### 内存碎片是如何形成的？

   - 内因是操作系统的内存分配机制。
   - 外因是 Redis 的负载特征。

<a name="j1Xdf"></a>
#### 内因：内存分配器的分配策略
 Redis 每次向分配器申请的内存空间大小不一样，这种分配方式就会有形成碎片的风险。
<a name="UFwUK"></a>
#### 外因：键值对大小不一样和删改操作
内存分配器策略是内因，而 Redis 的负载属于外因，包括了大小不一的键值 对 和键值对修改删除带来的内存空间变化。

<a name="bLfLF"></a>
### 如何清理内存碎片？
重启Redis实例<br />搬家让位，合并空间思想


<a name="rCVAS"></a>
## 21-缓存区：一个可能发生惨案的地方
分别聊聊服务器端和客户端、主从集群间的缓冲区溢出问题，以及应对方案。
<a name="AqSGy"></a>
### 客户端输入和输出缓冲
为了避免客户端和服务器端的请求发送和处理速度不匹配，服务器端给每个连接的客户端都设置了一个输入缓冲区和输出缓冲区，我们称之为客户端输入缓冲区和输出缓冲区。

- 输入缓存区：会把客户端发来的命令暂存起来，主线程在从中读取命令。
- 输出缓冲区：暂存的是 Redis 主线程要返回给客户端的数据。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639050281515-343b5e28-9a29-4ee1-bc01-78984ab9c169.png#clientId=u75ee1d21-304a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=832&id=ue14165ed&margin=%5Bobject%20Object%5D&name=image.png&originHeight=832&originWidth=1334&originalType=binary&ratio=1&rotation=0&showTitle=false&size=502696&status=done&style=none&taskId=u36a26a30-ac68-4a94-92ed-0a04ba083fc&title=&width=1334)
<a name="qRW8X"></a>
### 如何应对客户端输入缓存区溢出？
<a name="vSPHG"></a>
#### 溢出原因主要有两种：

1. 写入BigKey，比如一下子写入百万级的集合数据。
1. 服务端处理速度编码，主线程阻塞，无法及时处理正常发来的请求，导致积压。
<a name="bobPC"></a>
#### 解决方案：
可以从两个角度去考虑如何避免：<br />一是把缓冲区调大。<br />二是从数据命令的发送和处理速度入手。

<a name="bXiRF"></a>
### 如何应对客户端输出缓冲区溢出？
<a name="kSdAt"></a>
#### 客户端输出缓冲区也包括两部分：
一部分，是一个大小为 16KB的固定缓冲空间，用来暂存 OK 响应和出错信息；<br />另一部分，是一个可以动态增加的缓冲空间，用来暂存大小可变的响应结果。

<a name="s4Wu8"></a>
#### 输出端溢出原因主要三种：

1. 服务器端返回 bigkey 的大量结果；
1. 执行了 MONITOR 命令；（检测命令执行情况时使用，一般在调试环境采用）
1. 缓冲区大小设置得不合理。
<a name="HKnfJ"></a>
#### 解决方案：

1. 避免 bigkey 操作返回大量数据结果；
1. 避免在线上环境中持续使用 MONITOR 命令。
1. 使用 client-output-buffer-limit 设置合理的缓冲区大小上限，或是缓冲区连续写入时间和写入量上限。

<a name="oPHP3"></a>
### 主从集群中的缓冲区
（了解一下就好了）<br />在全量复制过程中，主节点在向从节点传输 RDB 文件的同时，会继续接收客户端发送的写命令请求。这些写命令就会先保存在复制缓冲区中，等 RDB 文件传输完成后，再发送给从节点去执行。主节点上会为每个从节点都维护一个复制缓冲区，来保证主从节点间的数据同步。
<a name="KCl9l"></a>
#### 复制缓冲区的溢出问题
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639051064839-334dbb85-da40-46a4-8ce7-bf4291e3a611.png#clientId=u75ee1d21-304a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=1048&id=u0f0b2733&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1048&originWidth=1424&originalType=binary&ratio=1&rotation=0&showTitle=false&size=685540&status=done&style=none&taskId=ua28b7227-e4c2-4089-a994-419b87d35e7&title=&width=1424)
<a name="AeUUf"></a>
#### 复制积压缓冲区的溢出问题
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639051089560-cd8503e1-1e71-4a4a-862b-93b773171318.png#clientId=u75ee1d21-304a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=1096&id=u722bf5ac&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1096&originWidth=1388&originalType=binary&ratio=1&rotation=0&showTitle=false&size=875074&status=done&style=none&taskId=ue55c2c63-fe6a-4dbb-8801-bad142e34c1&title=&width=1388)

<a name="XQw5r"></a>
## 22-问题答疑分析11-21章节
重要！！！

如何排查Redis的bigKey使用？<br />要尽量避免 bigkey 的使用，这是因为，Redis 主线程在操作bigkey 时，会被阻塞。那么，一旦业务应用中使用了 bigkey，我们该如何进行排查呢？<br />Redis 可以在执行 redis-cli 命令时带上–bigkeys 选项，进而对整个数据库中的键值对大小情况进行统计分析，比如说，统计每种数据类型的键值对个数以及平均大小。此外，这个命令执行后，会输出每种数据类型中最大的 bigkey 的信息，对于 String 类型来说，会输出最大 bigkey 的字节长度，对于集合类型来说，会输出最大 bigkey 的元素个数，如下图所示： （不要在线上写库上执行，可能影响主线程阻塞）<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639376493563-f61cfb01-9067-4a7f-bd14-58a8c59f37d2.png#clientId=u700cea98-df73-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=444&id=u5d936c0a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=888&originWidth=1004&originalType=binary&ratio=1&rotation=0&showTitle=false&size=696357&status=done&style=none&taskId=u5ea85cc2-ea9e-4ba4-9b40-df08fa5073f&title=&width=502)

<a name="SCLGO"></a>
## 23-旁路缓存：Redis是如何工作的？

- 了解一下就行：只读缓存，读写缓存。
- 同步MySQL策略，读写串行同步MySQL方案，读写异步同步策略。
- 按照使用场景权衡，是保证接口性能，还是数据完整性能。

<a name="MYCr7"></a>
## 24-缓存策略：缓存满了怎么办？
如果按照“八二原理”来设置缓存空间容量，也就是把缓存空间容量设置为总数据量的 20% 的话，就有可能拦截到 80% 的访问。<br />也就是你100G的内存数据，设置个20G的环境就差不多了，没必要初始化很大的缓存空间。

<a name="TpKMC"></a>
### Redis缓存有哪些淘汰策略？
Redis4.0版本之前有6种实现策略，4.0版本之后又增加了两种策略，主要区分为两类：一种是淘汰数据，一种是不同淘汰数据。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639378747348-c1e48276-54c7-4f50-b554-732a96f3d8b3.png#clientId=u700cea98-df73-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=282&id=uc3f83110&margin=%5Bobject%20Object%5D&name=image.png&originHeight=564&originWidth=1358&originalType=binary&ratio=1&rotation=0&showTitle=false&size=223504&status=done&style=none&taskId=udd8c3034-38d1-4c5d-b1a9-8b148c91f52&title=&width=679)
<a name="LhRBa"></a>
#### noevction：不淘汰数据，直接报错（默认）
缓存被写满了，再有写请求来时，Redis 不再提供服务，而是直接返回错误。

<a name="z8rrB"></a>
### 有过期时间：4种淘汰策略
volatile-random、volatile-ttl、volatile-lru 和 volatile-lfu 这四种淘汰策略。它们筛选的候选数据范围，被限制在已经设置了过期时间的键值对上。
<a name="bPTXm"></a>
#### volatile-ttl
volatile-ttl筛选时，会针对设置过期时间的键值对，按过期时间的先后进行删除，越早过期的越先被删除。
<a name="HjsYi"></a>
#### volatile-random
volatile-random 就像它的名称一样，在设置了过期时间的键值对中，进行随机删除。
<a name="HY9LH"></a>
#### volatile-lru
volatile-lru 会使用 LRU 算法筛选设置了过期时间的键值对。
<a name="fa3uz"></a>
#### volatile-lfu
volatile-lfu 会使用 LFU 算法选择设置了过期时间的键值对。

<a name="wmuto"></a>
### LRU算法简单说明
LRU 算法背后的想法非常朴素：它认为刚刚被访问的数据，肯定还会被再次访问，所以就把它放在 MRU 端；长久不访问的数据，就不会再被访问了，所以就让它逐渐后移到 LRU 端，在缓存满时，就优先删除它。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639379427539-8a95f808-b25d-4314-bf0c-c37ebff9d360.png#clientId=u700cea98-df73-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=600&id=uc9aaaedf&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1200&originWidth=962&originalType=binary&ratio=1&rotation=0&showTitle=false&size=433770&status=done&style=none&taskId=u6180b0a8-92c1-43d1-ba5b-1b0633d57e2&title=&width=481)<br />当请求量增大的时候，频繁移动链表，会影响Redis性能。<br />由于这种算法简单粗暴，后面做了个优化，对每个RedisObject缓存对象记录了一个 修改时间戳，在淘汰数据的时候，会随机选出N个数据，把他们作为一个集合，选出修改时间戳最小的，当做排除对象。

<a name="BixFa"></a>
### 淘汰策略选择建议

1. 优先使用 allkeys-lru 策略。把最近最常访问的数据留在缓存中，提升访问性能。
1. 如果你的业务数据中有明显的冷热数据区分，我建议你使用 allkeys-lru 策略。
1. 如果业务应用中的数据访问频率相差不大，没有明显的冷热数据区分，建议使用allkeys-random 策略，随机选择淘汰的数据就行。
1. 如果业务中有置顶的需求，比如置顶新闻、置顶视频，那么可以使用 volatile-lru策略，同时不给这些置顶数据设置过期时间。这些需要置顶的数据一直不会被删除，而其他数据会在过期时根据 LRU 规则进行筛选。


<a name="aWHj4"></a>
### 如何处理淘汰的数据？
主要还是看这些淘汰的数据是否发生修改，要是修改了就得持久化到MySQL当中，没修改直接删除就好了。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639380598616-549114bb-d9ea-420b-abff-e8e89f87bc2d.png#clientId=u700cea98-df73-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=322&id=u541e4208&margin=%5Bobject%20Object%5D&name=image.png&originHeight=644&originWidth=876&originalType=binary&ratio=1&rotation=0&showTitle=false&size=204403&status=done&style=none&taskId=u71b5910d-96cc-47b2-a24d-f987ec10068&title=&width=438)


<a name="gUOUp"></a>
## 25-缓存异常：如何解决缓存和数据库不一致问题？
<a name="s5vz3"></a>
### 两种回写数据库的方案

- 同步直写策略：写缓存时，也同步写数据库，缓存和数据库中的数据一致；
- 异步写回策略：写缓存时不同步写数据库，等到数据从缓存中淘汰时，再写回数据库。使用这种策略时，如果数据还没有写回数据库，缓存就发生了故障，那么，此时，数据库就没有最新的数据了。

看使用场景去选择：只要保证两个操作的原子性。

<a name="HUiON"></a>
#### 不同策略存在的问题：
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639381070916-5f6be138-ec7c-4956-b630-ee764ac75b9c.png#clientId=u700cea98-df73-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=203&id=u2088cac0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=406&originWidth=1326&originalType=binary&ratio=1&rotation=0&showTitle=false&size=402478&status=done&style=none&taskId=u80402a97-b542-45af-ad7c-ec7515ded91&title=&width=663)

<a name="jttuc"></a>
### 如何解决数据不一致问题？
<a name="XPqbi"></a>
#### 1.重试机制：
可以把要修改的缓存值暂存到消息队列，当应用没能成功修改，则利用消息队列重新读取这些值，然后再次进行删除OR修改。
<a name="TQV8E"></a>
#### 2.先删除缓存，再更新数据库

1. 线程 B 读取到了旧值；
2. 线程 B 是在缓存缺失的情况下读取的数据库，所以，它还会把旧值写入缓存，这可能会

导致其他线程从缓存中读到旧值。

等到线程 B 从数据库读取完数据、更新了缓存后，线程 A 才开始更新数据库，此时，缓存中的数据是旧值，而数据库中的是最新值，两者就不一致了。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639381611967-426c066e-b275-4a78-8aef-b9dcbd2218c9.png#clientId=u700cea98-df73-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=310&id=ub77a00c7&margin=%5Bobject%20Object%5D&name=image.png&originHeight=620&originWidth=1354&originalType=binary&ratio=1&rotation=0&showTitle=false&size=436286&status=done&style=none&taskId=u66f42188-4553-4521-adb2-a2df84f7874&title=&width=677)
<a name="ZRBLT"></a>
#### 解决方案：延迟双删除 策略
之所以要加上 sleep 的这段时间，就是为了让线程 B 能够先从数据库读取数据，再把缺失的数据写入缓存，然后，线程 A 再进行删除。所以，线程 A sleep 的时间，就需要大于线程 B 读取数据再写入缓存的时间。这个时间怎么确定呢？建议你在业务程序运行的时候，统计下线程读数据和写缓存的操作时间，以此为基础来进行估算。<br />其它线程读取数据时，会发现缓存缺失，所以会从数据库中读取最新值。因为这个方案会在第一次删除缓存值后，延迟一段时间再次进行删除，所以我们也把它叫做“延迟双删”。<br />（为啥不采用时间戳对比，在最后修改的时候对比一下时间戳，然后在写入？）<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639381885720-d8aac77f-81bd-40d7-af53-a2c783b9e216.png#clientId=u700cea98-df73-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=133&id=ucb83b480&margin=%5Bobject%20Object%5D&name=image.png&originHeight=266&originWidth=406&originalType=binary&ratio=1&rotation=0&showTitle=false&size=52777&status=done&style=none&taskId=uded422b3-98b6-4fd1-9d49-e80c7297efe&title=&width=203)
<a name="UxK1I"></a>
#### 3.先更新数据库值，再删除缓存值 
如果线程 A 删除了数据库中的值，但还没来得及删除缓存值，线程 B 就开始读取数据了，此时，线程 B 查询缓存时，发现缓存命中，就会直接从缓存中读取旧值。（请求量不大的情况下，业务影响较小）<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639382035706-2d016171-0a19-4e87-af5f-8f6724076071.png#clientId=u700cea98-df73-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=224&id=ua75e2180&margin=%5Bobject%20Object%5D&name=image.png&originHeight=448&originWidth=1352&originalType=binary&ratio=1&rotation=0&showTitle=false&size=268141&status=done&style=none&taskId=u5fb93d9a-c039-49eb-b7c9-2f7a579daaf&title=&width=676)

<a name="RTWs4"></a>
### 只读缓存，删改操作顺序问题总结
对于读写缓存来说，如果采用同步写回策略，那么可以保证缓存和数据库中的数据一致。只读缓存的情况比较复杂，我总结了一张表，以便于你更加清晰地了解数据不一致的问题原因、现象和应对方案。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639382779714-d678e982-24cc-4fc4-8680-ab7eef96e181.png#clientId=u700cea98-df73-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=317&id=uc8f1c909&margin=%5Bobject%20Object%5D&name=image.png&originHeight=634&originWidth=1348&originalType=binary&ratio=1&rotation=0&showTitle=false&size=662891&status=done&style=none&taskId=udb179d49-7996-42f7-8987-c8502508fe8&title=&width=674)<br />大多数场景，只会采用Redis只读场景。


<a name="nUvnv"></a>
## 26-缓存异常：如何解决缓存雪崩，击穿，穿透问题？
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639383983704-a203bb46-63a8-43c7-9bbc-39bf2e34e930.png#clientId=u700cea98-df73-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=295&id=uc243aba4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=590&originWidth=1350&originalType=binary&ratio=1&rotation=0&showTitle=false&size=480978&status=done&style=none&taskId=u00c91031-fb11-45d2-ac82-694dcaa3cac&title=&width=675)
<a name="IYvxW"></a>
### Redis雪崩

1. 突然出现大量缓存失效，导致请求无法处理。
1. 除了大量数据同时失效会导致缓存雪崩，还有一种情况是Redis故障宕机，无法处理请求。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639383207415-ef602ccd-931b-4832-a47b-f969ddae7590.png#clientId=u700cea98-df73-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=328&id=u65de44e9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=656&originWidth=1242&originalType=binary&ratio=1&rotation=0&showTitle=false&size=285414&status=done&style=none&taskId=u63cab630-08b4-4f00-ad06-7154922a93e&title=&width=621)
<a name="HecqP"></a>
#### 雪崩解决方案：
1.分散过期时间，在过期时间上加个随机值，在1分钟范围内，避免同时过期。<br />2.服务降级，当发现Redis不能用时，返回指定的默认值，或者请求限流等等。（在不影响业务的情况下）<br />3.请求限流，防止大量请求，怼到MySQL上面造成系统影响。<br />4.提前预防，增加Redis集群稳固性。

<a name="JA3Bw"></a>
### Redis缓存击穿
热点数据过期，频繁访问热点数据，无法命中缓存，导致频繁访问MySQL压力激增。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639383551733-174029b8-8c09-4f8f-88f9-8366ce7cdb07.png#clientId=u700cea98-df73-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=326&id=u3c5c7163&margin=%5Bobject%20Object%5D&name=image.png&originHeight=652&originWidth=1218&originalType=binary&ratio=1&rotation=0&showTitle=false&size=277931&status=done&style=none&taskId=uc97b72c1-8a2d-4a37-a552-3851a76d76b&title=&width=609)
<a name="Rn25v"></a>
#### 击穿解决方案：

- 设置系统二级缓存，比如SpingCache等。
- 不影响业务下，不要设置过期时间等。

<a name="ZipFk"></a>
### 缓存穿透
指故意查询缓存中没有，且数据库中不存在的数据，频繁访问MySQL，导致频繁访问MySQL压力激增。<br />一般出现在：数据误删场景，恶意攻击。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639383722678-6a72d02c-76d1-4191-85de-095f73dcd52c.png#clientId=u700cea98-df73-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=325&id=uacb9c887&margin=%5Bobject%20Object%5D&name=image.png&originHeight=650&originWidth=1250&originalType=binary&ratio=1&rotation=0&showTitle=false&size=301937&status=done&style=none&taskId=u387aedb1-57d5-432d-9d7b-855689e6749&title=&width=625)
<a name="NbBkt"></a>
#### 穿透解决方案：
1.缓存一个null值到Redis中。<br />2.boolFilter过滤器快速判断缓存是否存在，减少数据库查询。（有不一定有，无则一定无）<br />3.加强前端接口访问的拦截，减少无效请求。<br />4.接口限流也能预防一部分吧，避免频繁刷接口。

<a name="yOhja"></a>
## 27-缓存被污染了，该怎么办？
“污染”，说白了，就是很少用到，但一直在占用的缓存key，我们该怎么把他们淘汰，不要浪费内存。

<a name="gm0eP"></a>
### LFU缓存策略的优化（LRU算法的一个升级）
LFU 缓存策略是在 LRU 策略基础上，为每个数据增加了一个计数器，来统计这个数据的访问次数。<br />首先会根据数据的访问次数进行筛选，把访问次数最低的数据淘汰出缓存。如果两个数据的访问次数相同，LFU 策略再比较这两个数据的访问时效性，把距离上一次访问时间更久的数据淘汰出缓存。

<a name="OW0uy"></a>
#### LFU算法的底层实现：
为了避免操作链表的开销，Redis 在实现 LRU 策略时使用了两个近似方法：Redis 是用 RedisObject 结构来保存数据的，RedisObject 结构中设置了一个 lru 字段，用来记录数据的访问时间戳；<br />Redis 并没有为所有的数据维护一个全局的链表，而是通过随机采样方式，选取一定数量（例如 10 个）的数据放入候选集合，后续在候选集合中根据 lru 字段值的大小进行筛选。

<a name="VarM1"></a>
## 28-Pika:如何基于SSD实现大容量Redis？
基于大内存的大容量实例在实例恢复、主从同步过程中会引起一系列潜在问题，例如恢复时间增长、主从切换开销大、缓冲区易溢出。<br />那怎么办呢？我推荐你使用固态硬盘（Solid State Drive，SSD）。它的成本很低（每 GB的成本约是内存的十分之一），而且容量大，读写速度快，我们可以基于 SSD 来实现大容量的 Redis 实例。360 公司 DBA 和基础架构组联合开发的 Pika键值数据库，正好实现了这一需求。

<a name="eugIY"></a>
### 大内存Redis实例的潜在问题
实例内存容量大，RDB 文件相应增大，RDB 文件生成时的 fork 时长就会增加，这就会导致 Redis 实例阻塞。且RDB 文件增大后，使用RDB进行恢复的时长也会增加，会导致 Redis 较长时间无法对外提供服务。
<a name="ToVfH"></a>
### Pika的整体架构：
自己下去了解一下就好了<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639384990696-1a00077e-2048-4cae-a645-e88124b62a2e.png#clientId=u700cea98-df73-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=481&id=u6a9c0f6e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=962&originWidth=1386&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1075452&status=done&style=none&taskId=uf7253613-05aa-408d-b2a4-2218e20f6c2&title=&width=693)

29-无锁的原子操作：Redis如何应对并发访问？
<a name="slXwJ"></a>
### Redis的两种原子性操作方法？

1. 自带的原子操作：INCR，DECR
1. 使用Lua脚本，以原子方式执行单个命令。
<a name="wXVd8"></a>
#### 自带的原子操作：INCR，DECR
Redis单个命令是单线程执行，天生原子性，但是多个操作交互时，就不是原子性的了。<br />使用提供的 INCR、DECR命令，可以减少“读-改-写回”这三个步骤，直接在数据上做加减操作。
<a name="Z4Tax"></a>
#### Lua脚本封装多条命令
Redis 会把整个 Lua 脚本作为一个整体执行，在执行的过程中不会被其他命令打断，从而保证了 Lua 脚本中操作的原子性。（注意性能问题）<br />**注意性能问题！！！**<br />Redis 的 Lua 脚本可以包含多个操作，这些操作都会以原子性的方式执行，绕开了单命令操作的限制。不过，如果把很多操作都放在 Lua 脚本中原子执行，会导致 Redis 执行脚本的时间增加，同样也会降低 Redis 的并发性能。所以，在编写 Lua脚本时，你要避免把不需要做并发控制的操作写入脚本中。

<a name="ppsHF"></a>
## 30-如何用Redis实现分布式锁？
专栏讲的有点复杂，自己去看看视频理解一下吧。
<a name="i2CTI"></a>
### 总结：
在基于单个 Redis 实例实现分布式锁时，对于加锁操作，我们需要满足三个条件。<br />1.加锁包括了读取锁变量、检查锁变量值和设置锁变量值三个操作，但需要以原子操作的<br />方式完成，所以，我们使用 SET 命令带上 NX 选项来实现加锁；

2.锁变量需要设置过期时间，以免客户端拿到锁后发生异常，导致锁一直无法释放，所<br />以，我们在 SET 命令执行时加上 EX/PX 选项，设置其过期时间；

3.锁变量的值需要能区分来自不同客户端的加锁操作，以免在释放锁时，出现误释放操<br />作，所以，我们使用 SET 命令设置锁变量值时，每个客户端设置的值是一个唯一值，用<br />于标识客户端。

<a name="ApDvp"></a>
## 31-事务机制：Redis能实现ACID属性吗？
事务在执行时，会提供专门的属性保证，包括原子性（Atomicity）、一致性（Consistency）、隔离性（Isolation）和持久性（Durability），也就是 ACID 属性。这些属性既包括了对事务执行结果的要求，也有对数据库在事务执行前后的数据状态变化的要求。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639388669882-09fdc99f-7f64-4370-95b4-bd128b527e99.png#clientId=ud2205ab2-a81f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=416&id=ufa4d7859&margin=%5Bobject%20Object%5D&name=image.png&originHeight=416&originWidth=1370&originalType=binary&ratio=1&rotation=0&showTitle=false&size=400921&status=done&style=none&taskId=ue49d0451-074d-4e2f-b2f0-6de57c481e8&title=&width=1370)
<a name="eMHvk"></a>
### Redis如何实现事务？
支持柔性事物。不是强严格的事务。
<a name="dloTh"></a>
#### 提供了 MULTI、EXEC 两个命令来实现事务
分为这三个步骤：<br />1.在客户端命令显示的开启一个事务，命令就是 multi 。<br />2.把多个命令发送给Redis服务端，这些命令会暂存到命令队列中，并不会立即执行。<br />3.客户端发送提交事务命令：Exec命令，服务端收到后才开始执行队列中所有命令。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639387602307-17ee8958-53b5-4529-a075-3291857d79c8.png#clientId=ud2205ab2-a81f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=554&id=u41adda78&margin=%5Bobject%20Object%5D&name=image.png&originHeight=554&originWidth=548&originalType=binary&ratio=1&rotation=0&showTitle=false&size=195871&status=done&style=none&taskId=uf9cb0612-0051-456a-9113-5c6f257fb2b&title=&width=548)

<a name="tXxT6"></a>
### Redis事务机制能保证哪些属性？
<a name="B8p56"></a>
#### 能否保证原子性？ = 分情况

   - 命令入队时就报错，会放弃事务执行，保证原子性；
   - 命令入队时没报错，实际执行时报错，不保证原子性；
   - EXEC 命令执行时实例故障，如果开启了 AOF 日志，可以保证原子性。

**Redis没有提供回滚机制，但是可以用DISCARD命令来主动放弃事务。**

<a name="QWeqL"></a>
#### 能否保证事务一致性？ = 能保证
命令执行错误或 Redis 发生故障的情况下，Redis 事务机制对一致性属性是有保证的。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639388299934-e35c9045-65d2-4e1e-93db-7b411df0f80e.png#clientId=ud2205ab2-a81f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=1220&id=u28f49927&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1220&originWidth=1406&originalType=binary&ratio=1&rotation=0&showTitle=false&size=903247&status=done&style=none&taskId=u40a55c43-4d2c-4685-9df9-db11b99bd6d&title=&width=1406)

<a name="L6c8u"></a>
#### 能否保证隔离性？ = 能，如果没开启WATCH，就不一定
1.并发操作在 EXEC 命令前执行，此时，隔离性的保证要使用 WATCH 机制来实现，否则隔离性无法保证；<br />2.并发操作在 EXEC 命令后执行，此时，隔离性可以保证。

WATCH机制：监控机制
> WATCH 机制的作用是，在事务执行前，监控一个或多个键的值变化情况，当事务调用EXEC 命令执行时，WATCH 机制会先检查监控的键是否被其它客户端修改了。如果修改了，就放弃事务执行，避免事务的隔离性被破坏。然后，客户端可以再次执行事务，此时，如果没有并发修改事务数据的操作了，事务就能正常执行，隔离性也得到了保证。


<a name="rtKZG"></a>
#### 能否保证持久性？ = 不能

- Redis 是内存数据库，所以，数据是否持久化保存完全取决于 Redis 的持久化配置模式。
- 无论是配置成RDB，还是AOF，还是混合使用，都可能丢掉最后一次操作，并不是严格保证事务的持久性。


<a name="LYmsg"></a>
## 32-Redis主从同步 与 故障切换，有哪些坑？
介绍 3 个坑，分别是主从数据不一致、读到过期数据，以及配置项设置得不合理从而导致服务挂掉。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639389487017-42b4864b-ceb7-42fb-9017-4cfb0790ae4a.png#clientId=ud2205ab2-a81f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=572&id=u706a9b3b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=572&originWidth=1380&originalType=binary&ratio=1&rotation=0&showTitle=false&size=641318&status=done&style=none&taskId=uc7c614e1-4232-4fc0-910f-cffa6eb1944&title=&width=1380)
<a name="UXWcd"></a>
### 一、主从数据不一致
<a name="AHB1H"></a>
#### 为啥会出现主库读出的数据和从库不一样呢？
因为主从库间的复制命令是异步进行的。有时间延迟，网络延迟等。
<a name="xKHQR"></a>
#### 解决方案：

   1. 主从机器部署在同一个机房，减少网络延时。
   1. 开发外部程序，监控主从库的复制进度。

<a name="G9CD2"></a>
### 二、读取过期数据
<a name="sx4LU"></a>
#### Redis删除数据的两种策略
Redis 同时使用了两种策略来删除过期的数据，分别是惰性删除策略和定期删除策略。<br />**惰性删除策略：**<br />当一个数据过期，不会立即删除，而是等到有请求读取时，会判断是否过期，在删除。好处是减少CPU的使用，但是可能白白占用内存。

**定期删除策略：**<br />Redis每隔0.1S会检测一部分是否过期，会删掉部分数据。

<a name="akC0t"></a>
#### Redis3.2版本之前可能读取过期数据

- 因为在3.2版本之前， 从库读取数据时并不会判断数据是否过期，也会返回数据。
- 在3.2版本之后，如果读取到过期数据，从库虽然不删除，但是会返回控制，避免读到过期数据。

<a name="KtbvF"></a>
### 三、不合理配置项导致的服务挂掉
这里涉及到的配置项有两个，分别是 protected-mode 和 cluster-node-timeout。、
<a name="obFR4"></a>
#### 1.protected-mode配置项
我们在应用主从集群时，要注意将 protected-mode 配置项设置为 no，并且将bind 配置项设置为其它哨兵实例的 IP 地址。这样一来，只有在 bind 中设置了 IP 地址的哨兵，才可以访问当前实例，既保证了实例间能够通信进行主从切换，也保证了哨兵的安全性。
<a name="jpB4X"></a>
#### 2.cluster-node-timeout 配置项

   - 这个配置项设置了 Redis Cluster 中实例响应心跳消息的超时时间。
   - 建议你将 cluster-node-timeout 调大些（例如 10 到 20 秒）。

<a name="e9dy7"></a>
## 33-脑裂：Redis数据丢失？
背景：

- 1主，5从，3哨兵、使用过程中出现数据丢失，影响到了业务数据的可靠性，排查是脑裂问题。
- 脑裂是指在主从集群中，同时有两个主库都能接收写请求。在 Redis 的主从切换过程中，如果发生了脑裂，客户端数据就会写入到原主库，如果原主库被降为从库，这些新写入的数据就丢失了。
<a name="zJY3Q"></a>
### 什么是脑裂？
所谓的脑裂，就是指在主从集群中，同时有两个主节点，它们都能接收写请求。而脑裂最直接的影响，就是客户端不知道应该往哪个主节点写入数据，结果就是不同的客户端会往不同的主节点上写入数据。而且，严重的话，脑裂会进一步导致数据丢失。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639392799171-94ac639f-e69f-4665-bd11-8a8d348b7483.png#clientId=ud2205ab2-a81f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=912&id=u9e969925&margin=%5Bobject%20Object%5D&name=image.png&originHeight=912&originWidth=1208&originalType=binary&ratio=1&rotation=0&showTitle=false&size=426458&status=done&style=none&taskId=u0ecfc04b-fb55-490b-aa4c-18bcb6ca2fe&title=&width=1208)
<a name="yJbOI"></a>
### 脑裂为什么会丢失数据？
主从切换后，从库一旦升级为新主库，哨兵就会让原主库执行 slave of 命令，和新主库重新进行全量同步。而在全量同步执行的最后阶段，原主库需要清空本地的数据，加载新主库发送的 RDB 文件，这样一来，原主库在主从切换期间保存的新写数据就丢失了。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639392866459-92439f51-b809-4ef3-b187-590c04deb72c.png#clientId=ud2205ab2-a81f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=908&id=u167128f5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=908&originWidth=1180&originalType=binary&ratio=1&rotation=0&showTitle=false&size=381537&status=done&style=none&taskId=ubc90cc82-3195-46bc-b01a-6632dc961ca&title=&width=1180)

<a name="VQqQ2"></a>
### 如何应对脑裂问题？
设置两个参数，使主库连接的从库中至少有 N 个从库，和主库进行数据复制时的 ACK 消息延迟不能超过 T 秒，否则，主库就不会再接收客户端的请求了。
> 两个配置项： min-slaves-to-write 和 min-slaves-max-lag 这两个、

即使原主库是假故障，它在假故障期间也无法响应哨兵心跳，也不能和从库进行同步，自然也就无法和从库进行 ACK 确认了。


<a name="QUJPl"></a>
## 34-第23-33课思考与总结
<a name="GW3yh"></a>
### 1.只读缓存 和 直写策略的读写缓存，有什么区别呢？
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639393151905-df359434-97b1-4861-adaf-17a0872fcea7.png#clientId=ud2205ab2-a81f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=722&id=u9a5ca22c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=722&originWidth=1352&originalType=binary&ratio=1&rotation=0&showTitle=false&size=601040&status=done&style=none&taskId=u9f6f496e-a512-4d1e-80b6-37bd056137f&title=&width=1352)
<a name="JP0Hf"></a>
### 2.使用了 LFU 策略后，缓存还会被污染吗？
在一些极端情况下，LFU 策略使用的计数器可能会在短时间内达到一个很大值，而计数器的衰减配置项设置得较大，导致计数器值衰减很慢，在这种情况下，数据就可能在缓存中长期驻留。例如，一个数据在短时间内被高频访问，即使我们使用了 LFU 策略，这个数据也有可能滞留在缓存中，造成污染。


<a name="yEHdc"></a>
### 3.开启事务时,Redis宕机，使用RDB持久化机制，事务的原子性能得到保证嘛？
能保证操作的原子性，分三个阶段去分析：<br />1.假设事务在执行到一半时，实例发生了故障，在这种情况下，上一次 RDB 快照中不会包含事务所做的修改，而下一次 RDB 快照还没有执行。所以，实例恢复后，事务修改的数据会丢失，事务的原子性能得到保证。

2.假设事务执行完成后，RDB 快照已经生成了，如果实例发生了故障，事务修改的数据可以从 RDB 中恢复，事务的原子性也就得到了保证。

3.假设事务执行已经完成，但是 RDB 快照还没有生成，如果实例发生了故障，那么，事务修改的数据就会全部丢失，也就谈不上原子性了。


<a name="l9Csc"></a>
## 35-Codis 和 RedisCluster集群方案哪个好？
<a name="MJImg"></a>
### Codis集群架构
了解就好了。
<a name="e5Iwt"></a>
#### Codis 集群中包含了 4 类关键组件：

- codis server：这是进行了二次开发的 Redis 实例，其中增加了额外的数据结构，支持数据迁移操作，主要负责处理具体的数据读写请求。
- codis proxy：接收客户端请求，并把请求转发给 codis server。
- Zookeeper 集群：保存集群元数据，例如数据位置信息和 codis proxy 信息。
- codis dashboard 和 codis fe：共同组成了集群管理工具。其中，codis dashboard 负责执行集群管理工作，包括增删 codis server、codis proxy 和进行数据迁移。而 codisfe 负责提供 dashboard 的 Web 操作界面，便于我们直接在 Web 界面上进行集群管理。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639449344137-06f51a5e-97f0-462f-a76a-ed65088dd12e.png#clientId=u5e426742-449f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=800&id=u4436cafc&margin=%5Bobject%20Object%5D&name=image.png&originHeight=800&originWidth=1314&originalType=binary&ratio=1&rotation=0&showTitle=false&size=454529&status=done&style=none&taskId=ue9b94f9d-ff05-47e7-8a39-5d4f25ad4ee&title=&width=1314)

<a name="wEyUx"></a>
### Codis集群和Cluster集群的方案对比
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639449742791-7147520e-fad1-43f8-8945-639594d04d67.png#clientId=u5e426742-449f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=542&id=u83c6a5be&margin=%5Bobject%20Object%5D&name=image.png&originHeight=542&originWidth=1346&originalType=binary&ratio=1&rotation=0&showTitle=false&size=654572&status=done&style=none&taskId=u3ee1dca2-ecfe-4660-9ef6-3c8bd671040&title=&width=1346)<br />Codis：<br />出来的比较早，较为成熟，单实例连接，客户端只需要知道一个IP，但是支持Redis的版本比较低，对新版Redis的特性可能不支持，数据迁移更方便，但是集群复杂度高。<br />Cluster：<br />出来的晚，多实例连接，支持Redis新版本特性，集群复杂度低。

<a name="b1C43"></a>
## 36-Redis支持秒杀的关键技术和实践？
<a name="BJwgQ"></a>
### 秒杀场景分析

- 瞬时并发访问高
- 读多邪少，而且读操作是简单的查询
<a name="pi7k0"></a>
### Redis在场景中发挥哪些作用？

- 秒杀场景分成秒杀前、秒杀中和秒杀后三个阶段。
- 秒杀开始前后，高并发压力没有那么大，我们不需要使用 Redis，但在秒杀进行中，需要查验和扣减商品库存，库存查验面临大量的高并发请求，而库存扣减又需要和库存查验一起执行，以保证原子性。这就是秒杀
<a name="cf6Vg"></a>
### Redis如何基于原子操作来支撑秒杀？
因为库存查验和库存扣减这两个操作要保证一起执行，一个直接的方法就是使用 Redis 的原子操作。
<a name="xgO2m"></a>
#### 方案一：使用Lua脚本去保证库存查验和扣减操作原子性
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639451270826-844fbade-744f-498b-9630-839021cd4889.png#clientId=u5e426742-449f-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=534&id=u6c1df849&margin=%5Bobject%20Object%5D&name=image.png&originHeight=534&originWidth=1128&originalType=binary&ratio=1&rotation=0&showTitle=false&size=338617&status=done&style=none&taskId=ua8a383ab-ef3f-4936-bfde-ae93e5e3970&title=&width=1128)
<a name="H47dH"></a>
#### 方案二：采用Redis实现分布式锁去查验扣减库存
这种方案相当于锁住了库存，让每次都只有一个客户端能到锁，去扣减，影响效率。

<a name="qiNXc"></a>
### 总结：
对于秒杀场景来说，只用 Redis 是不够的。秒杀系统是一个系统性工程，Redis 实现了对库存查验和扣减这个环节的支撑，除此之外，还有 4 个环节需要我们处理好。<br />1.前端静态页面的设计。秒杀页面上能静态化处理的页面元素，我们都要尽量静态化，这样可以充分利用 CDN 或浏览器缓存服务秒杀开始前的请求。

2.请求拦截和流控。在秒杀系统的接入层，对恶意请求进行拦截，避免对系统的恶意攻击，例如使用黑名单禁止恶意 IP 进行访问。如果 Redis 实例的访问压力过大，为了避免实例崩溃，我们也需要在接入层进行限流，控制进入秒杀系统的请求数量。

3.库存信息过期时间处理。Redis 中保存的库存信息其实是数据库的缓存，为了避免缓存击穿问题，我们不要给库存信息设置过期时间。

4.数据库订单异常处理，数据库没能成功处理订单，可增加订单重试功能，保证订单最终能被成功处理。

<a name="Mc9xS"></a>
## 37-Redis数据分布优化：如何应对数据倾斜问题？
<a name="hUAZz"></a>
### 什么是数据倾斜？
说白了就是，Redis集群中，大量热点数据，存在一个实例上，导致负载不均衡。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639473487965-e05a32b3-1326-442a-a7c1-7b075a9210ef.png#clientId=u2295bcad-a4a5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=888&id=ub5724380&margin=%5Bobject%20Object%5D&name=image.png&originHeight=888&originWidth=1054&originalType=binary&ratio=1&rotation=0&showTitle=false&size=314615&status=done&style=none&taskId=u5eb03aab-f219-43e3-a54f-3efb6525b47&title=&width=1054)
<a name="ZZD6L"></a>
### Redis数据倾斜的三个主要原因？
1.数据中有 bigkey，导致某个实例的数据量增加；<br />2.Slot 手工分配不均，导致某个或某些实例上有大量数据；<br />3.使用了 Hash Tag，导致数据集中到某些实例上。
<a name="DgtlT"></a>
#### 数据倾斜解决办法：
一个小建议：在构建切片集群时，尽量使用大小配置相同的实例（例如实例内存配置保持相同），这样可以避免因实例资源不均衡而在不同实例上分配不同数量的 Slot。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639473322396-ab29c75d-58c5-4748-a173-22a0c28f0d41.png#clientId=u2295bcad-a4a5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=644&id=u7102bd9c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=644&originWidth=1364&originalType=binary&ratio=1&rotation=0&showTitle=false&size=556786&status=done&style=none&taskId=u79da831a-f2b3-4af4-91d5-e13e1cdaa5b&title=&width=1364)
<a name="ub76l"></a>
#### 什么是HashTag?
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639473470376-6560ff03-920a-45b3-8180-1c3688709e0a.png#clientId=u2295bcad-a4a5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=1482&id=u0a825b18&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1482&originWidth=1508&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1422394&status=done&style=none&taskId=ud6f0d2ea-44d0-4215-8d95-2a3a01bf8dc&title=&width=1508)

<a name="bAMz7"></a>
## 38-通信开销：限制RedisCluster规模的关键因素
Redis Cluster 能保存的数据量以及支撑的吞吐量，跟集群的实例规模密切相关。<br />Redis 官方给出了 Redis Cluster 的规模上限，就是一个集群运行 1000 个实例。

Redis Cluster 在运行时，每个实例上都会保存 Slot 和实例的对应关系（也就是 Slot 映射表），以及自身的状态信息。
<a name="hUuwa"></a>
### GossIP协议
为了让集群中的每个实例都知道其它所有实例的状态信息，实例之间会按照一定的规则进行通信。这个规则就是 Gossip 协议。
<a name="iYWeh"></a>
#### Gossip 协议的两点工作原理
1.每个实例定期发送PING消息给集群中的 部分实例 ，PIng消息中封装了 实例自身的状态信息，部分其他实例信息，和Slot映射表。<br />2.实例接收到Ping消息后，会返回一个PONG消息，包含PONG消息一样的内容。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639473829723-02096c87-5aed-44d3-995a-470a738bd4de.png#clientId=u2295bcad-a4a5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=484&id=ua0939dc5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=484&originWidth=1334&originalType=binary&ratio=1&rotation=0&showTitle=false&size=279462&status=done&style=none&taskId=ua59cd9dd-7b83-41ea-82c5-017ba8ae09a&title=&width=1334)
<a name="ysEK4"></a>
#### Gossip通信对集群的影响
**网络带宽占用：**<br />实例间使用 Gossip 协议进行通信时，通信开销受到 消息大小 和 通信频率 这两方面的影响。消息越大，频率越高，开销也越大。导致网络带宽占用。<br />**随机对实例通信，无法及时更新实例状态**<br />还有一个问题：实例选出来的这个最久没有通信的实例，也是随机选出的 5个实例中挑选的，这并不能保证这个实例就一定是整个集群中最久没有通信的实例。所以，这有可能会出现，有些实例一直没有被发送 PING 消息，导致它们维护的集群状态。<br />已经过期了。
> Redis 实例发送的 PING 消息的消息体是由 clusterMsgDataGossip 结构体组成的


<a name="s5NdY"></a>
### Redis大集群建议：
最后，我也给你一个小建议，虽然我们可以通过调整 cluster-node-timeout 配置项减少心跳消息的占用带宽情况，但是，在实际应用中，如果不是特别需要大容量集群，我建议你把 Redis Cluster 的规模控制在 400~500 个实例。



<a name="n2Vc2"></a>
## 39-Redis 6.0版本新特性
<a name="LiBjG"></a>
### Redis6.0新版本主要在四个方面做了升级：
多IO线程<br />客户端缓存<br />访问权限控制<br />RESP3协议<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639475609503-de05921a-e387-4fdf-b566-75a18cc0beab.png#clientId=u2295bcad-a4a5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=962&id=u734d2cb2&margin=%5Bobject%20Object%5D&name=image.png&originHeight=962&originWidth=1342&originalType=binary&ratio=1&rotation=0&showTitle=false&size=984914&status=done&style=none&taskId=u811e25c7-dc15-4c06-9b54-8a3aa02cb39&title=&width=1342)
<a name="y9iBp"></a>
#### 请求的三个阶段：
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639475641701-24888be1-3384-43fd-ae79-675bdc921066.png#clientId=u2295bcad-a4a5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=884&id=u635d027e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=884&originWidth=1346&originalType=binary&ratio=1&rotation=0&showTitle=false&size=484277&status=done&style=none&taskId=uc5efeadb-dfca-4615-a40a-ec840b3a4ef&title=&width=1346)
<a name="gwVt5"></a>
## 40-Redis基于NVM内存的实践
看看就好。。。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639475773432-fc010492-c7d7-4684-be78-28127ffdbc4e.png#clientId=u2295bcad-a4a5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=798&id=ud18eef17&margin=%5Bobject%20Object%5D&name=image.png&originHeight=798&originWidth=1308&originalType=binary&ratio=1&rotation=0&showTitle=false&size=984706&status=done&style=none&taskId=u71045baf-c484-4213-8bcb-d68cefb3b04&title=&width=1308)

<a name="noFFh"></a>
## 41-第35-40讲问题与总结
<a name="SGMCR"></a>
### Redis使用规范
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639654391860-299fe806-dfca-4183-b314-48a3d07c0351.png#clientId=u9e7b62b4-44d6-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=952&id=u6c8a9d56&margin=%5Bobject%20Object%5D&name=image.png&originHeight=952&originWidth=1352&originalType=binary&ratio=1&rotation=0&showTitle=false&size=857651&status=done&style=none&taskId=u52312c52-eb37-4029-b276-ac412a67633&title=&width=1352)

- 强制类别的规范：这表示，如果不按照规范内容来执行，就会给 Redis 的应用带来极大的负面影响，例如性能受损。
- 推荐类别的规范：这个规范的内容能有效提升性能、节省内存空间，或者是增加开发和运维的便捷性，你可以直接应用到实践中。
- 建议类别的规范：这类规范内容和实际业务应用相关，我只是从我的经历或经验给你一个建议，你需要结合自己的业务场景参考使用。

<a name="qmkow"></a>
### Redis客户端和服务端的通信协议交换命令：RESP
Redis 使用 RESP（REdis Serialization Protocol）协议定义了客户端和服务器端交互的命令、数据的编码格式。在 Redis 2.0 版本中，RESP 协议正式成为客户端和服务器端的标准通信协议。从 Redis 2.0 到 Redis 5.0，RESP 协议都称为 RESP 2 协议，从 Redis 6.0 开始，Redis 就采用 RESP 3 协议了。不过，6.0 版本是在今年 5 月刚推出的，所以，目前我们广泛使用的还是 RESP 2 协议。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1639654574571-257fc72a-b1d6-42bd-8081-db05276d9019.png#clientId=u9e7b62b4-44d6-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=1400&id=u7c4c170d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1400&originWidth=1368&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1394275&status=done&style=none&taskId=u12362abd-ffa7-400e-90d5-ddbe0e3f8f5&title=&width=1368)




















