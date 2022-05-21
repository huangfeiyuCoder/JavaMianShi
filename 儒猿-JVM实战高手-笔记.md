<a name="vRhye"></a>
## 前言
<a name="jSoQ0"></a>
### 1-复习总结
<a name="JhZoQ"></a>
#### 本次章节主要带来三个部分：
第一个部分<br />JVM运行所写系统的工作原理，包括：

   - 内存各个部分的划分
   - 代码在执行的过程中，各个内存区域是如何配合协调工作的
   - 对象是如何分配的
   - GC如何触发
   - GC执行的原理是什么
   - 常见的用于控制JVM工作行为的一些核心参数都有哪些？

第二部分ww

   - JVM如何设置合理参数？
   - 如何对JVM进行监控？
   - 如何对有GC性能问题的JVM，作出合理优化？
   - 带来大量实战案例，自己去总结排查思路。

第三部分<br />对OOM溢出等问题，如何进行分析，定位，解决。<br />提供一些OOM案例，做排查和演示，帮助你总结问题，分析问题。
<a name="uF3db"></a>
#### 总结
希望能把上面的排查案例总结出来，运用到自己的系统里面，最好总结出一套排查方案出来。
<a name="med2T"></a>
## JVM运行基础
<a name="Z593V"></a>
### 2-JAVA代码到底是如何运行起来的？
<a name="bBLuB"></a>
#### 笔记

- 由.java文件编译成.class文件，然后 java -jar XX.jar 包启动，此时JVM会开启一个Java进程，负责运行这些class文件。
- 然后由“类加载器” 把那些class 文件夹在加载到JVM虚拟机中，以 类 的形式出现。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1628500738125-a974cd94-d9d9-4c9f-b5a7-ff006d3a46c0.png#crop=0&crop=0&crop=1&crop=1&height=390&id=u4b458bec&margin=%5Bobject%20Object%5D&name=image.png&originHeight=382&originWidth=520&originalType=binary&ratio=1&rotation=0&showTitle=false&size=94883&status=done&style=none&title=&width=531)
<a name="qrZL6"></a>
#### 问题与总结

- [x] 怎么把class文件翻译成Java文件？

答： Javap 反编译

- [x] 加载java文件经历了哪些过程？

答：加载，验证，准备，解析，初始化，使用，卸载

<a name="blp5E"></a>
### 3-面试官对于 JVM 类加载机制的猛烈炮火，你能顶住吗？
<a name="d8AoF"></a>
#### 笔记
一个类加载到使用，经历了什么？<br />加载-验证-准备-解析-初始化-使用-卸载

1JVM在什么情况下会加载一个类？<br />**啥时候把class 字节码加载到JVM内存里来？**

- 在你用到这个类的时候，或者有相关的引用时候，就会加载到内存中。
- 也又可能是JIT的加载刺探，提前把一些常用的代码编译加载到CodeCache来。

- 什么是验证阶段？

 JVM校验class 指令语法，内容是否规范等，判断是否class文件发生篡改等。

- 什么是准备阶段？

给代码中的类变量（static修饰的）变量或引用分配内存空间，给默认值，比如0。

- 什么是解析阶段？

简单来说就是 符号引用替换为直接引用，后面在细说。

- 什么是初始化阶段？

就是给类变量赋 真实值，此时static代码块逻辑也会执行。
<a name="nSWNg"></a>
#### 问题与总结

- [x] 什么时候会初始化一个类？
   - 包含 main() 方法的，会立马初始化的。
   - 运行中有 new Object() 指令的，会触发类 加载 到 初始化 的全部过程，然后在实例化一个对象出来。同时会先把父类也初始化进来。
   - <br />
- [x] 了解类的懒加载机制，为什么要提前加载？

答：JIT里面有热点代码刺探，提前把一些常用的代码加载到CodeCache中，方便在运行时，快速拿到class对象信息，是空间换时间的一个操作。也是属于堆外内存。

- [x] 父类子类加载顺序是如何的？

答：先加载父类，以及在子类。

<a name="lkh9N"></a>
## JVM内存结构
<a name="j1vFS"></a>
### 4-JVM有哪些内存区域，作用是啥？
<a name="gfCtk"></a>
#### 笔记
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1628648361435-16611ea8-a5d8-4511-992a-4b5bd367f3e4.png#crop=0&crop=0&crop=1&crop=1&height=363&id=u10adf2b7&margin=%5Bobject%20Object%5D&name=image.png&originHeight=417&originWidth=577&originalType=binary&ratio=1&rotation=0&showTitle=false&size=133998&status=done&style=none&title=&width=502.4886169433594)<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1628652256660-08f3f9dd-396d-4c73-98a1-dc1e5a8e81c0.png#crop=0&crop=0&crop=1&crop=1&height=292&id=u21699595&margin=%5Bobject%20Object%5D&name=image.png&originHeight=338&originWidth=563&originalType=binary&ratio=1&rotation=0&showTitle=false&size=95719&status=done&style=none&title=&width=486.491455078125)<br />**方法区（Metaspace||元空间）**：存放.class文件加载后的类，（理解为创建对象的类模具），常量等。<br />**常量池**：存放.class文件加载后的一些常量，也属于 元空间。<br />**字节码执行引擎**：执行代码编译出来的代码指令。<br />**程序计数器**：记录当前线程执行的字节码指令位置，唯一不会溢出的区域，（线程私有）。<br />**栈：**先进后出，线程每执行一个方法，就在对应的栈里创建一个栈帧，栈帧里存放局部变量表，和引用指针（线程私有）。<br />**堆：**先进先出，存放new 出来的对象，栈中的变量指向的就是该堆中的地址资源（线程共享）。<br />**堆外空间（其他）**：源码中用native声明的方法，主要是运行C类库的函数，会有线程对于的本地方法栈，这不属于JVM中的内存。以及NIO通过allocateDirect这种操作，可以在堆外分配内存空间，虚拟机通过DirectByteBuffer来引用和操作对外内存空间。（NIO，Socket网络相关，可以提高性能）
<a name="WjfKm"></a>
#### 问题与总结

- [x] Tomcat容器中的类加载器是如何实现设计的？

Tomcat自定义Common，Catalina等类加载器，用来加载自有的核心基础类库。<br />Tomcat为每个Web应用都分配一个WebApp类加载器，负责部署。<br />Tomcat是**打破**了**双亲委派机制**的。

- [x] Tomcat为什么要打破双亲委派机制？

因为要在一个Tomcat容器里面部署多个应用，多个应用之间会有同名的类名，避免冲突就需要不能共用一个类加载器，不能向上委托，避免共用一个类加载器，所以不能按照双亲委派的做法来实现。

- [x] Tomcat中怎么实现类加载的？

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1632469132135-a02061c5-b03c-426c-a9f8-cf8b26dacf11.png#clientId=uec6659b6-77df-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=971&id=u30a7fc83&margin=%5Bobject%20Object%5D&name=image.png&originHeight=872&originWidth=583&originalType=binary&ratio=1&rotation=0&showTitle=false&size=329457&status=done&style=none&taskId=u36ac63ea-ef05-40ec-9808-af38f413f3e&title=&width=649.491455078125)

- **commonClassLoader**: tomcat最基本的类加载器, 加载路径中的class可以被tomcat容器本身和各个webapp访问;（双亲委派机制）
- catalinaClassLoader: tomcat容器中私有的类加载器, 加载路径中的class对于webapp不可见（符合双亲委派机制）
- sharedClassLoader: 各个webapps共享的类加载器, 加载路径中的class对于所有的webapp都可见, 但是对于tomcat容器不可见.   （违背双亲委派机制）
| 加载器/特征 | 用途 | 是否所有<br />WebApp可访问 | 是否遵循<br />双亲委派机制 |  |
| --- | --- | --- | --- | --- |
| **common** | Tomcat最基本的加载器，主要是加载自己的一些类 | 可 | 遵循 |  |
| catalina | 私有的类加载器，主要加载咱们写的Java应用 |      不可 | 尊循 |  |
| shared | 共享的类加载器，主要加载对于所有APP可见的程序 |       可 | 不遵循 |  |
| JSP加载器 | 为每一个JSP文件都生成一个独有的类加载器，并且如果JSP有修改，则会监听到变化，重新加载class信息。 |      可 |   不遵循 |  |


- [x] 延伸：tomcat自定义的类加载器中, 还有一个jsp类加载器. jsp是可以实现热部署的, 那么他是如何实现的呢?

jsp其实是一个servlet容器, 有tomcat加载. tomcat会为每一个jsp生成一个类加载器. 每个类加载器都加载自己的jsp, 不会加载别人的. 当jsp文件内容修改时, tomcat会有一个监听程序来监听jsp的改动，比如文件夹的修改时间, 一旦时间变了, 就重新加载文件夹中的内容. <br />具体tomcat是怎么实现的呢? tomcat自定义了一个thread, 用来监听不同文件夹中文件的内容是否修改, 如何监听呢? 就看文件夹的update time有没有变化, 如果有变化了, 那么就会重新加载.

- [x] Java堆中的对象，会占用多少内存？（这个在做GC分析的时候会用到）

分两部，第一部分：对象内部包含了又对象头，哈希码，锁，填充物。第二部分：看对象你们包含了哪些对象属性，比如long=8字节，int=4字节，char=一个字节，double = 8字节等等，按照对象内容去估算。

- [x] 怎么估算，计算系统创建的对象内存占用的压力呢？

看你JVM里面的堆内存，新老年代的分布情况，Eden，S能最多容纳多少，会隔多长时间触发一次YGC,一次FGC，会造成多久的STW停顿。<br />查看GC日志，观察新生代多久一次，每次回收的量是多少。

- [x] 一个线程大概占用多少内存？
   - 内存分两块：1.对象本身的一些信息，对象实例变量作为数据占用的空间。
   - 比如对象头在64位操作系统上，会占用16个字节，如果里面有一个int 类型变量，会多占用4个字节，如果是long 就是8个字节。另外JVM有很多优化的地方，比如 补全机制，指针压缩机制 ，这些后面讲解？
   - 线程分配的栈空间，一般会主动设置200M，因为默认分配的空间很小。

- [x] 栈里头的指针是怎么指向对象的？

答：栈帧里面有一个局部变量表，变量表里面有块 reference 区域，里面存的就是对象在堆中的引用指针。<br />有可能是直接的引用指针，也有可能是堆中的句柄池的对象句柄。

- [x] 递归，在栈里是如何体现的？

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1635305760525-3e54b233-37d5-4a41-b826-3f6400813a47.png#clientId=u8997ebce-e422-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=631&id=u6d28d38d&margin=%5Bobject%20Object%5D&name=image.png&originHeight=778&originWidth=927&originalType=binary&ratio=1&rotation=0&showTitle=false&size=316934&status=done&style=none&taskId=ueebafa8e-e693-4644-a692-9e00e9d2243&title=&width=751.4942932128906)

- [ ] 如何保证不会栈击穿？<br />1，比较通用的方法就是通过循环和迭代的手法来化解递归。一般能用递归实现的需求，都能<br />转化为相应的循环迭代模型。<br />2，第二种处理方式，就是基于Lambda表达式来构造尾递归模型。尾递归通过记录上次递归的<br />结果避免了堆栈溢出。

- [x] 递归解释

就是在运行的过程中调用自己。<br />构成递归需具备的条件：<br />1. 子问题须与原始问题为同样的事，且更为简单；<br />2. 不能无限制地调用本身，须有个出口，化简为非递归状况处理。

- [ ] **什么是单向（普通）递归，什么是尾递归？**

单向递归：就是普通的递归，在函数里面调用自己。<br />尾递归：意思是在函数代码里的最后一行代码，才调用自己，且直接返回返回值，返回值不需要经过其他操作。

- [x] **尾递归的解释，原理和作用？**

**尾递归的解释**（百科来的，供理解）：<br />     如果一个函数中所有递归形式的调用都出现在函数的末尾，我们称这个递归函数是尾递归的。当递归调用是整个函数体中最后执行的语句且它的返回值不属于表达式的一部分时，这个递归调用就是尾递归。尾递归函数的特点是在回归过程中不用做任何操作，这个特性很重要，因为大多数现代的编译器会利用这种特点自动生成优化的代码。

**尾递归的原理：**<br />     当编译器检测到一个函数调用是尾递归的时候，它就覆盖当前的活动记录而不是在栈中去创建一个新的。编译器可以做到这点，因为递归调用是当前活跃期内最后一条待执行的语句，于是当这个调用返回时栈帧中并没有其他事情可做，因此也就没有保存栈帧的必要了。通过覆盖当前的栈帧而不是在其之上重新添加一个，这样所使用的栈空间就大大缩减了，这使得实际的运行效率会变得更高。

**尾递归的作用：**<br />就是防止栈溢出，浪费空间。如果是尾递归的话，编译器就会优化指令，让你的递归调用复用同一个栈空间，减少空间的浪费和OOM风险。但是前提是得符合尾递归的原则，直接返回返回值。

- [x] JVM中什么是内存补全机制？

虚拟机的内存管理要求**对象的内存大小必须是8字节的整数倍，**用来充数的区域。

- [x] JVM中什么是指针压缩机制？

大概意思就是在JDK7之后，HotSpot虚拟机默认开启指针压缩，本来一个指针要占用64bit的压缩成了32位Bit，可以减少JVM中内存占用率，但是也会带来一个指针失效问题，64位机器上的如果堆内存超过了32G，会导致寻址失效，即使扩大内存到48G，也有可能会出现OOM。<br />具体详细的解释，在另一篇“JVM图解白话笔记”中有详细的解释。这里简单介绍一下。

<a name="jqS5E"></a>
### 5-JVM垃圾回收机制干嘛用？为啥要回收？
对象的分配与引用<br />一个方法执行完毕后会怎样？<br />这个方法对应的栈帧移出去，后面栈中的资源被回收。<br />我们创建的Java对象其实都是占用内存资源的，不再需要的那些对象应该怎么处理？<br />垃圾回收守护线程，会检查对象引用情况，判断是否被回收？
<a name="w8TYc"></a>
#### 问题与总结

- [x] GC是为什么会突然知道要回收收垃圾了的？

守护线程，在不断循环的执行扫描任务，判断是否要回收垃圾

- [x] 检查对象是否引用，是否为垃圾的算法是什么？

引用计数算法：就是看是否引用，然后判断为垃圾，有循环引用的漏洞，没人使用。<br />可达性分析算法：从 GCRoot节点开始去分析引用，看是否为垃圾对象。 <br />注意几个关键点：“三色指针”，GCROOT节点有哪些？ 老年代中跨代引用是怎么解决的？ G1回收器中的卡表，RSet是用来干什么的？

- [x] 栈帧弹出去后是怎么回收的？

当这个方法返回之后，就弹出栈帧，自动销毁了。

<a name="CuW3l"></a>
### 7-第一周问题答疑

1. 不要在for循环里创建对象的操作，可以在循环外创建一个对象，在里面修改，这样减少对象头内存的占用。
1. Java虚拟机栈和本地方法栈，都是线程私有。
1. 新建的实例堆内存，实例变量也是在堆里。
1. 父类子类是如何加载的？加载父类就是父类，除非用到子类才会加载子类；但是加载子类要初始化之前，必须先加载父类，初始化父类。
1. 对象内存结构：Object Header（4字节） + Class Pointer（4字节）+ Fields（看存放类型），但是jvm内存占用是8的倍数，所以结果要向上取整到8的倍数。
1. 类是按需加载，如果要实现一次性全部加载，需要自己实现类加载器。
1. 类加载要一层一层往上找，按照 双亲委派机制的规则实现的好处是，避免重复记载某个类。类加载器本身就是父子关系模型实现的，所以不会从下往上找，这样设计保证程序的可拓展性。
1. 允许代码new replicamanager()的时候才会加载replicamanger类，而不是加载kafka的时候会同时加载。（有问题啊，不是会懒加载一些类的嘛？后面验证一下？？）
1. class文件分配内存是在准备阶段，那类的class对象是在准备阶段创建的吗？ 
   1. 类是在准备阶段分配内存空间的
10. 如果实例变量有初始值，那实例变量是和类变量一同在初始化阶段赋值的吗？
   1. 实例变量得在你创建类的实例对象时才会初始化
11. 初始化之后是不是就有实例了
   1. 类的初始化阶段，仅仅是初始化类而已，跟对象无关，用new关键字才会构造一个对象出来。
12. class文件可以反编译成Java文件，可能会有代码安全问题，有商用的代码可以考虑特殊混淆处理。
12. 一个包里面有多个main方法时，启动jar包时，会让你指定那个main方法，会优先加载它。
12. 自定义的类加载器如何实现？ 自己写一个类，继承ClassLoader类，重写类加载的方法，在里面指定用自己的类加载器去加载某个路径下的类，加载到内存里来。
12. -XX:+TraceClassLoading 可以看加载了哪些类到内存中。rt.jar包下的类运行时就会加载，自己写的类一般是用到了才会加载。
12. 比如jvmti小工具就可以实现class文件的加密，解密的话可以基于自定义的类加载器来实现。

<a name="cjBpA"></a>
### 8-JVM中的分代模型：年轻代，老年代，永久代
<a name="fxUAP"></a>
#### 笔记

