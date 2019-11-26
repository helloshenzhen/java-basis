# java基础知识

复习java基础知识的笔记


#### 进程和线程:

    进程和线程主要区别在于他们是操作系统不同的资源管理方式.
  
    进程是程序的一次执行过程(运行中的程序),是系统运行程序的基本单位.
    一个进程至少包含一个线程(main),
    
    可以包含多个线程.(换言之,线程是进程内的执行单元)
  
    线程与进程相似,它是比进程更小的执行单位.一个进程在执行过程中可以产生多个线程.
    同类线程有共享的堆和方法区(jdk8之后的元空间(MetaSpace)),
    每个线程又有自己的程序计数器,虚拟机栈,本地方法栈.
    系统在各个线程之间的切换工作要比进程负担低,因此线程又被称为轻量级进程

#### 线程的几种状态:(见:jdk Thread类源码中的state枚举类)
      NEW,RUNNABLE,BLOCKED,WAITING,TIMED_WAITING,TERMINATED

#### 并发和并行:
    并发是指计算机在同一时间段内处理多任务的执行. 
    如我和小明同时访问淘宝网站,那么淘宝服务器就同时处理我和小明的访问请求
    
    并行是指多任务同时执行,但是任务之间没有任何关系,不涉及共享资源.
    比如我一边看电视一边喝水,2件事互不干扰

##### 公平锁: 
      指根据线程在队列中的优先级获取锁,比如线程优先加入阻塞队列,那么线程就优先获取锁

##### 非公平锁:
      指在获取锁的时候,每个线程都会去争抢,并且都有机会获取到锁,无关线程的优先级 

##### 可重入锁:
       一个线程获取到锁后,如果继续遇到被相同锁修饰的资源或方法,那么可以继续获取该锁.
       对synchronized来说,每个锁都有线程持有者和锁计数器,每次线程获取到锁,会记录下
       改线程,并且锁的计数器就+1,当线程退出synchronized代码块的时候,线程计数就会-1,
       当锁计数为0的时候,就释放锁.
       
##### 自旋锁:
       指当锁被获取后,其他线程并不会停止获取,而是一直去尝试获取.这样做的好处是减少上下文开销,
       缺点是增加cpu消耗. 
       CAS底层就使用了自旋操作(不是自旋锁,而是如果预期值和原值比较不成功就会一直比较)       
     
##### 独占锁:
      　锁一次只能被一个线程占有使用,Synchronized和ReetrantLock都是独占锁
    
##### 共享锁:
       锁可以被多个线程持有,对于ReentrantReadWriteLock而言,它的读锁是共享锁,写锁是独占锁      
            

#### volatile:
     Volatile是JVM提供的轻量级的同步机制
     
  ##### 1: volatile保证内存可见性
        JMM内存模型实现总是线程从主内存(共享内存)读取数据,线程把主存的变量存储到本地,
        在本地进行修改,然后写回主内存,而不是直接在主存中进行操作.
        
        那么这就可能造成可见性问题:假设2个线程从主存读取同一个变量,一个线程修改了它的本地变量,
        并写回了主存,但是另一个线程仍然使用的是之前的值,这就造成了数据的不一致.
        volatile关键字修饰的变量就解决了这个问题:被volatile修饰的变量要求线程使用时,
        都从主内存中读取,而不使用本地的拷贝.
          
  ##### 2:　volatile不保证原子性
          原子性指一个操作的完整性,它是不可分割的,要么同时成功执行,要么同时失败回滚. 
          当多个线程同时对volatie关键字修饰的变量进行非原子性操作(++,--)的时候,
          变量可能会被多++一次,少++一次,多--一次,少--一次.
          volatile并不能保证操作的原子性,最后得到的结果可能不尽人意
          
  ##### 如何解决原子性:
        1: 最简单的方法就是加锁
        2: 使用CAS原子类
  
  ##### 3:　volatile禁止指令重排序   
        指令重排序是编译器和cpu为了尽可能高效的执行程序而采取的一种优化手段,
        它会导致程序实际执行的顺序和代码的顺序不一定相符,
        而volatil就是在执行的代码前后加入屏障,使cpu在执行时
        无法重排序,就按代码的顺序执行
  
