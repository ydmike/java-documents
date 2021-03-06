#### 概念
将java代码编译成class文件并执行；
#### vm参数
- java -verbose:gc中参数-verbose:gc表示输出虚拟机中GC的详细情况.
- -XX:+PrintGC与-verbose:gc是一样的，可以认为-verbose:gc 是 -XX:+PrintGC的别名.
-XX:+PrintGCDetails 在启动脚本可以自动开启

-XX:+PrintGC , 如果在命令行使用jinfo开启的话，不会自动开启-XX:+PrintGC
- -Xmx5g：设置堆最大内存为5G

- -Xms5g：设置堆最小内存为5G，将最大和最小值设置一样，可以避免堆自动扩展，即垃圾回收后会重新分配堆内存空间，提高性能，一般也推荐这么做

- -Xmn2g：设置堆中的年轻代大小为2G。整个堆大小=年轻代大小+老年代大小+持久代大小。持久代一般固定位64M，所以增大年轻代后，将会减少老年代大小，当老年代内存用完会引发Full GC，相当严重。此值对系统性能影响较大，Sun官方推荐配置为整个堆的3/8

- -XX:SurvivorRatio=8：设置年轻代中Eden区与一个Survivor区的比例为8:1，默认为8

- -XX:NewRatio=2：设置老年代和年轻代比例大小2:1，默认为2

- -Xss128k：设置每个线程的栈大小。JDK5.0以后每个线程栈大小为1M，以前每个线程栈大小为256K。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000-5000左右

- -XX:PermSize -XX:MaxPermSize（JDK1.7）：设置永久代大小，jvm启动时，永久区一开始就占用了PermSize大小的空间，如果空间还不够，可以继续扩展，但是不能超过MaxPermSize，否则会OOM PermSiz space

- -XX:MetaspaceSize -XX:MaxMetaspaceSize（JDK1.8）：设置元空间的初始值和最大值

- -XX:+HeapDumpOnOutOfMemoryError –XX:HeapDumpPath=F:\ ：调试用，可以让虚拟机在出现溢出内存异常时Dump出当前的内存堆转储快照以便时候进行分析, HeapDumpPath是指定文件存放路径。

- -XX:MaxDirectMemorySize=10M：设置直接内存大小，如果不指定，则默认与Java堆最大值（-Xmx）一样。NIO操作会占用直接内存，因此大量的NIO操作可能引起直接内存溢出：Direct buffer memory

- -XX:PretenureSizeThreshold=3145728：表示超过3M的数据直接在老年代中保存

- -XX:MaxTenuringThreshold=30：设置年轻代中的对象存活多少次Minor GC后进入老年代。如果设置为0的话，则年轻代对象不经过Survivor区直接进入年老代。对于年老代比较多的应用，可以提高效率。如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象在年轻代的存活时间，增加在年轻代即被回收的概率。设置为30表示一个对象如果在Survivor空间移动30次还没有被回收就放入年老代。

- -XX:+UseParNewGC：设置年轻代为并行收集。可与CMS收集同时使用。JDK5.0以上，JVM会根据系统配置自行设置，所以无需再设置此值

- -XX:ParallelGCThreads=8：配置并行收集器的线程数，即：同时多少个线程一起进行垃圾回收。此值最好配置与处理器数目相等

- -XX:+PrintGCTimeStamps：打印GC停顿时间，一般测试用

- -XX:+PrintGCDetails：打印GC详细日志，一般测试用

- 如果满足下面的指标，则一般不需要进行GC优化：

    Minor GC 执行时间不到50ms
    Minor GC 执行不频繁，约10秒一次
    Full GC 执行时间不到1s
    Full GC 执行频率不算频繁，不低于10分钟1次

#### java内存区域
- 程序计数器
- java虚拟机栈
- 本地方法栈
- java堆
- 方法区
- 运行时常量池
- 直接内存
##### 程序计数器
- 记录当前程序正在执行的字节码行号的指示器。方法的跳转，异常会用到。
- 线程私有
- 可以看做当前线程所执行的字节码的行号指示器。字节码解释器工作时就是通过改变这个计数器的值来选取下一条需要执行的字节码指令，分支、循环、跳转、异常处理、线程恢复等都需要依赖这个计数器来完成。
- 此区域无内存溢出
#### java虚拟机栈
- 用来描述java方法执行的内存模型：java执行方法时，会创建一个栈帧，栈帧局部变量表，操作数栈，动态链接，方法入口等。方法从调用到完成的过程对应一个栈帧从入栈到出栈的过程。局部变量表所需内存空间在编译时确定，方法运行期间不会改变。
- 可能发生栈内存或堆内存溢出。
    - StackOverflowError:线程请求的栈深度大于虚拟机允许的深度
    - OutOfMemoryError:大部分虚拟机栈都是可扩展的，当无法申请足够的内存时，会发生堆内存溢出