- 长期存活的对象存在于老年代。
- 短期存活的对象存在于新生代。
- JVM中的方法区，就是永久代。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1628756904762-dac35390-0467-4569-b635-71c3711f3654.png#crop=0&crop=0&crop=1&crop=1&height=658&id=uac924a23&margin=%5Bobject%20Object%5D&name=image.png&originHeight=658&originWidth=631&originalType=binary&ratio=1&rotation=0&showTitle=false&size=205397&status=done&style=none&title=&width=631)<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1628756897597-b6d70289-21ca-41aa-8ded-25f4c6692aac.png#crop=0&crop=0&crop=1&crop=1&height=920&id=u1769b253&margin=%5Bobject%20Object%5D&name=image.png&originHeight=920&originWidth=837&originalType=binary&ratio=1&rotation=0&showTitle=false&size=234573&status=done&style=none&title=&width=837)

- [x] 方法区里的类会不会进行垃圾回收？要满足以下条件才能被回收
- 首先该类的所有实例对象都已经从Java堆内存里被回收
- 其次加载这个类的ClassLoader已经被回收
- 最后，对该类的Class对象没有任何引用

<a name="zf1dB"></a>
#### 问题与总结

- [x] 每个线程都有Java虚拟机栈，里面也有方法的局部变量等数据，这个Java虚拟机栈需要进行垃圾回收吗？

栈帧出栈后，里面的局部变量等信息，会直接清理掉。

<a name="XIg8t"></a>
### 9-对象在JVM内存中如何分配？如何流转？
<a name="leb08"></a>
#### 笔记

- 大部分是先在新生代对象里，只有新生代内存不足的时候，或者超大对象才会直接丢老年代。
- 哪怕是静态变量引用的对象也是先进入新生代，后面才进入老年代中。
- 新生代触发垃圾回收叫 YoungGC, 或者叫 MinorGC 。主要回收生命周期短的对象。
- 对象在新生代里经历了15次（默认，可调）垃圾回收之后，还存在，那么就会被放到老年代，长期存在。
- 注意区分永久代 和 老年代，老年代和新生代是在堆里，永久带是在方法区当中，也叫元空间，存放static变量和一些class对象信息，长期存在。
<a name="Jk29m"></a>
#### 问题与总结

- [x] 多大内存的对象才会直接进入老年代？

看你的JVM参数中是如何设置的，一般默认为0，生产中视情况设置为2-3M。

- [x] 新生代内存区域的切换？

先在Eden区，当满了之后，进行YGC，会进入S1区，第二次YGC后可能会进入S2区，反复几次，如果还存活，就会进入到老年代当中。

- [ ] 什么是动态对象年龄判断机制？

- [ ] 什么是空间担保机制？

- [ ] 怎么查看堆中的对象大小？

<a name="pKDu6"></a>
### 10-线上系统部署时如何设置JVM内存大小？
<a name="hD3P1"></a>
#### 笔记
新上线的系统，应该怎么去根据系统线上的负载情况，预估内存使用的压力，然后设置JVM大小。<br />可以根据可能发生的OOM的场景，做压测，比如并发请求大时做压测，大数据导出的时候，可以在本地查看内存运行情况大致给一个估算值。
<a name="yr9ig"></a>
#### 问题与总结

- [x] 学会评估一个系统需要多大的JVM内存？

找到系统核心业务，最耗内存的业务，估算一下对象熟悉，和数据量大小，根据所占字节去估算一下，最直观的方式就是在本地打开JVM的内存Debug，模拟一下看看堆飙高到什么位置，什么时候触发GC。

- [ ] 学会怎么设置？记住常用的几个关键字。

Xms  Xmx  userG1....等

<a name="wMXwP"></a>
## JVM垃圾回收器
<a name="w4yDs"></a>
### 11-什么情况下会对JVM中的对象做回收
<a name="VL1gL"></a>
#### 笔记

- 什么时候会触发垃圾回收？

没有变量引用的时候就会被GC判断为垃圾。

- 被哪些变量引用的对象是不能回收的？

static 静态变量引用了就无法回首，或者说是被方法区（永久代）中的变量引用就无法被回收。

- 常见的引用类型？
   - 强引用 （永远不会被回收）
   - 软引用（可能不会回收，只用在堆內存不够用的时候才会被回收，常用于缓存场景当中）
   - 弱引用（相当于没有引用，直接回收）
- Object类中finalize() 方法的作用？
   - 对象将要被回收时，会执行这个方法，看是否把自己这个实例对象给了某个GCRoot变量，如果引用了自己就不会被回收掉。
<a name="TgDNB"></a>
#### 问题与总结
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1628850003784-f9ca4154-c260-4db8-9205-f7799a1f8fa3.png#crop=0&crop=0&crop=1&crop=1&height=205&id=ue7f100fd&margin=%5Bobject%20Object%5D&name=image.png&originHeight=205&originWidth=532&originalType=binary&ratio=1&rotation=0&showTitle=false&size=63103&status=done&style=none&title=&width=532)

- [x]  回答上述题目：不会被清除，因为有最外层有静态变量引用，不会视为垃圾。

- [x] 判断是垃圾的两种算法，利弊区别是什么吗？ 
- [ ] 什么是引用计数判断？

- [ ] 什么是可达性分析判断？

来源于离散数学中的原理，从GCRoot节点开始分析，判断哪些对象被引用，就视为正常对象，否则视为垃圾。

- [x] 哪些可以视为GCRoot节点？

*虚拟机栈中引用的对象（本地变量表）<br />* 本地方法栈中引用的对象<br />* 方法区中静态属性引用的对象<br />* 方法区中常量引用的对象

- [x] HotSpot虚拟机是怎么实现可达性分析算法的？

简单来说，如果要枚举方法区中所有的GCRoot节点，遍历一次会非常慢，所以不能这么玩。<br />于是虚拟机用了一种 OoMap的数据结构，里面存储对象偏移量，只需要遍历根节点时访问所有的OoMap即可。<br />如果把所有符合GCRoot节点的对象都存进去，也会非常大，难以维护变化，所以并不是在每个对象都创基了OoMap结构，而是在特定位置上创建了 安全节点(SafePoint)，保证虚拟机的安全点个数不算太多，也不算少。在上面还增加了**安全区域**，为了实现安全节点达不到的效果？

- 使用OoMap记录并枚举根节点
- 使用安全节点SafePoint约束根节点

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1632474851682-1c813540-db6f-4d22-8126-1b51a171d939.png#clientId=uec6659b6-77df-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=277&id=u3cbd5d42&margin=%5Bobject%20Object%5D&name=image.png&originHeight=553&originWidth=990&originalType=binary&ratio=1&rotation=0&showTitle=false&size=271720&status=done&style=none&taskId=uf4e61942-28aa-41c8-a3a8-4b7f3a173b5&title=&width=495)

- [ ] 什么是复制算法？

- [ ] 什么是标记清除算法？

- [ ] 什么是标记整理算法？


- [x] 拓展：列举Object类中有哪些常见的方法？
- **clone(): **实现对象的浅复制，只有实现Cloneable接口才可以调用该方法,否则异常。主要是除了8种基本类型传参数是值传递，其他的类对象传参数都是引用传递，我们有时候不希望在方法里讲参数改变，这是就需要在类中复写clone方法。
- **eqlase() ：**实现对象的比较
- **hashCode(): 获取对象的哈希码**
- **wait()：**使当前线程等待该对象的锁
- notify(): 唤醒线程锁
- **finalize()：**GC回收前必须调用一次，但是不会立马回收内存，会在下一次GC时做回收，一般在JVM中的“本地方法”中使用到
<a name="bOIU6"></a>
### 
<a name="dJcH6"></a>
### 12-JVM中有哪些垃圾回收算法？各自优缺点是什么？
<a name="m5x04"></a>
#### 笔记
1、前文回顾<br />2、复制算法的背景引入<br />3、一种不太好的垃圾回收思路<br />4、一个合理的垃圾回收思路<br />5、复制算法有什么缺点？<br />6、复制算法的优化：Eden区和Survivor区<br />7、新生代垃圾回收的各种“万一”怎么处理？

- 堆的新生代中采用复制算法 回收垃圾；
- 复制算法：里面会有两块内存 S1和S2，每次只用其中1块，清除时，把内存中存活的对象摞到另有块，然后清除掉刚开始占用的块内存。好处是避免了内存碎片，坏处是内存占用效率低。
- 新生代中的复制算法：1个Eden区，2个Survivor区，按照：8:1:1的默认比例分配。
- 具体回收步骤是
   - 对象先进入Eden区
   - 触发垃圾回收，把存活的对象放到S1区域
   - 清除Eden区中的所有对象
   - 后面的对象继续进入Eden区，然后一阵子后触发第二次垃圾回收。
   - 检查S1中还存活的对象。
   - 然后把Enden区的存活对象 和 上次S1中存活的对象放到S2区域。
   - 然后Enden继续接客。S1空出来继续接客。
- 这样做的好处是内存使用效率高，碎片控制的好。

<a name="H0pPg"></a>
#### 问题与总结

- [ ] 新生代中的对象要来回多少次才能被转移到老年代里去？
- [ ] 如果S区域内存放不下大对象，会丢到老年代里去？


<a name="bHo5r"></a>
### 13-年轻代和老年代分别适合什么样垃圾回收算法？
<a name="E69tS"></a>
#### 笔记

- 目录：
   - 躲过15次GC之后进入老年代
   - 动态对象年龄判断
      - 15次
   - 大对象直接进入老年代
      - 避免S区的白白占有
   - Minor GC后的对象太多，无法放入Survivor区怎么办？
      - 进入老年代
   - 老年代空间分配担保规则
   - 老年代垃圾回收算法

**进入老年代的方式：**

- 对象每次在新生代里躲过一次GC被转移到一块Survivor区域中，此时他的年龄就会增长一岁。
   - 默认情况下，达到15岁（次）的时候，就会转移到老年代中。
   - 通过JVM参数“-XX:MaxTenuringThreshold”（最大任期阀值）来设置，默认是15次进入老年代。
- “年龄1+年龄2+年龄n 的多个年龄对象总和 超过 了Survivor区域的50%”，此时就会把年龄n以上的对象都转移到老年代。这就是动态年龄判断 规则。
- 超大对象，直接进入老年代，有个参数“-XX:PretenureSizeThreshold” 就是设置直接进入老年代的 大对象阀值，超过这个字节内存数，就会直接进入了老年代。
- 如果Eden区中存活的对象总数大小，超过S区的对象，也会直接进入老年代。

老年代空间分配担保规则

- 如果新生代有大量对象存活，Survivor区放不下，必须转移到老年代去，但是老年代也放不下，咋整？
- 执行任何一次Minor GC，（YoungGC）之前，JVM会先检查老年代可用的内存空间，是否大于新生代所有对象的总大小（避免极端情况，新生代全部存活，全进入老年代）
- 如果大于，就直接转移到老年代。
- 如果小于，那就会看“-XX:-HandlePromotionFailure”的参数是否设置了，
   - 如果没有设置，会触发Full GC ,对老年代做回收，腾出点空间，然后MinorGc。
      - 如果MGc之后，S区能存下，万事大吉。
      - 如果MGc之后，S区不能存下，就放老年代空间中。
      - 如果MGc之后，S区和老年代都存不下，那么会发生“Handle Promotion Failure”（晋级失败），会再次触发 Full GC 对老年代回收, 如果还存不下那就OOM了。
   - 如果有设置该参数，下一步，看老年代的内存大小，是否大于之**前每一次**Minor GC后**进入**老年代的**对象的平均大小**。如果大于则会直接进入老年代，小于就会触发FullGC
- [ ] 后面补充GC时序图

老年代垃圾回收算法 = 标记整理算法

   - 把存活对象移动到一起，然后进行回收，避免内存碎片。
   - 这种回收算法比 新生代中FullGc的算法慢10倍，会影响性能。
<a name="cM63q"></a>
#### 问题与总结

- [ ] 新生代和老年的的内存占比是多少？
<a name="mkzE2"></a>
### 14-JVM中有哪些常见的垃圾回收器，各自有啥特点？
<a name="HMgX9"></a>
#### 笔记

- Serial和Serial Old垃圾回收器：
   - 分别用来回收新生代和老年代的垃圾对象
   - 单线程回收，会暂停用户线程，几乎不用。
- ParNew和CMS垃圾回收器：
   - ParNew都是用在新生代的垃圾回收器，针对服务器都是多核CPU做了优化。
   - CMS是用在老年代的垃圾回收器。
   - 多线程并发的机制，标配组合
- G1垃圾回收器：
   - 作用于 新生代 和老年代
   - 并发，且和用户线程并行。
<a name="EzpvO"></a>
#### 问题与总结

- [ ] 学会自己根据现有财务确收系统，分析一篇确收是GC分析报告。

<a name="YH1ut"></a>
### 15-JVM的痛点：Stop the World
<a name="DeRwV"></a>
#### 笔记
在垃圾回收时，JVM会在后台直接进入“Stop the World”状态，停止我们写所有工作线程，不在运行。<br />所以优化的目的就是要，避免GC频率不要太高，且时间不要过长。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1629270492669-492e9d44-08b6-455a-b9ec-095c40671764.png#crop=0&crop=0&crop=1&crop=1&height=330&id=u59b3a99a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=330&originWidth=545&originalType=binary&ratio=1&rotation=0&showTitle=false&size=76129&status=done&style=none&title=&width=545)
<a name="uTeek"></a>
#### 问题与总结

- [ ] 到底是单线程进行垃圾回收好呢？还是多线程进行垃圾回收好呢？在不同的场景下有各自的优缺点吗？

<a name="C7Hsn"></a>
### 16-第二周问题答疑
<a name="P2OAi"></a>
#### 笔记

1. 一个面试题：**parnew+cms的gc，如何保证只做ygc，jvm参数如何配置？**（消灭FullGC,只做MinorGC）
   - 同学回答：
      - 1)加大分代年龄，比如默认15加到30;
      - 2)修改新生代老年代比例，比如新生代老年代比例改成2:1 
      - 3)修改e区和s区比例，比如改成6:2:2 面试官说他们内部服务已经做到fullgc次数为0，只做ygc
   - 老师回答：
      - 先借住工具观察每秒钟会有多少对象在新生代里，计算平均触发时间，平均回存活多少对象。出一个内存GC分析报告。
      - 关键点是新生代中的S区要放的下，不会进入老年代。

2. eden区和survivor区 对象是通过什么规则被分配到这两个区的呢？

优先分配到Eden区，Survivor区是用来放每次GC过后的那些存活对象的

3. 老年代为什么适合标记整理算法？
   1. 因为老年代存活对象多，复制占用内存，且耗费时间。
<a name="QTVoD"></a>
#### 问题与总结

- [ ] 分析财务系统适合怎么配置GC？
- [ ] FullGC过多会造成停顿，那是不是高并发系统都要做到0 FullGC ?
- [ ] 结合确收逻辑，分析占用对象有多少？ 

<a name="K8uDF"></a>
### 17-揭秘JVM年轻代垃圾回收器ParNew的工作原理
<a name="SoP9G"></a>
#### 笔记