#### CAS:
      CAS:CompareAndSet,比较并交换,它将指定内存位置的值与给定值进行比较,
      如果两个值相等,就将内存位置的值
      改为给定值.CAS涉及3个元素:内存地址,期盼值和目标值,
      只有内存地址对应的值和期望的值相同时,才把内存地址对应的值修改为目标值.
 ##### CAS在JAVA中的底层实现(Atomic原子类实现)        
 ######     1:Unsafe类:
    Unsafe类是CAS的核心类,由jdk自动加载,它的方法都是native方法.
    因为Java无法像c/c++一样直接使用底层指针操作对象
    内存,Unsafe类的作用就是专门解决这个问题,它可以直接操作对象在内存中的地址.
    具体步骤是:首先获取当前Atomic对象的value在内存中真实的偏移地址,再根据这个偏移地址
    获取value的真实值,然后再重复这个步骤,把两次获取到的值进行比较,
    如果比较成功,则继续操作,否则继续循环比较.
    而获取value在内存中真实的偏移地址和比较设置值方法都是native的.
    
 ######    2: volatile:
    Atomic原子类内部的value值是volatile修饰的,这就保证了value的可见性.
 
 ##### CAS的缺点:
 
 ###### 1: 循环时间开销大
          如果预期的值和当前值比较不成功,那么CAS会一直进行循环.如果长时间比较不成功
          就会一直循环,导致CPU开销过大    
           
 ###### 2:　只能保证一个共享变量的原子操作
          在获取内存地址和设置值的时候都是当前Atomic对象的volatile值,如果要保证多个共享变量,
          那么可以通过加锁来保证线程安全和原子性.
          
 ###### 3: ABA问题
           尽管一个线程CAS操作成功,但并不代表这个过程就是没有问题的.
           假设2个线程读取了主内存中的共享变量,如果一个线程对主内存中的值进行了修改后,
           又把新值改回了原来的值,而此时另一个线程进行CAS操作,发现原值和期盼的值是
           一样的,就顺利的进行了CAS操作.这就是CAS引发的ABA问题.
           
 ###### 4:　解决ABA问题
            juc的atomic包下提供了AtomicStampedReference这个类来解决CAS的原子引用
            更新的ABA问题,它相较于普通的Atomic原子类多增加了一个版本号的字段,
            每次修改引用就更新版本号,这样即使发生ABA问题,也能通过版本号判断引用
            是否被修改过了.                 
                     
#### 线程池的好处:
     池化技术屡见不鲜:数据库连接池,Http连接池,线程池都是这种思想．
     池化技术的好处非常明显,以往单个的new Thread,不易于线程之间的管理,
     而池化技术把所有线程都放在一个池子里,要用就取出,用完就回收,这样非常易于管理,
     并且可以降低资源的消耗,使线程可以重复利用,提高任务的响应速度,当任务到来时,就可以
     处理.

#### 线程池构造参数:
````
 ThreadPoolExecutor
(int corePoolSize,
 int maximumPoolSize, 
 long keepAliveTime,
 TimeUnit unit,
 BlockingQueue<Runnable> workQueue,
 ThreadFactory threadFactory,
 RejectedExecutionHandler handler)
````

##### corePoolSize:
      线程池的核心线程数(常驻线程数),也就是线程池的最小线程数,这部分线程不会被回收.
      
##### maximumPoolSize:
      线程池最大线程数,线程池中允许同时执行的最大线程数量
      
##### keepAliveTime:
      当线程池中的数量超过 corePoolSize(最小线程池数量),并且此时没有新的任务执行,那么
      会保持    keepAliveTime 的时间才会回收线程
      
##### unit
      keepAliveTime的时间单位
  
##### workQueue:
      任务队列,当有新任务来临时,如果核心线程数corePoolSize被用完,此时如果workQueue有空间,
      任务就会被放入workQueue            

##### threadFactory:
      创建工作线程的工厂,也就是如何创建线程的,一般采用默认的

##### handler:
      拒绝策略. 如果线程池陷入一种极端情况:工作队列满了,无法再容纳新的任务,最大工作线程也到达限制了,
      此时线程池如何处理这种极端情况.
      ThreadPoolExecutor 提供了四种策略:
   ###### AbortPolicy(是线程池的默认拒绝策略): 
         如果还有新任务到来,那么拒绝,并抛出RejectedExecutionException异常
   ###### CallerRunsPolicy: 
         这种策略不会拒绝执行新任务,但是由发出任务的线程执行,也就是说当线程池无法
         执行新任务的时候,就由请求线程自己执行任务
   ###### DiscardPolicy:
          这种策略会拒绝新任务,但是不会抛出异常
   ###### DiscardOldestPolicy:
         这种策略不会拒绝策略,他会抛弃队列中等待最久那个任务,来执行新任务      
 