#### 本地方法栈
和虚拟机栈类似，不过一个为java方法服务，一个为navtive方法服务。
#### java堆
存储对象的实例
#### 方法区
存储：
- 类信息
    - 类的版本
    - 字段
    - 方法
    - 接口
- 常量
- 静态变量
- 即时编译器即时编译的代码数据
hotspot虚拟机使用java堆的永久代来实现方法区。
#### 运行时常量池
运行时常量池是方法区的一部分。Class文件中除了有类的版本、字段、方法、接口等描述信息外，还有一项信息时常量池，用于存放编译期生成的各种字面量和符号引用，这部分内容将在类加载后进入方法区的运行时常量池中存放。

运行时常量池相对于Class文件常量池的另外一个重要特征是具备动态性，Java语言并不要求常量一定只有编译期才能产生，也就是并非预置入Class文件中常量池的内容才能进入方法区运行时常量池，运行时常量池，运行期间也可能将新的常量放入池中，这种特性被开发人员利用比较多的就是String类的intern()方法。

优点：
- 常量池是为了避免频繁的创建和销毁对象而影响系统性能，其实现了对象的共享。
例如字符串常量池，在编译阶段就把所有的字符串放到一个常量池中。
- （1）节省内存空间：常量池中所有相同的字符串常量被合并，只占用一个空间。
- （2）节省运行时间：比较字符串时，比equals()快。对于两个引用变量，只用 ""判断引用是否相等，也就可以判断实际值相等。

#### 堆外内存（直接内存）
    分配在Java虚拟机的堆以外的内存，这些内存直接受操作系统管理（而不是虚拟机），这样做的结果就是能够在一定程度上减少垃圾回收对应用程序造成的影响。使用未公开的Unsafe和NIO包下ByteBuffer来创建堆外内存。
1、减少了垃圾回收
- 使用堆外内存的话，堆外内存是直接受操作系统管理( 而不是虚拟机 )。这样做的结果就是能保持一个较小的堆内内存，以减少垃圾收集对应用的影响。

2、提升复制速度(io效率)
- 堆内内存由JVM管理，属于“用户态”；而堆外内存由OS管理，属于“内核态”。如果从堆内向磁盘写数据时，数据会被先复制到堆外内存，即内核缓冲区，然后再由OS写入磁盘，使用堆外内存避免了这个操作。
#### 对象创建的过程--内存方面
new 类名 -> 根据new的参数在常量池中定位到一个类的符号引用 -> 没有找到符号引用说明类没有被加载，进行类的加载，解析，初始化 -> 虚拟机分配堆内存 -> 将分配的内存置初始化为0 -> 调用对象的init方法。
#### java对象结构
在HotSpot虚拟机中，对象在内存中存储的布局可以分为3块区域：
- 对象头（Header）
    1. markword 
第一部分markword,用于存储对象自身的运行时数据，如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等，这部分数据的长度在32位和64位的虚拟机（未开启压缩指针）中分别为32bit和64bit，官方称它为“MarkWord”。
    2. klass 
对象头的另外一部分是klass类型指针，即对象指向它的类元数据的指针，虚拟机通过这个指针来确定这个对象是哪个类的实例.
    3. 数组长度（只有数组对象有） 
    如果对象是一个数组, 那在对象头中还必须有一块数据用于记录数组长度.
- 实例数据（Instance Data）
    实例数据部分是对象真正存储的有效信息，也是在程序代码中所定义的各种类型的字段内容。无论是从父类继承下来的，还是在子类中定义的，都需要记录起来。
- 对齐填充（Padding）
    第三部分对齐填充并不是必然存在的，也没有特别的含义，它仅仅起着占位符的作用。由于HotSpot VM的自动内存管理系统要求对象起始地址必须是8字节的整数倍，换句话说，就是对象的大小必须是8字节的整数倍。而对象头部分正好是8字节的倍数（1倍或者2倍），因此，当对象实例数据部分没有对齐时，就需要通过对齐填充来补全。

https://www.cnblogs.com/maxigang/p/9040088.html
#### 对象的访问定位
- 使用句柄池
    栈帧中的引用是固定的，不用改变，地址指向句柄池，句柄池指向对象地址
- 直接指针
    直接指向对象地址 优点速度快 hotspot采用直接指针。
### 垃圾回收
#### 如何判定对象是垃圾对象
- 引用计数法
    对象添加一个引用计数器，有地方引用这个对应，计数器+1，时效为-1
    一缺点：堆内存的对象相互引用之后，不会被垃圾回收。
- 可达性分析法
    根据GCroot节点往下找引用琏
    作为GCroots的对象
        - 虚拟栈 一般指局部变量表
        - 方法区的类属性引用的对象
        - 方法区常量引用的对象
        - 本地方法栈引用的对象
