<a name="g8Cgx"></a>
# 课程说明
<a name="Sg3Sn"></a>
## 课程地址
[https://www.aliyundrive.com/s/eyvaRYnFDUJ](https://www.aliyundrive.com/s/eyvaRYnFDUJ)
<a name="v3CWh"></a>
## 代码地址
[https://gitee.com/niverkk/java-common-mistakes?_from=gitee_search](https://gitee.com/niverkk/java-common-mistakes?_from=gitee_search)
<a name="uU2OJ"></a>
## 章节脑图![Java业务开发常见错误100例-脑图.jpeg](https://cdn.nlark.com/yuque/0/2021/jpeg/1461694/1627033156180-291ebbeb-2a6f-4f0c-a12a-eeb26ba953ef.jpeg#clientId=u5afdea04-7550-4&crop=0&crop=0&crop=1&crop=1&from=drop&height=4499&id=u7c671cdd&name=Java%E4%B8%9A%E5%8A%A1%E5%BC%80%E5%8F%91%E5%B8%B8%E8%A7%81%E9%94%99%E8%AF%AF100%E4%BE%8B-%E8%84%91%E5%9B%BE.jpeg&originHeight=5999&originWidth=2995&originalType=binary&ratio=1&rotation=0&showTitle=false&size=2594422&status=done&style=none&taskId=uabd77b15-7e9c-4493-9420-5f3f5a7e7cb&title=&width=2246)

<a name="E79CU"></a>
# 章节
<a name="tdTZ2"></a>
## 1-使用了线程并发工具类库，线程安全就高枕无忧了吗？
<a name="Y2IB7"></a>
### 1.ThreadLocal做缓存信息错乱问题
<a name="my0sL"></a>
#### 场景描述
ThreadLocal 来缓存获取到的用户信息，错误的取到了别人的用户信息。
<a name="uVREC"></a>
#### 问题分析

1. 需要理解ThreadLocal的来龙去脉，知道线程的生命周期，和线程池中的原理。
1. 以为ThreadLocal在线程之间做了隔离，就不会有线程安全问题， 但是很多组件（Tomcat）内部会线程池，出现线程复用，导致ThreadLocal对象复用，出现缓存信息错乱问题。
1. 利用线程池，指定核心线程数为1，去复现场景。
<a name="YZ8nc"></a>
#### 解决方案
注意线程在线程中复用的问题，使用完ThreadLocal后需要显示清空缓存数据<br /> 
<a name="JH62e"></a>
### 2.ConcurrentHashMap使用不当出现的线程并发问题
<a name="hvUfK"></a>
#### 场景描述
ConcurrentHashMap容器已有900个元素，需要添加至1000 个元素，开启10个线程，填充剩下的100个元素，出现填充后的size大于1000，出现了线程并发问题。
<a name="CwZTy"></a>
#### 问题分析

1. 误以为使用并发容器能解决一切线程安全问题，有些复合操作， 会引起并非问题
1. 因为每个线程先通过size函数拿到元素个数，计算，后添加元素，在get,insert两个动作中，不是原子操作，没有加锁，导致线程安全问题。
<a name="qq53N"></a>
#### 解决方案

1. 在代码添加元素的逻辑上面加锁，保证‘取’和‘改’两个操作原子性，则能保证线程安全。
1. 使用容器自带的工具方法：	
   - CourrentHashMap.computeIfAbsent() 方法 判断Key是否存在，不存在则插入，否则新建一个。
   - LongAdder(线程安全累加器)对象做Value计数，用increment方法做累加，CAS实现，高效率。
<a name="dQ8Df"></a>
#### 其他
拓展了解一下为什么HashMap是线程不安全的？并且不安全的体现在哪里？<br />参考链接：[https://javadoop.com/post/hashmap](https://javadoop.com/post/hashmap)

<a name="kzNdv"></a>
### 3.CopyOnWriteArrayList线程安全但效率低
<a name="ZNuTR"></a>
#### 场景描述
修改非数据库操作场景下，使用了CopyOnWriteArrayList线程安全容器做缓存，耗时长，性能差。
<a name="OypkH"></a>
#### 问题分析
CopyOnWriteArrayList作为缓存，修改频繁，耗时长，因为它只适合“读多写少”的场景。<br />CopyOnWriteArrayList是线程安全的 ArrayList，但是内部是通过修改数据时，复制一份数据出来，所以只适合“读多写少”，或者“无锁读”的场景。
<a name="lWOTd"></a>
#### 解决方案
如果修改操作多，直接用ArrayList+锁的方式去实现，性能好一百倍。<br />所以，在使用并发容器的时候，一定要搞清楚场景！！！不要为了代码装逼而瞎用。

<a name="PmKZP"></a>
### 4.思考与讨论

1. ThreadLocalRandom是否可以设置为静态变量，在多线程下重用？
1. ConcurrentHashMap提供了putIfAbsent方法，作用是什么？和computeIfAbsent方法的区别？
1. 为什么HashMap是线程不安全的？并且不安全的体现在哪里？参考[https://javadoop.com/post/hashmap](https://javadoop.com/post/hashmap)
<a name="ai9m4"></a>
#### 
<a name="daxfV"></a>
## 2-代码加锁，不要让锁成为烦心事
<a name="ERHMx"></a>
### 1.为什么在add方法上加了锁也线程不安全？
<a name="zmSgW"></a>
#### 场景描述

- ![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1627229076879-2f813213-ba80-4faf-b223-e4424341df5c.png#clientId=ub19a029a-2961-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=397&id=u15186bbd&margin=%5Bobject%20Object%5D&name=image.png&originHeight=794&originWidth=520&originalType=binary&ratio=1&rotation=0&showTitle=false&size=158855&status=done&style=none&taskId=u1f5ccefa-dd72-44a2-b7ff-bf704207059&title=&width=260)
- 按道理，a 和 b 一起进行累加,应始终相等，compare中的第一次判断应该始终不会成立，不会输出任何日志。但, 执行后发现不但输出了日志,而且更诡异的是,compare方法在判断 a<b成立的情况下还输出了a>b也成立。
- 后续在add方法上加了锁，也没有解决问题。
<a name="Rp4Oi"></a>
#### 问题分析

1. 因为两个线程是交错执行 add 和 compare 方法中的业务逻辑，而且这些业务逻辑不是原子性的：a++ 和 b++操作中可以穿插在 compare 方法的比较代码中。
1. 要注意的是,a<b这比较操作在字节码层面是加载 a，加载b和比较三步，代码虽然是一行但也不是原子性的。
1. 案例中的add方法始终只有一个线程操作，只为add方法加锁是没用的，只对Compare加锁也是没用的。
<a name="yQSx7"></a>
#### 解决案例

- 正确的做法应该是为 add 和 compare 都加上方法锁，确保 add 方法执行时，compare 无法读取 a 和 b；
<a name="DARhG"></a>
#### 其他

- 使用锁解决问题之前一定要理清楚，要保护的是什么逻辑，多线程执行的情况又是怎样的。
- 加锁前要清楚 锁 和 被保护的对象 是不是一个层面的避免线程，业务逻辑，锁这三者间添加无效方法锁。
- [ ] 了解一下什么是指令重排？
- [ ] 了解一下JVM在中内存，和CPU高速缓存区交互的8个步骤。
- [ ] 了解以及多线程如何共享JVM堆内存？

<a name="wxSt0"></a>
### 2.非static方法加了锁，依旧有线程安全问题
<a name="hcSsQ"></a>
#### 场景描述
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1627288922494-593b91c2-4b87-49be-8d90-f612a84d7c57.png#clientId=u80b695a0-9d01-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=376&id=u77740817&margin=%5Bobject%20Object%5D&name=image.png&originHeight=752&originWidth=713&originalType=binary&ratio=1&rotation=0&showTitle=false&size=165328&status=done&style=none&taskId=ua99a0b1a-0cf3-4752-b6e8-32e51549ac4&title=&width=356.5)
<a name="X06C9"></a>
#### 问题分析

1. 非静态方法上的锁，在多线程下只能锁住同一个new出来的实例，不能锁住不同的实例对象，所以静态的counter在多个实例中共享，必然会出现线程安全问题。
1. 静态字段属于类，类级别的锁才能保护；而非静态字段属于类实例，实例级别的锁就可以保护。
<a name="UVIJC"></a>
#### 解决案例

1. 把wrong方法定义为静态方法，把锁升级为 类 级别锁（但不建议，尽量不要改变原有代码结构）
1. 同样在类中定义一个 Object 类型的静态字段，在操作counter 之前对这个字段加锁。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1627289351689-d8758c3e-13c4-4544-85cc-5dafa9c49f4a.png#clientId=u80b695a0-9d01-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=176&id=u8414d1a8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=352&originWidth=708&originalType=binary&ratio=1&rotation=0&showTitle=false&size=80133&status=done&style=none&taskId=u9f181090-dfef-4c45-8c2a-3217fda9143&title=&width=354)
<a name="ifaIN"></a>
#### 其他

1. 了解代码块级别的synchronized 和方法上标记 synchronized 关键字，在实现上有什么区别？
1. 加锁要考虑，锁 和 被保护的对象 是不是一个层面。

<a name="e3kiu"></a>
### 3.在锁里放入了线程安全的代码，导致锁粒度过大
<a name="ViRzF"></a>
#### 场景描述
在一个先查询接口（耗时长），然后做List容器add操作，发现线程被锁的时间过长，影响效率。
<a name="bDP3l"></a>
#### 问题分析
将不涉及共享资源的代码操作，也放入锁中，导致锁的粒度过大，其他线程等待耗时长，效率低
<a name="YIoxD"></a>
#### 解决案例

- 将不涉及线程安全问题的代码提前抽出来，降低锁的粒度，提高性能。
- 尽可能降低锁的粒度，仅对必要的代码块甚至是需要保护的资源本身加锁。
<a name="vWzSx"></a>
#### 其他

- 了解一下HashTable 和 CorentHashMap 的底层锁粒度问题，为效率更高？
- 什么是公平锁？
- synchronized加锁简单，但是避免滥用。
- Spring框架中很多Bean是单例的，加上Synch锁之后，可能导致整个程序只支持单线程，要慎重考虑清楚锁的对象和例度，避免造成性能问题。
- 如果细化了粒度还有性能问题，可考虑区分读写场景以及资源访问冲突，考虑使用悲观锁，乐观锁来实现。

加锁的3点参考：

      1. 读写差异明显的场景，考虑使用ReentrantReadWriteLock 细化区分读写锁，来提高性能。
      1. 如JDK1.8以上，共享资源冲突概率不大，可使用StampedLock 的乐观读的特性，来提高性能。
      1. JDK 里 ReentrantLock 和 ReentrantReadWriteLock 都提供了公平锁的版本，在没有明确需求的情况	下不要轻易开启公平锁特性，在任务很轻的情况下开启公平锁可能会让性能下降上百倍。

<a name="z9LrC"></a>
### 4.多锁操作小心死锁问题
<a name="FkCiQ"></a>
#### 场景描述
下单时，需锁住商品库存，在多线程场景下，A线程勾选Item1,Item2商品，B线程勾选了Item1,Item2商品，然后A线程先锁住了Item1，B线程先锁住了Item2，然后A等待Item2的锁，B等待Item1的锁，出现了死锁问题。
<a name="bRB4K"></a>
#### 问题分析

1. JDK 自带的 VisualVM 工具来跟踪一下线程的状态。
1. 锁的粒度在商品上，出现了共享资源相互占用的情况出现了锁死。
<a name="SSzXC"></a>
#### 解决案例

1. 提高锁的粒度，在整个商品库存上加锁（不可取，因为效率低）
1. 对勾选的商品排个序，然后上锁，解决死锁问题。（可取，效率高）
1. 上锁的时候，要考虑无限等待死锁问题，避免循环等待。

<a name="idHfO"></a>
### 5.思考与讨论

1. synchronized加锁方式简单，但要清楚锁的共享资源，是类级别，还是实力级别，会被哪些线程操作，sync关联的锁对象或防范又是什么范围？
1. 加锁需要考虑粒度和场景，要尽量降低锁粒度，提高效率，对性能要求高的系统，需要考虑读写锁分离，考虑悲观锁，乐观锁，尽可能针对业务去实现，考虑 ReentrantReadWriteLock、StampedLock 等高级的锁工具类。
1. 加锁需要考虑死锁问题，避免无限等待，和循环等待问题。
1. 业务逻辑中锁的实现复杂的话，要看看加锁和释放是否配对，是否有遗漏释放或重复释放的可能性；并且要考虑锁自动超时释放了，而业务逻辑却还在进行的情况下，如果别的线线程或进程拿到了相同的锁，可能会导致重复执行。
1. 建议对复杂锁操作的场景，进行压测，避免误用带来的性能问题和死锁问题。
1. volatile关键字的原理？
1. 开启了一个线程无限循环来跑一些任务，有一个 bool 类型的变量来控制循环的退出，默认为 true 代表执行，一段时间后主线程将这个变量设置为了false。如果这个变量不是 volatile 修饰的，子线程可以退出吗？你能否解释其中的原因呢？
1. 一是加锁和释放没有配对的问题，二是锁自动释放导致的重复逻辑执行的问题。你有什么方法来发现和解决这两种问题吗？

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1627293975418-584ed849-e078-4f53-9a8a-3f003b0ee786.png#clientId=uce8e24a2-5948-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=112&id=u3db512fc&margin=%5Bobject%20Object%5D&name=image.png&originHeight=224&originWidth=654&originalType=binary&ratio=1&rotation=0&showTitle=false&size=75728&status=done&style=none&taskId=ud6973c53-52b9-4d47-9ec4-35be3ae25fa&title=&width=327)<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1627293990603-4e8fb520-3bf7-4175-9979-bc6b85fd55c3.png#clientId=uce8e24a2-5948-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=79&id=ud5a68b76&margin=%5Bobject%20Object%5D&name=image.png&originHeight=157&originWidth=629&originalType=binary&ratio=1&rotation=0&showTitle=false&size=48590&status=done&style=none&taskId=u9ce1b530-99b2-4e79-a3f9-5b51f6d90b7&title=&width=314.5)<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1627293936230-ddaf113a-533d-4802-9bc1-f49a85dacf28.png#clientId=uce8e24a2-5948-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=77&id=u367b7367&margin=%5Bobject%20Object%5D&name=image.png&originHeight=154&originWidth=662&originalType=binary&ratio=1&rotation=0&showTitle=false&size=52187&status=done&style=none&taskId=u87d7edd4-3d1d-4bed-9333-5af2e413056&title=&width=331)

<a name="QgCkG"></a>
## 3-线程池：最容易犯错的组件
<a name="mdC7o"></a>
### 1.Executors类快捷工具创建线程池的OOM坑
<a name="a8cWn"></a>
#### 场景描述

- 场景1:初始化一个单线程的newFixedThreadPool，循环1亿次向线程池提交任务，每个任务创建一个较大字符休眠一小时，不久则OOM。
- 场景2:将场景1中的线程池用newCachedThreadPool，替换，不久出现OOM。
- ![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1627436591147-c08c7efd-4147-4684-8c86-657d73d3358d.png#clientId=uf12cbec4-e6d4-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=230&id=u9f01856c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=460&originWidth=667&originalType=binary&ratio=1&rotation=0&showTitle=false&size=122144&status=done&style=none&taskId=u22c9c658-727b-4eb7-a235-d889e398f96&title=&width=333.5)
<a name="VlO7s"></a>
#### 问题分析

1. newFixedThreadPool的源码中，工作队列直接new了一个Integer.MAX_VALUE 长度的LinkedBlockingQueue队列，可以默认是无界的。虽然newFIxedThreadPool可以吧工作线程数固定，但是任务队列是无限的，会使队列积压，造成内存OOM。
1. newCachedThreadPool，其工作队列是一个没有存储空间的阻塞队列,SynchronousQueue,但是池中的最大线程数为Integer.MaxValue.所以导致线程创建过多，出现内存空间OOM。
<a name="S08BB"></a>
#### 解决方案

1. 不建议使用 Executors 提供的两种快捷的线程池，需根据场景，并发情况，评估线程池的几个核心参数（核心线程数，最大线程数，线程回收策略，任务队列，拒绝策略），一般都要设置有界队列，和可控线程数。
1. 修改线程池名称，方便排查，线程数暴增，线程死锁，占用CPU，程序异常等情况。
1. 建议手动声明线程池，可用一些监控手段来观察线程池状态。
<a name="qKmp9"></a>
#### 其他

1. 了解池化思想，利用池化技术缓存昂贵对象，直取直用，通过调整策略改变对象数量，实现池动态伸缩。
1. 为什么《阿里巴巴 Java 开发手册》中提到，禁止使用Executors这些方法来创建线程池？
1. 总结默认线程池的工作行为：
   - 不会初始化 corePoolSize 个线程，有任务来了才创建工作线程；
   - 当核心线程满了之后不会立即扩容线程池，而是把任务堆积到工作队列中；
   - 当工作队列满了后扩容线程池，一直到线程个数达到 maximumPoolSize 为止；
   - 如果队列已满且达到了最大线程后还有任务进来，按照拒绝策略处理；
   - 当线程数大于核心线程数时，线程等待 keepAliveTime 后还是没有任务需要处理的话，收缩线程到核心线程数；

<a name="nAuxu"></a>
### 2.线程池没有复用，导致线程数飙高
<a name="bhbh4"></a>
#### 场景描述
报警线程数过多，忽高忽低，抖动厉害，但是访问量并不大。
<a name="fCxe1"></a>
#### 问题分析

1. 在线程数高的时候，抓去线程栈，发现内存中有1000+个自定义**线程池。**
1. 一般5个左右才正常，于是乎开始排查代码，发现有段获取线程池代码里，直接返回了Executors.newCachedThreadPool();没有对该线程池做复用，回到 newCachedThreadPool 的定义，发现核心线程数是0，而 keepAliveTime是 60 秒，是在60秒后所有的线程都是可以回收的，程序才没死。
<a name="BpnLQ"></a>
#### 解决方案

1. 使用一个静态字段存放线程池的引用，在获取线程池的代码中直接返回静态线程池的引用，但是要注意线程池混用死锁问题。
<a name="gHUs4"></a>
#### 其他

1. 合理设置池参数，并不是一个程序仅用一个线程池，要根据场景来合理定义线程池。比如：
   - （IO密集型）IO操作多的，核心线程数可以多点，等待队列短点，可以提高CPU的利用率。
   - （CPU密集型）CPU计算多的，可以线程数少点，等待队列长短，减少CPU的时间片切换成本。线程数理论设置值为： CPU核数*2  。
   - 拒绝策略选型也慎重，CallerRunsPolicy 拒绝处理策略可能会在任务多的时候，把程序从异步处理，变成同步处理的原因。
2. Java8的parallel stream功能，可以很方便地并行处理集合中的元素，其背后是共享同一个 ForkJoinPool，默认并行度是CPU 核数 -1。对于 CPU 绑定的任务来说，使用这样的配置比较合适，但如果集合操作涉及同步 IO 操作的话（比如数据库操作、外部服务调用等），建议自定义一个ForkJoinPool（或普通线程池）。

<a name="JmXMd"></a>
### 3.思考与讨论
1.根据场景和需求配置合理的线程数，队列大小，拒绝策略，线程回收策略，线程Name（方便排查问题）。<br />2.线程池的合理复用，避免死锁，使用类库提供的线程池，切记查看源码，是否符合场景，避免有坑。<br />3.根据需求场景区分：IO密集型，还是CPU密集型。合理分配，避免相互干扰。<br />4.必要时，对线程池封装成监控，可以实时观察状态。

<a name="I4bRu"></a>
## 4-连接池：别让连接池帮了倒忙
<a name="DCs16"></a>
### 1.多线程下使用Jedis，出现莫名奇妙的报错
<a name="YFMji"></a>
#### 场景描述
程序中直接：Jedisjedis=newJedis("127.0.0.1",6379)； 在多线程下操作，会出现数据获取错乱。比如new Jedis个去操作，出现：线程1，2先后get A和 B，redis返回了值1和2，但是线程AB先后读取发生了错乱问题。
<a name="yHOOn"></a>
#### 问题分析
多线程下，使用Jedis操作Redis，无法确保整条命令以一个院子操作写入Socket通信管道中，出现多个操作相互干扰，穿插的情况。其实就是多个请求源码立共用了一个socket连接。
<a name="UIVeN"></a>
#### 解决方案
使用Jedis提供的另一个线程安全类JedisPool来获取Jedis实例，JedisPool 可声明为 static 在多个线程之间共享，扮演连接池的角色。使用时，按需使用try-with-resources 模式从 JedisPool 获得和归还 Jedis 实例。
<a name="oPqfQ"></a>
#### 其他

1. TCP 面向连接的基于字节流的协议：

面向连接，意味着连接需要先创建再使用，创建连接的三次握手有一定开销；<br />基于字节流，意味着字节是发送数据的最小单元，TCP 协议本身无法区分哪几个字节是完整的消息体，也无法感知是否有多个客户端在使用同一个 TCP 连接，TCP 只是一个读写数据的管道。

2. 常见的ClientSDK的API有哪三种形式?
- 连接池和连接分离的 API：有XXXPool 类负责连接池实现，先从其获得连接Connection，然后进行服务端请求，需考虑使用完成后归还问题，使用Pool一般是线程安全的。
- 内部带有连接池的 API：对外提供一个 XXXClient 类，直接请求服务端，内部会维护了连接池，无需考虑归还问题，内部线程安全。
- 非连接池的 API：一般命名为 XXXConnection，以区分其是基于连接池还是单连接的，直接使用的时候会建立连接，断开动作。性能一般，，无需归还，通常线程不安全。

3. 连接池对外提供获得连接、归还连接的接口给客户端使用，并暴露最小空闲连接数、最大连接数等可配置参数，在内部建立连接、连接心跳保持、连接管理、空闲连接回收、连接可用性检测等功能。

4. 连接池的结构示意图：

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1627615094052-910fcc7e-4eaa-40a2-8c24-5e470a03be9a.png#clientId=u1338180e-fd9c-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=214&id=u0cc19df5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=427&originWidth=706&originalType=binary&ratio=1&rotation=0&showTitle=false&size=62999&status=done&style=none&taskId=uadd1c248-98e4-4a61-bf7c-1d3ecf357d6&title=&width=353)
<a name="gwqTx"></a>
### 2.连接池不复用，配置不合理，出现线程泄露
<a name="lSYJt"></a>
#### 场景描述

   - 每次使用的时候，都new一个连接池出来，出现了大量连接池资源浪费。
<a name="ja0gs"></a>
#### 问题分析

   - 错误的设置了参数，导致性能下降，或者连接数过多。
   - 没有复用创建的线程池，导致每次都创建线程池
<a name="Q5EpC"></a>
#### 解决方案

   - 解决就是，定义一个static线程池，实现复用。
   - 池是用来复用的，原因如：创建连接池时，可能初始化建立了多个连接，维护一定最小连接数，如果每次使用都创建一个连接池，但实际只用到了一个连接，造成其他连接，和连接池中的维护线程浪费。
   - 最大连接数不是设置的越大越好，可能会给服务端带来压力，最大连接数不是越小越好，可能导致吞吐量降低，出现超时等待，无法获取连接。
<a name="arwTh"></a>
#### 其他

1. 对类似数据库连接池的重要资源进行持续检测，并设置一半的使用量作为报警阈值，出现预警后及时扩容。
1. 修改配置参数务必验证是否生效，并且在监控系统中确认参数是否生效、是否合理。避免有坑。

<a name="MDmAW"></a>
### 3.思考与讨论

1. 使用连接池注意三点：
   1. 确保连接池的复用。
   1. 尽可能在程序退出之前，显示关闭连接池，或释放资源。
   1. 连接池配置参数，根据场景来，避免过多过少的情况。
2. 获取连接操作往往有两个超时时间：（区分连接等待时间和连接超时时间）
   1. 一个是从连接池获取连接的最长等待时间，通常叫作请求超时 **connectRequestTimeout** 或等待超时 **connectWaitTimeout**；
   1. 一个是连接池新建 TCP 连接三次握手的连接超时，通常叫作连接超时 **connectTimeout**。针对JedisPool、Apache HttpClient 和 Hikari 数据库连接池，你知道如何设置这 2 个参数吗？

<a name="A0ADI"></a>
## 5-HTTP调用：你考虑超市，重试，并发了吗？
<a name="g9fwr"></a>
### 1.Feign 和 Ribbon 配合使用，设置超时的三个坑
<a name="MHg1R"></a>
#### 场景描述

   - 坑点一 默认超时时间1秒，过短：没有设置超时参数，默认情况下 Feign 的读取超时是 1 秒。
   - 如要修改 Feign 客户端默认的两个全局超时时间，你可以设置feign.client.config.default.readTimeout 和feign.client.config.default.connectTimeout 参数。
   - 坑点二 如要配置 Feign 的读取超时，只配置readTimeout其一，则超时配置无效，必须两个同时配置。
      - ![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1627634954210-fd702c6f-59aa-4692-8220-6ed61888517b.png#clientId=u3ad926b3-58e2-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=47&id=u2789e664&margin=%5Bobject%20Object%5D&name=image.png&originHeight=93&originWidth=458&originalType=binary&ratio=1&rotation=0&showTitle=false&size=21731&status=done&style=none&taskId=uea4da9a4-bccf-47cd-b1e4-4807b3c62e1&title=&width=229)
   - 注意单独设置FeginClient超时时间，可以把 default 替换为Client 的 name，单独的超时可以覆盖全局超时，这符合预期。
      - ![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1627635266759-ba829071-4ffa-46bc-9342-a533a41962fa.png#clientId=u3ad926b3-58e2-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=78&id=ub1803e10&margin=%5Bobject%20Object%5D&name=image.png&originHeight=155&originWidth=461&originalType=binary&ratio=1&rotation=0&showTitle=false&size=38444&status=done&style=none&taskId=u2f186492-780e-48fe-980d-fe9b72ededd&title=&width=230.5)
   - 坑点三 除了可配置Feign，也可配置Ribbon组件的参数修改两个超时时间。参数首字母要大写，和 Feign 的配置不同。（同时配置 Feign 和 Ribbon 的超时，以 Feign 为准）
<a name="rb9bS"></a>
#### 问题分析
为Feign配置超时参数的复杂在于，Feign有两个超时参数，内部的负载均衡组件Ribbon本身还有相关配置。

<a name="nZynC"></a>
#### 2.注意 Ribbon 会自动重试请求呢？
<a name="YuDtZ"></a>
#### 场景描述

   - Feign调用接口，使用出现了短信重发现象，接口定义成Get请求。
<a name="wKG83"></a>
#### 问题分析

   - HTTP 客户端往往会内置一些重试策略，可能造成一些不可预估影响。
   - Feign 内部有一个 Ribbon 组件负责客户端负载均衡，当负载时，其中一个出现超时情况，可能Ribbon自己做了重试，导致重复发送。
<a name="mPbg4"></a>
#### 解决方案

1. 方案一把发短信接口从 Get 改为 Post，避免这种有状态的API接口，定义成Get方式请求，Get仅仅用于查询当中，有可能部分Nginx服务中或浏览器中，会对Get返回的东西做缓存。
1. 方案二将 MaxAutoRetriesNextServer 参数配置为 0，禁用服务调用失败后在下一个服务端节点的自动重试。在配置文件中添加一行即可：ribbon.MaxAutoRetriesNextServer=0
1. 注意！！！默认重试机制，是否对接口有幂等性问题。

<a name="XAbnp"></a>
### 2.注意HttpClient的并发限制，避免出现性能瓶颈
<a name="mOJg9"></a>
#### 场景描述
并发限制了爬虫的抓取能力：进行 HTTP 请求调用还有一个常见的问题是，并发数的限制导致程序的处理能力上不去。
<a name="CfwjT"></a>
#### 问题分析

   - 使用Http中默认的PoolingHttpClientConnectionManager构造的CloseableHttpClient，并发性能低。
   - 看源码发现，同一个主机 / 域名的最大并发请求数为 2，显然是默认值太小限制了爬虫的效率。
   - 很多浏览器内置也默认对同1个域名限制了2个并发请求。
<a name="yhz4d"></a>
#### 解决方案
声明一个新的 HttpClient 放开相关限制，设置maxPerRoute 为 50、maxTotal 为 100：<br />httpClient2=HttpClients.custom().setMaxConnPerRoute(10).setMaxConnTotal(20)
<a name="cwboh"></a>
### 思考与讨论

1. HTTP调用本质就是进行一次网络请求（网络层是 TCP/IP 协议），请求必然有超时可能性，需考虑三点：
   - 框架设置的默认超时是否合理
   - 网络不稳定的情况下，是否有重试，重试是否能保证幂等性。
   - 用的框架是否像浏览器一样限制并发连接数，避免服务并非很大的情况下，HTTP请求并发数成了瓶颈。

2. 常见的网络请求框架提供的两个超时参数：
   - 连接超时参数 ConnectTimeout，让用户配置建连阶段的最长等待时间；
   - 读取超时参数 ReadTimeout，用来控制从 Socket 上读取数据的最长等待时间。

3. 连接超时参数和连接超时的2个误区：
   - 连接超时配置的过长：一般TCP建立3次握手，只需毫秒级别，如果很久都没连上，则网络出现了隔离。
   - 排查超时，需理清楚方向：注意多个服务点下，会有Nginx代理和负载均衡技术来连接服务端，很有可能出现超时的情况是服务端。

4. 读取超时出现的误区：
   - 读取超时，并不代表服务端执行就会中断：类似Tmcat Web服务器都是把服务端请求提交到线程池处理，网络层超时断开，不会影响服务端的正常执行。
   - 认为读取超时只是Socket网络层面的概念：无法区分是服务端没有返回数据，还是网络上耗时久或丢包。有可能是网络环境问题，有可能是业务处理时间过长。
   - 认为超时时间越长，成功率就高：Http一般属于同步调用，如果等待时间过长，回耗费线程资源，影响性能，超时等待，导致程序崩溃。（通常设置超时时间不过30秒）

5. Nginx 也有类似的重试功能。你了解 Nginx 相关的配置吗？

6. 总结：注意超时时间配置，注意框架默认Get请求重试，注意接口幂等性，注意框架的同域名下并发限流。


6-业务代码中的Spring声明式事务，如何正确处理？

问题：<br />增加了事物注解，没有生效<br />1.可能是注解所加的方法是private的<br />2.可能是调用这个加了注解的方法，是在内部调用，属于this.method调用方式，必须通过代理过的类调用才能生效，spring利用切面做的事物机制，没有切到，所以没有生效。<br />3.在标记有注解方法里面做了try catch 操作，没有把异常抛出来，所以无法自动执行回滚生效。

4. 复杂的业务逻辑，由IO操作，数据库操作时，细化事务的粒度，多个事物嵌套的场景下，或在调用别的时候，需要确认事物的传播机制是否由影响和生效。
   1. （如果方法涉及多次数据库操作，并希望将它们作为**独立的事务**进行提交或回滚，那么我们需要考虑进一步细化配置事务传播方式，也就是 @Transactional 注解的Propagation 属性。）

了解：JTA，JPA 等事物API的实现。<br />了解事务的原理。

7-数据库索引：不是万能药<br /> 不要盲目建立索引，一般数据超过1万后，才建立索引，如果数据增长的快，才在建设表的时候加索引。<br />尽量使用轻量级索引，能用int 就不要用char， 因为索引也占用内存。<br />减少select * 的查询，减少没有必要的回表。<br />不要在where 中使用函数。<br />重点，数据库基于成本决定是否走索引，会考虑各个索引的执行计划，所需要的IO成本，和CPU成本，相比较，有可能在比较之下，哪怕符合索引规则，也不会走，会全表扫描。<br />这里的成本，包括 IO 成本和 CPU 成本：IO 成本，是从磁盘把数据加载到内存的成本。默认情况下，读取数据页的 IO 成本常数是 1（也就是读取 1 个页成本是 1）。<br />CPU 成本，是检测数据是否满足条件和排序等 CPU 操作的成本。默认情况下，检测记录的成本是 0.2。<br />我们可以得到两个结论：其原因就是，MySQL 并不是猜拳决定是否走索引的，而是根据成本来判断的。虽然表的统<br />计信息不完全准确，但足够用于策略的判断了。<br />MySQL 选择索引，并不是按照 WHERE 条件中列的顺序进行的；<br />即便列有索引，甚至有多个可能的索引方案，MySQL 也可能不走索引。<br />使用 optimizer trace 来分析 成本差异。



**08-判断问题：程序里如何确定你就是你**<br /> equals、compareTo 和 Java 的数值缓存、字符串驻留等问题展开讨论，希望你可以理解其原理，彻底消除业务代码中的相关 Bug。<br />1.基本类型 int long ,char 只能使用 == ，比较的是直接值。<br />2.引用类型如：Long,String 需要使用equals方法，因为变量比较的是指针指向的地址，判断是否为同一个对象，而不是比较对象内容。

我先和你说说 Java 的字符串常量池机制。首先要明确的是其设计初衷是节省内存。当代码中出现双引号形式创建字符串对象时，JVM 会先对这个字符串进行检查，如果字符串常量池中存在相同内容的字符串对象的引用，则将这个引用返回；否则，创建新的字符串对象，然后将这个引用放入字符串常量池，并返回该引用。这种机制，就是**字符串驻留或池化。 **<br />**（**但在业务代码中滥用 intern，可能会产生性能问题。**）**字符串常量池是一个固定容量的 Map。如果容量太小（Number ofbuckets=60013）、字符串太多（1000 万个字符串），那么每一个桶中的字符串数量会非常多，所以搜索起来就很慢。输出结果中的 Average bucket size=167，代表了 Map<br />中桶的平均长度是 167。 （没事别轻易用 intern，如果要用一定要注意控制驻留的字符串的数量，并留意常量表的各项指标。）<br /> 2. 如果对于对象的排序，需要定义 comparrTo方法。

3. 注意使用equals 方法比较对象，有一个重要因素是，该类对象的类加载器也必须是同一个，不然返回false。

特别是在 有热加载的时候，因为可能修改了某些对象，热加载后，没重启。 （需要理解为什么！！！）

4. HastSet 底层是HashMap,TreeSet 底层是TreeMap，HashSet使用的是HashMap调用equals方法，判断两对象的HashCode是否相等。TressSet底层是一个树形结构，需要考虑左右问题，重写compareTo计算政府值，确定排序问题。

思考与讨论：<br />在实现 equals 时，我是先通过 getClass 方法判断两个对象的类型，你可能会想到还可以使用 instanceof 来判断。你能说说这两种实现方式的区别吗？


<a name="oEdqZ"></a>
## 09-数值计算：注意精度，舍入和益处问题

1. 金额计算不要用double类型，只能使用BigDecimal,不是因为没有精度问题，而是精度更高一些。
1. 使用 BigDecimal 表示和计算浮点数，且务必**使用字符串的构造方法来初始化BigDecimal**，因为用数字构造出的BigDecimal对象依旧是精读不准确的。（使用 BigDecimal 表示和计算浮点数，且务必使用字符串的构造方法来初始化BigDecimal）
1. 避免用equals比较BigDecimail对象， 希望只比较 BigDecimal 的 value，可以使用 compareTo 方法。
1. 即使通过 DecimalFormat 来精确控制舍入方式，double 和 float 的问题也可能产生意想不到的结果，所以浮点数避坑第二原则：浮点数的字符串格式化也要通过 BigDecimal 进行
1. 总结进行数值运算时要**小心溢出问题**，虽然溢出后不会出现异常，但得到的计算结果是完全错误的。我们考虑使用 Math.xxxExact 方法来进行运算，在溢出时能抛出异常，更建议对于可能会出现溢出的大数运算使用 BigInteger 类。总之，对于金融、科学计算等场景，请尽可能使用 BigDecimal 和 BigInteger，避免由精度和溢出问题引发难以发现，但影响重大的 Bug。
<a name="LOtrd"></a>
#### 思考讨论：

- [ ] BigDecimal提供了 8 种舍入模式，你能通过一些例子说说它们的区别吗？
- [ ] 1. MySQL中的浮点数和整型数字，知道应该怎样定义？又如何实现浮点数的准确计算呢？


<a name="ctWSI"></a>
## 10- 集合类：坑满地的List列表操作
<a name="ltHfj"></a>
### 1.不能直接使用 Arrays.asList 来转换基本类型数组。	
错误例子：<br />int[]arr={1,2,3};<br />Listlist=Arrays.asList(arr);<br />正确例子：<br />Integer[]arr2={1,2,3};<br />Listlist2=Arrays.asList(arr2);

问题二：<br />String[]arr={"1","2","3"};Listlist=Arrays.asList(arr);<br />arr[1]="4";<br />try{<br />list.add("5");<br />}catch(Exceptionex){<br />ex.printStackTrace();<br />}<br />发现 arr 的结构也发生了变化，且  还出现了异常。

<a name="niELA"></a>
### 2.第二个坑，Arrays.asList 返回的 List 不支持增删操作。
Arrays.asList 返回的 List 并不是我们期望的 java.util.ArrayList，而是 Arrays 的内部类 ArrayList。ArrayList 内部类继承自AbstractList 类，并没有覆写父类的 add 方法，而父类中 add 方法的实现，就是抛出<br />UnsupportedOperationException。<br />第三个坑，对原始数组的修改会影响到我们获得的那个 List。看一下 ArrayList 的实现，可以发现 ArrayList 其实是直接使用了原始的数组。所以，我们要特别小心，把通过Arrays.asList 获得的 List 交给其他方法处理，很容易因为共享了数组，相互修改产生Bug。

误区：使用数据结构不考虑平衡时间和空间。<br />搜索 ArrayList 的时间复杂度是 O(n)，而 HashMap 的 get 操作的时间复杂度是 O(1)。所以，要对大 List 进行单值搜索的话，可以考虑使用 HashMap，其中 Key 是要搜索的值，Value 是原始对象，会比使用 ArrayList 有非常明显的性能优势。 （但是List占用内存比较小，Map占用内存大，这里是空间换时间的思想）
<a name="IbTEm"></a>
#### 思考和结论
1.调用类型是 Integer 的 ArrayList 的 remove 方法删除元素，传入一个 Integer 包装类的数字和传入一个 int 基本类型的数字，结果一样吗？不一样，Integer删除的是对象，int 删除的数组元素<br />2.循环遍历 List，调用 remove 方法删除元素，往往会遇到ConcurrentModificationException 异常，原因是什么，修复方式又是什么呢？  （用迭代器）

3.书上说 arrayList 适合查询场景，LinkedList 适合插入多的场景，但是大量实验证明，ArrayList全面碾压他，无论插入和查询都优于LinkdList 这款指针列表，因为书上说，指针插入块，但是只考虑了插入的性能，没有考虑指针移动的性能。为什么没把LinkList去掉呢，因为SDK中有别的场景需要用到这个结构。


<a name="MwrJt"></a>
## 11 空值处理：分不清楚的null 和空指针。
<a name="PfIAZ"></a>
### 1.Java代码最常见5种的Null异常
Java 代码中最常见的Null异常，场景归为以下5 种：

1. 参数值是 Integer 等包装类型，使用时因为自动拆箱出现了空指针异常；( Integer i=nll 进行 i+1 操作；)
1. 字符串比较出现空指针异常；
1. 诸如ConcurrentHashMap容器不支持 Key和Value为null,强行 put null 的Key 或 Value 会出现空指针异常；
1. A 对象包含 B，在通过 A 对象的字段获得 B 之后，没有对字段判空就级联调用 B 的方法出现空指针异常；
1. 方法或远程服务返回的 List 不是空而是 null，没有进行判空就直接调用 List 的方法出现空指针异常。

<a name="zgekp"></a>
#### 注意：
1.使用判空方式或 Optional 方式来避免出现空指针异常，不一定是解决问题的最好方式，空指针没出现可能隐藏了更深的 Bug。而处理时也并非判空然后进行正常业务流程这么简单，同样需要考虑为空的时候是应该出异常、设默认值还是记录日志等。

2. 注意在Entitly对象中，保存的时候，要注意为null属性的默认填值。可实现动态 SQL，只更新必要的字段。
2. Mysql 中为null 的坑。

测试下面三个用例，来看看结合数据库中的 null 值可能会出现的坑：

1. 通过 sum 函数统计一个只有 NULL 值的列的总和，比如 SUM(score)；（sum 函数没统计到任何记录时，会返回 null 而不是 0）
1. select 记录数量，count 使用一个允许 NULL 的字段，比如 COUNT(score)；( count 字段不统计 null 值)
1. 使用 =NULL 条件查询字段值为 NULL 的记录，比如 score=null 条件。MySQL 中 =NULL 并不是判断条件而是赋值，对 NULL 进行判断只能使用 IS NULL 或者 IS NOT NULL。


<a name="O4dpE"></a>
## 12-异常处理：别让自己出问题的时候变瞎子
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1628066860190-a08a6477-ae9b-4752-91c2-03dd791462ee.png#clientId=u68a75449-b0a7-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=632&id=u1a63e577&margin=%5Bobject%20Object%5D&name=image.png&originHeight=631&originWidth=565&originalType=binary&ratio=1&rotation=0&showTitle=false&size=72787&status=done&style=none&taskId=uda6ab261-867d-4b74-90b7-5ad54111e73&title=&width=565.5)
<a name="vjKTK"></a>
#### 注意事项：

1. 别把异常定义为静态变量，原因是会导致异常栈信息错乱，
1. 避免异常生吞，要打印日志，在抛出业务异常，避免问题排查。
1. 小心finally ，或者 catch中的，出现二次异常，导致原本异常被覆盖。
1. 使用实现了AutoCloseable 接口的资源，务必使用 try-with-resources 模式来使用资源，确保资源可以正确释放，也同时确保异常可以正确处理。
1. 如果任务通过 execute 提交，那么出现异常会导致线程退出，大量的异常会导致线程重复创建引起性能问题，我们应该尽可能确保任务不出异常，同时设置默认的未捕获异常处理程序来兜底； （如果使用了线程池，建议在开启业务代码的模块中，做好异常处理，避免导致出现异常，被线程池吞掉，没有打印日志。而且如果线程没有处理，会导致线程死掉，无法复用。）


<a name="Vsh6v"></a>
## 13-日志记录没有那么简单
几个常用的 API 方法，比如 debug、info、warn、error；
<a name="hk8Bz"></a>
### 1.不同日志框架的兼容问题
在追加日志的时候，是直接把日志写入 OutputStream 中，属于同步记录日志：<br />大量日志写入会耗时很久，会消耗IO操作，属于同步操作，高并发下影响吞吐量。
<a name="Kjbsb"></a>
### 2.关于 AsyncAppender 异步日志的三类坑
<a name="cYnqn"></a>
#### Logback 提供的AsyncAppender 即可实现异步的日志记录。
**关于 AsyncAppender 异步日志的坑**，这些坑可以归结为三类：

1. 记录异步日志撑爆内存；
1. 记录异步日志出现日志丢失；
1. 记录异步日志出现阻塞。

其实，我们只是换成了 Log4j2 API，真正的日志记录还是走的 Logback 框架。没错，这就是 SLF4J 适配的一个好处。 （门面模式）<br />总结：使用异步日志解决性能问题，是用空间换时间。但空间毕竟有限，当空间满了之后，我们要考虑是阻塞等待，还是丢弃日志。如果更希望不丢弃重要日志，那么选择阻塞等待；如果更<br />希望程序不要因为日志记录而阻塞，那么就需要丢弃日志。

<a name="QNxgG"></a>
## 14-文件IO：实现高效正确的文件读写
<a name="hMVH2"></a>
#### 问题背景：
读取第三方文件，在新部署的服务器上，出现乱码，旧的服务器没有乱码。
<a name="vw4xX"></a>
#### 原因分析：
注意代码里种的是FIleReader 读取文件流，而FileReader 是以当前机器的默认字符集来读取文件的，恰好旧服务器编码方式和新服务器编码方式不一样。

<a name="nMx2n"></a>
#### 总结、解决：
“文件读写需要确保字符编码一致”。<br />1.直接使用 FileInputStream 拿文件流，然后使用 InputStreamReader 读取字符流，并指定字符集。<br />如果你觉得这种方式比较麻烦的话，使用 JDK1.7 推出的 Files 类的 readAllLines 方法，可以很方便地用一行代码完成文件内容读取：但这种方式有个问题是，读取超出内存大小的大文件时会出现 OOM，readAllLines 方法的源码可以看到，readAllLines 读取文件所有内容后，放到一个List<String> 中返回，如果内存无法容纳这个 List，就会 OOM：

2. 使用Files类的一些流式操作，注意使用try-with-resources 包装Stream，明确底层文件资源可以释放，避免产生 too many open files 的问题。
2. 进行文件字节流操作的时候，一般考虑使用缓冲字节流，或者自定义缓冲字节数组，或者追求性能极限的话使用FileChannel进行流转发。

总结问题注意：<br />最后我要强调的是，文件操作因为涉及操作系统和文件系统的实现，JDK 并不能确保所有IO API 在所有平台的逻辑一致性，代码迁移到新的操作系统或文件系统时，要重新进行功能测试和性能测试。

4.Java 的 File 类和 Files 类提供的文件复制、重命名、删除等操作，是原子性的吗？<br />非原子性，没有锁，也没有异常后的回滚。需要调用方进行事务控制<br />**5.按需求实现流式读取，Files.lines**<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1628080992531-87d9ba80-fd4a-440d-b3f7-643a3eb4ee92.png#clientId=u68a75449-b0a7-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=694&id=u9a260e2d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=964&originWidth=800&originalType=binary&ratio=1&rotation=0&showTitle=false&size=365474&status=done&style=none&taskId=u289ba8df-6675-45cb-bca2-be84ebdfc62&title=&width=576)<br />15-序列化：一来一回还是原来的你？

<a name="ve46i"></a>
## 21-类似点击流这样的海量数据应该如何存储？
1.两种方案： 1Kafka 。 2 HDFS + hive <br />其他方案：分布式流数据存储 （Pravega, Apache BookKeeper）； 时序数据库存储（InfluxDB ，OpenTSDB等）。 <br />现在对这种行文跟踪日志，都是 先存储，后计算，因为存储便宜。<br />问题：KafKa 受制于单节点的存储容量，Kafka 实际能存储的数据容量并不是无限的。<br />现代的消息队列，本质上就是分布式的流数据存储系统。

<a name="TkKTi"></a>
## 22-面对海量数据，如何查的快。
根据查询来选择存储系统 和数据结构<br />对于海量数据来说，选择存储系统没有银弹，重要的是转变思想，根据业务对数据的查询方式，反推数据应该使用什么存储系统、如何分片，以及如何组织。

<a name="VhIkg"></a>
## 23Mysql经常遇到高可用，分片问题，NewSQL是如何解决的？
 New SQL 是新一代的分布式数据库，它具备原生分布式存储系统高性能、高可靠、高可用和弹性扩容的能力，同时还兼顾了传统关系型数据库的 SQL 支持。更厉害的是，它还提供<br />了和传统关系型数据库不相上下的、真正的事务支持，具备了支撑在线交易类业务的能力。

<a name="GeeBw"></a>
## 24-RocksDB：不丢数据的高性能KV存储。
RocksDB 是一个高性能持久化的 KV 存储，每秒20万查询，相比Redis，每秒50万查询要差一些，但是Redis在持久化磁盘时又可能丢失数据，但是RocksDB 是必须落到磁盘才算写入成功。实现原理是 次用磁盘+ 内存缓存相结合的方式实现。

<a name="EgN59"></a>
## 加餐3-定位应用问题套路
DEBUG用于开发调试、INFO 用于重要流程信息、WARN 用于需要关注的问题、ERROR 用于<br />阻断流程的错误。