##### 阿里巴巴开发者手册不建议开发者使用Executors创建线程池:
      newFixedThreadPool和newSingleThreadExecutor:
      会创建固定数量线程的线程池和单线程线程池,尽管二者线程池数量有限,
      但是它会创建长度为Integer.MAX_VALUE长度的阻塞队列,这样可能会导致阻塞队列
      的任务过多而导致OOM(OutOfMemoryError)
      newCachedThreadPool和newScheduledThreadPool:
      会创建缓存线程池和周期任务线程池,二者线程池的最大线程为Integer.MAX_VALUE,
      也可能会导致OOM   
    
#### JVM运行时内存分区:.
     JDK8之前:线程私有的部分有:程序计数器(PC寄存器),JAVA虚拟机栈,
     本地方法栈(native),线程共享部分有: GC堆,方法区(永久代包含运行时常量池)
     JDK8之后:线程私有的部分不变, 线程共享部分的方法区改为了元空间(MetaSpace),
     运行时常量池也移动到了 heap空间
     
##### 程序计数器:
      程序计数器又称PC寄存器,它是记录着当前线程执行的字节码的行号指示器.
      cpu是通过时间片轮换制度执行每个线程的任务,而在多个线程之间切换时,
      程序计数器就记录当前线程执行的位置,当cpu又开始执行此线程的时候,它需要知道
      上次运行的位置,那么就是通过程序计数器直到上次字节码的执行位置,
      所以每个线程都会有属于自己的程序计数器
      
##### Java虚拟机栈:
       Java虚拟机栈描述的是方法执行的内存模型,每个方法在执行时会在虚拟机栈中创建一个
       栈帧,每个方法的执行,就对应着栈帧在虚拟机栈中的入栈和出栈.
       栈帧由局部变量表,操作数栈,方法出口,动态链接等数据组成.
    
#####　Java虚拟机栈的2种错误      
   ###### StackOverflowError:
          当Java虚拟机栈无法动态扩容的时候,当前线程执行或请求的栈的大小超过了Java虚拟机栈的最大空间
          (比如递归嵌套调用太深),那么抛出StackOverflowError错误
           
   ###### OutOfMemoryError:
           1: 当Java虚拟机栈允许动态扩容的时候,当前虚拟机栈执行请求的栈的大小仍然超过了扩容之后的最大空间,
             无法继续为栈分配空间(堆内存分配空间过小),那么抛出OutOfMemoryError错误
           2: Java堆存放对象实例,当需要为对象分配内存时,而堆空间大小已经达到最大值,
              无法为对象实例继续分配空间时,抛出 OutOfMemoryError错误
   
   ###### 虚拟机栈栈的动态扩容:
           上面说过如果当虚拟机栈允许动态扩容,当动态扩容的空间都不够用的时候,就抛出OutOfMemory异常.
           虚拟机栈的动态扩容是指在栈空间不够用的时候,自动增加栈的内存大小.
           
           动态扩容栈有2种方法:
           Stack Copying: 就是分配一个更大的栈空间,把原来的栈拷贝到新栈空间去.
           Segmented Stack: 可以理解为一个双向链表把多个栈链接起来,一开始只分配一个栈,当
                            这个空间不够时.再分配一个栈空间,用链表链接起来.
           
  ###### 局部变量表:存放编译期可知的各种数据类型(常见的8大数据类型)和引用类型(reference,可以是指向对象的指针,也可以是句柄)
  
  ###### 操作数栈:与局部变量类似,它也可以存放任意类型的数据,但它是作为数据在计算时的临时存储空间,入站和出栈
  
  ###### 动态链接:因为每个方法在运行时都会创建相应的栈帧,那么栈帧也会保存一份方法的引用,栈帧保存方法的引用是为了支持方法调用过程中的动态链接.
  
      动态链接是指将符号引用转为直接引用的过程.
      如果方法调用另一个方法或者调用另一个类的成员变量就需要知道其名字,
      符号引用就相当于名字,运行时就将这个名字解析成相应的直接引用.
  
  ###### 方法出口:方法执行后,有2种方式退出: 1: 执行方法的过程中遇到了异常; 2: 遇到正常的返回字节码指令.
        无论何种方式退出,在方法退出后,都需要回到方法被调用时的位置,
        方法在返回时就需要在栈帧中保存一些信息,用来帮助它恢复上层方法的执行状态.
##### 本地方法栈:
      与Java虚拟机栈类似,Java虚拟机栈是对Java方法执行的描述,
      但是本地方法栈是对jvm的native方法描述,也就是第三方c/c++
      编写的方法,本地方法栈也有相应的 局部变量表,操作数栈,方法出口,动态链接
   
