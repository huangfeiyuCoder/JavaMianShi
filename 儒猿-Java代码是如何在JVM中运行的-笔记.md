<a name="HQ98L"></a>
## 概要
本次分享会先从⼀个java⽂件开始，⼀路跟踪它的编译、类加载器加载、以及在JVM中被字节码执⾏引擎执⾏， <br />了解执⾏时是如何与JVM中的各种内存结构交互的。 <br />**先思考以下问题：**

1. 在JVM层面代码是如何运行的？
1. Java代码可以变异成class信息，也可以反编译窃取信息，可以采用哪些安全措施？
1. 什么时候回加载一个类？什么时候会初始化一个类？
1. 如何自定义一个类加载器？
1. JVM中的class字节码对象什么时候才能被卸载？

<a name="BxA6p"></a>
## 1.java运行代码编译
一行代码加载，JVM运行流程图如下：<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1632655356830-a3cbe924-ce7a-42bc-b4bb-ce795ccbdf58.png#clientId=u588b93f8-3e3a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=446&id=ueefaaf66&margin=%5Bobject%20Object%5D&name=image.png&originHeight=891&originWidth=841&originalType=binary&ratio=1&rotation=0&showTitle=false&size=495800&status=done&style=none&taskId=udc562523-8c30-4846-a6f5-e0c8c4556dd&title=&width=420.5)
<a name="bSLUr"></a>
## 2.Java代码是如何运⾏起来的 
我们在⽇常项⽬开发过程中写的java代码，都是以.java后缀的⽂件，然后编译器如 Eclipse或 IDEA就会⾃动帮我们将java代码编译成以.class为后缀的字节码⽂件，字节码⽂件才是JVM可以读取的 ⽂件。当整个项⽬都开发完成了之后，接下来就要将项⽬中的java代码编译打包成jar或war，然后通过jar -jar命令或tomcat等web容器部署启动项⽬了。 <br />当然不管是jar -jar命令启动还是web容器启动，其实效果都是⼀样的，都会启动⼀个JVM进程，然 后代码的各种逻辑的执⾏都会在这个JVM进程中开展，如下图所示：<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1632655613573-fa91df85-d2c1-402b-9df3-5d43af762806.png#clientId=u588b93f8-3e3a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=181&id=uf4f2929a&margin=%5Bobject%20Object%5D&name=image.png&originHeight=362&originWidth=164&originalType=binary&ratio=1&rotation=0&showTitle=false&size=63403&status=done&style=none&taskId=u23d9e0ac-fa20-4da8-9134-8b0bb4b4a4f&title=&width=82)

<a name="udpvj"></a>
## 3.类加载器⼯作原理 
java代码通过编译打包后⽣成jar/war包，之所以能被JVM处理的，就是jar/war包中的java代码已经被编译为.class后缀的字节码⽂件，但是问题来了，.class后缀的⽂件也只是个⽂件啊，它是如何进⼊到JVM进程中的呢，我们继续看。 
<a name="fN6J0"></a>
### 3.1 类加载器是如何加载⼀个类的
为将class⽂件加载到JVM中，就要⽤到类加载器了，类加载器说⽩了就是实现了将.class后缀⽂件加载到JVM功能的的代码组件⽽已，**类加载器加载⼀个类分为以下五个步骤：** 

