<a name="qtgnF"></a>
# 非运行时
主要聊聊：代码编译，类加载，JIT及时编译，Java对象创建过程
<a name="etcHE"></a>
## 代码编译
java文件通过JavaC编译，经历语法解析器，语意解析器等，变成class字节码文件。<br />这里还可以通过JavaP反编译成Java文件，这样就衍生出一个问题，打包之后的代码安全问题？<br />其实有那种加密生成jar的工具，一般是针对类加载的过程入手，自定义类加载器，实现源文件的加解密。<br />具体工具Github上多的是，这里不多BB了，了解一下就行。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634039321737-1413adaf-d256-48d9-af70-772ec2d2acde.png#clientId=ue4769271-855f-4&from=paste&height=252&id=u00b4f075&margin=%5Bobject%20Object%5D&name=image.png&originHeight=292&originWidth=647&originalType=binary&ratio=1&size=87100&status=done&style=none&taskId=u24219f72-9610-4d99-9231-76b1cbd48f6&width=558.4942932128906)
<a name="cDpl8"></a>
## 类加载器
<a name="vdIzN"></a>
### class文件加载过程

1. 从磁盘中拿到文件流
1. 将磁盘文件静态结构载入内存方法区（元空间），转换为运行时数据结构《类信息》
1. 把载入后的类信息进行组装，在堆空间中生成类对象（class对象），做为数据入口。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634039371655-782c517f-27e8-49ec-a66b-73b576c1843e.png#clientId=ue4769271-855f-4&from=paste&height=514&id=u7d804f46&margin=%5Bobject%20Object%5D&name=image.png&originHeight=503&originWidth=757&originalType=binary&ratio=1&size=426104&status=done&style=none&taskId=ub2878e9e-086a-47a3-98f2-b206ee40ee4&width=773.48291015625)<br />简单来说：<br />双亲委派机制：就是拿到字节码信息之后不会直接加载，会丢给父类去加载，直到没有符合加载的父类为止，这样的好处就是避免同一个文件出现重复加载的情况。<br />但是在Tomcat里面的加载机制是违背双亲委派原则的，因为有可能一个Tomcat容器里面要部署多个系统，这样不同系统里面会有同名的文件，就会出现加载混乱。

<a name="fPAmv"></a>
## JIT及时编译
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634040021554-3cc00aed-93fb-4ab3-a532-c7296d0e957b.png#clientId=ue4769271-855f-4&from=paste&height=188&id=uc859439f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=197&originWidth=832&originalType=binary&ratio=1&size=353103&status=done&style=none&taskId=ub5088658-35d6-40de-a43f-d46b451c1a0&width=791.9857788085938)
<a name="zBnBy"></a>
### 简单来说
HotSpot中的Java程序通常情况下都是通过解释器，一边解释，一边执行，并不是全部把Class字节码文件全部加载进去。也不是每次使用的时候才解释加载，里面有个热点代码检测机制，当部分代码被识别为热点代码就会提前加载变成机器码。

**其实JIT编译只是一个统称，具体要看jvm是client端还是server端的，不同的端会分为C1，C2编译器**<br />C1:追求编译速度，  C2:追求编译质量

**记得以前有个概率叫 codeCache 空间，不知道是不是放这玩意的。**
> 简言之codeCache是存放JIT生成的机器码(native code)。当然JNI(java本地接口)的机器码也放在codeCache里，不过JIT编译生成的native code占主要部分。
> 都是属于非堆空间。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634040039976-951c8571-34d8-4adb-84d2-ea92ae30097a.png#clientId=ue4769271-855f-4&from=paste&height=384&id=u415afccb&margin=%5Bobject%20Object%5D&name=image.png&originHeight=295&originWidth=393&originalType=binary&ratio=1&size=99663&status=done&style=none&taskId=u9a66cec7-551c-4864-bb60-e3e96967768&width=511.48577880859375)
<a name="xAHxo"></a>
### 几个关键字
方法调用计数器：顾名思义就是记录调用次数。<br />回边计数器：统记方法体内的循环体。<br />而且：编译的最小单元是方法，运行前要先检查是否编译。<br />如果编译了就直接运行。<br />如果没有编译就需要先经过JIT的加载，提交OSR编译请求，执行编译。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634040055360-68db4da8-c25e-473a-8e9c-268f8f7ddaee.png#clientId=ue4769271-855f-4&from=paste&height=345&id=ub7dbde46&margin=%5Bobject%20Object%5D&name=image.png&originHeight=300&originWidth=443&originalType=binary&ratio=1&size=170120&status=done&style=none&taskId=u467db7a9-47cd-473d-ad47-a6f84aa6a34&width=509.49147033691406)<br />**记住几个关键点：**

- 方法是编译的最小单元。
- JIT里面会做**“热点测探”**，通过技术器的方式记录方法的执行次数等。
- JIT内部会做**“栈上替换”**，通把原本的方法调用地址替换为编译后的方法地址。

<a name="N611v"></a>
## Java对象创建的过程
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634042769951-bbd1c03e-f254-4eb9-9b75-84c47bda9b93.png#clientId=ue4769271-855f-4&from=paste&height=579&id=ub0316f28&margin=%5Bobject%20Object%5D&name=image.png&originHeight=557&originWidth=823&originalType=binary&ratio=1&size=743127&status=done&style=none&taskId=u0c99f859-3cf0-4a9f-8977-e247f6b6a89&width=854.9971313476562)

<a name="eQ8x5"></a>
### 这里先要解释一下什么是TLAB？
> TLAB线程本地分配缓存区是什么？工作原理分析，TLAB全称Thread Local Allocation Buffer，即**线程本地分配缓存区**，是一个线程专用的内存分配区域。在线程初始化时，虚拟机会为每个线程分配一块TLAB空间，只给当前线程使用。

**简单来说：**<br />就是JVM为每个线程在堆内存中划分的一个“**可分配空间**”，这样的好处就是为了避免多线程下，在堆空间上的分配错乱问题。其实内存读取是共享的，只是有独有的**分配权**。

TLAB空间的内存也非常小，默认仅占有整个Eden空间的1%，所以一些大对象是不经过TLAB区域分配的，这种分配就需要进行同步控制，所以经常说的小的对象比大的对象分配起来更加高效。

这样可以提升分配效率，且不影响对象的移动和回收。<br />也可以通过选项-XX:TLABWasteTargetPercent设置TLAB空间所占用Eden空间的百分比大小。

<a name="mz5Ow"></a>
### Java对象创建过程？
这里其实也是运行时实现的，只是为了方便配合阅读，排版就就加在这里。
> **new ->  类加载检测  ->  对象内存分配  ->  值初始化    ->  设置对象头   ->  执行init 方法。**