##### 堆
      堆是jvm内存区域中最大的一块区域.
      
      堆区的作用是为对象分配内存,存储他们,并负责回收它们之中无用的对象.
      堆可以分为新生代和老年代,新生代占堆区的1/3,老年代占堆区的2/3.
      新生代包括eden,from survivor, to survivor三个空间
      其中 eden空间最大占新生代的80%内存,from和to都是1:1.
      
      新生代是GC发生最频繁的区域.
      发生在新生代的GC被称为MinorGC,老年代的GC被称为Major GC / Full GC
      
   
     因为老年代的空间比新生代的空间大,所以通常老年代的GC时间会比新生代的GC时间慢很多.
     对象优先在eden区域分配,当eden区域没有空间时,虚拟机就会发起一次Minor GC
    
#### 判断对象存活的方法

#####  1: 引用计数法:
          给每个对象添加一个引用计数器,当对象被引用的时候,引用计数器就+1,当引用失效时,引用计数器
          就-1,直到引用计数器为0,就代表对象不再被引用.
          引用计数的主要缺陷是很难解决循环引用的问题:也就是当2个对象互相引用的时候,除了彼此,
          就没有其他地方引用这2个对象,那么他们的引用计数都为1,就无法被回收
#####  2: 可达性算法:
          通过一系列被称为GC ROOTS的对象节点往下搜索,节点走过的地方被称为引用链,
          如果一个对象不被任何引用链走过,那么称
          此对象不可达.
          
 ###### 什么是GC Root
        
        上面说通过GC Root对象搜索引用链,那么GC Root对象是什么对象,或者什么样的对象是GC Root对象.
        
        可以作为GC Root对象的有: 
        1: 虚拟机栈和本地方法栈区(native)的引用对象;
        2: 堆区里的静态变量引用的对象;
        3: 堆区里的常量池的常量引用的对象         
        
#### 垃圾回收算法：

  ##### 复制算法:
         将内存分为2块大小的内存空间,每次使用其中一块空间,当一块使用完后,
         将还存活的对象复制到另一块空间去,然后清楚已经使用过的空间.
         根据GC角度来说就是:在新生代的eden空间和From Survivor空间经历过MinorGC后,
         仍然存活的对象采用复制算法复制到To Survivor,
         并将To Survivor(其实就是空间不空闲的那块Survivor区域)的对象年龄+1,
         默认对象年龄撑过15岁,那么进入老年代.
   
         复制算法的缺点就是太耗空间内存.
         
  ##### 标记-清除算法:
         标记出所有需要被回收的对象,然后统一回收所有被标记的对象.
         
         标记清除算法的最大缺点就是会造成不连续的内存空间,
         也就是内存碎片,因为对象在内存中的分布是不均匀的.
      
  ##### 标记-整理算法:
         是对标记-清除算法做出的改进,标记整理算法也是首先标记出所有需要被回收的对象,
         不同的是,它会使所有仍然存活的对象向空间的一段移动,然后对其它端进行清理.
         
         此算法虽然不会产生内存碎片,但是它的效率会比标记清楚算法慢
         
  ##### 分代收集算法:
         分代收集算法不是一种具体的收集算法.
         因为堆是分为新生代(包括eden空间,from survivor,to survivor)
         老年代的,分代收集算法就是在不同的分代空间采用不同的垃圾回收算法:
         如复制算法应用于新生代,标记清除和标记整理应用于老年代
     
  ####  强引用:
        一般常用的new方式创建对象,创建的就是强引用. 只要强引用存在,
        垃圾回收器就不会回收.        
  
  #### 软引用
         SoftReference , 非必须引用,如果内存足够或正常,就不回收,但是当
         内存不够,快发生OOM的时候就回收掉软引用对象.
  
  #### 弱引用
        WeakReference , 对于弱引用的对象来说,只要垃圾回收器开始回收,\
        无论空间是否充足,都回收弱引用的对象.
        
  ####  虚引用(幽灵引用)
        和其他几种引用不同,虚引用不会决定对象生命周期,垃圾回收时,无法通过虚引用获取对象
        值.虚引用在任何时候都可能被垃圾回收掉.虚引用必须和引用队列(ReferenceQueue)使用.   
      
  ##### 软引用,弱引用,虚引用在被GC前会被加入到与其关联的引用队列中.     
         
          
              
    
      

   
  
 
  