1. 加载：通过类的全限定名、获取字节码⽂件的⼆进制字节流并加载到JVM内存中; 
1. 验证：处理前需要检查下字节流中的信息是否符合JVM规范、字节码⽂件内容是否被篡改等检查操作; 
1. 准备：该阶段主要为类分配内存，并为⼀些静态的变量设置默认值，如类的静态变量如果是int类型，暂时先分配0值，如果是引⽤类型则暂时分配null值； 
1. 解析：这⾥主要是将符号引⽤转换为直接引⽤，直接引⽤就是直接指向引⽤对象的内存地址了; 
1. 初始化：初始化阶段就是准备阶段的续期了，准备阶段主要完成了前两步：为类和静态变量分配内存、设置静态变量的默认值，但是实际值的赋值还得初始化阶段完成，⽐如int类型值只有现在才能被复制，⽽静态变量的引⽤类型变量在该阶段后才不会为null； 经过以上五个阶段后，才在JVM中创建了⼀个可⽤的class字节码对象，如下图所示：

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1632655866895-0db19738-542a-48d5-88ad-8b7a557b4ab6.png#clientId=u588b93f8-3e3a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=205&id=u9d5aa5a2&margin=%5Bobject%20Object%5D&name=image.png&originHeight=410&originWidth=524&originalType=binary&ratio=1&rotation=0&showTitle=false&size=81892&status=done&style=none&taskId=uac974fe6-1c5f-46eb-8731-baa43371eb6&title=&width=262)

<a name="x54Zq"></a>
### 3.2类加载器的双亲委派机制
下类加载器的双亲委派机制，java中有**三种类加载器**，分别是： 

