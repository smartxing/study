## jvm 内存模型
### 这些其实是老生常谈的事情了，这边就自己想到的知识点 ，网上搜一下太多了
![内存](https://github.com/smartxing/study/blob/master/image/vm.png)
 ```html
 1 新生代（YoungGeneration）：大多数对象在新生代中被创建，其中很多对象的生命周期很短。每次新生代的垃圾回收（又称Minor GC）后只有少量对象存活，所以选用复制算法，只需要少量的复制成本就可以完成回收。新生代内又分三个区：一个Eden区，两个Survivor区（一般而言），大部分对象在Eden区中生成。当Eden区满时，还存活的对象将被复制到两个Survivor区（中的一个）。当这个Survivor区满时，此区的存活且不满足“晋升”条件的对象将被复制到另外一个Survivor区。对象每经历一次Minor GC，年龄加1，达到“晋升年龄阈值”后，被放到老年代，这个过程也称为“晋升”。显然，“晋升年龄阈值”的大小直接影响着对象在新生代中的停留时间，在Serial和ParNew GC两种回收器中，“晋升年龄阈值”通过参数MaxTenuringThreshold设定，默认值为15。             
 2 老年代（OldGeneration）：在新生代中经历了N次垃圾回收后仍然存活的对象，就会被放到年老代，该区域中对象存活率高。老年代的垃圾回收（又称Major GC）通常使用“标记-清理”或“标记-整理”算法。整堆包括新生代和老年代的垃圾回收称为Full GC（HotSpot VM里，除了CMS之外，其它能收集老年代的GC都会同时收集整个GC堆，包括新生代）。               
 3 永久代（PermGeneration）：java8已经改成元空间(Metaspace) 主要存放元数据，例如Class、Method的元信息，与垃圾回收要回收的Java对象关系不大。相对于新生代和年老代来说，该区域的划分对垃圾回收影响比较小。              
```
## 类加载
### 类加载子系统
JVM的类加载是通过ClassLoader及其子类来完成的，类的层次关系和加载顺序可以由下图来描述：
![PNG](https://github.com/smartxing/study/blob/master/image/apploader.png)

```html
    1.Bootstrap ClassLoader负责加载$JAVA_HOME/jre/lib里所有的类库到内存，Bootstrap ClassLoader是JVM级别的，由C++实现，不是ClassLoader的子类，开发者也无法直接获取到启动类加载器的引用，所以不允许直接通过引用进行操作。

    2.Extension ClassLoader负责加载java平台中扩展功能的一些jar包，主要是由    sun.misc.Launcher$ExtClassLoader实现的，是一个java类，继承自URLClassLoader超类。它将负责%JRE_HOME/lib/ext目录下的jar和class加载到内存，开发者可以直接使用该加载器。

    3.App ClassLoader负责加载环境变量classpath中指定的jar包及目录中class到内存中，开发者也可以直接使用系统类加载器。

    4.Custom ClassLoader属于应用程序根据自身需要自定义的ClassLoader(一般为java.lang.ClassLoader的子类)在程序运行期间，通过java.lang.ClassLoader的子类动态加载class文件，体现java动态实时类装入特性，如tomcat、jboss都会根据j2ee规范自行实现ClassLoader。自定义ClassLoader在某些应用场景还是比较适用，特别是需要灵活地动态加载class的时候。
    全盘负责委托机制，自顶向下加载。每次只要一个class被loaded，系统的class loader就首先被调用。然而它不会立即去load这个这个类。取而代之的是，他会把这个task委托给他的parent class loader，也就是extension class loader;同样的，extension class loader也会委托给它的parent class loader也就是bootstrap class loader。
    因此，bootstrap class loader总是被给第一个去load class的机会。如果bootstrap class loader找不到类的话，那么extension class loader将会load，如果extension class loader也没有找到对应的类的话，system class loader将会执行这个task，如果system class loader也没有找到的话，java.lang.ClassNotFoundException将会被抛出。另外一个原因是避免了重复加载类，
    每一次都是从底向上检查类是否已经被加载，然后从顶向下加载类，保证每一个类只被加载一次。
```
加载顺序
![classloader](https://github.com/smartxing/study/blob/master/image/classloader.png)

## gc算法
```html
标记-清除算法
复制算法
标记压缩算法
分代回收算法

垃圾收集器
SerialGC    串行（Serial）回收器是单线程的一个回收器，简单、易实现、效率高。
ParallelGC  并行（ParNew）回收器是Serial的多线程版，可以充分的利用CPU资源，减少回收的时间。    
ParallelCMSThreads  吞吐量优先（Parallel Scavenge）回收器，侧重于吞吐量的控制。
G1GC                G1垃圾回收器
ConcMarkSweepGC  并发标记清除（CMS，Concurrent Mark Sweep）回收器是一种以获取最短回收停顿时间为目标的回收器，该回收器是基于“标记-清除”算法实现的               

```
##  jvm 参数
```html
-Xmx 最大堆大小
-Xms 最小堆大小
-Xmn 新生代 大小
-Xss 线程栈大小
-XX:NewRatio 设置年轻代（包括Eden和两个Survivor区）与年老代的比值（除去持久代）
-XX:SurvivorRatio   Eden区与Survivor区的大小比值， 设置为8,则两个Survivor区与一个Eden区的比值为2:8,一个Survivor区占整个年轻代的1/10
jdk8 后永久代变成元空间 会自动扩容 
-XX:MaxMetaspaceSize参数可以设置元空间的最大值，默认是没有上限的，也就是说你的系统内存上限是多少它就是多少。
-XX:MetaspaceSize选项指定的是元空间的初始大小


-XX:-OmitStackTraceInFastThrow  当大量抛出同样的异常的后，后面的异常输出将不打印堆栈
-XX:+PrintGCDetails 打印 GC 详情
-XX:+PrintGCDateStamps  打印 GC 时间戳
-XX:+UseGCLogFileRotation   GC 日志轮循
-XX:NumberOfGCLogFiles  GC 日志文件数
-XX:GCLogFileSize   文件大小
-XX:+HeapDumpOnOutOfMemoryError 当JVM发生OOM时，自动生成DUMP文件
-XX:HeapDumpPath=生成DUMP文件的路径
-XX:MaxDirectMemorySize来指定最大的堆外内存 (-Xmx的值里除去一个survivor的大小就是默认的堆外内存的大小了)

-XX:+UseSerialGC    串行垃圾回收器
-XX:+UseParallelGC  并行垃圾回收器
-XX:ParallelCMSThreads= 并发标记扫描垃圾回收器 =为使用的线程数量
-XX:+UseG1GC            G1垃圾回收器
常用 -XX:+UseConcMarkSweepGC 并发标记扫描垃圾回收器 -XX:CMSInitiatingOccupancyFraction=70 -XX:+ExplicitGCInvokesConcurrent  只有在cms下才生效，不是触发一个 完全stop-the-world的full GC，而是一次并发GC周期（注：一次并发周期其实就是在CMS下的一次gc,CMS只能用在full gc 中，所以，也是一次full gc 只不过效率比较高罢了）。


example：-Xmn1024m -Xms2500m -Xmx2500m -XX:+UseConcMarkSweepGC -XX:CMSInitiatingOccupancyFraction=70 -XX:MaxDirectMemorySize=256m -XX:+UseCMSInitiatingOccupancyOnly -XX:SurvivorRatio=8 -XX:+ExplicitGCInvokesConcurrent -XX:MetaspaceSize=128m -XX:MaxMetaspaceSize=256m -XX:-OmitStackTraceInFastThrow -XX:+PrintGCDetails -XX:+PrintGCDateStamps -Xloggc:/var/www/logs/gc-%%t.log -XX:+UseGCLogFileRotation -XX:NumberOfGCLogFiles=5 -XX:GCLogFileSize=10m -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/tmp/log -Djava.io.tmpdir=/tmp                                            

```
## gc日志分析
[understanding-cms-gc-logs](https://blogs.oracle.com/poonam/understanding-cms-gc-logs)

```html


82551.569:[GC [1 CMS-initial-mark: 2027280K(2516992K)] 2084513K(3088576K), 0.0344000secs] [Times: user=0.03 sys=0.01, real=0.03 secs]
第一阶段：初始标记阶段（Initial mark）标志着CMS收集老生代（Old Generation）的开始，所有从根部直接可达的对象会被标记，此时其他线程被阻断，这个阶段称为stop-the-world。此记录中，老生代的大小是2516992K，CMS在占用内存达到2027280K时触发，初始标记引起的pause time是0.0344s。 
82551.604:[CMS-concurrent-mark-start]
第二阶段：并发标记阶段（concurrent mark），所有第一个阶段被停掉的线程重新启动，此阶段内，从第一阶段标记对象出发所有间接可达的对象将被标记。
82553.887:[CMS-concurrent-mark: 2.272/2.283 secs] [Times: user=5.27 sys=0.14, real=2.29secs]
并发标记消耗2.272s CPU时间和2.283s 实际时间，实际时间包含CPU时间和阻断其他线程的时间。
82553.887:[CMS-concurrent-preclean-start]
第三阶段：并发预清理阶段（concurrent preclean），目的是减轻下一个阶段：重标记（remark）的工作量，因为预清理是并发的，而重标记是stop-the-world的，这样可以减小对其他线程的影响。此阶段内，收集器查看在并发标记过程中，CMS堆内得到晋升（promotion）的对象。
82553.928:[CMS-concurrent-preclean: 0.040/0.041 secs] [Times: user=0.04 sys=0.00,real=0.04 secs]
并发预清理消耗0.040s CPU时间和0.041s实际时间。
82553.929:[CMS-concurrent-abortable-preclean-start]
CMS: abort preclean due to time 82558.942: [CMS-concurrent-abortable-preclean: 1.311/5.014secs] [Times: user=1.41 sys=0.05, real=5.01 secs]
82558.959:[GC[YG occupancy: 338653 K (571584 K)]82558.959: [Rescan (parallel) , 0.3058990secs]82559.265: [weak refs processing, 0.0667480 secs]82559.332: [classunloading, 0.0868270 secs]82559.419: [scrub symbol & string tables,0.1176240 secs] [1 CMS-remark: 2027283K(2516992K)] 2365936K(3088576K),0.6602340 secs] [Times: user=4.88 sys=0.37, real=0.66 secs]
第四阶段：经过了并发预清理阶段，可中断预清理（abortable preclean）开始了（笔者不确定这么翻译是否合理）。从上面的记录可以看出，新生代的容量是571584K，新生代占有内存达到338653K时此预清理过程就被中断了，重标记阶段开始，由于重标记是stop-the-world的，所以其他线程被阻断。
第五阶段：重标记（remark）阶段，此阶段重新扫描CMS堆中剩余的且状态更新过的对象，重新从根部遍历，并且执行被引用的对象。这里，重扫描消耗0.3058990s，弱引用对象执行消耗0.667480s，重标记总体消耗了0.6602340s。
需要说明的是：如果新生代中Eden的占有内存达到了参数XX:CMSScheduleRemarkEdenSizeThreschold=<n>的值，可中断预清理就会启动，直到Eden占有内存达到参数XX:CMSScheduleRemarkEdenPenetration=<n>才会结束，所以说它是可以被中断的。如果它执行的时间超过了参数XX:CMSMaxAbortablePrecleanTime的值，无论如何也是会立即停止的。以上这些参数是为了限制预清理执行时间过长，但是预清理减轻了重标记的工作量。
82559.619:[CMS-concurrent-sweep-start]
第六阶段：并发清理阶段，重标记过后CMS开始清理没有标记的也就是已经死亡的对象。这一过程不会阻断其他进程。所用时间如下一条记录所示。
82560.976:[CMS-concurrent-sweep: 1.320/1.357 secs] [Times: user=1.76 sys=0.23, real=1.36secs]
82560.976:[CMS-concurrent-reset-start]
82561.000:[CMS-concurrent-reset: 0.024/0.024 secs] [Times: user=0.02 sys=0.00, real=0.02secs]
第七阶段：重置阶段，CMS内数据再一次初始化，进入下一个清理循环，重置消耗0.024s。
下面是两种清理错误的情况：promotion failed和concurrentmode failure。
250169.767:[GC 250169.767: [ParNew (promotion failed): 571584K->571584K(571584K),0.6487910 secs]250170.416: [CMS250173.050: [CMS-concurrent-mark: 2.887/3.777 secs][Times: user=10.86 sys=0.56, real=3.78 secs]
(concurrentmode failure): 2268975K->2111899K(2516992K), 8.3732150 secs]2766660K->2111899K(3088576K), [CMS Perm : 562899K->562896K(1048576K)],9.0223120 secs] [Times: user=9.78 sys=0.28, real=9.02 secs]
promotion failure表示从新生代晋升到老生代时发生了错误，因为老生代内存占用快满了，所以放不下晋升上来的对象。
有时promotion failure会引起concurrentmode failure，原因还是老生代内存不够用了，这样就引起了Full GC，也就是记录中的CMS Perm，Full GC是一个stop-the-world的过程。
```
![ygc](https://github.com/smartxing/study/blob/master/image/ygc.png)
![fullgc](https://github.com/smartxing/study/blob/master/image/fullgc.png)

## 基本策略
```html
jvm 调优 一般是最后的调优手段了
各分区的大小对GC的性能影响很大。如何将各分区调整到合适的大小，分析活跃数据的大小是很好的切入点。

活跃数据的大小是指，应用程序稳定运行时长期存活对象在堆中占用的空间大小，也就是Full GC后堆中老年代占用空间的大小。可以通过GC日志中Full GC之后老年代数据大小得出，比较准确的方法是在程序稳定后，多次获取GC数据，通过取平均值的方式计算活跃数据的大小。活跃数据和各分区之间的比例关系如下：

空间	倍数
总大小	3-4 倍活跃数据的大小
新生代	1-1.5 活跃数据的大小
老年代	2-3 倍活跃数据的大小
永久代	1.2-1.5 倍Full GC后的永久代空间占用
例如，根据GC日志获得老年代的活跃数据大小为300M，那么各分区大小可以设为：

总堆：1200MB = 300MB × 4
新生代：450MB = 300MB × 1.5
老年代： 750MB = 1200MB - 450MB*

这部分设置仅仅是堆大小的初始值，后面的优化中，可能会调整这些值，具体情况取决于应用程序的特性和需求。

```
## 常用排查命令
### jinfo:
查看Java进程的栈空间大小:               /jinfo - ThreadStackSize 14750
查看是否使用了压缩指针:                 /jinfo -flag UseCompressedOops 14750
查看系统属性:                          /jinfo -sysprops 14750
###  jstack: 
查看堆栈                              /bin/jstack -l 14750
###  [jstat](http://blog.csdn.net/zhaozheng7758/article/details/8623549):
查看gc的信息:                         /bin/jstat -gcutil 14750
###  jmap&mat
空间中各个年龄段的空间的使用情况:        /bin/jmap -heap 14750
jmap指定的dump                       /bin/jmap -dump:live,format=b,file=/var/www/dump.20131125.hprof 14750  (jmap -dump:format=b,file=<filename> <pid>)
### 其他
```html
最耗时cpu 线程查找 top
ps -mp 1904 -o THREAD,tid,time : 列出cpu最高的线程  / top -Hp pid
printf "%x\n" tid 转成16进制
jstack pid |grep tid -A 30 既可以找到cpu耗时最高的

[vmstat](https://www.cnblogs.com/ggjucheng/archive/2012/01/05/2312625.html) 是Virtual Meomory Statistics（虚拟内存统计）的缩写，可对操作系统的虚拟内存、进程、IO读写、CPU活动等进行监视。
常常用来查看上下文切换
```

### 杀手锏
[greys-anatomy](https://github.com/oldmanpushcart/greys-anatomy)



## 案例
[美团案例](https://tech.meituan.com/jvm_optimize.html) 把核心的东西都写在外面了
```html
1 增大新生代 减少对象复制时间
2 增大CMSScavengeBeforeRemark 由于跨代引用的存在，CMS在Remark阶段必须扫描整个堆，同时为了避免扫描时新生代有很多对象，增加了可中断的预清理阶段用来等待Minor GC的发生。只是该阶段有时间限制，如果超时等不到Minor GC，Remark时新生代仍然有很多对象，我们的调优策略是，通过参数强制Remark前进行一次Minor GC，从而降低Remark阶段的时间。
3 对于性能要求很高的服务，建议将MaxPermSize和MinPermSize设置成一致（JDK8开始，Perm区完全消失，转而使用元空间。而元空间是直接存在内存中，不在JVM中），Xms和Xmx也设置为相同，这样可以减少内存自动扩容和收缩带来的性能损失。虚拟机启动的时候就会把参数中所设定的内存全部化为私有，即使扩容前有一部分内存不会被用户代码用到，这部分内存在虚拟机中被标识为虚拟内存，也不会交给其他进程使用

结合上述GC优化案例做个总结：
首先再次声明，在进行GC优化之前，需要确认项目的架构和代码等已经没有优化空间。我们不能指望一个系统架构有缺陷或者代码层次优化没有穷尽的应用，通过GC优化令其性能达到一个质的飞跃。
其次，通过上述分析，可以看出虚拟机内部已有很多优化来保证应用的稳定运行，所以不要为了调优而调优，不当的调优可能适得其反。
最后，GC优化是一个系统而复杂的工作，没有万能的调优策略可以满足所有的性能指标。GC优化必须建立在我们深入理解各种垃圾回收器的基础上，才能有事半功倍的效果。
```