<br />
<a name="wgmFD"></a>
#### 遇到new 指令时
先看是否能在常量池中找定位 **类的符号引用 **

<a name="yBnAR"></a>
#### 类加载检测  
后看涉及的class相信是否已经加载。

<a name="Udpl2"></a>
#### 对象内存分配
分配堆内存，具体分配方式看堆的整齐性，这个🈶️GC机制决定。<br />分配方式：<br />1.指针碰撞（堆整齐的话） 啥叫指针碰撞，是不是可以理解为指针移动，指向空闲内存地址就行。。<br />2.空闲内存列表 （堆不整齐的话）就是一张记录空闲内存的列表，相当于可用库存账本。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634045363659-f4cc0c14-6c1d-4654-96da-42182064de3c.png#clientId=ue4769271-855f-4&from=paste&height=280&id=u26aa698e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=173&originWidth=321&originalType=binary&ratio=1&size=75259&status=done&style=none&taskId=ua34244fd-4315-480a-968e-32ff169d274&width=519.4971466064453)

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634045376417-37975d70-d9fe-4432-a929-1f9c6b00c33f.png#clientId=ue4769271-855f-4&from=paste&height=267&id=u9db7e007&margin=%5Bobject%20Object%5D&name=image.png&originHeight=160&originWidth=336&originalType=binary&ratio=1&size=69518&status=done&style=none&taskId=u2915c73a-4400-4426-9f83-82212506db4&width=559.9885864257812)<br />**分配时并发问题及解决方案**：<br />1.CAS自旋锁（比较交换机制，乐观锁）  这就是上面所说的大对象分配机制。。？<br />2.TLAB本地内存，指定一块区域的分配权限。

<a name="GKq2F"></a>
#### 值初始化
为对象的属性分配一些初始化值，比如int = 0, long = 0 等默认值。

<a name="s1lsp"></a>
#### 设置对象头
填充对象头信息，比如HashCode, GC 年龄，锁标记，锁信息，Class类型指针等等。。<br />说白了就是内存也整好了，初始化也正好了，要复盘记录自己的财产信息了。。
<a name="Zmei9"></a>
#### 执行Init方法
最后一步执行init函数，也就是类的构造函数，主要对属性赋值。

仔细理解下面分配过程：  细品！！！<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634044665010-6c7ce3bb-5a86-4262-a429-6cd6674484e0.png#clientId=ue4769271-855f-4&from=paste&height=138&id=u06422a5e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=147&originWidth=827&originalType=binary&ratio=1&size=241849&status=done&style=none&taskId=u8cdf39cc-eee9-4bb8-bc03-1912802ae36&width=776.4942932128906)


<a name="nYVwL"></a>
# 运行时区
主要按照，线程私有区，和 线程共享区 划分。<br />里面有栈，和堆，元空间，主要是这三块区域。
<a name="UJIoB"></a>
## 线程私有区
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634126182037-0e706bcf-818e-4047-8396-0e0484d66477.png#clientId=u3f4b921e-4f2a-4&from=paste&height=731&id=u9b01ce7d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=904&originWidth=1068&originalType=binary&ratio=1&size=1671625&status=done&style=none&taskId=ua1c491ca-74a5-4694-be41-8be91cce674&width=863.9971313476562)<br />该区域：线程独享，线程安全区域，主要包含：虚拟机栈，本地方法栈，程序计数器

<a name="OUX4O"></a>
### 虚拟机栈
主要包含：局部变量表，操作数栈，动态链接表，方法出口，等等。<br />一个方法就是一个栈帧<br />这里注意，对象引用信息和数据存储，是放在局部变量表里面的。<br />主要理解局部变量里面的区域，！！！
<a name="iqDCC"></a>
#### 局部变量表
包含：基本数据类型数据，reference, returnAddress，等等。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634126160152-d904aa8f-2aa8-4f92-b2f0-23a1d7325c9c.png#clientId=u3f4b921e-4f2a-4&from=paste&height=136&id=u8dbf06d5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=137&originWidth=466&originalType=binary&ratio=1&size=143206&status=done&style=none&taskId=u20b481c9-9e3d-49a7-91ef-21b98d073e9&width=463.3267059326172)<br />简单来说，里面存放基本数据类型，和引用对象的信息，其中引用对象信息。<br />而引用对象方式又分为两种：<br />**句柄池引用 **： 可理解为直接连接的是堆内存中的句柄<br />**地址引用**：可理解为直接连接的是对象的物理地址<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634126470453-42533d0f-01f3-4d12-9c40-ac89c53b2dbd.png#clientId=u3f4b921e-4f2a-4&from=paste&height=48&id=ub41fcf44&margin=%5Bobject%20Object%5D&name=image.png&originHeight=55&originWidth=471&originalType=binary&ratio=1&size=28216&status=done&style=none&taskId=u43b1b4be-dfc9-4ded-b4d8-317ce4b8df9&width=410.98863220214844)
> 对象创建出来之后，可以通过栈上的 reference 数据来操作堆上的对象了，
> 访问一般有两种方式：**句柄和直接指针**
> 句柄：java堆中有一块区域叫做句柄池，句柄池存储的是对象的句柄信息，reference存储的就是对象的句柄地址。这种方式的好处就是即使对象在堆中频繁地移动，改变的也只是句柄中的实例数据指针，而对栈中的reference数据不影响；
> 直接指针： 这种方式reference中存储的就是对象的内存地址；
> 对比：这两种方式各有优缺点，其中直接指针优点就是访问速度快，小缺点是对象实例数据频繁移动，维护这个直接内存地址也需要开销；句柄的优点就是稳定，但是要在堆中创建维护一个句柄池，这样访问性能可能会稍差。


<a name="iLiTP"></a>
#### 操作数栈
可以理解为PC寄存器，就是CPU运行时可能需要临时存储的一些鬼东西。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634127018193-596cd2fe-1fdd-4b1b-983a-0f333a9cdc59.png#clientId=u3f4b921e-4f2a-4&from=paste&height=62&id=ue6499aff&margin=%5Bobject%20Object%5D&name=image.png&originHeight=52&originWidth=448&originalType=binary&ratio=1&size=57681&status=done&style=none&taskId=u82a58b31-81a2-481a-ae7c-6cf3dedd9e5&width=535.3323822021484)