#### 垃圾回收算法
- 标记清除算法
    标记被清除的内存区域，进行清除
    - 效率问题
    - 空间问题，清除后的内存不连续
- 复制算法
    堆内存分为：
    - 新生代
        - Eden 伊甸园
        - survivor 存活区
        - Tenured Gen
    - 老年代
    垃圾回收器主要光顾eden区，它占百分之八十左右比较大，两个survivor区，进行垃圾回收的时候，会把存活的内存数据复制到survivor区中，把eden全部清空，下次回收垃圾时，会把eden和survivor的存活的数据复制到另外一个survivor中。内存不够，通过内存担保算法申请内存。
- 标记-整理算法
    针对需要回收的区域不多，将回收的较多的区域的有用数据移动到回收较少的区域，并将回收较少的垃圾数据移动到回收较多的区域，再将回收较多区域清空。
- 分代收集算法
    新生代回收垃圾较多使用复制算法，老年代回收数据较少，使用标记整理算法。
#### 垃圾收集器
- serial收集器
    - 最基本，最悠久
    - 单线程垃圾收集器  
    - 桌面应用，内存比较小的，收集比较快
    - 垃圾收集的时候其他线程会停止
- ParNew收集器
    - 多线程
    - 
- Parllel Scvenge收集器
    - 复制算法（新生代收集器）
    - 多线程
    - 达到可控制的吞吐量
    - XX:MaxGCPauseMills 垃圾收集器最大停顿时间
    - XX:CGTimeRatio 吞吐量大小
- CMS收集器 Concurrent Mark Sweep 并发标记清除收集器
   - 初始标记
   - 并发标记
   - 重新标记
   - 并发清理
   初始标记和重新标记，会停顿（停止其他的线程），但时间较短，初始标记是标记GCroots，并发标记是根据GCroots进行追踪（使用的是可达性分析法），重新标记是在并发标记时用户程序，改变的内存对象，对变动的进行重新标记，并发清理，可以和用户程序同时进行。
CMS收集器一般使用在老年代。
    缺点：
        - 占用大量的cpu资源
        - 空间碎片 （采用的是标记清除算法）  
        - 无法处理浮动垃圾(在清理垃圾时产生的垃圾)
        - 出现Concurrent mode failure(在垃圾回收时，有专门的一块区域，用来存并发时的对象，如果这个区域在回收的时候满了，就会报这个错，之后会采用serial收集器，导致回收时间加长，影响性能。)
- G1收集器
    JDK9 默认使用G1，适用于比较大的项目。
    你之前收集器之和：
    优势：
        - 并行并发，
        - 分代收集，
        - 空间 整合，
        - 可预测的停顿，

    步骤：
        - 初始标记
        - 并发标记
        - 最终标记
        - 筛选回收
没有很强新生代和老年代，会把内存分成一块一块区域，在一个remember set表里面会记录，进行筛选，哪个区域更容易回收。

### 内存分配策略
- 优先分配到eden区
- 大对象直接分配到老年代
- 长期存活的对象直接分配到老年代
        eden区经常被gc回收，执行的是复制算法，频繁移动数据，将大对象分配在老年代可以提高性能。
- 空间分配担保
    - 先解释YGC：
        当对象生成在EDEN区失败时，出发一次YGC，先扫描EDEN区中的存活对象，进入S0区，S0放不下的进入OLD区，再扫描S1区，若存活次数超过阀值则进入OLD区，其它进入S0区，然后S0和S1交换一次。

        那么当发生YGC时，JVM会首先检查老年代最大的可用连续空间是否大于新生代所有对象的总和，如果大于，那么这次YGC是安全的，如果不大于的话，JVM就需要判断HandlePromotionFailure是否允许空间分配担保。
    - 允许分配担保：
        JVM继续检查老年代最大的可用连续空间是否大于历次晋升到老年代的对象的平均大小，如果大于，则正常进行一次YGC，尽管有风险（因为判断的是平均大小，有可能这次的晋升对象比平均值大很多）；
        如果小于，或者HandlePromotionFailure设置不允许空间分配担保，这时要进行一次FGC。

        新生代采用的是复制收集算法，S0和S1始终只是用其中一块内存区，当出现YGC后大部分对象仍然存活的话，就需要老年代进行分配担保，把survior区无法容纳的对象直接晋升到老年代。
        那么这种空间分配担保的前提是老年代还有容纳的空间，一共有多少对象会活下来，在实际完成内存回收之前是无法明确知道的，所以只好取之前每次回收晋升到老年代对象容量的平均值大小作为经验值，与老年代的剩余空间比较，决定是否进行FGC来让老年代腾出更多空间。
    
- 动态对象年龄判断