- 新生代ParNew垃圾回收器主打的就是多线程垃圾回收机制，另外一种Serial垃圾回收器主打的是单线程垃圾回收，这俩都是回收新生代的，唯一的区别就是单线程和多线程的区别，但是垃圾回收算法是一样的。
- ![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1629451087746-e895d126-2a71-4435-8fa8-59d7acf7137b.png#crop=0&crop=0&crop=1&crop=1&height=335&id=u66a85f2e&margin=%5Bobject%20Object%5D&name=image.png&originHeight=335&originWidth=538&originalType=binary&ratio=1&rotation=0&showTitle=false&size=70181&status=done&style=none&title=&width=538)
- 通过启动时设置JVM参数“-XX:+UseParNewGC”指定GC垃圾回收器为ParNew。
- ParNew默认给自己设置的**回收线程的数量**就是跟CPU的核数是一样的。
- 也可以通过“-XX:ParallelGCThreads”参数调节线程的数量。（不建议变动）
<a name="ORSzO"></a>
#### 问题与总结

- [x] 到底是用单线程回收好，还是多线程垃圾回收好？到底是Serial垃圾回收器好还是ParNew垃圾回收器好？

如果在单核CPU系统上，还用ParNew多线程回收，会导致CPU来回多个线程切换浪费。<br />Serial垃圾回收器一般应用于PC客户端上，因为很多Windos上只会给你分配一个线程来使用；

- [x] Java -Server服务 和 Client 服务的区别？（如果没有指定启动参数，JVM会自己检查所在系统是否为服务器系统，配置是否为4C8G的服务器以上配置）
   - HotSpot Client ：（启动快，性能差点）为在客户端环境中减少启动时间而优化；比较适合桌面程序，它会做一些例如像快速初始化，懒加载这一类的事件来适应桌面程序的特点。
   - HotSpot Server：启动慢，稳定，性能好）为在服务器环境中最大化程序执行速度而设计；适合做服务器程序，一些针对服务器特点的事情，比如预加载，尤其在一些并发的处理上，是会做更多的优化。

<a name="SkFs3"></a>
### 18-JVM老年代CMS垃圾回收器内部干了啥？
<a name="rAV3E"></a>
#### 笔记

- 老年代内存空间小于了历次Minor GC后升入老年代对象的平均大小，判断Minor GC有风险，可能就会提前触发Full GC回收老年代的垃圾对象。
- 追踪GCRoots的方法，看各对象是否被GCRoots给引用，如是的话，那就是存活对象，否则就是垃圾对象。
- **标记清除算法** 最大的问题就是：造成 内存碎片。

- CMS工作原理：

1.回收算发：标记清除算法<br />2.回收线程 和 用户工作线程 同时执行处理。（简称并发处理）

- **CMS执行一次垃圾回收的 4个阶段**
1. 初始标记：（暂停）｜追踪，检查GCRoots直接引用的对象，让用户线程全部暂停。
1. 并发标记：（并发）｜用户线程和回收线程 同时运行，可随意创建新对象，对老年代所有对象进行GC Roots追踪看是否被引用，这是最耗时的部分，但是并发进行不影响性能。
1. 重新标记：（暂停）
   1. 因为第二阶段里，一边标记存活对象和垃圾对象，一边系统不停运行创建新对象，让老对象变成垃圾。
   1. 所以第二阶段会产生一些新的 存活对象 和 垃圾对象。所以要再次进入 “Stop the World” 状态，让用户线程，回收线程都停止工作。
   1. 然后标记出 最终需要清除的对象 ，其实是对第二阶段中系统运行变动过的少数对象进行标记，所以运行速度很快。
4. 并发清理：（并发）｜JVM一边回收垃圾，一边正常工作。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1629454148625-626aa1ef-62da-46e5-95ae-3259a98d2c78.png#crop=0&crop=0&crop=1&crop=1&height=424&id=u1779dcee&margin=%5Bobject%20Object%5D&name=image.png&originHeight=424&originWidth=640&originalType=binary&ratio=1&rotation=0&showTitle=false&size=107538&status=done&style=none&title=&width=640)<br />对老年代所有对象进行GC Roots追踪看是否被引用，最耗时的部分。也就第二部分，但是不影响性能。
<a name="AtWho"></a>
#### 问题与总结

- [ ] CMS的各细节参数怎么设置？

<a name="Wydfu"></a>
## JVM项目实战案例
<a name="GS1AA"></a>
### 19-线上部署系统，如何设置垃圾回收相关参数
<a name="lfBG0"></a>
#### 笔记
**CMS垃圾回收器可能存在的3个问题？**<br />**消耗CPU资源问题**

- CMS回收器一个最大的问题，就是垃圾回收线程和用户工作线程同时进行，导致有限的CPU资源被占用。
- CMS默认启动的垃圾回收线程的数量是（CPU核数 + 3）/ 4。（这我觉得物极必反，不可能做到又不用CPU是吧？这算不上是缺陷。）

**Concurrent Mode Failure问题**

- **引发问题的条件一**：CMS并发清理阶段，也可能同时触发MinorGC，并发进行的同时也会产生新垃圾对象，称之为 **浮动垃圾**，但是这种垃圾不会在本次FullGC中回收，
- <br />
- **引发问题的条件二**：CMS垃圾回收的触发时机，其中有一个就是当老年代内存占用达到一定比例了，就自动执行GC。“-XX:CMSInitiatingOccupancyFaction”可以设置老年代占用多少比例时触发CMS垃圾回收。（JDK 1.6里面默认的值是92%时，触发Full GC。）
- **原因**：当紧接着FullGC后的YongGC，产生的新对象也会进入老年代，如果此时没有一块完整的内存地址放得下，那就触发**Concurrent Mode Failure **问题，GC会切换为单线程的“Serial Old”垃圾回收器替代CMS回收器，该回收器是单线程，会让所有用户线程停顿，清理完所有垃圾对象后，才能继续使用。
- <br />
- 解决：设置参数（设置多少次FullGC后整理内存，设置是否FullGC后要整理碎片）
- “-XX:CMSFullGCsBeforeCompaction” 设置多少次FullGC后就整理一次内存碎片，默认是0，意思没执行一次FullGC后就会整理内存碎片。
- 但是！！！ 每次整理碎片，都要去暂停用户线程，会影响性能，切记！！！

**内存碎片问题**

- **如内存碎片太多**，会导致后续对象进入老年代找不到可用的连续内存空间了，内存不足，频繁触发Full GC.
- CMS参数“-XX:+UseCMSCompactAtFullCollection”，默认打开，意思是在Full GC之后要再次进行“Stop the World”，停止工作线程，然后进行碎片整理，就是把存活对象挪到一起，空出来大片连续内存空间，避免内存碎片。
<a name="ET935"></a>
#### 问题与总结

- [x] 为什么老年代的FullGC比新生代的YoungGC要慢很多？
   - 新生代直接从GCRoots开始追踪活对象，内存少，对象数量也少。
   - 老年代內存多，数量也多，且对象分布零散，回收起来慢。

- [x] 总结触发老年代GC的几个时机？
1. 老年代可用内存 < 新生代全部对象大小，如果没有担保机制，会直接FullGC，所以一般会打开。
1. 老年代可用内存 < 历次新生代GC后进入老年代 的平均对象大小，此时会提前触发FullGC。
1. 新生代MinorGC后的存活对象 > S1区域，进入老年代发现内存不足，也会出发FullGC。
1. 老年代已经使用内存占比，超过设置的比例，也会自定出发FullGC，默认95%。

<a name="n5Imj"></a>
### 20-上亿请求电商系统，新生代垃圾回收参数如何优化？
<a name="SFZk8"></a>
#### 笔记
**背景：**<br />一番数据量化，最终结论，4核8G配置下，达到300个订单/秒的订单系统， 300个订单就大约300kb的大小，加上库存，优惠券，商品等业务对象，一般需要对单个对象开销放大10-20倍的量.<br />所以，300kb * 20 * 10 = 60mb, 每秒钟大概有60 mb的垃圾开销。

**内存如何优化：**

- 假设4核8G，一般JVM内存会给4G，剩下4个G会给操作系统使用。
- **JVM中内存分配：**
   - 堆总共给3G，新生代= 2G，老年代=1G
   - 永久代256M内存
   - 每个线程的虚拟机栈1M，会有几百个线程，会占用剩下的几百M。

“-Xms3072M -Xmx3072M -Xmn1536M -Xss1M -XX:PermSize=256M -XX:MaxPermSize=256M”

**运行分析：**

- 上面说1S产生60MB，所以新生代1.5G内存，大约25S就会占满，就要进行youngGC, 所以99%新生代的对象被回收，还有当前1S的对象还在里面，可能存活的就100MB左右。
- 但是，S和Eden区占比是1:1:8，所以S区只有150M内存，运行两次就S区域不足了，会频繁进入老年代。
- 所以此时要**调整为 新生代2G，老年代1G，防止S区不足，频繁进入Old区**。

**如何设置多少岁后进入老年代？**

- 默认15岁后进入，但是按照高并发系统，一般很少出现几分钟还存活的对象，这种可能也是SpringIOC 容器中的@service，@config 等常驻对象。
- 所以在高并发系统中可以适当降低岁数，比如5岁，就进入老年代，但没有绝对，因为FullGC也容易Stop。

**设置多大的对象可以直接进入老年代？**

- 一般参考系统中是否有缓存内存占用，需要计算一下，没有的话一般 直接设置 1M 。

**如何在JVM启动时指定回收器？**

- “-Xms3072M -Xmx3072M -Xmn2048M -Xss1M  -XX:PermSize=256M -XX:MaxPermSize=256M  -XX:SurvivorRatio=8 -XX:MaxTenuringThreshold=5 -XX:PretenureSizeThreshold=1M -XX:+UseParNewGC -XX:+UseConcMarkSweepGC”
- 新生代中ParNew垃圾回收器的核心参数，其实就是配套的新生代内存大小、Eden和Survivor的比例，
   - 主要是避免：YGC后在S区放不下对象，指定岁数，尽量让对象别长期占用S区。
<a name="jx5wv"></a>
#### 问题与总结

1. 主要是三方面：
   1. 调整新生代 和 老年代的比例 2 : 1
   1. 预估新生代中 E区 和 S 区的内存。
   1. 降低 对象被回收的年龄 ，避免过久占用新生代。

 

2. 评估一下线上财务系统的运行模型：
- [ ] 每秒占用多少内存？
- [ ] 多长时间触发一次Minor GC？
- [ ] 一般Minor GC后有多少存活对象？
- [ ] Survivor能放的下吗？
- [ ] 会不会频繁因为Survivor放不下导致对象进入老年代？
- [ ] 会不会因动态年龄判断规则进入老年代？

<a name="Gwbd6"></a>
### 21-上亿请求电商系统，老年代垃圾回收参数如何优化？
<a name="IQ7PB"></a>
#### 笔记

**上篇的背景下：一般什么情况下对象会进入老年代呢？**

- 经过上一批的计算，大约每隔5分钟会有200M的对象进入老年代。
- 年龄超过5岁的
- 超过1M的大对象

**大促期间多久会出发一次FullGC？**<br />（总结：一般要半个多小时后才触发一次FullGC）<br />（1）没有打开“ -XX:HandlePromotionFailure”（晋级失败，就会对比以前进入老年代的平均年龄）

   - 结果老年代可用内存最多也就1G，新生代对象总大小最多可以有1.8G，会导致每次Minor GC前检查，都发现“老年代可用内存” < “新生代总对象大小”，导致每次Minor GC前都触发Full GC。
   - JDK 1.6以废弃该参数，只要满足下面第二个条件就可以直接触发Minor GC，不需要触发Full GC。

（2）每次Minor GC之前，都检查一下“老年代可用内存空间” < “历次Minor GC后升入老年代的**平均对象大小**”<br />按照目前设定的背景，要很多次Minor GC之后才可能有一两次碰巧会有200MB对象升入老年代，所以这个“历次Minor GC后升入老年代的平均对象大小”，基本是很小的。<br />（3）可能某次Minor GC后要升入老年代的对象有几百MB，但是老年代可用空间不足了。<br />（4）设置了“-XX:CMSInitiatingOccupancyFaction”比例，比如设定值为92%，那么此时可能前面几个条件都没满足，但是刚好发现这个条件满足了，比如就是老年代空间使用超过92%了，此时就会自行触发Full GC。

**老年代GC的时候会发生“Concurrent Mode Failure”吗？**

- 必须是CMS触发Full GC的时候，系统运行期间还让200MB对象进入老年代
- **概率很小，没必要刻意去调整参数。**

**CMS垃圾回收之后进行内存碎片整理的频率应该多高？**

- 半个多小时才做一次FullGC,也没必要刻意调整，默认0，每次都整理就完事了

<a name="txecg"></a>
#### 问题与总结

1. 去估算一下你的财务系统运行模型：
- [ ] 每秒占用多少内存，
- [ ] 多长时间触发一次Minor GC，
- [ ] 一般Minor GC后有多少存活对象，Survivor能放的下吗？
- [ ] 会不会频繁因为Survivor放不下导致对象进入老年代？会不会因动态年龄判断规则进入老年代？

<a name="Kp2qa"></a>
### 22-第三周问题答疑
<a name="rZCrb"></a>
#### 笔记

1. 如果没有主动开启设置 分配内存担保失败 参数，那么老年代内存 < 新生代对象内存，会直接FullGC，可能会导致频繁FullGC，所以一般都是要开启的。

2. 4核, 系统开启线程数量50个左右，大概所有线程100+，高峰期同时工作，CPU基本上满荷负载了。

3. 选择了g1 作为收集器，存在不一样的回收效果。

4. Tomcat下部署多个WEB应用，那么就只会有一个JVM，你的web应用只是一堆代码。

5. 不同垃圾回收器，只是性能和吞吐有所求别，不影响回收的时机。
5. ![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1629653194849-1b5fdd79-2c4f-46d0-927e-bbfc6f29f6fa.png#crop=0&crop=0&crop=1&crop=1&height=344&id=u5e2f60bd&margin=%5Bobject%20Object%5D&name=image.png&originHeight=688&originWidth=1073&originalType=binary&ratio=1&rotation=0&showTitle=false&size=226267&status=done&style=none&title=&width=536.5)

7. 针对web服务，POJO类一般都是在新生代，而通过@service @controller @component 等注解创建的对象一般都是在老年代。针对纯java代码的后台跑批服务，基本都是新生代，除了一些通过静态变量或者常量引用的类，或者通过单例创建的类（本质也是通过静态变量引用）。
7. <br />

<a name="dx9uG"></a>
#### 问题与总结

- [ ] 查询**一下GC回收器里面的实现原理，而不是执行原理。**
- [ ] 怎么查看线上GC回收的次数，和频率？
- [ ] 重新标记只会对并发标记改动的对象进行标记，是由什么结构存储了并发标记改动的对象吗？ 为啥这么快？
- [ ] 照着第6个问题，分析一下自己的系统？
<a name="CHpL5"></a>
### 23-G1垃圾回收器的工作原理？
<a name="fpy3Z"></a>
#### 笔记

- 目录

1、ParNew + CMS的组合让我们有哪些痛点？

   -  STW，会暂停用户工作线程

2、G1垃圾回收器。<br />特点：把Java堆内存拆分为多个大小相等的Region，<br />也有新生代，老年代的区分，不过是逻辑上的区分。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1629706143693-65446cca-6c41-49a8-9abd-0eed18f0cb9e.png#crop=0&crop=0&crop=1&crop=1&height=201&id=u86cbe133&margin=%5Bobject%20Object%5D&name=image.png&originHeight=201&originWidth=313&originalType=binary&ratio=1&rotation=0&showTitle=false&size=48798&status=done&style=none&title=&width=313)<br />3、G1是如何做到对垃圾回收导致的系统停顿可控的？

   - G1内部记载了每一个Region的对象数量，和垃圾数量，能清楚的知道需要清除需要耗费多少时间。
   - 所以，G1回收器能够在你设置的时间范围內，去选择性清除垃圾。

4、Region可能属于新生代也可能属于老年代。
<a name="AICpC"></a>
#### 问题与总结
G1主要的好处就是减少了用户线程停顿的场景，实现高并发场景，也就是说，一般对高并发情况下才有作用。<br />设置最大可停顿时间，避免长时间的停顿。

- [ ] G1是如何工作的？
- [ ] 对象什么时候进入新生代的Region？
- [ ] 什么时候触发Region GC？
- [ ] 什么时候对象进入老年代的Region？
- [ ] 什么时候触发老年代的Region GC？

<a name="CYd09"></a>
### 24-G1分代回收原理图解，为什么回收性能比传统GC更好？
<a name="WL56T"></a>
#### 笔记

- “-XX:+UseG1GC”来指定使用G1垃圾回收器
- Region进行了垃圾回收，新生代也会减少，这个是一个动态的过程。
- 新生代里还是有Eden和Survivor的划分的，也就是说新生代也分为S区，和区，也是用的复制算法。

**思考：**

- 到底有多少个Region呢？
   - 最多 2048 个Region （区域）个数
   - JVM启动时，自动按照：堆总内存/2048个 = Region每个区域的内存大小
- 每个Region的大小是多大呢？
   - Region块大小必须是2的倍数，按照上面总内存除以个数，得到每个区域的大小，约1MVB，2MB，4MB之类的大小。

**参数设置：**

- “-XX:G1NewSizePercent”来设置新生代 和 老年代的初始占比。
- “-XX:G1MaxNewSizePercent”来设置新生代最大的占比，默认不会超过60%。

**什么时候触发新生代的回收？**

- 新生代达到设定的堆内存大小的60%，就会触发YoungGC.

**对象什么时候进入老年代？**

- 年龄到达设置的值。
- 动态年龄判断，发现某次新生代GC后，存活对象超过了S区的50%。

**大对象存储**：

- G1提供了专门的Region来存放大对象，而不是去占用老年代的Region.
- 在G1中，大对象的判定规则就是一个大对象超过了一个Region大小的50%.
- 如果一个Region存不下，会用几个来合并存储，因为Region数量和大小在运行当中不断变化的。
- 大对象Region，也会随着新生代，老年代一起回收。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1629713054666-a66e0dd4-ca2d-4329-a763-64657e4ae3c5.png#crop=0&crop=0&crop=1&crop=1&height=386&id=u14c90f8c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=386&originWidth=495&originalType=binary&ratio=1&rotation=0&showTitle=false&size=98524&status=done&style=none&title=&width=495)
<a name="iYgyl"></a>
#### 问题与总结

- [ ] G1垃圾回收器在新生代垃圾回收过程中，相比之前的ParNew而言，最大的进步在哪里？

- [ ] G1这种垃圾回收器到底在什么场景下适用呢？有了G1以后，是不是还有一些场景采用“ParNew+CMS”垃圾回收器也可以呢？

1、G1压缩内存空间会比较有优势，适合会产生大量碎片的应用； <br />2、G1能够可预期的GC停顿时间，对高并发应用更有优势 <br />3、其他垃圾收集器对大内存回收耗时较长，G1对内存分成多块区域，能够根据预期停顿时间选择性的对垃圾多的区域进行回收，适用多核、jvm内存占用大的应用 <br />4、parNew+cms回收器比较适用内存小，对象能够在新生代中存活周期短的应用

   - 总结：g1适合 大堆的场景 或者有业务不能有太高的延时 如果业务上不需要特别大的堆 或者 业务属于不需要及时反馈用户的比如贷款业务 申请额度之后就后台处理了 有额度以后 在通知你这个时候 par new 加 cms可以用的

<a name="aDGEO"></a>
### 25-G1垃圾回收器，应该如何设置参数？
<a name="pA2QX"></a>
#### 笔记

- **什么时候触发新生代+老年代的混合垃圾回收？（学名又叫：Mixed GC）**
   - G1有一个参数，是“-XX:InitiatingHeapOccupancyPercent”，他的默认值是45%，也就是说老年代占据堆内存的45%时，就会触发。但是由于设置停顿时间的限制，只能从新生代 老年代中各自挑选一些Region区域来回收。

- **G1混合垃圾回收过程？**
   - 初始标记：会暂停用户线程，GCRoot节点追踪，检查是否为垃圾。
   - 混合标记：和用户线程并发进行，耗时长，但是不影响性能。
   - 最终标记：会暂停用户线程，也是对上一步并发过程中产生的新垃圾做一个最终标记，速度也不慢。
   - **回收阶段**：（和CMS最大区别的地方）会暂停用户线程，开始回收垃圾，但是会选择性的去回收Region区域，避免超过限制回收时间。

**过程简述：**<br />G1回收器，在最后回收阶段，可以按照时间粒度化，如果回收时，发现超过了停顿时间限制，会回收一部分，恢复系统，在进行回收，按时间粒度化，但是次数也不会太多，默认8次， ，最终实现避免停顿时间过长。至于中间停顿 多长时间，时间间隔有GC自己决定。

**参数调节：**<br />**回收的Region区域，必须的是存活对象低于85%。**

- 还有一个参数，“-XX:G1MixedGCLiveThresholdPercent”，他的默认值是85%，意思就是确定要回收的Region的时候，必须是存活对象低于85%的Region才可以进行回收，避免过多存活对象需要复制保留。

**回收阶段里，回收总停顿次数，默认不超过8次。**

- 可设置最多会回收几次：“-XX:G1MixedGCCountTarget”（默认：8）

**回收阶段如果清空内存，超过总堆内存的5%，也会停止回收，默认5%**

- 可设置回收时，如果空出来的新Region个数超过堆内存的5% ，也会停止回收。“-XX:G1HeapWastePercent”，默认值是5%。

**“-XX:MaxGCPauseMills”，他的默认值是200毫秒，停顿时间不超过这个值**

**混合回收失败时的Full GC，没有剩余空闲空间去复制，会失败，就只能单线程标记清除。**

- 如果在进行Mixed回收时，年轻代老年代都基于复制算法进行回收，都要把各个Region的存活对象拷贝到别的Region里去，此时拷贝中发现没有空闲Region可以承载自己的存活对象了，就会触发 一次失败。
- 一旦失败，立马就会切换为停止系统程序，然后采用单线程进行标记、清理和压缩整理，空闲出部分Region，但是过程很慢。
<a name="ca1Pi"></a>
#### 问题与总结

- [ ] 如**果使用G1垃圾回收的时候，应该值得优化的是什么地方？**
- 对于新生代，和之前的理论差不多，主要目标是避免短期存活的对象进入老年代。 
   - 1. 预估系统每次GC后存活对象，确保Survivor能放得下。 
   - 2. 避免高峰期间，新生代对象满足动态年龄判断条件，导致短期存活对象进入老年代。 
   - 3. 大对象有大对象Region，不占用老年代空间，基本不用考虑。 
- 对于老年代； 
   - 1. 对于可预测停顿时间，需要合理设置，并不是越小越好，如果过小，有可能多次回收效果不大，最终导致回收失败FullGC，停顿系统线程。 
   - 2. G1HeapWastePercent 这个参数，我觉得应该可以适当提高，避免万一真的遇到了高峰期，短期存活对象进入老年代，但是回收的时候，进行了几次混合回收的时候， 刚好达到了5%,但是在老年代Region中可能还是存在某些短期存活对象没有被回收。过早结束混合回收。 
   - 3. -XX:G1MixedGCLiveThresholdPercent这个参数，暂时不用管，降低这个参数可能会让Region的回收效率更高，但是也可能导致短期存活对象驻留内存时间过长，进入老年代的风险。

- [ ] 什么时候可能会导致G1频繁的触发Mixed混合垃圾回收？
- [ ] 如何尽量减少Mixed GC的频率？

<a name="nULxm"></a>
### 26-百万级用户的在线教育平台，G1垃圾回收性能优化
<a name="f1Fg4"></a>
#### 笔记

- 背景：百万级注册用户，日活20万用户在线上课。会记录一些相关动作等。并发出现在晚上2小时，初略统计20万每小时进行1200万次互动，平均每秒3000TPS，至少需要 5台4C*8G的机器才能抗的住。
- 所有估算下，一次互动请求大致会连带创建几个对象，占几KB的内存，比如我们就认为是5KB吧那么一秒600请求会占用3MB左右的内存。

- **分配4G给堆内存，其中新生代默认初始占比为5%，最大占比为60%，每个Java线程的栈内存为1MB，**元数据区域（永久代）的内存为256M，此时JVM参数如下：
   - “-Xms4096M -Xmx4096M  -Xss1M  -XX:PermSize=256M -XX:MaxPermSize=256M -XX:+UseG1GC“
   - “-XX:G1NewSizePercent”参数是用来设置新生代初始占比的，不用设置，维持默认值为5%即可。
   - “-XX:G1MaxNewSizePercent”参数是用来设置新生代最大占比的，也不用设置，维持默认值为60%即可。

- 实G1里是很动态灵活的，会根据你设定的gc停顿时间给你的新生代不停分配更多Region，到达一定程度才会触发GC，而不是固定的。
- 既不会频繁GC，也不会长时间GC，这才是最终的目的。
- 主要核心在于调节“-XX:MaxGCPauseMills”这个参数，保证新生代gc别太频繁的同时，还得考虑每次gc过后的存活对象有多少，避免存活对象太多快速进入老年代，频繁触发mixed gc。

<a name="Kt2gz"></a>
#### 问题与总结

- 基于百万用户在线教育平台的G1垃圾回收优化案例，把整个系统的背景、核心业务流程、高峰运行压力、机器部署、每秒请求压力、每秒内存使用压力，都给分析了出来。基于这个每秒内存使用压力，结合G1垃圾回收器的运行原理，来给大家分析在这个压力之下，G1垃圾回收机制会如何来运行，这个过程中可能会产生哪些问题，如何对G1的一些参数进行基本的优化来调整垃圾回收的性能。

<a name="wK5LV"></a>
### 27-第一阶段复习，开发完一个系统如何评估JVM的参数？
<a name="W7BMN"></a>
#### 笔记
**从新生代垃圾回收来看，G1垃圾回收器在新生代垃圾回收过程中，相比之前的ParNew而言，进步在哪里？**

- 最大进步就是STW可控，然各个Region所属区域是动态变化的，但不是随意变化的，还是会为Eden、Survivor、老年代保留各自需要的空间。不会让Eden空间的分配超过系统设定的值。
- G1适合大内存机器，因为32G内存，要是用ParNew+CMS，每次gc都是内存快满了，此时要回收对象太多了，就会导致gc停顿时间很长，所以针对那种**大内存机器，短停顿**，用G1是很合适的。
<a name="Hk3Or"></a>
#### 问题与总结

- [ ] 对上面部分，做画图总结。
- [ ] 对上面学习的知识，做一次头脑风暴，想到什么写什么？

<a name="K5JoA"></a>
### 28-GC导致线上系统突然卡死无法访问
<a name="O2ZQy"></a>
#### 笔记

- 一般对年轻代的gc进行调优，只要你给系统分配足够的内存即可，核心点在于堆内存的分配、新生代内存的分配内存足够的话，通常来说系统可能在低峰时期在几个小时才有一次新生代gc，高峰期最多也就几分钟一次新生代gc。

- 什么时候新生代GC对系统影响很大？
   - 大内存机器，比如几十个G，在大数据系统情况下，如ES，KafKa，此时Jvm的Eden区分配了很大内存，假设一分钟被塞满一次就会停顿几秒去回收。

- G1天生就适合这种大内存机器的JVM运行，完美解决大内存垃圾回收时间过长的问题。

- **GC名词解释**
   - YoungGC/MinorGC ： 新生代GC
   - OldGC/FullGC：老年代GC
   - MajorGC：可以理解为针对老年代的GC
   - Mixed GC：仅仅在G1回收器里才有的概念。
<a name="uM2Id"></a>
#### 问题与总结

- [x] YoungGC和FullGC分别在什么情况下发生？（具体过程看前面的章节描述）
   - YoungGC会在Enden区域满了之后触发，采用复制算法。
   - OldGC/一句话概括就是老年代空间也不够了

- [x] 永久代（元空间）满了怎么办？
- 主要存储Class信息，和Meta（元数据）的信息
- 当永久代满时也会引发Full GC，会导致Class、Method元信息的卸载。
- 永久代Java8之后又叫元空间，但是元空间最大的区别是不在Java虚拟机中，class元数据存在本地内存，所以不受虚拟机大小限制，字符串池和类的静态变量放入java堆中，受大小限制。
- 如果FullGC之后，永久代还存不下，那就触发OOM异常。

<a name="k84yh"></a>
### 29-每秒10万并发的BI系统是如何频繁发生YoungGC？
<a name="OUoFv"></a>
#### 笔记

- 系统数据流程图

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1630490481205-7184b39d-ed8f-4857-b14e-c6d8056b608c.png#clientId=u05b4f55d-fff5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=430&id=u59e904ef&margin=%5Bobject%20Object%5D&name=image.png&originHeight=860&originWidth=1304&originalType=binary&ratio=1&rotation=0&showTitle=false&size=385985&status=done&style=none&taskId=u82258661-add0-4129-8ba3-f15490b65ea&title=&width=652)<br />**配置**：开始系统部署简单，2台机器，4C8G配置，新生代内存分配1.5G左右，Eden区占用1G左右。<br />**背景**：前端轮训动态刷新。<br />**技术痛点**：

   - 开始数据量少时运行正常，商家数量暴涨的时候，前端“实时数据报表”会自动刷新，几万个商家同时在线就会导致约500/S的并发请求，量化下来，每秒50M的对象会进入Eden区。
   - 频繁YoungGC，按照上面估算，约3分钟左右，Eden区就会填满，触发YoungGC。
   - 如果按照1G的Eden区计算，100ms时间内就能搞定回收，也不会太多停顿，影响不大。
   - 随着商家增多，需要达到每秒10S并发，就得增加 单节点内存，因为BI系统很吃内存。
- **问题来了**：假如新生代分配到20G内存的时候，Eden区也会占据16G以上的内存，每秒产生几百M数据，如果YoungGC的话会导致1S左右的停顿，这样有可能就会导致前端查询超时。
- **解决**：用G1来优化大内存机器的YoungGC性能，确保不会停顿过久。

<a name="rpYly"></a>
#### 问题与总结

- [ ] 怎么把上面BI系统应用到我们的财务系统里面来？面试的可以说道说道。
<a name="N6XTG"></a>
#### 
<a name="ESySo"></a>
### 30-每日百亿数据量的实时分析引擎，为啥频繁发生FullGC?
<a name="wBLIo"></a>
#### 笔记
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1630498765208-7d78f297-fc27-46c5-97fd-4b4edf8954dd.png#clientId=u05b4f55d-fff5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=302&id=u9c49be8c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=604&originWidth=638&originalType=binary&ratio=1&rotation=0&showTitle=false&size=201708&status=done&style=none&taskId=uef0506d7-a1ac-4abd-a996-dcac2c77cf9&title=&width=319)

- **背景：**
   - 系统会不停的从MySQL数据库以及其他数据源里提取大量的数据加载到自己的JVM内存里来进行计算处理。分布式部署多台，每台机器每分钟执行100次数据提取和计算任务。
- **假设：**
   - 次提起1万条内存计算，耗费10S，一台4C8G的机器，JVM给了4G，其中新生代和老年代分别是1.5G。

- **评估这个系统到底多久会塞满新生代？**
   - 如果按照S区和E区默认比例1:1:8分配，那么每块S区就是100M，Eden区就是1.2G，假设每分钟计算100次，基本上1min就会把Eden区塞满。

- **触发YoungGC会有多少对象进入老年代？**
   - 计算1万条约10S，假设20个还在计算中，那么就会有200M的数据存活，那么就无法YoungGC,会进入老年代。

- **系统运行多久，老年代会填满？多久会触发FullGC？**
   - 大概每分钟会有200M进入老年代，那么2分钟过去了，400M进入老年代，按照这样计算7分钟后就会塞满Old区。
   - 按照上面计算，基本上平均7，8分钟就触发一次FullGC,频率很高。

- **这种案例怎么优化？**
   - 主要原因是S区域存放不下存活的对象，假设3G的堆内存，2G分配给新生代，调节S区域的比例，减少对象进入老年代的比例，避免频繁FullGC.

- **如果工作负载扩大10倍呢？**
   - 按照上面的估算，十多秒就会放1G的对象到老年代，每分钟会产生2-3次FullGC.

- **使用大内存机器优化上述场景**
   - 通过提升机器配置，增加S区的空间，避免频繁进入老年代，减少FullGC频率。

<a name="i6vqs"></a>
#### 问题与总结

- [x] 针对大内存机器，就一定要用G1来减少每次YoungGC的停顿时间吗？
- 不用，后台自动计算系统，不是to C ,允许稍微停顿，不影响体验。

<a name="Q6Nf9"></a>
### 31-第四周问题答疑
<a name="JmHfE"></a>
#### 笔记
TODO 
<a name="u3Duu"></a>
#### 问题与总结

<a name="LntIx"></a>
## JVM垃圾回收GC日志分析
<a name="wONEU"></a>
### 32-模拟出YoungGc的场景，GC日志分析
<a name="qhyBE"></a>
#### 笔记
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1630921689896-9529b42f-a11f-4978-a3e6-b460910f33de.png#clientId=u05b4f55d-fff5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=234&id=u2e392f0b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=468&originWidth=858&originalType=binary&ratio=1&rotation=0&showTitle=false&size=304717&status=done&style=none&taskId=ua5991c3b-6e37-4eaa-bf2c-27a6e62ef37&title=&width=429)<br />**导读**<br />本章节，演示年轻代的Young GC是如何发生的，同时告诉如何在JVM参数中去配置打印对应的GC日志，然后通过GC日志来慢慢的分析JVM的GC到底是如何运行的。

- <br />

**如何配置打印出JVMGC日志？**

- -XX:+PrintGCDetils：打印详细的gc日志
- -XX:+PrintGCTimeStamps：这个参数可以打印出来每次GC发生的时间
- -Xloggc:gc.log：这个参数可以设置将gc日志写入一个磁盘文件

**GC日志里会打印出JVM当前运行下的参数**<br />CommandLine flags: -XX:InitialHeapSize=10485760 -XX:MaxHeapSize=10485760 -XX:MaxNewSize=5242880 .........

GC日志分析过程<br />日志内容
```
0.268: [GC (Allocation Failure) 0.269: [ParNew: 4030K->512K(4608K), 0.0015734 secs] 4030K->574K(9728K), 0.0017518 secs]
[Times: user=0.00 sys=0.00, real=0.00 secs]
```

- “GC (Allocation Failure)” ：表示对象分配失败，所以触发YoungGC,因为上面错误代码里当要分配一个2M对象，Eden区存不下就会触发分配失败，进行YoungGC.
- “0.268” : 表示系统运行后过了多少秒，发生了本次GC，表示发生的时间。
- “ParNew: 4030K->512K(4608K), 0.0015734 secs” 
   - “ParNew”：表示用ParNewGc垃圾回收器执行的。
   - “4030K->512K(4608K)”：轻代可用空间是4608KB，也就是4.5MB，因为2个S区有也是可放空间，但是另一个要保持空闲，所以最终可用空间就是Eden区+1个S区的空间。也就是4.5M。
   - “4030K->512K”：对年轻代执行了一次GC，GC之前都使用了4030KB了，但是GC之后只有512KB的对象是存活下来的。
- “4030K->574K(9728K), 0.0017518 secs：意思是整个Java堆内存是总可用空间9728KB（9.5MB），也就是年轻代4.5MB+老年代5M。
   - “4030K->574K”：GC前整个Java堆内存里使用了4030KB，GC之后Java堆内存使用了574KB。
- “[Times: user=0.00 sys=0.00, real=0.00 secs] ”：意思就是本次gc消耗的时间最小单位是小数点后两位，这里0.00secs，可以忽略不计。

**总体堆内存情况分析**

- 原日志记录
```

Heap
par new generation   total 4608K, used 2601K [0x00000000ff600000, 0x00000000ffb00000, 0x00000000ffb00000)
eden space 4096K,  51% used [0x00000000ff600000, 0x00000000ff80a558, 0x00000000ffa00000)
S-from space 512K, 100% used [0x00000000ffa80000, 0x00000000ffb00000, 0x00000000ffb00000)
S-to   space 512K,   0% used [0x00000000ffa00000, 0x00000000ffa00000, 0x00000000ffa80000)
concurrent mark-sweep generation total 5120K, used 62K [0x00000000ffb00000, 0x0000000100000000,0x0000000100000000)
Metaspace       used 2782K, capacity 4486K, committed 4864K, reserved 1056768K
 class space    used 300K, capacity 386K, committed 512K, reserved 1048576K
```
这段日志是在JVM退出的时候打印出来的当前堆内存的使用情况.<br />每个数组他会额外占据一些内存来存放一些自己这个对象的元数据,占用内存不仅仅是代码中设置的大小。

- 解释后的记录
```
Heap
par new generation   total 4608K, used 2601K  // “ParNew”垃圾回收器负责的年轻代总共有4608KB（4.5MB）可用内存，目前是使用了2601KB（2.5MB）。
eden space 4096K,  51% used 	// Eden区此时4MB的内存被使用了51%，就是因为有一个2MB的数组在里面。
S-from space 512K, 100% used // From Survivor区，512KB是100%的使用率，此时被之前gc后存活下来的512KB的未知对象给占据了。
S-to   space 512K,   0% used // To Survivor区，空闲状态。
concurrent mark-sweep generation total 5120K, used 62K // CMS垃圾回收器，管理的老年代内存空间一共是5MB，此时使用了62KB的空间
Metaspace       used 2782K, capacity 4486K, committed 4864K, reserved 1056768K     // Metaspace元数据空间和Class空间，存放一些类信息、常量池之类的东西，此时的总容量，使用内存，等等。
 class space    used 300K, capacity 386K, committed 512K, reserved 1048576K   // 同上
```
<a name="Cm5GO"></a>
#### 问题与总结
```
Metaspace       used 2782K, capacity 4486K, committed 4864K, reserved 1056768K
class space    used 300K, capacity 386K, committed 512K, reserved 1048576K
```

- [ ] 对JDK 1.8以后的Metaspace和Classspace，这里都是存放什么内容的？
- [ ] 总结：上面日志主要记录了回收器的哪些信息？

<a name="tAWXc"></a>
### 33-模拟出对象进入Old区域的场景
<a name="gcH8T"></a>
#### 笔记
**模拟对象从新生代进入老年代**

- 年龄达到15岁进入老年代
- **动态年龄判定规划**：Survivor区域内年龄1+年龄2+年龄3+年龄n的对象总和大于Survivor区的50%，此时年龄n以上的对象会进入老年代，不一定要达到15岁。
- 大对象直接进入老年代。

我们通过代码模拟，也就是动态年龄判断规定<br />JVM参数设置，注意⚠️ 新生代我们通过“-XX:NewSize”设置为10MB了，Eden区8MB，Survivor区是1MB，堆总大小是20MB，老年代是10MB，大对象必须超过10MB才会直接进入老年代
```
“-XX:NewSize=10485760 -XX:MaxNewSize=10485760 -XX:InitialHeapSize=20971520 -XX:MaxHeapSize=20971520 -
XX:SurvivorRatio=8  -XX:MaxTenuringThreshold=15 -XX:PretenureSizeThreshold=10485760 -XX:+UseParNewGC -
XX:+UseConcMarkSweepGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:gc.log”
```
动态年龄判定规则的部分示例代码<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1630922165890-1b7dd9d4-d686-47e1-8ec0-6ac41cac8799.png#clientId=u05b4f55d-fff5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=308&id=u5e973c6b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=616&originWidth=974&originalType=binary&ratio=1&rotation=0&showTitle=false&size=437124&status=done&style=none&taskId=u01c5a517-36ec-49db-973f-2151a254889&title=&width=487)<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1630926040199-da742aee-0f10-44f6-9d5a-e29a50841604.png#clientId=u05b4f55d-fff5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=466&id=u2d3b3dac&margin=%5Bobject%20Object%5D&name=image.png&originHeight=932&originWidth=1148&originalType=binary&ratio=1&rotation=0&showTitle=false&size=453136&status=done&style=none&taskId=ud09f581b-ea4f-464d-bb73-3b690109400&title=&width=574)<br />上面会导致：分析了对象是如何通过动态年龄判定规则进入老年代的。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1630926194655-a0bc66ed-9898-443a-8acf-8da8ce7770a8.png#clientId=u05b4f55d-fff5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=156&id=u527bf1a8&margin=%5Bobject%20Object%5D&name=image.png&originHeight=312&originWidth=876&originalType=binary&ratio=1&rotation=0&showTitle=false&size=287209&status=done&style=none&taskId=u07846a5e-372e-467d-abbb-c3f7bdd83f6&title=&width=438) <br />上面会会导致：Young GC过后存活对象放不下Survivor区域，从而部分对象会进入老年代
<a name="kqMxy"></a>
#### 问题与总结
具体分析的细节，请看原文本第46章节

<a name="cx2of"></a>
### 34-JVM的FullGC日志咋看？
<a name="L0sMc"></a>
#### 笔记
导读：老年代的GC是如何触发的。<br />代码示例<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1630926398497-ab275eb2-addc-4fe3-81e5-81aef66766e2.png#clientId=u05b4f55d-fff5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=274&id=u7b609e37&margin=%5Bobject%20Object%5D&name=image.png&originHeight=548&originWidth=994&originalType=binary&ratio=1&rotation=0&showTitle=false&size=531524&status=done&style=none&taskId=u07e1c165-dc34-4883-96cf-07437291b72&title=&width=497)<br />**分析过程**

- 设置大对象阈值为3MB，也就是超过3MB，就直接进入老年代。
- 拿到GC日志
- ![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1630926469401-705115b5-8eec-4186-8d9e-b38426115142.png#clientId=u05b4f55d-fff5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=69&id=ue6d55e6c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=138&originWidth=902&originalType=binary&ratio=1&rotation=0&showTitle=false&size=100644&status=done&style=none&taskId=u340e6212-73b2-4b93-b851-870919dae08&title=&width=451)
- 上面这行代码直接分配一个4MB大对象，会直接进入老年代，然后失去引用变为垃圾。
- ![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1630926532896-7daf4beb-e60e-47ee-b69e-06cdfd298cc8.png#clientId=u05b4f55d-fff5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=98&id=u0668f5b9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=196&originWidth=878&originalType=binary&ratio=1&rotation=0&showTitle=false&size=289236&status=done&style=none&taskId=u2aac75f9-90cf-43b6-aead-5df424e71dc&title=&width=439)
- 上面这四行代码，2个是2MB，1个是128Kb,全部进入Eden区域
- ![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1630926593111-a84eb9e2-6724-4c63-b533-644b2536d4a9.png#clientId=u05b4f55d-fff5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=18&id=u35feffe9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=35&originWidth=441&originalType=binary&ratio=1&rotation=0&showTitle=false&size=28978&status=done&style=none&taskId=u16b8125a-5d8b-46e5-8bd1-768816b3d7b&title=&width=220.5)
- 接着，上面这行代码创建了一个2MB的对象，Eden区放不下，会触发一次YoungGC
- 看下面的GC日志：ParNew (promotion failed): 7260K->7970K(9216K), 0.0048975 secs
- 显示Eden区有7000多Kb的对象，都无法回收，还在引用状态。
- 此时，老年代也无法把这些对象放进去，因为里面已有4mb数组，无法吧3个2M的对象放进去，因为总共才10MB的大小。
- 观察日志：[CMS: 8194K->6836K(10240K), 0.0049920 secs] 11356K->6836K(19456K), [Metaspace: 2776K->2776K(1056768K)],0.0106074 secs]
- 显示此时执行了FullGC,一般会跟进一次YoungGC,还会触发一次元空间（永久代）的GC。
- 回收之后，调整剩余空间，把Eden区的一部分对象放入了老年代。

<a name="Fd43Y"></a>
#### 问题与总结
本章节讲解的是一个触发老年代GC的案例，年轻代存活的对象太多放不下老年代了，此时就会触发CMS的Full GC。

- [ ] 思考模拟出另外几种老年代FullGC的场景？
- [ ] 触发Young GC之前，可能老年代可用空间小于了历次Young GC后升入老年代的对象的平均大小，就会在Young GC之前，提前触发Full GC。
- [ ] 老年代被使用率达到了92%的阈值，也会触发Full GC。

作业》

- [ ] 何去结合GC日志去分析自己的系统平时的GC情况？
- [ ] 看看自己线上系统的GC日志，然后分析分析每次GC的触发时机，以及每次GC之后的对象变化情况。

<a name="JiDZe"></a>
### 35-第七周问题答疑
<a name="PTQFe"></a>
#### 笔记
TODO，后面在看，20210906跳过
<a name="lwNbM"></a>
#### 问题与总结

<a name="We7CP"></a>
## JVM排查工具使用
<a name="FFOwn"></a>
### 36-使用jstat摸清线上JVM的运行状况
<a name="HRYor"></a>
#### 笔记

- 检查JVM的整体运行情况，比较实用的工具之一，就是jstat，能看到当前运行中的JVM内的Eden、Survivor、老年代的内存使用情况，还有Young GC和Full gC的执行次数以及耗时。
- 分析出当前系统的运行情况，判断当前系统的内存使用压力以及GC压力，还有就是内存分配是否合理。
- jstat -gc PID。可以看到Java进程（其实本质就是一个JVM）的内存和GC情况了。
- 显示的列解释：
   - S0C：这是From Survivor区的大小
   - S1C：这是To Survivor区的大小
   - S0U：这是From Survivor区当前使用的内存大小
   - S1U：这是To Survivor区当前使用的内存大小
   - EC：这是Eden区的大小
   - EU：这是Eden区当前使用的内存大小
   - OC：这是老年代的大小
   - OU：这是老年代当前使用的内存大小
   - MC：这是方法区（永久代、元数据区）的大小
   - MU：这是方法区（永久代、元数据区）的当前使用的内存大小
   - YGC：这是系统运行迄今为止的Young GC次数
   - YGCT：这是Young GC的耗时
   - FGC：这是系统运行迄今为止的Full GC次数
   - FGCT：这是Full GC的耗时
   - GCT：这是所有GC的总耗时

- [x] **Jstat可以的到哪些信息？**

新生代对象增长的速率，Young GC的触发频率，Young GC的耗时，每次Young GC后有多少对象是存活下来的，每次Young GC过后有多少对象进入了老年代，老年代对象增长的速率，Full GC的触发频率，Full GC的耗时。

- [x] **查询：新生代对象增长的速率？**

linux机器上运行如下命令：jstat -gc PID 1000 10   表示每1S更新出来最新的一行jstat统计信息，一共执行10次统计。可以固定频率数据Jvm内存统计信息，观察变化差异。分系统场景区缩短时间和次数，能判断出对象的增长速率。

- [x] **查询：YoungG次的触发频率和每次耗时？**

根据上面的到每秒新增的对象速率，可以判断Enden区何时能塞满，比如Eden区总共800M，5M/S的增长速度，则约3min会触发一次YoungGc.

- [x] **查询：Young GC的平均耗时？**

前面jstat -gc pid  能查询到 总耗时和次数

   - YGC：系统运行迄今为止的Young GC次数
   - YGCT：这是Young GC的耗时
   - YGC/YGCT = 平均每次的耗时

- [x] **每次Young GC后有多少对象是存活和进入老年代？**

假设YGC每三分钟触发一次，则jstat -gc PID 180000 10。每三分钟打印一次信息。观察老年代的对象增长速率。和S区域对象减少的量，一般老年代对象增长是很慢的，只能粗略估计。

- [ ] **Full GC的触发时机和耗时？**
   - 如果上面知道了老年代的增长速度，那么总共FullGC的好触发时间和耗时就能计算出来了
   - 总Old内存对象/old对象增长速度=触发的时机。

- [x] **FullGC的平均耗时？**

总耗时/总次数=平均每次耗时
<a name="N3qQr"></a>
#### 问题与总结
其他排查工具：JConsole、VisualVM等可视化的监控工具，还有其他一些开源的监控系统，都是可视化的，自己了解一下。

<a name="f1soI"></a>
### 37-使用Jmap和Jhat摸清JVM对象分布情况
<a name="zQkax"></a>
#### 笔记

- 观察线上JVM中的对象分布，知道运行过程中主要哪些对象占据了内存，多少空间等。
- 如果只要了解运行状况，优化GC，则使用Jstat完全够用了。
- 有时候发现对象增长速度很快，想看看啥对象占据了这么多内存？
- “jmap -heap PID” ：打印出来堆内存相关的一些参数设置，然后就是当前堆内存里的一些基本各个区域的情况。
- “jmap -histo PID”：会按照各种对象占用内存空间的大小降序排列，占用最多的在上面
- “jmap -dump:live,format=b,file=dump.hprof PID”：使用jmap生成堆内存转储快照，可以看到更详细的内存占用情况。
- “jhat dump.hprof -port 7000”：使用jhat在浏览器中分析堆转出快照，就在上面保存的.hprof文件目录下执行命令。（默认端口号7000）
<a name="YIWGN"></a>
#### 问题与总结

- [ ] 知道怎么排查一下，放到在线分析网站上分析内存结构。
- [ ] 知道jstat和Jmap的区别在哪里？
- [ ] 知道Jstat和Jmap这些工具内部是怎么实现的？

<a name="qAkSO"></a>
### 38-测试到上线：如何分析JVM运行状况及合理优化？
<a name="QVR5O"></a>
#### 笔记

- 开发好系统之后预估，优化。
- 优化思路简单来说：
   - 尽量让每次Young GC后的存活对象小于Survivor区域的50%，都留存在年轻代里。
   - 尽量别让对象进入老年代。尽量减少Full GC的频率，避免频繁Full GC对JVM性能的影响。
- 了解一下Java压力测试，用Jstat，Jmap查看运行情况。
- 上线后的监控：Zabbix、OpenFalcon、Ganglia，等等。了解一下。
<a name="jZaMe"></a>
#### 问题与总结
总结：对线上运行的系统，要不然用命令行工具手动监控，发现问题就优化，要不然就是依托公司的监控系统进行自动监控，可视化查看日常系统的运行状态。

- [ ] 公司系统开发流程里有压测环节么？大致了解一下是怎么做的？
- [ ] 对自己负责的项目，观察一下JVM的运行情况。


<a name="d93se"></a>
### 39-模拟10万并发的BI系统，定位解决频繁YGC问题。
<a name="zLkoY"></a>
#### 笔记
模拟频繁YGC的场景<br />设置JVM参数：注意堆内存设置为了200MB，把年轻代设置为了100MB，然后Eden区是80MB，每块Survivor区是10MB，老年代也是100MB。
```
-XX:NewSize=104857600 -XX:MaxNewSize=104857600 -XX:InitialHeapSize=209715200 -XX:MaxHeapSize=209715200 -
XX:SurvivorRatio=8  -XX:MaxTenuringThreshold=15 -XX:PretenureSizeThreshold=3145728 -XX:+UseParNewGC -
XX:+UseConcMarkSweepGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:gc.log
```
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631004382951-ad1b8e9e-eddc-4ed2-b8c0-7dd01e62aaa1.png#clientId=u05b4f55d-fff5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=494&id=u69591aa2&margin=%5Bobject%20Object%5D&name=image.png&originHeight=988&originWidth=1314&originalType=binary&ratio=1&rotation=0&showTitle=false&size=662278&status=done&style=none&taskId=u8c601a3a-5efb-4df6-8dba-f14a6188529&title=&width=657)

1. ![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631004855989-122d7483-dd80-47c9-9daf-1bd5d27b3340.png#clientId=u05b4f55d-fff5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=612&id=u5b2fbd49&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1224&originWidth=832&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1661415&status=done&style=none&taskId=u36c050db-f385-47d3-9ee1-6069e66d3bb&title=&width=416)
1. 先停顿30S，方便找到PID
1. 然后循环50次，模拟50个请求，每个分配100K的数组，接着休眠1S。
1. 使用Jstat观察运行状态，执行：“jstat -gc pid 1000 1000 ” 每隔1S打印一次
1. 观察EU区数据，发现每秒增长5MB对象，当到达70MB+的时候，在分配5MB就会失败了，此时就会触发YGC，清理。
1. 发现：OU，是老年代的内存量，在YGC前后都是0，证明不需要优化。
<a name="nnnXj"></a>
#### 问题与总结

- [ ] 针对上面的代码，演练一下，分析一下，算出其他信息。

<a name="RydO3"></a>
### 40-模拟日百亿数据量的实时分析引擎，定位频繁FullGC
<a name="uoLC4"></a>
#### 笔记
Demo背景：大概每秒执行一次loadData方法，分配4个10MB数组，立马变为垃圾，大是有data1,data2两个100M的数组被引用存活，此时Eden区已占70M左右空间，接着data3变量一次指向了两个100M的数组，为了在1S内触发YoungGC。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631005417625-ddc8166b-5ea6-489e-8f9a-ce8370102763.png#clientId=u05b4f55d-fff5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=540&id=uebf03384&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1080&originWidth=1208&originalType=binary&ratio=1&rotation=0&showTitle=false&size=956708&status=done&style=none&taskId=u34d6eeea-f838-41fb-8f79-ad607bb5efe&title=&width=604)<br />JVM参数设置：大对象伐值设置为20MB，避免大对象直接进入Old区，影响测试效果。
```
-XX:NewSize=104857600 -XX:MaxNewSize=104857600 -XX:InitialHeapSize=209715200 -XX:MaxHeapSize=209715200 -
XX:SurvivorRatio=8  -XX:MaxTenuringThreshold=15 -XX:PretenureSizeThreshold=20971520 -XX:+UseParNewGC -
XX:+UseConcMarkSweepGC -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:gc.log
天下没有难
```
**基于jstat分析程序运行的状态：**

1. ![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631005721324-70f7cc77-757f-4e98-816c-72fca66cf94c.png#clientId=u05b4f55d-fff5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=359&id=u1adbedc1&margin=%5Bobject%20Object%5D&name=image.png&originHeight=718&originWidth=1288&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1468364&status=done&style=none&taskId=uad188222-8757-4350-99d4-8b9a84e6b74&title=&width=644)
1. 序运行起来之后，突然在一秒内就发生了一次Young GC。
1. 因为每次有30MB对象存活，S区放不下，直接进入老年代了。
1. 红圈部分，明显每秒会发生一次Young GC，都会导致20MB~30MB左右的对象进入老年代。
1. 观察老年代对象从30K到60M之后，就发生了FullGC,做了回收，变成了30MB。
1. ![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631006025233-04d7e0a0-6070-4e3b-bc74-2b00f26fade4.png#clientId=u05b4f55d-fff5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=762&id=u0a15a934&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1524&originWidth=786&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1430416&status=done&style=none&taskId=ub9eeef14-a44c-4f63-84ac-2c5bff1f7ab&title=&width=393)
1. 观察28次YGC总耗费时间189ms,平均一次YGC要6ms，而FullGC总共才14，才总耗费34ms。
1. 按照上面程序，每次FullGC都是有YGC触发的，因为YGC过后存活对象太多要放老年代，老年代内存不够触发FullGC。

**优化：**

   - 最大的问题就是每次Young GC过后存活对象太多了，导致频繁进入老年代，频繁触发Full GC。
   - 年轻代的内存空间，增加Survivor的内存即可。
   - 把Old：S1：S2的比例调节成 2:1:1 即可，扩大S区域的空间。
<a name="vFyEh"></a>
#### 问题与总结

- [ ] 模拟一下上述的场景，用工具分析一下呗？
- [ ] 能不能对财务系统的JVM参数做个优化？

<a name="fEyWY"></a>
### 41-第八周问题与答疑
<a name="aY2Gd"></a>
#### 笔记
TODO 跳过，后面复习的时候在看呗。
<a name="rUF55"></a>
#### 问题与总结

<a name="YkiLF"></a>
## 性能优化案例实战
<a name="UmwJ1"></a>
### 42-每秒10万QPS社交APP，GC如何优化，性能提升3倍
<a name="ytmz4"></a>
#### 笔记
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631010448249-5ff4538a-5875-4e00-9189-90a04c6279e4.png#clientId=u05b4f55d-fff5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=393&id=u92f901bb&margin=%5Bobject%20Object%5D&name=image.png&originHeight=786&originWidth=928&originalType=binary&ratio=1&rotation=0&showTitle=false&size=213827&status=done&style=none&taskId=uc79d0987-afe9-4e76-ac59-34e12404921&title=&width=464)<br />**背景：**

   - 并发高，通过jstat监控工具发现Eden区会倍迅速填满，并且频繁触发YGC。
   - 高峰期，经常出现YGC过后，存活对象过多，在S区放不下，导致对象进入老年代。
   - 老年代必然触发频繁GC，导致系统停顿，影响性能。

**优化：**

1. 增加机器，减少每台机器的并发请求，减少压力。
1. 给年轻代的S区分配更多的空间，让YGC后对象尽量存放在S区中，别进入老年代。，具体分配多少就用Jstat等工具，区估算一下，尽量做到一个小时一次FullGC。
1. 上面案例用的是CMS回收器，正常垃圾回收，使用标记-清理算法，所以必然导致大量的内存碎片，而他们设置的是5次之后才调整碎片，会导致这5次内有一些大对象存不下，会增加FGC的触发，所以需要设置成，每次FullGC后就调整碎片，增加真实可用堆空间。
<a name="d0xfk"></a>
#### 问题与总结

- [ ] 怎么把这种高并发场景应用到自己负责过的项目上面？

<a name="H7KoV"></a>
### 43-垂直电商APP后台，对FullGC进行深度优化？
<a name="pdavu"></a>
#### 笔记
**背景**<br />APP大致注册用户量有就数百万的规模，日活几十万。日请求也就小几千万。高峰期QPS几百。<br />**公司级别的JVM参数模板**<br />为团队或者公司定制一套基本的JVM参数模板，尽量让大部分系统套用，基本保证JVM性能别太差，避免很多初中级工程师直接使用默认的JVM参数。<br />**可以参考当时我定制出来的适合他们公司APP的JVM参数模板：**
```
-Xms4096M
-Xmx4096M 
-Xmn3072M 
-Xss1M  
-XX:PermSize=256M 
-XX:MaxPermSize=256M 
-XX:+UseParNewGC 
-XX:+UseConcMarkSweepGC 
-XX:CMSInitiatingOccupancyFaction=92 
-XX:+UseCMSCompactAtFullCollection 
-XX:CMSFullGCsBeforeCompaction=0
```
**为何如此定制参数呢？**

   - 8G的机器，堆内存分配4G就差不多了，可能别的进程还会使用。
   - 年轻代给3G，让每个S区都到300M左右。避免对象进入老年代。
   - 不同系统针对上面稍微调整，基本上都能扛住，保证FullGC不会过于频繁。
   - 加入Compaction相关的参数，保证每次FullGC后都会执行一次压缩，解决内存碎片问题。

**如何优化FullGC的性能？**

   - “-XX:+CMSParallelInitialMarkEnabled” CMS回收时“初始标记”开启多线程并发执行，减少STW停顿时间，
   - “-XX:+CMSScavengeBeforeRemark” CMS回收时“重新标记”阶段，先尽量执行一次YGC，回收掉一些年轻代没人引用的对象，在CMS重新标记阶段，可以少扫描一些对象，提高性能。
<a name="Vzpc0"></a>
#### 问题与总结

- [ ] 了解公司内部的JVM参数模板？
- [ ] 假如你是架构师，如何结合实际情况去定制一套JVM模板？
- [ ] 了解不同配置的机器，是如何配置机器的？（K8S容器又是如何设定的？）
- [ ] 有没有特例系统，是如何进行优化的？

<a name="cPfo3"></a>
### 44-新手不合理设置JVM参数，如何导致频繁FGC的？
<a name="LPF35"></a>
#### 笔记
**问题的产生背景：**<br />新手百度了一个参数，导致线上频繁FullGC。<br />**分析：**

- 可以通过一些命令，让Jstat自动对JVM进行监控，把监控结果输出到某个文件里，第二天自己去看。
- 查看GC日志，发现“Full GC（Metadata GC  Threshold）xxxxx”的日志记录，表示Metadata（永久代）区域频繁被塞满，触发了FullGC.
- 用Jstat查看永久代占用情况，发现是是波状图，到达一定值就降下被回收了。

**分析思路：**

1. 表示不停的有新类产生被加载到MetaSpace区域，然后占🈵️空间，触发FullGC,回收掉部分类。
1. 设置两个参数：“-XX:TraceClassLoading -XX:TraceClassUnloading” 就是追踪类加载和卸载的情况，通过日志记录哪些Class被加载，被卸载。
1. 明显可以看到，JVM在运行期间不停的加载了大量的所谓“GeneratedSerializationConstructorAccessor”类到了Metaspace区域里去。
1. 而这个类通常是软引用导致，进而排查JVM参数，新手竟然动了SoftRefLRUPolicyMSPerMB这个JVM启动参数，他直接设置为0了。导致任何软引用都会被释放掉。
1. 却本以为是想释放掉软引用的而扩大内存空间，结果导致刚创建出来就被YGC回收了，正常是要50分钟才会回收一次的。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631020394483-56084689-c447-4ecc-8883-2b0fd1ceed1d.png#clientId=u05b4f55d-fff5-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=394&id=u45e929e0&margin=%5Bobject%20Object%5D&name=image.png&originHeight=788&originWidth=1002&originalType=binary&ratio=1&rotation=0&showTitle=false&size=186644&status=done&style=none&taskId=ud23c9abe-8daa-4868-8307-f9255a1ac2d&title=&width=501)<br />**结论：**

- 有大量反射代码的场景下，大家只要把-XX:SoftRefLRUPolicyMSPerMB=0这个参数设置大一些即可，千万别设置为0，可以设置个1000，2000，3000，或者5000毫秒，都可以。
- 提高这个数值，就是让反射过程中JVM自动创建的软引用的一些类的Class对象不要被随便回收，当优化这个参数之后，就可以系统稳定运行了。
- 基本上Metaspace区域的内存占用是稳定的，不会来回大幅度波动了。
<a name="mplUI"></a>
#### 问题与总结

- [ ] 百度了解用Jstat对JVM进行监控，把结果已文件输出。
- [ ] <br />
- [ ] 在代码里大量用了类似上面的反射的东西，那么JVM就是会动态的去生成一些类放入Metaspace区域里的，这个时候要注意永久代的内存空间使用情况。
- [ ] 查询一下元空间是怎么进行垃圾回收的？

<a name="lLL9k"></a>
### 45-每天10+次FullGC导致频繁卡死的优化实战
<a name="obfzR"></a>
#### 笔记
**背景：**<br />真实案例，新系统，上线后每天的FullGC次数超过10次，甚至上百次，一般良好JVM的性能，是几天发生一次。所以需要调优。<br />**没优化前：**

   - 配置：2核4G，JVM堆2G，系统运行实践6天，系统运行6天内发生FullGC次数和耗时70+S，YoungGC次数和耗时2.6万次，1400秒。
   - JVM参数：
```
-Xms1536M -Xmx1536M -Xmn512M -Xss256K -XX:SurvivorRatio=5 -XX:+UseParNewGC -XX:+UseConcMarkSweepGC -
XX:CMSInitiatingOccupancyFraction=68 -XX:+CMSParallelRemarkEnabled -XX:+UseCMSInitiatingOccupancyOnly -
XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintHeapAtGC
```

      - 4G的机器上，堆设置1.5G，新生代512M，老年代1G。
      - 比较关键的是“-XX:SurvivorRatio”设置为了5，也就是说，Eden:Survivor1:Survivor2的比例是5:1:1
      - 所以此时Eden区域大致为365M，每个Survivor区域大致为70MB。
      - 这里有一个非常关键的参数，那就是“-XX:CMSInitiatingOccupancyFraction”参数设置为了68，一旦老年代内存占用达到68%，也就是大概有680MB左右的对象时，就会触发一次Full GC。

**推测出的内存GC模型**<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631514000861-a408a568-0bf9-45f1-80d3-5643bd4b1255.png#clientId=u8f17f207-826c-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=378&id=OQ7VE&margin=%5Bobject%20Object%5D&name=image.png&originHeight=756&originWidth=1602&originalType=binary&ratio=1&rotation=0&showTitle=false&size=297017&status=done&style=none&taskId=u81fa45c6-1a43-4776-9a58-488c7b5bb7b&title=&width=801)<br />结合分析图标观察，发现是每隔一段时间就老年代突然增加到了几百M，于是判读可能是有大对象进入，于是在老年代对象增多时，打印出Dump日志，分析里面的具体对象信息，找到了一个超大的数据对象，<br />通过分析，发现是有个傻逼select查询后面没有加Where条件，导致全部查的信息过多。<br />**解决：**<br />1.先让傻逼修改代码，实现分页查询，不要全部怼出来。<br />2.修改JVM参数，调整新生代的大小到600M，让每个S区域从70M提升到150M。<br />3.Old区比例超过65%就FullGC，调整到当内存到占比95%后才做FullGC<br />按照上面修改，基本上就是10天才发生一次FullGC，比之前好了很多。
<a name="XzUJe"></a>
#### 问题与总结

- [ ] 尝试一下自己画出一个GC时的内存模型？

<a name="DZOkn"></a>
### 46-电商大促活动下，严重的FullGC导致系统直接卡死的优化实战
<a name="Um1Kn"></a>
#### 笔记
背景：<br />在大促活动下，发现系统每秒都在FullGC,导致系统卡死。<br />**发现问题：**<br />  用jmap工具排查，发现新生代，老年代，永久代都占用内存很正常，不会触发GC.<br /> 	于是乎，代码里搜，是不是有人显示调用了“System.gc()”? 结果还真有傻逼这么干了。<br />**为什么这样干？**<br />这个傻逼，本是好心， 每次操作完大对象之后，都会显示调用GC，自认为释放掉一些内存。结果在正常场景下是看似没问题的，但是在大促场景下，就这种人为触发GC就出现了过于频繁，导致卡死。<br />**解决：**<br />把代码注视掉！！杜绝在代码里显示调用GC。<br />在JVM启动参数里面设置“-XX:+DisableExplicitGC”。// 这个参数意思是，禁止显示执行GC，不允许你通过代码来触发GC。
<a name="v1IDI"></a>
#### 问题与总结

- [ ] 在财务系统或者公司模板文档里加入这个禁止显示调用的参数！
- [ ] 去查查这个参数，还有没有别的作用，防止翻车哦！！

<a name="LXm69"></a>
### 47-第九周问题与答疑
<a name="Gp1pw"></a>
#### 笔记
TODO 后面复习在看。
<a name="LY6jV"></a>
#### 问题与总结

- [ ] 复习题，自己总结本周生产案例的问题定位、原因分析以及解决思路！

<a name="wyLIW"></a>
### 48-线上大促营销活动，导致的内存泄漏（OOM）和FullGC
<a name="pEcMz"></a>
#### 笔记
**线上故障场景：**<br />节日促销，发现系统CPU使用率飙升，无法处理请求，出现卡死现象。重启之后，发现CPU又出现飙高。<br />**排查：**<br />CPU飙高的常见两个场景：<br />1.系统里创建了大量的线程，同时并发运行，负载更重导致。<br />2.JVM在执行频繁的FullGC，耗费CPU资源，而且导致系统又短暂的停顿。<br />所以根据上面的场景，先通过监控平台观察FullGC频率是否正常。检查得出，发现几乎每分钟都会有FullGC.<br />**排查FullGC的原因？**

   - 频繁FullGC的问题，一般可能性有三个？

1.内存分配不合理，导致频繁进入老年代。<br />2.存在内存泄漏问题，内存驻留了大量的对象，塞满老年代，稍微有些对象进去就引发FullGC.<br />3.永久代的类过多，触发了FullGC.<br />4.人工执行：System.gc();

   - 检查得知，并不是内存不合理，也不是频繁进入老年代，永久代也正常使用，排除上面几个问题。
   - 怀疑是：老年代驻留大量的对象给塞满了。
   - 通过Jmap + jhat的组合分析内存对象，也可以通过MAT这个内存分析工具，去检查快照对象分布情况。
   - 通过工具，找到这些系统自己生产的大量对象，检查代码！！发现这些对象为啥没有被释放。
- 这就是典型的“内存泄漏”，系统创建了大量的对象占用内存，又不使用，还无法收回。
- **原因**是在系统里做了一个JVM本地缓存，但是没有限制缓存的大小，没有使用LRU之类的算法， 淘汰数据。
<a name="e1w13"></a>
#### 问题与总结

- [ ] **MAT堆快照分析工具**是如何使用的？了解一下，玩一下！！

<a name="G2AJZ"></a>
### 50-百万级数据误处理导致的频繁FullGC问题优化
<a name="JuHJZ"></a>
#### 笔记
**背景：**

- 这个系统是来进行大量的数据处理，然后提供数据给用户查看的
- 线上系统进行版本升级，收到反馈说前端页面无法访问，用户看到的全是空白和错误信息。
- 监控报警也是现实CPU负载高，机器都宕机了。

**CPU负载高**

   - 前面说过CPU负责高的两种情况，这里是因为FullGC频繁，每2分钟就会执行一次，每次耗时10S左右。

**FullGC频繁的原因分析**

   - 配置情况：4核CPU，20G的堆内存，10G给老年代，10G给年轻代，（其中S区域各1G，E区8G）。
   - jstat分析发现，E区大概1分钟就塞满，就会触发一次YGC,而且YGC过后有几个G的对象进入老年代。
   - 说明代码运行的时候生成大量对象，处理很慢，1分钟后很多对象还存活，所以导致每2分钟就会执行一次FullGC,每次都要处理10G的对象,很慢，且CPU占用高。

**解决方案分析：**<br />这案例，因为是发版本，肯定是由于新写的代码引起的！所以一定要先优化代码！<br />普通JVM参数修改难起作用，每块S区域已经很大了，GC也会很慢，每次还有几G存活去老年代。

**模拟场景，使用Dump快照分析内存**<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631526418444-cdef23a8-b41d-49a7-be77-878b038f786a.png#clientId=ud368ad80-43a8-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=282&id=u537c1965&margin=%5Bobject%20Object%5D&name=image.png&originHeight=564&originWidth=1310&originalType=binary&ratio=1&rotation=0&showTitle=false&size=420279&status=done&style=none&taskId=u9c6a1d89-4307-4c1a-b4a5-bf77c6a3ebe&title=&width=655)<br />在代码里创建10000个对象，然后进入阻塞状态，然后获取Dump快照文件。<br />获取JVM进程Pid,然后打印dump文件
```yaml
jps -l
// 得到PID
jmap -dump:live,format=b,file=dump.hprof pid 
打印出dump文件
```
使用MAT分析快照文件（这里具体怎么操作就不多BB了，自行百度更详细）<br />追踪线程执行堆栈，找到问题代码，发现是 String.spit()这里面有问题。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631527758807-53e85fb2-f0b2-4a10-b057-2ec421a44921.png#clientId=ud368ad80-43a8-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=670&id=ucf907f4b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1340&originWidth=2252&originalType=binary&ratio=1&rotation=0&showTitle=false&size=883008&status=done&style=none&taskId=ua2701825-928d-4dda-a50a-0872eb5c241&title=&width=1126)<br />所以导致字符串数量暴增几倍，几十倍，这就是频繁产生大量对象的原因！！！<br />**解决：**<br />针对代码上做优化！<br />总结：这种大数据量处理的系统，一次性加载过多到内存来，要小心内存泄漏，比较核心的思路是，开启多线程并发处理大量的数据，提升数据处理完毕的速度，这样避免YoungGC的时候还有过多对象存活。
<a name="LPH6n"></a>
#### 问题与总结

- [ ] 模拟一下上面Dump打印分析的过程？
- [ ] 仔细看看Spit函数里面是怎么实现的，有些什么坑？

<a name="a6uPv"></a>
## 阶段性复习
<a name="CxZPx"></a>
### 51-阶段性复习：JVM运行原理和GC搞懂了么？
<a name="ARYwj"></a>
#### 笔记
顺着思路，学习过往的知识，自己整理一下。

- JVM的内存区域划分，最核心的就是这么几块了：年轻代、老年代、Metaspace（也就是以前的永久代）。
- Eden区和S1，S2区默认比例为 8:1:1
- 一般创建对象都是在各种方法执行的，一旦方法运行完毕，局部变量的引用就成为了Eden区的垃圾对象。
- 一旦Eden区塞满，就会触发一次YoungGC。
- Young GC为复制算法，从GC Roots（方法的局部变量、类的静态变量）开始追踪，标记出来存活的对象。

**常见的4种对象时候进入老年代？**

1. 对象在年轻代躲过了15次GC
1. 对象过大，超过了阀值，直接进入老年代
1. 一次YoungGC之后，发现S区放不下，这批对象会直接进入老年代
1. 几次YGC之后，S区域的对象占比超过了50%，动态年龄判断，会把超过N年龄以上的对象都放老年代。

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631686710521-a15d5917-7c1d-4a45-b26a-97252b26a358.png#clientId=uea14c2db-bb90-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=347&id=uf99cad16&margin=%5Bobject%20Object%5D&name=image.png&originHeight=694&originWidth=1170&originalType=binary&ratio=1&rotation=0&showTitle=false&size=211998&status=done&style=none&taskId=ue1099ce2-62a8-4c5e-8e58-36975dc4f77&title=&width=585)<br />**老年代的GC是如何触发的？**<br />老年代一多，就会引发FullGC,FullGC必然会带着OldGC,一般也会跟着YGC，也会触发永久代的GC。<br />**FullGC触发的3个条件？**

1. 超过自身设置的内存占用阀值，比如设置超过92%
1. 执行YGC之前判断 老年代可用空间 是否大于历次YG后进入老年代的平均对象大小，那么就可能会在YGC之前，触发FullGC。
1. 如果YGC存活对象太多，S区放不下，就要进入老年代，这时有担保机制，如果老年代也放不下，就会触发FullGC,回收掉一批对象，再把这批对象放入老年代。

总而言之：就是老年代一旦发现空间不足，就会触发FullGC.

**正常情况下的系统是什么样子的呢？**<br />几分钟 乃至 几十分钟 触发一次YGC / 耗时几毫秒<br />几十分钟 乃至 几小时 触发FullGC  / 耗时几百毫秒以内  

<a name="Ygjft"></a>
#### 问题与总结

- [ ] 针对前面的好好复习一下，画流程图

<a name="mvjHD"></a>
### 52-JVM性能优化到底如何做？
<a name="V6S0z"></a>
#### 笔记
**一个新系统开发完后，如何设置JVM参数？**

- 一般每个公司都会有JVM参数模板，在模板的基础上改
- 先估算系统的每个核心借口每秒QPS多少，每次会产生多少对象，每个对象多大，每秒会用多少空间。能估算出来Eden区大概多长时间会占满。
- 通过上面估算，就可以合理的分配年轻代，老年代的空间，还有E区S区的比例
- 最后上线前进行压测，调整JVM参数

原则是：尽量让每次YGC后存活对象远远小于S区空间，避免频繁进入老年代FullGC。

**压测调整参数需要关注的指标**<br />用Jstat观察JVM运行的内存模型，得到如下指标：

- Eden区的对象增长速率多块？
- Young GC频率多高？
- 一次Young GC多长耗时？
- Young GC过后多少对象存活？
- 老年代的对象增长速率多高？
- Full GC频率多高？
- 一次Full GC耗时？

注意：线上系统的监控和优化，可以用Zabbix、Open-Falcon之类的工具来监控机器和JVM的运行。

**线上频繁FullGC的几种常见表现**

1. 高并发请求，处理数据量大，导致YGC频繁，每次YGC后存活对象太多，内存分配不合理，S区过小，对象频繁进入老年代，频繁出发FullGC。
1. 系统发生了内存泄漏，莫名创建大量的对象，无法回首   
1. MetaSpace 永久代用了反射等机制，加载类过多，触发FullGC. （永久代默认只有几十M，需要手动设置）
1. 代码里面误用了System.gc();

**解决方案**<br />如果是第一种就：调大S区即可<br />如果是第二，三种就：dump快照，找出大对象，优化代码。

<a name="HHzqJ"></a>
#### 问题与总结

- [ ] 找到自己公司的JVM模板，看有哪些地方需要优化的？

<a name="XLQoF"></a>
### 53-如何为面试准备JVM优化案例
<a name="JfDx8"></a>
#### 笔记
需要熟悉JVM中，内存模型，垃圾回收算法，垃圾判断算法，常见回收器，类加载等。<br />**如何应付面试中的优化问题？**<br />常见做法，归纳总结一套通用方法，写写看看，面试的时候说出来，最好结合场景去编。

**如果你的系统访问量暴增10-100倍，此时会不会频繁FullGC?**<br />**如果会的话，你怎么避免，发现，定位，分析，和解决问题？**

- 应该从问题 预判，发现，定位，分析，解决 这5个纬度去吹牛逼。
- 应该把FGC频繁的问题和业务系统结合起来，整理可能发生的性能问题，出一套解决方案
- 在面试的时候，结合自己的系统去吹一吹，说哪些情况可能发生，你准备咋做。

<br />
<a name="xHCmG"></a>
#### 问题与总结
真正的JVM优化，一般是：

   - 内存分配+垃圾回收器的选择（ParNew、CMS、G1），
   - 垃圾回收器的常见参数设置，
   - 代码层面的内存泄漏问题，

其实搞定这些问题，99%的JVM性能问题你都能搞定了！所以千万别瞎鸡吧设置奇怪的参数。

<a name="RVhYx"></a>
### 54-第10周问题与答疑
<a name="qVBrp"></a>
#### 笔记
TODO 后面复习在看
<a name="sXE7C"></a>
#### 问题与总结


<a name="cvOjj"></a>
## OOM翻车演练分析



<a name="ubP8S"></a>
### 55-程序员的梦魇，系统宕机，OOM内存溢出讲解
<a name="letZP"></a>
#### 笔记
什么是内存溢出（OutOfMemory）：简而言之就是你的容器塞不下了，就溢出了。<br />程序运行时会涉及到哪些容器？？<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631697357727-36fe4fe0-6e5c-4b38-afcf-6eb0dd414c5d.png#clientId=u0edf89e1-4f7a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=190&id=u0019f942&margin=%5Bobject%20Object%5D&name=image.png&originHeight=380&originWidth=1558&originalType=binary&ratio=1&rotation=0&showTitle=false&size=170265&status=done&style=none&taskId=uf2f7a5c8-c6db-401d-9b45-2493fccf96f&title=&width=779)<br />**哪些区域会出现内存溢出？**<br />MetaSpace方法区，里面存放类信息，内存默认几十M，空间有限制会溢出。<br />每个线程有个栈内存，每个方法里面有栈桢：默认是一个栈内存占1M，空间有限制会溢出的。<br />JVM存放对象的堆空间，也是有限制，会溢出。

<a name="UFxKy"></a>
#### 问题与总结

1. 要搞清楚
- [ ] 什么是内存溢出？
- [ ] 什么是内存泄漏？
- [ ] 什么事对象逃逸？

- [ ] 程序计数器为什么不回溢出呢？

<a name="UNriE"></a>
#### 
<a name="AOOkz"></a>
### 56-MetaSpace内存溢出-是因为类太多？
<a name="OgrAT"></a>
#### 笔记
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631697998204-dad16df2-6da4-4b56-a0a8-21c9fc97a97d.png#clientId=u0edf89e1-4f7a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=329&id=u5c19cafd&margin=%5Bobject%20Object%5D&name=image.png&originHeight=658&originWidth=1564&originalType=binary&ratio=1&rotation=0&showTitle=false&size=390056&status=done&style=none&taskId=u9fa00153-2193-46b0-a80d-3af21cfd4bc&title=&width=782)<br />**设置MetaSpace的两个参数**

- -XX:MetaspaceSize=512m 
- -XX:MaxMetaspaceSize=512m

一旦MetaSpace不足，就会引发FullGC,一旦频繁就会CPU飙高，一旦回收不过来，存不下就会溢出。<br />**类信息要如何才能被回收？**<br />是一个很严苛的过程，总是回收不了多少东西。比如：<br />1.这个类所有对象实例要被回收<br />2.这个类的类加载器要被回收掉？

**什么情况下会发生MetaSpace的内存溢出？**

1. 没经验的程序狗，没有合理设置MetaSpace空间大小，直接用默认的才几十M，导致不足溢出。
1. 代码里用了cglib之类的技术，动态生成一些代理类，导致类信息过多，引发溢出

**溢出演示请看第59节**
<a name="rXa5n"></a>
#### 问题与总结

- [ ] 百度一下，那种热部署工具，会涉及到类加载器，这个会不会回收掉？
- [ ] 具体百度一下，类信息要如何才能被回收掉？
- [ ] MeataSpace默认到底是多少M，自己去看看？

<a name="QFiHZ"></a>
### 57-栈内存溢出，无限制的调用方法
<a name="GdDLd"></a>
#### 笔记
切记：<br />一个线程对应一个栈，一个栈默认占用1M，里面的每个栈贞也会占用几百个字节。<br />一般来说，栈128KB，就足够正常用了，只要没有递归骚操作。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631698906889-5a1cfc74-41c7-4e83-b5d5-73f70d4a5409.png#clientId=u0edf89e1-4f7a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=83&id=u2e48c42c&margin=%5Bobject%20Object%5D&name=image.png&originHeight=166&originWidth=912&originalType=binary&ratio=1&rotation=0&showTitle=false&size=160315&status=done&style=none&taskId=u8a0902fb-26ee-46f4-8c0d-2b84401b59f&title=&width=456)<br />递归用法，可能导致无限的栈贞压入，出现OOM<br />**溢出演示请看第60节**
<a name="JZbQI"></a>
#### 问题与总结

- [ ] 为什么高并发的系统里面要降低线程栈的默认空间？
- [ ] 了解一下递归的几种实现方式？和存在的其他问题。这里有个坑点的，暂时想不起来了。
<a name="Ep3QI"></a>
#### 
<a name="PjaRr"></a>
### 58-堆内存溢出，对象太多
<a name="Bj28I"></a>
#### 笔记
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631775567308-88177531-cb20-4270-b380-69c7b8cf489c.png#clientId=u8eaccf4c-4e06-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=1044&id=ub89eeff9&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1044&originWidth=1660&originalType=binary&ratio=1&rotation=0&showTitle=false&size=738561&status=done&style=none&taskId=ua42af7f6-8915-4137-934f-81f471069c1&title=&width=1660)<br />简而言之，有限的堆内存，里面即使GC之后还是会大部分存活，导致信对象无法进入，就只能发生内存溢出。<br />**堆内存发生溢出的两个场景**

1. 高并发请求，QPS高，导致大量对象存活，GC过后依旧存活很多对象，会引发OOM。
1. 系统发生泄露，莫名其妙出现很多对象，都没有释放，占用堆内存资源导致新的对象放不下，引发OOM。这个一般是代码层面上的问题。

**溢出演示请看第61节**
<a name="bdkxr"></a>
#### 问题与总结

- [ ] 还有做数据处理的时候，由于数据量过大也会引发OOM吧？
- [ ] 系统加载文件的时候，这个是已什么形式存在堆内存里的？

<a name="O8tzk"></a>
### 59-模拟:MetaSpace内存溢出
<a name="I5uvS"></a>
#### 笔记
上面说到，是MetaSpace溢出产生的一个场景，是因为程序不停的产生类，且还无法被回收。<br />到底什么是动态生类？<br />由程序，框架自己运行过程中产生的累，类似于cglib框架。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631776813695-5ec4859e-248c-49f0-b3d9-0a27e0d9ad42.png#clientId=u8eaccf4c-4e06-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=1110&id=ueb75e11f&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1110&originWidth=1976&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1514394&status=done&style=none&taskId=ub9ab3b26-3735-4a88-9b0e-3b1bb704602&title=&width=1976)

- 具体程序解释就不多BB了，反正这里死循环就会频繁创建大量的Class信息。
- 设置元空间的大小限制10M：-XX:MetaspaceSize=10m -XX:MaxMetaspaceSize=10m

**此时运行程序跑出异常。**<br />Caused by: java.lang.OutOfMemoryError: Metaspace。<br />这里显示的就是 元空间出现OOM溢出。
<a name="XOwLS"></a>
#### 问题与总结

- [ ] 去复习一下什么是动态代理？
- [ ] 复习一下cglib底层是怎么实现的？
- [ ] 什么是Java探针？
- [ ] 动态生成代理类有哪些方式？
- [ ] Spring框架里面哪些地方用到了代理对象？

<a name="XA73c"></a>
### 60-模拟:栈内存溢出
<a name="OSphK"></a>
#### 笔记
**背景：**

- 一般4核8G的配置，MetaSpace给512M，堆（新老）给4G，其他3G+給操作系统和其他Tomcat程序用，通常来说，我们设置每个线程栈1M，假设系统1000个线程，每个1M，就是1G空间，所以这套配置合理。
- 所以可能你分配的栈空间过大，你机器上的线程数量就会变少。

**溢出的原理**<br />就是栈内存不够，有傻逼在里面写了死循环，不停的调用方法。或者生成变量超级多。<br />**复现：**

- 先设置栈的默认大小为1M，-XX:ThreadStackSize=1m
- 接着运行代码，发现调用方法到5675就出现异常了，这里可以记一下，1M，**大约6000个栈贞就溢出了**
- 异常日志：“java.lang.StackOverflowError”

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631777949514-8dc34d82-bd48-408b-9dd9-cbd980c3181d.png#clientId=u8eaccf4c-4e06-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=454&id=ffp8S&margin=%5Bobject%20Object%5D&name=image.png&originHeight=454&originWidth=1254&originalType=binary&ratio=1&rotation=0&showTitle=false&size=393662&status=done&style=none&taskId=u35e1e8d3-3e14-4df3-af35-e777b279d74&title=&width=1254)
<a name="ku5NA"></a>
#### 问题与总结

- [ ] 反思：机器上的线程是无限产生吗？最多能有多少个线程？
- [ ] 反思：栈贞里面是些什么鬼？也会溢出吗？
- [ ] 如果局部变量超级多，也会溢出么？

<a name="gjPSK"></a>
### 61-模拟:堆内存溢出
<a name="xlBLC"></a>
#### 笔记
模拟一个比较普版的现象，系统负载过高，并发过大，或数量过大，出现内存溢出，崩溃。<br />先设置堆空间：-Xms10m -Xmx10m<br />运行后，创建了第360145个出现了如下异常：<br />Exception in thread "main" java.lang.OutOfMemoryError: Java heap space<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631781253147-abba65f9-7248-41ee-9c41-b75ae4058d56.png#clientId=u8eaccf4c-4e06-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=536&id=u60a946e6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=536&originWidth=1378&originalType=binary&ratio=1&rotation=0&showTitle=false&size=404801&status=done&style=none&taskId=uea2bd24e-9be7-41b9-a87b-683ac02af06&title=&width=1378)
<a name="V46CK"></a>
#### 问题与总结

- [ ] 自己去模拟一下这种溢出的场景，最好结合Jstat,Dump日志去观察现象。


<a name="pDO69"></a>
## OOM案例实战
上一单元讲述了什么事内存溢出，JVM中堆，栈，元空间是怎么出现内存溢出的，并且模拟了一下发生OOM的代码，这章节主要讲述线上真实OOM案例，希望你能够在里面吸取经验。


<a name="fYtJm"></a>
### 62-超大数据处理系统如何不负重堪OOM的？
<a name="bEFwj"></a>
#### 笔记
**背景**<br />数据量处理PB级的计算引擎系统，这里会解释当时线上OOM的问题，前面章节也对这个系统分析过GC问题。所以系统背景就不多BB了。这个系统每次加载到数据后，内存里计算，然后推送给另一个系统。完整的流程图如下：![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631783279952-ad36bbf6-037c-47c8-be83-67ef617b8547.png#clientId=u8eaccf4c-4e06-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=570&id=jZM5M&margin=%5Bobject%20Object%5D&name=image.png&originHeight=570&originWidth=1302&originalType=binary&ratio=1&rotation=0&showTitle=false&size=119243&status=done&style=none&taskId=ud161b609-204e-45f5-8bf9-34db12fc4b1&title=&width=1302)<br />上面系统里面有个设计，就是当Kafka推送失败后，会不断重试，所以当Kafka长时间宕机之后，就会把数据存在内存里，不断重试，导致对象无法释放，出现OOM。<br />临时解决方案：取消重试机制，直接丢掉本地计算结果，释放内存<br />后面解决方案：一旦出现KafKa故障，在有限重试次数之后，就把结果写到本地磁盘中，后续再异常处理。
<a name="FTbO3"></a>
#### 问题与总结
这个案例最主要的问题就是，在那种极端情况下，导致的异常现象，以后开发中要注意中间价的不可用问题。和服务的熔断降级问题。

<a name="qFQ2N"></a>
### 63-两个傻雕新手误写代码导致的OOM
<a name="cG4VG"></a>
#### 笔记
**第一个：无限循环调用，ES异常，导致栈溢出**<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631863445337-9cd7cd99-d99d-4950-abff-a690af2abd62.png#clientId=ue7210b96-b87d-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=359&id=u644fd01b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=718&originWidth=582&originalType=binary&ratio=1&rotation=0&showTitle=false&size=294317&status=done&style=none&taskId=ueefe7d17-524a-440f-a58c-3bb48d69b79&title=&width=291)<br />当出现ES异常时，会进入catch里面，导致反反复复的log()函数循环调用，出现OOM。<br />解决：

1. 加强同事之间的CoderReview.
1. 增加代码的配套单元测试，在单元测试+集成测试中，针对catch情况做一些测试。

**第二个：没有缓存的动态代理，导致MetaSpace溢出**<br />背景：<br />一个分布式事务框架，里面要对某个类生成一个动态代理类，对那个类的一些方法做额外的处理。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631864716049-6ef845c4-dfe7-499e-8cdb-d02dd1a9950f.png#clientId=ue7210b96-b87d-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=422&id=ua40d60b7&margin=%5Bobject%20Object%5D&name=image.png&originHeight=844&originWidth=2116&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1366017&status=done&style=none&taskId=ue5db4c7e-6b36-4598-ac55-1c21e9d98c1&title=&width=1058)<br />其实这里**可以对Enhancer 对象做一个缓存的**，没必要每次都创建。类似下面这样。。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631864874121-d8f62f64-65f0-47ed-b198-fea11d185eb0.png#clientId=ue7210b96-b87d-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=519&id=uf16aeb26&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1038&originWidth=1476&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1126281&status=done&style=none&taskId=uca9c7800-9416-4972-9247-1e8d3ad6349&title=&width=738)<br />然鹅。这个工程师每次都创建，没有做缓存处理，在某次负载很高的情况下，直接导致元空间内存溢出了。<br />所以：在用到反射，动态代理等场景的时候，千万注意一下缓存问题，和MetaSpace溢出的问题。
<a name="WjeB4"></a>
#### 问题与总结

- [ ] 注意复习一下，反射，动态代理相关知识？
- [ ] 百度一下在用动态代理的时候，要注意哪些坑？

<a name="ZKtFq"></a>
### 64-Tomcat出现的OOM问题和监控报警使用
<a name="pA2Jr"></a>
#### 笔记
**如何对线上的OOM异常进行监控和报警？**<br />公司最好是应该有一种监控平台，比如Zabbix、Open-Falcon之类的监控平台。<br />如何发现OOM：<br />1.线上监控平台提醒<br />2.客服通知你

**如何在JVM溢出的时候自动Dump内存快照？**<br />JVM启动的时候配置如下参数：

- -XX:+HeapDumpOnOutOfMemoryError   // OOM出来的时候自动Dump内存快照
- -XX:HeapDumpPath=/usr/local/app/oom   // 把Dump文件存储放到哪

**模拟MetaSpace内存溢出，如何用MAT工具去分析Dump日志**

- 具体里面怎么玩的就不多BB了，前面有GC日志分析章节，去复习复习。

**栈内存溢出时，如何解决？**

- 栈溢出的时候看，GC日志是没有用的，这里和GC日志是没有关系的。
- 内存快照主要是分析内存占用的，同样是针对堆内存和Metaspace的，对线程的栈内存而言，没有用。
- 这里主要看异常日志，去判断是哪个方法出了问题。

**每秒上百请求的系统为什么会OOM？**<br />**简单看一下Tomcat的原理，和本次导致OOM的流程图**<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631879296898-5c7a7dc3-9bc5-480a-a6cc-879c4c5a12b0.png#clientId=ue7210b96-b87d-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=406&id=u9247b03a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=812&originWidth=1770&originalType=binary&ratio=1&rotation=0&showTitle=false&size=663616&status=done&style=none&taskId=ufd005f9f-1f19-40d9-aa21-bc689ab0444&title=&width=885)<br />大概问题就是，发现有OOM，然后发现堆内存出现大量的数组，导致堆溢出，通过MAT工具发现这些数组大小都是10M，排查所属类发现，是因为Tomcat自身创建的线程，设置了10M。<br />另一边发现QPS才100M，但是发现每秒钟进来400个请求，于是导致400*10M瞬间把内存占用了。<br />那为啥是400个请求呢？后来发现代码里通过RPC调用，发现有请求超时时间设置为4S，于是当下游系统出现异常时，会占用4秒时间，导致1秒内出现了400个工作线程。<br />解决：设置Tomcat设置的线程大小，设置超时时间短一点。<br />**总结：针对这种比较复杂的OOM情况，要多方面分析，多方面挖掘**

<a name="wup7B"></a>
#### 问题与总结

- [ ] 去了解一下MAT是怎么玩的？
- [ ] 如果是栈溢出，会导致JVM服务宕机嘛？还是受影响的只有当前线程。

<a name="QnSYN"></a>
### 65-Jetty服务器的NIO机制导致 堆外内存溢出？
<a name="YQS0A"></a>
#### 笔记
背景：<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631880520730-e883e5be-9291-4717-892a-09a79aea077a.png#clientId=ue7210b96-b87d-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=532&id=u30967cee&margin=%5Bobject%20Object%5D&name=image.png&originHeight=1064&originWidth=1528&originalType=binary&ratio=1&rotation=0&showTitle=false&size=1098336&status=done&style=none&taskId=ud6d24248-94c7-4acc-b817-46250731b5b&title=&width=764)

背景：<br /> 系统采用Jetty服务器作为Web容器，某天收到告警，服务突然无法访问，发现如下日志：
```yaml
nio handle failed java.lang.OutOfMemoryError: Direct buffer memory
at org.eclipse.jetty.io.nio.xxxx
at org.eclipse.jetty.io.nio.xxxx
at org.eclipse.jetty.io.nio.xxxx
```
发现出现Direct buffer memory （堆外内存）区域出现OOM，下面显示一堆Jetty的调用函数。<br />**初步分析：**<br />Direct buffer memory 英文翻译叫直接内存，我们称之为** 堆外内存**，这块内存不属于JVM管理，所以这里明确一点，这次OOM是Jetty这个Web容器在使用堆外内存的时候导致的。

**题外话：关于解决OOM问题的底层技术修为建议**<br />希望在学习JVM本身的之余，多去其他核心主流技术做一些深层次的研究，有利于提高分析解决问题能力

**堆外内存是如何申请，又如何释放的？**<br />堆外内存是通过DirectByteBuffer这个类来实现的，可以在代码里创建一个DirectByteBuffer对象，这个对象本身是在JVM堆里的，但是也会在 堆外内存 划分一块内存空间跟这个对象关联起来，堆外内存是直接由操作系统管理的。<br />所以，当你的DirectByteBuffer对象无人引用时，就变成垃圾，自然在YGC或者FGC的时候被回收。

**为什么会出现堆外内存溢出的情况？**<br />🈶️两种可能<br />第一种：高并发情况下，请求过多，导致对象来不及释放就溢出了。（实际情况并非不高，排除这种情况）<br />第二种：用Jstat工具观察实际运行情况，发现一些请求的耗时比较长，平均1S多时间返回。<br />然后通过Jstat发现，Jetty不停的创建DBBuffer对象，且不停申请堆外空间，接着Eden区满了，就会触发YGC,很多请求还没处理完毕，DBBF对象处于存活状态，不能被回收满，进入了S区。<br />恰巧，线上S区分配极不合理，老年代几百M，年轻代只有100M，其中S区只有10M，往往YGC过后一些DBBuffer对象都超过了10M，无法进入S区，直接YGC后进入了老年代。<br />而此时，老年代有几百M，长时间下来，DBBuffer对象一直在老年代存活着，远远没有达到FullGC的条件，而GC回收器也无法感知到堆外空间的局限，反反复复下来，堆外内存一直没有释放，最终导致了堆外OOM。

**难道JavaNIO没考虑过这个问题嘛？**<br />在Jetty源码里，其实有考虑到，可能会有很多DBBuffer对象无人使用，就主动调用了System.gc(), 提醒主动回收。   <br />但是我们在JVM参数设置了-XX:+DisableExplicitGC，导致无法用代码显示调用GC回收。于是出现了上么的这种情况。

**解决方案：**<br />合理分配S区域的大小<br />把-XX:+DisableExplicitGC关掉，让他可以手动触发FGC。

<a name="fzvZB"></a>
#### 问题与总结
**注意：**Jetty里面针对使用堆外内存时，会主动调用System.gc()，这可能也是个坑，这是目前看到框架里面 唯一一个主动去调GC回收的框架，这样会不会导致系统频繁触发FGC，和之前🈲️用GC调用的案例相冲突。

- [ ] 复习一下System.gc()相关内容？
- [ ] System.gc()是触发的YGC还是FGC？
- [ ] 触发的是FGC，通常FGC也会顺带一次YGC。但是为啥会顺带触发一次YGC，就要看前面复习一下了？
- [ ] 了解一下什么是堆外内存 里面有什么不同，最大限制有多少？
- [ ] 堆外内存是由操作系统怎么回收的，里面有什么回收器嘛？
- [ ] DirectByteBuffer对象里面有些什么属性，里面是什么结构？

<a name="VojQ1"></a>
### 66-微服务架构下RPC引发的OOM故障排查实践
<a name="DGvI0"></a>
#### 笔记
![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631884330672-a0583e84-2f4b-4590-af6b-36543792cd4e.png#clientId=uddc2d75e-0b4e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=396&id=ub0889507&margin=%5Bobject%20Object%5D&name=image.png&originHeight=792&originWidth=1500&originalType=binary&ratio=1&rotation=0&showTitle=false&size=651394&status=done&style=none&taskId=u347ecc2f-3a8b-4f6c-bda4-c9551ccec33&title=&width=750)<br />**背景：**<br />一次奇怪故障案例，基于Thrift框架自己封装的RPC框架。系统A上线后没过多久导致系统B就OOM了。<br />**分析：**

- 因为系统A对发送的类做了修改，系统B又没有及时更新最新版本，导致序列化解析对象的失败，类似与我们返回枚举结构，最终版本不一致出现报错的问题一样。
- 恰巧的是这个RPC框架，只要在序列化失败的时候，都会初始化一个4G的byte数组去装这个解析失败的对象，所以默认4G内存设置的过大，导致出现了OOM。

**解决：**

1. 先对系统B里面的引用版本做紧急升级，和系统A中的对象保持一致。
1. 后面对RPC框架里面装载错误对象信息的数组，从4G改成4M

<a name="f0TEs"></a>
#### 问题与总结

- [ ] 了解一下Thrift是什么鬼东西？

<a name="pZ03y"></a>
### 67-没加Where条件引发的OOM问题
<a name="H3EVU"></a>
#### 笔记
这章节主要介绍了MAT工具的使用，和其中Histogram按钮的使用，至于怎么使用就不多BB了，我建议直接自己动手去操作一波，可以结合线上的Dump分析网站区分析故障，找到方法。<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631884784964-d0a39e09-35ee-4dab-9bbc-28f58187dcbe.png#clientId=uddc2d75e-0b4e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=449&id=u8759c505&margin=%5Bobject%20Object%5D&name=image.png&originHeight=898&originWidth=1728&originalType=binary&ratio=1&rotation=0&showTitle=false&size=965221&status=done&style=none&taskId=uc710f525-558e-49a5-9a81-9e8dadf8d2f&title=&width=864)
<a name="DuL3f"></a>
#### 问题与总结

- [ ] 一定要自己去写个Demo,做一次故障排查演练，看看这些工具是怎么使用的。

<a name="JYHwn"></a>
### 68-第11周问题与答疑
<a name="c6iyh"></a>
#### 笔记
TODO 后面复习的时候再看
<a name="rk8cN"></a>
#### 问题与总结

<a name="CRFMm"></a>
### 69-OOM问题排查实践汇总
<a name="jleR9"></a>
#### 92-每天10亿数据的日志分析系统的OOM问题排查实践
**笔记**<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631885822020-56e27248-988a-44c9-96ca-12c18f75748b.png#clientId=uddc2d75e-0b4e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=455&id=u5ac8e0d6&margin=%5Bobject%20Object%5D&name=image.png&originHeight=910&originWidth=1348&originalType=binary&ratio=1&rotation=0&showTitle=false&size=510248&status=done&style=none&taskId=u4180dbf9-06dc-4811-a379-783cb2c5d76&title=&width=674)<br />第一个大概就是在消费MQ消息的时候，出现了OOM堆异常，看日志XXClass.process()是这个函数出现了递归使用。<br />于是用MAT分析内存快照，是里面创建的大量的char数组，检查JVM内存配置，发现只给了1G。<br />分析发现可能是：YGC过后存活对象好太多，无法进入S区，直接进入老年代，只能进行FGC。但是每次FGC只能回收少量对象，最后发生了新对象无法进入堆内存，出现了OOM。<br />**优化：**<br />1.修改JVM堆空间，扩大内存。<br />2.修改代码，不使用递归频繁创建数组，而是用字符切割的方式去实现。

<a name="bCMBb"></a>
#### 93-一次服务类加载器过多引发的OOM问题排查实践
**笔记**<br />**背景：**<br />发现服务假死，接口无法调用，并不是直接出现OOM。<br />**分析：**<br />下面用Top对机器资源使用情况做个分析，看各进程对CPU和内存的消耗情况。<br />**为啥要看这些呢？**<br />第一种问题，可能是服务使用了大量内存，无法释放，导致频繁GC问题，出现停顿。<br />第二种问题，CPU负载过高，导致你的服务无法获得CPU资源，而停顿。<br />检查发现，是这个服务消耗了50% 的进程，始终没有被GC回收。

**在内存使用这么高的情况下会发生什么呢？**

1. 频繁FGC,导致系统停顿。
1. 内存使用率高，JVM自己发生OOM？  （这里没说为啥自己会发生OOM，是什么原因导致的）
1. 内存使用率高，这个进程申请内存不足，直接被操作系统给杀掉。

**检查Dump快照，用MAT排查问题对象**

- 发现有大量ClassLoader对象,都是傻逼写的自定义类加载器，用来加载配置文件，和一些其他资源。
- 在代码里没有限制，大量的创建，导致内存占用。（这里没有说明，为啥自定义类加载器，就不能被回收？）

<a name="T5ibp"></a>
#### 94-一个数据同步系统频繁OOM内存溢出的排查实践
**笔记**<br />问题系统的流程图<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631886924516-c2a5e2a3-8a2d-46e8-bf22-7d95c661430c.png#clientId=uddc2d75e-0b4e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=412&id=ufaa6baba&margin=%5Bobject%20Object%5D&name=image.png&originHeight=824&originWidth=1462&originalType=binary&ratio=1&rotation=0&showTitle=false&size=357283&status=done&style=none&taskId=uca44f89e-ca49-45d8-aa47-9c63c3cd8bc&title=&width=731)<br />优化之后的流程图<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1631887063015-f11b26b0-cee4-45e3-a9f7-4c82edcd19c6.png#clientId=uddc2d75e-0b4e-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=421&id=u9fdfaf36&margin=%5Bobject%20Object%5D&name=image.png&originHeight=842&originWidth=1496&originalType=binary&ratio=1&rotation=0&showTitle=false&size=384140&status=done&style=none&taskId=u8c5e41c0-d673-496e-ac67-dd2b4c975f8&title=&width=748)<br />简而言之就是数据同步系统，开始每次MQ里消费100条数据，先以List的形式存在无界队列里，然后长久下来，导致消费无界队列的速度很慢，里面的内存一直无法释放，引发了OOM。<br />**优化：**<br />对上面的无界队列改成了  有界的阻塞队列，比如里面最多1024个数据，超过了就会停顿MQ的消费，不会让内存无限扩张。
<a name="KJGWl"></a>
#### 问题与总结

- [ ] 好好反思一下上面这些案例，找到分析问题的套路
- [ ] 复习一下那些队列的使用，怎么把这些递归场景优化，内存队列优化案例，套在财务系统里。

<a name="z7lJy"></a>
## 总结复习
<a name="a7lew"></a>
### 70-总结复习，准备面试问题

- [ ] **总结复习好上面的问题？**

在我们的这个专栏推出之前，往往有三个问题在面试的时候是几乎所有人都回答的非常不好的。<br />**针对下面这三个问题，自己编几个案例出来，用于应付面试。**

- [ ] 第一个是你们生产环境的系统的JVM参数怎么设置的？为什么要这么设置？
- [ ] 还有一个是你在生产环境中的JVM优化经验可以聊聊？
- [ ] 另外一个是说说你在生产环境解决过的JVM OOM问题？

去网上搜罗相关的JVM面试题，最少10个不太相同的问题，解答，并且记录在下面<br />。。。。 TODO 。。。。