<a name="xGeO7"></a>
#### 动态链接表
OOP核心思想:封装,继承,多态<br />大概意思就是，在编译的时候无法确定具体的对象类型，需要在运行的时候才能确定对象。<br />这块动态链接表区域，就是解决这块区域的。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634127063203-12a1cf59-a1b2-4e35-afd3-0a3a5138f729.png#clientId=u3f4b921e-4f2a-4&from=paste&height=78&id=u2f4f5c0b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=73&originWidth=439&originalType=binary&ratio=1&size=35399&status=done&style=none&taskId=u821644ea-41df-4796-95d7-acc0eff8df2&width=471.3210144042969)
<a name="LDFsY"></a>
#### 方法出口区
记录方法出栈地址的区域<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634127162687-0386686a-2371-4f56-9ff9-9123ec3d6c08.png#clientId=u3f4b921e-4f2a-4&from=paste&height=53&id=u224b4362&margin=%5Bobject%20Object%5D&name=image.png&originHeight=50&originWidth=438&originalType=binary&ratio=1&size=54409&status=done&style=none&taskId=u49dd3a71-8f31-40d5-976e-e9d86d12fe1&width=464.9943084716797)

<a name="C0rvl"></a>
### 本地方法栈
与上面的虚拟机栈类似，执行一些Native 标记的方法，里面是C，C++执行的空间。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634127292267-4000a0b7-c522-4866-aa08-aa11dc6e77b2.png#clientId=u3f4b921e-4f2a-4&from=paste&height=58&id=ub7f3c498&margin=%5Bobject%20Object%5D&name=image.png&originHeight=53&originWidth=436&originalType=binary&ratio=1&size=61090&status=done&style=none&taskId=u4b65b0b8-51ad-412f-8cb2-e763834d19a&width=478.3267059326172)
<a name="dZre6"></a>
### 程序计数器
就是CPU是按时间片执行的，可能线程运行到一半就要停止，这玩意就是记录代码指令运行到哪个位置的。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634127367063-a3710cf5-287d-4fef-8651-19801d237fa4.png#clientId=u3f4b921e-4f2a-4&from=paste&height=123&id=u52e1d448&margin=%5Bobject%20Object%5D&name=image.png&originHeight=91&originWidth=446&originalType=binary&ratio=1&size=118996&status=done&style=none&taskId=u5a8ce33d-3676-4d08-ad40-966a3816d0c&width=604.6647338867188)


<a name="USOGw"></a>
## 线程共享区
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634128391691-e683aabb-928a-426e-bfaa-0714b64ca172.png#clientId=u3f4b921e-4f2a-4&from=paste&height=514&id=u184d29b5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=624&originWidth=1045&originalType=binary&ratio=1&size=959734&status=done&style=none&taskId=u552b257a-9c8f-4955-b7ef-19b4317949f&width=860.9971313476562)<br />此区域大分为两块区域： 本地内存， 直接内存 <br />本地内存： JVM虚拟机中使用的内存，里面有堆，方法区，常连池等等。。。<br />直接内存：操作系统所管的区域，也可能会出现OOM，直接被操作系统kill掉进程。<br />   在Java1.5只有，NIO操作类里面有个函数byteBuffer()，可以供程序员使用这块区域。<br />这里不多逼逼，主要讲解本地内存区域的划分。

<a name="nnVgz"></a>
### 元数据
Java8之后定义的元数据区，其实里面也就是，常量池，方法区，都是属于永久代，一般不会被GC清楚，如果真的因为频繁反射生成class信息，也有可能出现内存不足，会触发FullGC，也可能会导致OOM。<br />这里默认的空间很少，在生产环境的JVM参数模板中一般都会调整到200M。
<a name="Zgu99"></a>
#### 常量池
就是存储的是运行时的一些常量，比如int= 1, int =127 之间的一些区域。<br />也是内存共享的哦。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634128616723-5eb0cd5b-a9e6-4082-943f-d4ceb2437c84.png#clientId=u3f4b921e-4f2a-4&from=paste&height=149&id=ueb901646&margin=%5Bobject%20Object%5D&name=image.png&originHeight=121&originWidth=263&originalType=binary&ratio=1&size=84164&status=done&style=none&taskId=u287193bb-5166-4631-a458-2d36afd736c&width=324.6505432128906)
<a name="Uzp9Y"></a>
#### 方法区
说白了就是存储一些，类相关的信息，Class模板，Class信息等等。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634128603965-38be4309-4f01-4c05-987a-31d2c45a97ae.png#clientId=u3f4b921e-4f2a-4&from=paste&height=130&id=u5663c3b7&margin=%5Bobject%20Object%5D&name=image.png&originHeight=137&originWidth=570&originalType=binary&ratio=1&size=104038&status=done&style=none&taskId=ua10dfdb0-efdc-4b49-9b4e-9a6a5dcd1bf&width=541.9942932128906)
<a name="pZCCA"></a>
### 堆空间
堆里面设计的内容就有点多了，什么新老年代的空间分布啊，GC回收算法啊，对象内部存储结构啊，垃圾识别算法啊，JVM调优啊，OOM排查啊，TLAB啊，对象句柄池啊，各种乱七八糟的，这里就简单聊一聊。<br />主要分为：新生代，老年代（一般的默认大小比例是1：2 ）<br />新生代分为：Eden 区 和 S1 和 S2 区（一般默认比例是 8：1：1）<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634128896026-2abcec37-3ac3-4dbb-9583-13a89cefe230.png#clientId=u3f4b921e-4f2a-4&from=paste&height=270&id=u9ca45eec&margin=%5Bobject%20Object%5D&name=image.png&originHeight=345&originWidth=1030&originalType=binary&ratio=1&size=541616&status=done&style=none&taskId=u7fc3eb5d-beda-40ce-a11f-617f86ce3a9&width=806.3209838867188)

<a name="E219z"></a>
#### 新生代

新生代主要记住四块区域：Eden区域，S1，S2区域。<br />**Eden区**又包含**：**<br />**实例对象区域**<br />**TLAB分配区**<br />**字符常量池区域**<br />Survivor区包含：（其实两块区域一样的，只是叫法不同。）<br />S-To区域<br />S-Form区域

**Eden区域**<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634131851062-cfc43b82-e869-426e-b80f-367b946df602.png#clientId=u3f4b921e-4f2a-4&from=paste&height=288&id=ue71ae26e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=277&originWidth=561&originalType=binary&ratio=1&size=232319&status=done&style=none&taskId=ud1ad6801-64b7-42e7-8a0e-b402a9e0b69&width=582.9801025390625)<br />**E-1. TLAB区域：**<br />**属于Eden区域内**<br />上面说了，TLAB（线程本地内存）就是为了避免线程分配对象时内存冲突的一个区域，就是为每个线程绑定了对象地址的分配权限区域。这里就不多BB。