1. **启动类加载器 **
1. **扩展类加载器 **
1. **应⽤类加载器 **
- 其中启动类加载器和扩展类加载器都是负责加载JDK⾃带的⼀些基础组件，⽽应⽤类加载器就负责加载我们⾃⼰的写的那些代码。 
- 双亲委派模式，特点在于委派⽅式，当有⼀个类需要加载时，先委派给应⽤类加载器；
- 应⽤类加载器就委派给⽗类扩展类加载器，⽽扩展类加载器也毫不犹疑委派给了它的⽗类 启动类加载器加载； 
- 现在启动类加载器已经没有⽗类加载器了，只能⾃⼰加载。 
- 启动类加载器如果⾃⼰能够加载就成功加载了，⽐如java.lang.Object这种JDK底层对象，如果是我们⾃⼰写的代码，不管是启动类加载器还是扩展类加载器当然都是加载不了的，此时就会⼀层⼀层就向下返回给我们的应⽤类加载器加载，如下图所示：
- ![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1632656173181-6936a74f-954c-46d8-a994-4d8f6efd7b0e.png#clientId=u588b93f8-3e3a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=329&id=uaeebde18&margin=%5Bobject%20Object%5D&name=image.png&originHeight=657&originWidth=1154&originalType=binary&ratio=1&rotation=0&showTitle=false&size=447720&status=done&style=none&taskId=u7ffff4f5-7cab-4040-a533-a288332654c&title=&width=577)
- 以上的这种类加载器层⾯委派的模式就是双亲委派机制，它可以很好的保证不同类加载器加载的内容不会混乱、各⾃加载各⾃的。 

- 如归属启动类加载器加载的类，应⽤程序类加载器就不会被重复加载，假如有⼈在⾃⼰项⽬中写了⼀个java.lang.Object对象，想要修改JDK底层的Object对象是⽆法被加载的：

- 当应⽤类加载器加载java.lang.Object时，⾸先就会⼀路向上委派到启动类加载器中，启动类加载器检查发现该类是⾃⼰负责加载的，但是因为启动类加载器⼀开始就已经加载了⼀个java.lang.Object的类， 同⼀个类加载器、对于同⼀个类的全限定名的类只会加载⼀次，所以启动类加载器就不会重复加载了；所以通过这种双亲委派模式，可以让各个类加载器各司其职、各⾃负责加载的范围不会交叉跨越。 

<a name="PPfZa"></a>
## 4.JVM中的内存数据结构
通过类加载器的加载，此时class字节码⽂件，已经以class字节码对象的形式给加载到了JVM内存中了，那字节码对象存放在JVM内存的什么位置呢，我们继续看下。 
<a name="kL8gZ"></a>
### 4.1 ⽅法区 
存放类相关的信息的区域在JVM称为⽅法区，⽅法区同时也是常量池的所在地，如下图所示：<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1632656301450-c2e6ecd7-bc82-4762-a461-ad61fc38849e.png#clientId=u588b93f8-3e3a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=357&id=ued220908&margin=%5Bobject%20Object%5D&name=image.png&originHeight=713&originWidth=1151&originalType=binary&ratio=1&rotation=0&showTitle=false&size=449144&status=done&style=none&taskId=u5ec6e99e-1788-4344-a68d-10a0e6b2e0f&title=&width=575.5)

<a name="DnAA9"></a>
### 4.2程序计数器
当我们将⼀个字节码对象给加载到⽅法区后，下⼀步就是使⽤它了，因为java⽂件中对应的都是⼀⾏⾏的java代码，JVM根本就识别不了，所以才编译成了class字节码⽂件，字节码⽂件加载到JVM内存后称为字节码对象，⽽字节码对象中的、就是JVM可以识别的⼀⾏⼀⾏字节码指令，⽽这些字节码的指令则是由JVM中的字节码执⾏引擎来执⾏的。 <br />在执⾏字节码指令时，体现到具体的代码上的操作、可能就是在多个地⽅new对象，然后执⾏类中的各种各样的⽅法，⽅法执⾏过中可能存在多线程的操作，毕竟同⼀个类，我在多个线程中同时new对象是再正常不过的⼀个场景了。 <br />由于java多线程的执⾏，每个线程底层是根据CPU分配给它的时间⽚的⽅式、依次轮流来执⾏的，可能A线程执⾏⼀段时间后就切换为B线程来执⾏了，B线程执⾏时间结束后，再切换回A线程执⾏了，此时线程A肯定要知道⾃⼰上⼀次执⾏到字节码指令的哪个位置了，才能在上次的位置继续执⾏下去。 <br />此时程序计数器就扮演了这样⼀个、记录每个线程执⾏字节码指令位置的⻆⾊： <br />程序计数器每个线程都是私有的，专⻔为各⾃线程记录线程、每次执⾏字节码指令的位置，⽅便下次线程切换回来时还能找的到上次执⾏的位置继续执⾏，如下图所示：<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1632656433503-c004ac7b-7f0d-481b-ae79-fc05e30f4daf.png#clientId=u588b93f8-3e3a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=393&id=ud14bea91&margin=%5Bobject%20Object%5D&name=image.png&originHeight=785&originWidth=922&originalType=binary&ratio=1&rotation=0&showTitle=false&size=388331&status=done&style=none&taskId=u556ccca3-e223-41e3-b09f-107a17d61c0&title=&width=461)
<a name="hWkr9"></a>
### 4.3 JVM栈 

- JVM栈和程序计数器⼀样，也是每个线程私有的； 
- 当字节码执⾏引擎开始执⾏字节码指令时，对应就是执⾏java类中的⼀个个⽅法的代码逻辑，每执⾏⼀个⽅法就会⽣成⼀个栈帧、压到JVM栈中，每个栈帧中包括局部变量表、操作数栈、动态链接以及⽅法出⼝等信息，如下图示： 
- 当⼀个⽅法执⾏完毕后就会将栈帧给弹出JVM栈，对应的栈帧中的数据也就销毁了。 

![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1632656482050-241be5dd-0ebb-4723-b4e8-42a0e9c536b7.png#clientId=u588b93f8-3e3a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=357&id=u3104ca42&margin=%5Bobject%20Object%5D&name=image.png&originHeight=714&originWidth=701&originalType=binary&ratio=1&rotation=0&showTitle=false&size=343110&status=done&style=none&taskId=ub395331b-7f63-4938-920d-c16e4e68a53&title=&width=350.5)

<a name="rBury"></a>
### 4.3堆内存
在⽅法执⾏时，可能我们就会执⾏new操作创建对象，创建出来对象的内存空间的分配，就要分配到另外⼀个JVM区域即堆内存，<br />如下图所示：<br />![image.png](https://cdn.nlark.com/yuque/0/2021/png/1461694/1632656565451-5d6a6e2d-2807-4cb6-9ccb-324119b25015.png#clientId=u588b93f8-3e3a-4&crop=0&crop=0&crop=1&crop=1&from=paste&height=442&id=u86d4a61b&margin=%5Bobject%20Object%5D&name=image.png&originHeight=883&originWidth=802&originalType=binary&ratio=1&rotation=0&showTitle=false&size=467736&status=done&style=none&taskId=udb3dcf95-fc1d-437e-a63b-433bd12533b&title=&width=401)  

-   Java堆内存空间和⽅法区⼀样，都是多线程共享的区域； 
-   当⽅法执⾏完毕后，栈帧就会从JVM栈中弹出，JVM栈中的局部变量表的变量，之前可能指向了堆内存的对象，此时堆内存的对象就没有被局部变量引⽤了、就成为垃圾对象，下⼀次gc时就会被垃圾回收器给回收掉。 
-   讲解到这⾥，⼀个java类如何通过编译、类加载器加载到JVM中，然后通过JVM中的字节码执⾏引 擎，配合着JVM中的各种内存区域，完成了整个类的代码逻辑执⾏，整个流程都已经梳理通了。 


<a name="sMVNL"></a>
## 5.面试题解析
<a name="yqlol"></a>
### 5.1.在JVM层⾯java代码是如何运⾏的？ 

- 先java代码由⼯具编译成class⽂件，后由类加载器通过加载、验证、准备、解析过程给加载到JVM中的⽅法区中。 
- 当字节码执⾏引擎执⾏到class字节码对象的指令时、如new操作等，此时就会触发class字节码对象的初始化。 
- 然后线程⼀边通过字节码执⾏引擎执⾏字节码指令、⼀边程序计数器记录着线程执⾏指令的位置， 每开始执⾏⼀个⽅法就会创建⼀个栈帧压⼊到JVM栈中，每次在⽅法中创建⼀个对象就会相应在JVM堆内存中开辟⼀块内存创建对象，由栈帧中局部变量表中的局部变量指向堆内存对象的地址。 
- ⽅法执⾏完毕，⽅法对应的栈帧出栈，同时JVM对堆内存的对象此时可能已经没有局部变量引⽤了，下⼀次gc时就可以回收堆内存对象了，整个过程就是java代码运⾏的轨迹了。 

<a name="UmW6i"></a>
### 5.2.代码可编译成class⽂件，也可反编译窃取信息，可采⽤哪些安全措施呢？ 
可以在java代码编译时，对编译过程进⾏加密和混淆处理，然后加载类的时候通过我们**⾃定义的类加载器进⾏解密操作**、然后对类进⾏针对性的加载。 

<a name="WO28y"></a>
### 5.3.什么时候会加载⼀个类呢？什么时候会初始化⼀个类呢？ 
加载⼀个类的时机⽐较简单，只要⽤到⼀个类时就会加载； <br />⽽类的初始化时机⼀般有以下四种时机： <br />（1）执⾏了⼀些特殊的字节码指令，如 getstatic、putstatic、invokestatic、new 这些字节码指令分别对应着：获取⼀个类的静态变量值、设置类的静态变量值、执⾏类的静态 ⽅法和创建⼀个类的对象操作，这些操作发⽣时就会触发⼀个类的初始化。 <br />（2）通过反射API操作⼀个类时，会触发类的初始化； <br />（3）初始化⼀个类时，该类的⽗类⼀定会⾸先被初始化； <br />（4）当启动⼀个JVM进程时，JVM优先会找⼀个含main⽅法的类进⾏初始化； 

<a name="xlKGt"></a>
### 5.4.如何⾃定义⼀个类加载器？ 
可以通过创建⼀个类继承抽象类ClassLoader，然后覆写它的loadClass⽅法、在⽅法中⾃定义类的 加载逻辑。 
<a name="fq1wq"></a>
### 5.5.JVM中class字节码对象什么时候才能被卸载？ 
（1）class字节码对象所有的对象实例都要被卸载回收; <br />（2）加载该字节码对象的类加载器同时也要被卸载; <br />（3）class字节码对象同时不能被其他对象引⽤; <br />满足上面条件，此时才能卸载class字节码对象。


<a name="Gx4pP"></a>
## 总结：

- <br />
-  
- <br />


