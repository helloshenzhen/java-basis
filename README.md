# juc jvm 知识

复习JUC和JVM的笔记

#### 1:进程和线程:

    进程和线程主要区别在于他们是操作系统不同的资源管理方式.
  
    进程是程序的一次执行过程(运行中的程序),是系统运行程序的基本单位.一个进程至少包含一个线程(main),
    
    可以包含多个线程.(换言之,线程是进程内的执行单元)
  
    线程与进程相似,它是比进程更小的执行单位.一个进程在执行过程中可以产生多个线程.
    同类线程有共享的堆和方法区(jdk8之后的元空间(MetaSpace)),每个线程又有自己的程序计数器,虚拟机栈,本地方法栈.
    系统在各个线程之间的切换工作要比进程负担低,因此线程又被称为轻量级进程

#### 2:线程的几种状态(见:jdk Thread类源码中的state枚举类)
      NEW,RUNNABLE,BLOCKED,WAITING,TIMED_WAITING,TERMINATED

#### ３:并发和并行:
    并发是指计算机在同一时间段内处理多任务的执行. 如我和小明同时访问淘宝网站,那么淘宝服务器就同时处理我和小明的访问请求
    
    并行是指多任务同时执行,但是任务之间没有任何关系,不涉及共享资源.比如我一边看电视一边喝水,2件事互不干扰
    
    
#### JVM运行时内存分区:
     JDK8之前:线程私有的部分有:程序计数器(PC寄存器),JAVA虚拟机栈,本地方法栈(native)
             线程共享部分有: GC堆,方法区(永久代包含运行时常量池)
     JDK8之后:线程私有的部分不变, 线程共享部分的方法区改为了元空间(MetaSpace),运行时常量池也移动到了 heap空间
     
##### 程序计数器:
      程序计数器又称PC寄存器,它是记录着当前线程执行的字节码的行号指示器.
      cpu是通过时间片轮换制度执行每个线程的任务,而在多个线程之间切换时,程序计数器就记录当前线程执行的位置,当cpu又开始执行此线程的时候,它需要知道
      上次运行的位置,那么就是通过程序计数器直到上次字节码的执行位置,所以每个线程都会有属于自己的程序计数器
      
##### Java虚拟机栈:
       Java虚拟机栈描述的是方法执行的内存模型,每个方法在执行时会在虚拟机栈中创建一个
       栈帧,每个方法的执行,就对应着栈帧在虚拟机栈中的入栈和出栈.
       栈帧由局部变量表,操作数栈,方法出口,动态链接等数据组成.
      
  ###### 局部变量表:存放编译期可知的各种数据类型(常见的8大数据类型)和引用类型(reference,可以是指向对象的指针,也可以是句柄)
  
  ###### 操作数栈:与局部变量类似,它也可以存放任意类型的数据,但它是作为数据在计算时的临时存储空间,入站和出栈
  
  ###### 动态链接:因为每个方法在运行时都会创建相应的栈帧,那么栈帧也会保存一份方法的引用,栈帧保存方法的引用是为了支持方法调用过程中的动态链接.
  
      动态链接是指将符号引用转为直接引用的过程.如果方法调用另一个方法或者调用另一个类的成员变量就需要知道其名字,
      符号引用就相当于名字,运行时就将这个名字解析成相应的直接引用.
  
  ###### 方法出口:方法执行后,有2种方式退出: 1: 执行方法的过程中遇到了异常; 2: 遇到正常的返回字节码指令.
        无论何种方式退出,在方法退出后,都需要回到方法被调用时的位置,方法在返回时就需要在栈帧中保存一些信息,用来帮助它恢复上层方法的执行状态.
##### 本地方法栈:
      与Java虚拟机栈类似,Java虚拟机栈是对Java方法执行的描述,但是本地方法栈是对jvm的native方法描述,也就是第三方c/c++
      编写的方法,本地方法栈也有相应的 局部变量表,操作数栈,方法出口,动态链接
   
##### 堆
      堆是jvm内存区域中最大的一块区域,堆区的作用是为对象分配内存,存储他们,并负责回收它们之中无用的对象.
      堆可以分为新生代和老年代,新生代占堆区的1/3,老年代占堆区的2/3.新生代包括eden,from survivor, to survivor三个空间
      其中 eden空间最大占新生代的80%内存,from和to都是1:1.
      新生代是GC发生最频繁的区域,发生在新生代的GC被称为MinorGC,老年代的GC被称为Major GC / Full GC
      因为老年代的空间比新生代的空间大,所以通常老年代的GC时间会比新生代的GC时间慢很多.
      对象优先在eden区域分配,当eden区域没有空间时,虚拟机就会发起一次Minor GC
    
#### 判断对象存活的方法

#####  1: 引用计数法:
          给每个对象添加一个引用计数器,当对象被引用的时候,引用计数器就+1,当引用失效时,引用计数器
          就-1,直到引用计数器为0,就代表对象不再被引用.
          引用计数的主要缺陷是很难解决循环引用的问题:也就是当2个对象互相引用的时候,除了彼此,就没有其他地方引用这2个对象,那么他们的引用计数都为1,
          就无法被回收
       2: 可达性算法:
          通过一系列被称为GC ROOTS的对象节点往下搜索,节点走过的地方被称为引用链,如果一个对象不被任何引用链走过,那么称
          此对象不可达.
   
####　垃圾回收算法：
      1: 复制算法:将内存分为2块大小的内存空间,每次使用其中一块空间,当一块使用完后,将还存活的对象复制到另一块空间去,然后清楚已经使用过的空间.
         根据GC角度来说就是:在新手代的eden空间和From Survivor空间经历过MinorGC后,仍然存活的对象采用复制算法复制到To Survivor,
         并将To Survivor(其实就是空间不空闲的那块Survivor区域)的对象年龄+1,默认对象年龄撑过15岁,那么进入老年代.
   
         复制算法的缺点就是太耗空间内存.
         
      2: 标记-清除算法:标记出所有存活的对象,然后统一回收所有被标记的对象也就是仍然存活的对象.
         标记清除算法的最大缺点就是会造成不连续的内存空间,也就是内存碎片,因为对象在内存中的分布是不均匀的.
      
      3: 标记-整理算法:是对标记-清除算法做出的改进,标记整理算法也是首先对所有仍然存活的对象做出标记,
         不同的是,它会使所有仍然存活的对象向空间的一段移动,然后对其它端进行清理.
         此算法虽然不会产生内存碎片,但是它的效率会比标记清楚算法慢
         
      4: 分代收集算法:分代收集算法不是一种具体的收集算法.因为堆是分为新生代(包括eden空间,from survivor,to survivor)
         老年代的,分代收集算法就是在不同的分代空间采用不同的垃圾回收算法:如复制算法应用于新生代,标记清除和标记整理应用于老年代
         
       
          
              
    
      

   
  
 
  