**E-2. 字符串常量池：**<br />**也属于Eden区域内**<br />具体是啥玩意，参考一下这个博客：[https://www.cnblogs.com/weibanggang/p/9457999.html](https://www.cnblogs.com/weibanggang/p/9457999.html)<br />目的就是把一些常用的字符串常量加载到堆Eden区里，减少对象创建的时间，提高空间利用率。<br />简单来说，这个字符串常量池就是 String = "abcde"; 的时候，“abcde”是直接存储在这块区域的。但是如果是new String("abcde"); 的时候，就是直接存在Eden对象里的。**空间换时间 **的操作罢了！！<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634130546303-60f2049a-68ca-4d31-80f7-834de9f315e9.png#clientId=u3f4b921e-4f2a-4&from=paste&height=613&id=ub698b46c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1402&originWidth=1674&originalType=binary&ratio=1&size=927443&status=done&style=none&taskId=u53a1c213-9ee4-497d-9973-799bf75a3a4&width=731.9971313476562)<br />E-3.实例对象：<br />线程直接在Eden区域创建对象实例，对象内存块 里面又包含一大堆东西，<br />这里就简单说一下区域，后面单独拿出一个章节来复盘这个“对象块”的知识。

实例对象块 包含：<br />**对象头 **包含：<br />markword: 锁标记，GC年龄，Hash码，等等，在32，64位中存储结构还不同。<br />类型指针：指向Class类对象的一个指针<br />数组长度：数组是一块特殊区域，专门为 数组对象 存放内容的空间。<br />**实例数据**：对象真正存储内容的空间。<br />填充物：虚拟机的内存管理要求**对象的大小必须是8字节的整数倍，**用来充数的区域

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634131800887-1804dd0b-dae3-44a3-8432-cd9077c72e59.png#clientId=u3f4b921e-4f2a-4&from=paste&height=278&id=ud3c496d4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=193&originWidth=242&originalType=binary&ratio=1&size=92151&status=done&style=none&taskId=ud608bbbe-590b-4fb4-83a2-383318e20e4&width=348.64202880859375)<br />**MarkWord 区域明细**<br />具体存储的东西如下，里面那些锁标记，后面跟着synchroized锁升级的时候在细说。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634131992477-67d6c369-1234-4c38-aba2-bf87f63cb6cf.png#clientId=u3f4b921e-4f2a-4&from=paste&height=420&id=u8ca71a34&margin=%5Bobject%20Object%5D&name=image.png&originHeight=578&originWidth=938&originalType=binary&ratio=1&size=338285&status=done&style=none&taskId=u01f78169-a690-46bd-a0bb-5439b26edf2&width=681.65625)

**Survivor区域**<br />这块区域一般是经历了 YoungGC之后，对象从Eden区进入到S区域<br />注意：这里是 复制算法 来垃圾回收。每次回收时，永远有一个S区为空。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634132571667-358bd033-1fd0-403e-b8f2-9591c2f04f12.png#clientId=u3f4b921e-4f2a-4&from=paste&height=260&id=ue0390cba&margin=%5Bobject%20Object%5D&name=image.png&originHeight=279&originWidth=510&originalType=binary&ratio=1&size=238717&status=done&style=none&taskId=u096bfdb7-2dcb-4780-80fe-f4122d4aa65&width=475)


<a name="Wufgv"></a>
#### 老年代
这里主要是存储一些经常用的对象，比如Spring中的bean, 缓存对象等 都在这个区域。<br />主要是从新生代中晋升过来，里面涉及一些“年龄动态规划”，“空间分配担保机制”，"大对象直接从Eden区晋级"，“垃圾识别算法”，“垃圾清楚算法”，STW停顿，指针压缩，垃圾回收整理，JVM吞吐量等等。这里不多逼逼，后面单独拿出一个章节讲讲GC这一套东西。<br />本图主要描述的就是：<br />到达年龄的晋升，大对象的直接晋升，年龄动态规划，这三方面进入老年代的方式。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634132802632-91f150bf-6331-4ea7-9e33-8e9e46edd74f.png#clientId=u3f4b921e-4f2a-4&from=paste&height=421&id=u10f6378c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=277&originWidth=208&originalType=binary&ratio=1&size=165130&status=done&style=none&taskId=uc24a2704-9b26-4b1f-853c-dbb240e02ad&width=316.3210144042969)<br />分配担保机制：简单来说就是怕从新生代进入老年代的时候，放不下，要担保判断一下能放的下。<br />在YGC之前，老年代要担保现有可用连续空间，大于 新生代对象大小总或 历次晋升平均的大小，才能YGC，否则就要先FullGC。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634437495902-e0c17d71-e940-4401-b4ec-81d98e5957e4.png#clientId=ub478cf2a-78e5-4&from=paste&height=120&id=ufb7638e9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=240&originWidth=2208&originalType=binary&ratio=1&size=536269&status=done&style=none&taskId=uacdcea58-0647-4737-8834-f18ac4b0234&width=1104)


<a name="TZqwk"></a>
# JVM调优
<a name="HKX37"></a>
## JVM参数调优
没有十分完美的JVM参数模板，一般都是根据自己系统的业务场景去做调优，但是大部分公司都会出一套默认模板，指定部分参数，一些特殊系统，如高并发，高运算，大内存运算，多反射等场景下的需要特别调节。
<a name="my0Di"></a>
### 调优的指标

   1. 降低STW延迟：就是减少用户线程被停顿的时间。
   1. 提升最大吞吐量：就是为了系统多处理一些用户线程的事情，减少垃圾回收。
   1. 减少内存占用：就是少用内存，减少成本。 达到不给牛吃草还干活贼猛的状态。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634214270533-c7b59afe-4502-488f-8260-4dcc72092dd9.png#clientId=u16b3059d-d69b-4&from=paste&height=128&id=u3971943f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=156&originWidth=1085&originalType=binary&ratio=1&size=135739&status=done&style=none&taskId=u9e8214ff-1cf7-4c89-9aad-8ea0e5c4dcb&width=887.9971313476562)<br />下图罗列一堆参数，没有哪个面试官能完全背下来，一套JVM参数，有几百个，咱们只需记住常用的就行。
<a name="QYh3A"></a>
### 常用参数

1. 堆最大，最小空间，这里一般参数中设置的相同
1. 栈的大小， 这里一般默认2M，如果高并非系统下， 线程数量创建较多系统，可以设置少一点。
1. GC回收信息，Dump日志地址，这些日志配置，在OOM，CPU高负载时，可以作为排查依据。
1. 堆中的Eden，S区比例，young/Old区域的比例调节，一般会对系统的吞吐有影响，这要记住
1. 大对象进老年代的阀值大小，这个一般在多缓存系统下要特别设置。
1. -XX:UserG1GC，指定使用G1回收器，记住G1内部回收的原理。为啥要这么用
1. 堆空间占用到百分之多少时，触发FullGC这个比例也是需要提到的。
1. 还有FGC之后几次要做碎片调整，这个也是需要设置的。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634214285643-0c3ad8f6-0596-4945-9181-ac71bde693b6.png#clientId=u16b3059d-d69b-4&from=paste&height=359&id=u4b18c615&margin=%5Bobject%20Object%5D&name=image.png&originHeight=486&originWidth=1186&originalType=binary&ratio=1&size=498413&status=done&style=none&taskId=u03699886-9dd9-4ce8-b03a-c36a225dfcf&width=875.3266906738281)

多提那么几嘴，一般YGC的效率比较高，就是尽量去做YGC，减少OldGC，🈶️的公司甚至实现了0FullGC。<br />关于JVM优化可以参考另一篇笔记“JVM”，那里面介绍了大量的GC实战案例。
<a name="ESgsl"></a>
## GC回收器
<a name="eixlz"></a>
### 开胃小菜
在开始聊GC算法之前，先了解图中几个概念。
<a name="JV59B"></a>
#### STW
（用户线程停顿）<br />GC过程中，做垃圾对象的检测，需要暂停用户线程，导致JVM假死状态，影响系统正常工作。<br />为啥要暂停？ <br />就是避免：一边检测，一边生产垃圾，不能准确判断垃圾对象啦呀。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634215377252-e94e8f2b-f40e-483e-975e-47df28e8daf5.png#clientId=u16b3059d-d69b-4&from=paste&height=145&id=JE1OE&margin=%5Bobject%20Object%5D&name=image.png&originHeight=147&originWidth=668&originalType=binary&ratio=1&size=246888&status=done&style=none&taskId=ud588d375-e5db-4fe7-a008-3a0506eae7a&width=657.6562347412109)<br />这里衍生出一个问题：<br />**用户线程是如何停止的？  ** 简单的Stop 就不怕遇被死锁，CPU占用，对象混乱的情况嘛？

- 简单记住两个关键词：安全点   ，  安全区域   只有线程运行在这两个位置时，才能够停止。
- 就像高速上的汽车，你不能随便停车，只能去应急车道，或者去 休息站 去停车。不然会发生意外！

这里又思考一个问题，垃圾回收的时候暂停用户线程，和CPU时间片切换时暂停线程，有什么区别？


<a name="bYpUQ"></a>
#### 安全点/区域
Soft Point 	/   Soft Region<br /> 主要作用就是：让堆对象状态确定一致。<br /> <br />**安全点/区能解决的问题**：（为啥要有安全节点？）<br /> GC，偏向锁撤销，等等，好像后面的可达性分析的时候，也用到了 安全点，安全区域。

**安全点 有哪些：（简单来说就是在 “堆对象状态确定一致” 的时候，下面这些相信我，记不住的）**

   - 循环结束的末尾段
   - 方法调用之后
   - 抛出异常的位置
   - 方法返回之前

**如何进入安全节点？ **<br />主动式中断：就是自己主动在 应急车道点 停车。<br />抢断式中断：就是直接在高速上停车，然后被 拍照 后 在让你停到 应急车道点

安全区域 ：

- 休眠或者阻塞的线程，对堆中的对象没有操作修改，就可以视为在安全区域了。
- 用户线sleep状态或者Blocked状态，无法响应虚拟机中断请求！无法让他挪到安全点了，所以就把他们也称为安全区域了。
> - 说白了，就是为了一些 特殊线程，这些线程已经处于非运行状态，给他们戴了个安全的头衔而已。


![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634219166880-1075c070-cfe4-41e5-b590-d94073f81457.png#clientId=u16b3059d-d69b-4&from=paste&height=228&id=hG6MP&margin=%5Bobject%20Object%5D&name=image.png&originHeight=265&originWidth=1046&originalType=binary&ratio=1&size=310738&status=done&style=none&taskId=u07f15bd4-9cb1-4ccf-a097-0138eee2ef5&width=898.9971313476562)<br />这里还有一个概念叫OOMap，这个图里没有提起，但是也是和安全节点，相关的东西，下去自己查查。

<a name="qyxaK"></a>
#### 可达性分析算法
判断为垃圾对象的一种算法。这也是目前虚拟机中用的最多的算法。

**啥叫可达性分析算法？**

- 来源于离散数学中的原理，从GCRoot节点开始分析，判断哪些对象被引用，就为正常对象，否则为垃圾。

- [x] 哪些可以视为GCRoot节点？

*虚拟机栈中引用的对象（本地变量表）<br />* 本地方法栈中引用的对象<br />* 方法区中静态属性引用的对象<br />* 方法区中常量引用的对象

<a name="h0x1q"></a>
#### 对象引用信息存储：记忆集，卡表，写堆屏障
上面知道可达性分析算法需要快速找出垃圾对象，那么这里就出现几个问题？

- 这么多对象，如何快速查找？
- 老年代对象引用 新生代对象，如何实现跨代引用？
- 判断垃圾对象的时候，老年代对象是不是也得去全部遍历一遍？

下面主要记住几个关键字：<br />**记忆集思想，卡表数组存储，G1记忆集哈希思想，引用状态更改切面实现（**写堆屏障**）。**<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634558197192-d353ed63-412a-46d1-88d3-0da492ec6302.png#clientId=u929b6850-4fa1-4&from=paste&height=357&id=ua7c7ac5e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=573&originWidth=1410&originalType=binary&ratio=1&size=1641955&status=done&style=none&taskId=u2776d129-847b-4000-bbd3-333a16c5f01&width=877.9999694824219)简单来说：<br />新生代通过可达性分析算法判断对象的时候，会把老年代也算上扫描范围，但是这样效率很低，所以为了<br />**解决跨代问题，**在新生代引入了记录集，卡表的概念，去实现快速扫描垃圾对象**。**

**记忆集**<br />**一种抽象数据结构，为了记录 老年代对新生代的引用。解决跨代引用 的一种思想。**<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634559050455-369e19c4-ad6b-4695-a9cd-17468901939c.png#clientId=u929b6850-4fa1-4&from=paste&height=213&id=u9c353e0b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=286&originWidth=986&originalType=binary&ratio=1&size=136316&status=done&style=none&taskId=u4b56fdf0-0e6f-4de7-94f6-2a009dcc591&width=734.6647644042969)<br />卡表<br />记忆集合的一种具体实现，主要用在CMS回收器上面。底层是字节数组（底层数据结构是Bit Map），数组元素记录的是 老年代中的 引用状态，看是否存在跨代引用。<br />如果引用就相当于  arr[9] = 1 .当元素标记为1时，就代表这块区域有对象被跨代引用了。 <br />**主要记录的是：**我引用了谁的对象![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634559090957-43feaa3c-6f74-4b91-9f4e-6fe0e7cffa96.png#clientId=u929b6850-4fa1-4&from=paste&height=182&id=u96979133&margin=%5Bobject%20Object%5D&name=image.png&originHeight=248&originWidth=985&originalType=binary&ratio=1&size=107697&status=done&style=none&taskId=u1ccefe58-d16b-4334-bc06-6a7eacff8fe&width=723.3238525390625)
> - 在进行YoungGC时，我们在进行对一个对象是否被引用的过程，需要扫描整个Old区，所以JVM设计了CardTable，将Old区分为一个一个Card，一个Card有多个对象；如果一个Card中的对象有引用指向Young区，则将其标记为Dirty Card，下次需要进行YoungGC时，只需要去扫描Dirty Card即可。
> - Card Table 在底层数据结构以 Bit Map实现。


![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634561213221-41dd13dc-f55a-429d-9318-2b7177ddd1b3.png#clientId=u7f3ba72e-59ba-4&from=paste&height=549&id=u923f20cb&margin=%5Bobject%20Object%5D&name=image.png&originHeight=755&originWidth=993&originalType=binary&ratio=1&size=347455&status=done&style=none&taskId=u7f0f5ae2-3b06-4467-8bb5-81d457ff5b2&width=721.9942932128906)

**写屏障**<br />上面引用发生改变时，GC回收器中有 写屏障这个功能，相当于OOP切面思想，在对象赋值的时候，完成卡表状态的更新。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634559030178-02318122-eff4-4e57-87ec-79b3f36241d2.png#clientId=u929b6850-4fa1-4&from=paste&height=221&id=u37958ae5&margin=%5Bobject%20Object%5D&name=image.png&originHeight=343&originWidth=981&originalType=binary&ratio=1&size=115342&status=done&style=none&taskId=ue472cc46-51a3-486a-9d5e-a2e6fb0be11&width=631.9886169433594)

**G1的记忆集**<br />上面说到卡表机制主要应用这CMS垃圾回收器中，那么G1垃圾回收器，是怎么玩的呢？<br />因为G1垃圾回收器是基于分区模型的，所以只需要知道每个Region的引用指向了他，并且这些Region是不是回收的一部分。但是卡表这种记录结构只知道有没有引用，不知道具体引用是谁，所以需要另一种结构（RSet）。
> 所以在**G1垃圾回收器的记忆集的实现实际上是基于哈希表的，key代表的是其他region的起始地址，value是一集合，里面存放了对应区域的卡表的索引，因此G1的region能够通过记忆集知道，当前是哪个region有引用指向了它，并且能知道是哪块区域存在指针指向。**

因为**每个region都维护一个记忆集，内存占用量肯定很大，这也是为什么G1垃圾回收器比传统的其他垃圾回收器要有更高的内存占用。据统计G1至少要耗费大约10%-20%的Java堆空间来维护收集器的工作。**<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634559549894-4c1d2158-e9e3-411b-a724-05a548a01452.png#clientId=u929b6850-4fa1-4&from=paste&height=223&id=u1b9bd4b4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=275&originWidth=973&originalType=binary&ratio=1&size=164714&status=done&style=none&taskId=u42af8eca-fded-463f-93e1-d4ada526996&width=787.3323669433594)<br />**RSet**<br />可以理解为是卡表的一种升级，底层是哈希表实现。用来解决G1中跨代引用的问题。<br />**主要记录的是：**谁引用了我的对象
> G1的RSet是在Card Table的基础上实现的：每个Region会记录下别的Region有指向自己的指针，并标记这些指针分别在哪些Card的范围内。这个RSet其实是一个**Hash Table**，Key是别的Region的起始地址，Value是一个集合，里面的元素是Card Table的Index。每个Region中都有一个RSet，记录其他Region到本Region的引用信息；使得垃圾回收器不需要扫描整个堆找到谁引用当前分区中的对象，只需要扫描RSet即可。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634564148959-ee48bd9d-08d6-4811-9b56-4cc84bb149d9.png#clientId=udb394039-4352-4&from=paste&height=443&id=u65f3a9bc&margin=%5Bobject%20Object%5D&name=image.png&originHeight=526&originWidth=995&originalType=binary&ratio=1&size=224196&status=done&style=none&taskId=ue4b30682-b145-4c8f-a12a-01485ddd3a2&width=837.6619262695312)
<a name="oFD3g"></a>
#### 
<a name="ZRqai"></a>
#### 三色指针：（三色标记）：
可以理解为 可达性分析算法 中的一种具体实现细节，用来判断是否为垃圾对象的方法。
> 三色标记法是一种垃圾回收法，它可以让JVM不发生或仅短时间发生STW，从而达到清除JVM内存垃圾的目的。JVM中的**CMS、G1垃圾回收器**所使用垃圾回收算法即为三色标记法。

三色标记法将对象的颜色分为了黑、灰、白，三种颜色。<br />**白色**：该对象没有被标记过。（对象垃圾）<br />**灰色**：该对象已经被标记过了，但该对象下的属性没有全被标记完。（GC需要从此对象中去寻找垃圾）<br />**黑色**：该对象已经被标记过了，且该对象下的属性也全部都被标记过了。（程序所需要的对象）

算法详细过程就不多BB了。回头复习博客吧。

- 参考博客：[https://blog.csdn.net/u013256816/article/details/120695212](https://blog.csdn.net/u013256816/article/details/120695212)

三色标记算法存在的问题：<br />**1.浮动垃圾**<br />已经被标记成黑色或者灰色的对象，突然变成了垃圾，然后无法扫描到。<br />解决方案：只能下次去清理。<br />**2.对象漏标记**<br />并发过程中，导致该对象被程序所需要，却又要被GC回收，被标记成了白色对象。<br />解决方案：<br />CMS对 增加引用环节 的处理。<br />G1则对删除引用环节 做了处理。（SATB的实现）

<a name="BzAFY"></a>
#### 其他概念：
SATB：<br />SATB是维持并发GC的一种手段，G1之所以能实现并发就是基于STAB实现的。可以理解成开始前对对象做一次快照，认为当时是活对象就是活的，形成一个对象图。<br />增量更新：

有色指针：<br />多标-漏标：

新生代GC器解决漏标<br />CMS：<br />写屏障+增量更新<br />G1:<br />SATB + 写屏障<br />ZGC：读屏障<br />ZGC实现：有色指针 + 内存屏障


<a name="Qq2Pt"></a>
### 回收算法
<a name="itJbp"></a>
#### 复制算法
就是把正常对象复制到另一块空区域，然后剩下的垃圾对象清除掉。<br />这里一般是年轻代回收时使用，这就是S1，S2区域倒来倒去的原因。<br />**缺点：**

- 不适用于存活对象多的场景，倒腾开销大。
- 内存利用率低。

**优点**：

- 清理后内存依旧整齐。。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634214650241-f2187318-bb0a-4538-9e71-ae8d1c27d140.png#clientId=u16b3059d-d69b-4&from=paste&height=261&id=u52e10a78&margin=%5Bobject%20Object%5D&name=image.png&originHeight=245&originWidth=447&originalType=binary&ratio=1&size=235828&status=done&style=none&taskId=u35adc893-1b44-402c-83cc-006e3ed3948&width=476.99147033691406)
<a name="UkNjz"></a>
#### 标记-清除算法
说白了就是通过可达性算法判断，标记处垃圾对象，然后清除掉。<br />目前暂时没啥人用，一般被后面的标记整理算法给替代了。用于老年代<br />**缺点：**

- 两个阶段效率不高。停顿时间长
- 内存碎片。

**优点：**

- 简单粗暴

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634214880943-8f3a2095-01ba-4ceb-9133-1b5faff00df1.png#clientId=u16b3059d-d69b-4&from=paste&height=122&id=u1ee37bdd&margin=%5Bobject%20Object%5D&name=image.png&originHeight=121&originWidth=456&originalType=binary&ratio=1&size=106867&status=done&style=none&taskId=uf28b0f72-2ef2-4c37-bfa3-f9db8c31acf&width=459.98863220214844)
<a name="ODg8m"></a>
#### 标记-整理算法
就是上面标记清除算法的一个升级版，最后多一个整理过程。<br />一般在JVM参数里可以设置，当FGC之后几次就整理一次碎片，这个参数是可以调节的，一般生产环境上会设置每次FGC后执行整理。<br />因为当内存碎片过多的时候会暴露出一个问题：当到一定数量时，CMS无法清理这些碎片了，CMS会让Serial Old垃圾处理器来清理这些垃圾碎片，而Serial Old垃圾处理器是单线程操作进行清理垃圾的，效率很低。所以要设置垃圾回收时需要及时整理碎片。

**缺点：**

- 需要移动对象，STW停顿时间长

**有点：**

- 内存完整

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634215050902-fcd7553e-ca9b-4d4f-b11d-d05b1f2ac709.png#clientId=u16b3059d-d69b-4&from=paste&height=167&id=u797ec2d4&margin=%5Bobject%20Object%5D&name=image.png&originHeight=160&originWidth=545&originalType=binary&ratio=1&size=127313&status=done&style=none&taskId=u457d03fa-b528-486c-bb61-47cc4a18df9&width=568.6590881347656)


<a name="RIzsr"></a>
# 其他
<a name="ziJwW"></a>
## 对象逃逸和分配
知道什么是对象逃逸，对象是怎么分配的。

**对象分配流程图**<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634643940145-8154dd8e-e4e2-41f6-a3cf-5329af31693c.png#clientId=u72e36da2-7449-4&from=paste&height=289&id=ued58e87b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=866&originWidth=2130&originalType=binary&ratio=1&size=368792&status=done&style=none&taskId=u6c078c03-8642-44b9-933e-00724a339a5&width=710)
<a name="NjmiE"></a>
#### 什么是对象逃逸
说白了就是对象直接创建在栈帧里面了，没有存储在堆Eden区当中。<br />这样有利于减少堆空间，还有指针引用的过程，已经减少垃圾回收的作用。

<a name="lZmWc"></a>
#### 逃逸分析
就是对象被 其他线程 引用，或者 其他栈帧（方法）使用时，我们称这个对象指针发生了逃逸。<br />jdk6开始有逃逸分析。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634632570138-14797a15-35fe-4733-924b-91e257cf0d20.png#clientId=ufe45dbd7-bd54-4&from=paste&height=104&id=u097e0b16&margin=%5Bobject%20Object%5D&name=image.png&originHeight=82&originWidth=445&originalType=binary&ratio=1&size=44801&status=done&style=none&taskId=u3bcf44eb-962c-412e-8b61-ce56962892b&width=565.3238525390625)
<a name="ncMau"></a>
#### 逃逸的作用域
栈帧逃逸：简单来说，对象被其方法使用了，return返回出去了也算是逃逸。<br />线程逃逸：简单来说，对象被其他线程使用了

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634632409183-add9a0a0-a3e3-46cd-b5dc-264603fef1b1.png#clientId=ufe45dbd7-bd54-4&from=paste&height=289&id=u0abbecd2&margin=%5Bobject%20Object%5D&name=image.png&originHeight=238&originWidth=458&originalType=binary&ratio=1&size=131578&status=done&style=none&taskId=ue873bb99-59b8-49dc-a2fc-cb55a72f17b&width=555.6335144042969)
<a name="GBGEe"></a>
### 对象分配
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634633131017-9caeed09-6377-4fd4-833c-5c0cf721044a.png#clientId=ufe45dbd7-bd54-4&from=paste&height=264&id=u20a2f4fd&margin=%5Bobject%20Object%5D&name=image.png&originHeight=231&originWidth=453&originalType=binary&ratio=1&size=107798&status=done&style=none&taskId=u8f337544-b405-43d4-a246-a3770a3e550&width=516.9857788085938)<br />**栈上分配：**<br />对象一般是放在堆空间的，但是如果 **开启了** **逃逸分析**和 标量替换，也能直接在栈上面分配。<br />**堆上分配：**<br />对象正常在Eden区分配，或者 字符常量池，或者元空间分配。随着垃圾回收等操作，对象会去不同地方。

这里注意一下什么是“**标量替换**”？ 具体是啥玩意，看下面章节。

<a name="cXWDy"></a>
## 标量替换

相当于在栈帧中，利用基本数据类型，和引用类型，把对象里面属性值给替换了，可以在栈上面创建对象。
> 标量可以理解成一种不可分解的变量，如java内部的基本数据类型、引用类型等。 与之对应的聚合量是可以被拆解的，如对象。


**标量替换，对象不逃逸 的好处 就是：**

- 减少堆内存，直接在栈上分配。（栈内存默认2M）
- 运行过程中减少了对象引用的麻烦，速度更快。
- 栈弹出来后，就销毁了，不用GC垃圾回收器介入

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634635686378-190a392a-cbe3-4bab-acac-4ec8f0fb1012.png#clientId=ufe45dbd7-bd54-4&from=paste&height=283&id=ubb7e0151&margin=%5Bobject%20Object%5D&name=image.png&originHeight=242&originWidth=465&originalType=binary&ratio=1&size=124135&status=done&style=none&taskId=ub393697b-8ed6-4651-9e39-acdf56ab0dc&width=544)
<a name="S8QoT"></a>
## 
<a name="s2mOR"></a>
## 同步消除
可以理解为：<br />开启逃逸分析后，如果对象为线程级别的作用域，没有被其他线程使用，那就是线程安全的。那么，如果代码中这个对象被当做为“同步锁对象”，那么在编译时就会取消加锁。比如编译的时候就不会生成monitorenter，monitorexist 这样的关键字，因为本来就线程安全，何必多此一举呢。

> 通过-XX:+EliminateLocks可以开启同步消除,进行测试执行的效率
> 同步消除是java虚拟机提供的一种优化技术。消除的其实是对象的同步锁，堆被线程共享，那么堆上面的对象也会被线程共享，读写会出现同步的并发问题，所以对象也加了synchronized同步锁。但是栈是线程独享的，如果进行栈上分配，那么就根本不需要同步锁，提高了执行效率。这就是同步消除。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634636263387-6845b215-f245-4e81-8647-4257a86dee14.png#clientId=ufe45dbd7-bd54-4&from=paste&height=280&id=u9e7d1f75&margin=%5Bobject%20Object%5D&name=image.png&originHeight=217&originWidth=338&originalType=binary&ratio=1&size=88496&status=done&style=none&taskId=u8fdcc456-c19a-4d34-be36-aa9ea78b308&width=435.647705078125)
<a name="zGGxK"></a>
## 指针压缩
简单来说对JVM中所有的引用指针（类型指正，堆引用指针，栈帧中的变量指针 以及 锁记录指针等等），都做了压缩，在JDK1.7后默认开启。目的是为了在64位的虚拟机中提升内存使用率。

物极必反，月圆必缺！ 也**会出现一个问题：指针压缩失效**<br />指针被压缩到32bit之后，最大寻址为32GB，当如果你堆内存超过32G时，出现了OOM，内存扩大到48G时一样也可能出现OOM，因为内存超过32GB是，32Bit的指正无法寻址，会失效。<br />**关键字：**<br />**堆内存超过32G，无法寻址，指针膨胀， 指针失效，出现OOM。**
<a name="fiWOF"></a>
## ![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634636857212-d735642d-b183-401d-be91-639dc9ffee89.png#clientId=ufe45dbd7-bd54-4&from=paste&height=298&id=u4f43e1bb&margin=%5Bobject%20Object%5D&name=image.png&originHeight=239&originWidth=746&originalType=binary&ratio=1&size=216550&status=done&style=none&taskId=u8b1bfee0-e76a-450e-b2a6-3d94d70c14f&width=929.9971313476562)
<a name="Jb0B7"></a>
## synchronized锁的膨胀
<a name="u6OA3"></a>
#### 锁升级的流程图
底层原理就不多BB了，这里简单介绍一下锁的升级过程。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634645096228-eeaaee8b-56f2-40e1-bed0-68df9bbefaf3.png#clientId=u72e36da2-7449-4&from=paste&id=bhRE0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1807&originWidth=3867&originalType=binary&ratio=1&size=2041587&status=done&style=none&taskId=uc9ffe8f6-ad48-4897-8136-0078f98a146)<br />JDK1.6以前synchronized 关键字只表示重量级锁（依赖于操作系统的Mutex线程挂起，所以效率低）<br />JDK1.6之后区分为偏向锁、轻量级锁、重量级锁。
<a name="FgF8G"></a>
#### 关键字：
底层采用大量的 CAS操作。<br />偏向锁：MWd区域中的 轻量级锁ID<br />轻量级锁：MarkWord区域的 轻量级锁指针。以及MW中拷贝到栈帧中的“锁记录”。<br />重量级锁：操作系统中的Mutex互斥原则去挂起线程。

博客参考：[https://blog.csdn.net/hanghangaidoudou/article/details/113561482](https://blog.csdn.net/hanghangaidoudou/article/details/113561482)
> 锁的升级、降级，就是 JVM优化 synchronized 运行的机制，当 JVM 检测到不同的竞争状况时，会自动切换到适合的锁实现：
> - 当没有竞争出现时，默认会使用偏向锁。JVM 会利用 **CAS 操作**（compare and swap），在对象头上的 **Mark Word 部分设置线程 ID**，以表示这个对象偏向于当前线程，所以并不涉及真正的互斥锁。这样做的假设是基于在很多应用场景中，大部分对象生命周期中最多会被一个线程锁定，使用偏向锁可以降低无竞争开销。
> - 如果有另外的线程试图锁定某个已经被偏向过的对象，JVM 就需要撤销（revoke）偏向锁，并切换到轻量级锁实现。**轻量级锁** 依赖 CAS 操作 Mark Word 来试图获取锁，如果重试成功，就使用轻量级锁；否则，进一步升级为重量级锁

<a name="Mb2MA"></a>
#### 上锁的过程
无锁 ->  偏向锁  -> 轻量锁  -> 重量级锁

简单来说就是**：**<br />**（同步消除 阶段）**

- 编译前会有“**同步消除**”的判断，判断锁对象的使用域是否在一个线程内。安全就不需要加锁。

（偏向锁 阶段）

- 锁对象首先是处于**“偏向锁”**，这种是靠把 对象头MarkWord内存 中的  **偏向锁线程ID** 修改为自己的。
   - 修改的 **偏向锁线程ID** 过程是用 CAS 乐观锁机制。

（轻量级锁 阶段）

- 当有其他线程想要获取锁时，会升级为**“轻量级锁”，**此时会把MarkWord的中的锁标记 拷贝 到自己的栈帧中的“锁记录”（lock record）且让对象头MarkWord中的 “**轻量级锁指针”** 指向你的栈帧中。
   - 复制过程，指针指向过程都是采用CAS机制。

（重量级锁 阶段）

- 当上面出现把“轻量级锁指针” 指向栈时，如果失败，就不断自旋操作。		（重量级锁的出现过程？）
   - 默认会自选10次（可调节），超过次数就晋级为 “重量级锁”
   - 或 竞争锁线程 超过CPU核数的一半时，会升级成 “重量级锁”。
   - 或者 直接调用 wait() 方法也是直接变成 重量级锁。
   - JDK1.6之前默认都是重量级锁

**重量级锁实现原理的几个关键字？**

- 对象头中MarkWord区域内有指向 **重量级锁指针**（指向监视器对象Monitor的指针），靠这玩意去实现锁。
- 底层采用操作系统 Mutex (互斥) 原则挂起当前线程。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1634644785195-3eb14b64-2319-4f27-a402-6e7084b5db69.png#clientId=u72e36da2-7449-4&from=paste&height=555&id=u4a004243&margin=%5Bobject%20Object%5D&name=image.png&originHeight=814&originWidth=1547&originalType=binary&ratio=1&size=995065&status=done&style=none&taskId=ue254c7d0-6088-4ac7-ada9-95fe4936ee6&width=1054.9971313476562)


<a name="QuLcI"></a>
#### 
