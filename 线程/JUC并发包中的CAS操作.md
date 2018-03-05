在此对java并发包做一个大致总结，如有错误，请指正。 
juc包的总体结构大致如下 
 
1.外层框架主要有Lock(ReentrantLock、ReadWriteLock等)、同步器（semaphores等）、阻塞队列（BlockingQueue等）、Executor（线程池）、并发容器（ConcurrentHashMap等）、还有Fork/Join框架； 
2.内层有AQS（AbstractQueuedSynchronizer类，锁功能都由他实现）、非阻塞数据结构、原子变量类(AtomicInteger等无锁线程安全类)三种。 
3.底层就实现是volatile和CAS。整个并发包其实都是由这两种思想构成的。

1.1volatile
理解volatile特性的一个好方法是把对volatile变量的单个读/写，看成是使用同一个锁对这 
些单个读/写操作做了同步。一个volatile变量的单个读/写操作，与一个普通变量的读/写操作都 
是使用同一个锁来同步，它们之间的执行效果相同。 
volatile具有的特性。 
可见性。对一个volatile变量的读，总是能看到（任意线程）对这个volatile变量最后的写 
入。 
原子性：对任意单个volatile变量的读/写具有原子性，但类似于volatile++这种复合操作不 
具有原子性。
volatile读的内存语义如下。 
当读一个volatile变量时，JMM会把该线程对应的本地内存置为无效。线程接下来将从主 
内存中读取共享变量。 
volatile写的内存语义如下。 
当写一个volatile变量时，JMM会把该线程对应的本地内存中的共享变量值刷新到主内 
存。
为了实现volatile的内存语义，编译器在生成字节码时，会在指令序列中插入内存屏障来 
禁止特定类型的处理器重排序。对于编译器来说，发现一个最优布置来最小化插入屏障的总 
数几乎不可能。为此，JMM采取保守策略。下面是基于保守策略的JMM内存屏障插入策略。 
·在每个volatile写操作的前面插入一个StoreStore屏障。 
·在每个volatile写操作的后面插入一个StoreLoad屏障。 
·在每个volatile读操作的后面插入一个LoadLoad屏障。 
·在每个volatile读操作的后面插入一个LoadStore屏障。
上面内存屏障的简单意思就是：StoreStore屏障，禁止上面的写操作和下面的volatile写重排序；StoreLoad屏障，禁止上面的写操作和下面的volatile读重排序；LoadLoad屏障，禁止上面的读/写操作和下面的volatile读操作重排序；LoadStore屏障，禁止上面的读操作和下面的volatile写操作重排序。
由于Java的CAS同时具有volatile读和volatile写的内存语义，因此Java线程之间的通信现 
在有了下面4种方式。 
1）A线程写volatile变量，随后B线程读这个volatile变量。 
2）A线程写volatile变量，随后B线程用CAS更新这个volatile变量。 
3）A线程用CAS更新一个volatile变量，随后B线程用CAS更新这个volatile变量。 
4）A线程用CAS更新一个volatile变量，随后B线程读这个volatile变量。
总结一下适合使用volatile变量的使用条件（必须满足所有条件）： 
1、对变量的写操作不依赖变量的当前值，或者你能确保只有单线程更新变量的值。（简单来说就是单线程写，多线程读的场景） 
2、该变量不会与其他状态变量一起纳入不变性条件中。 
3、在访问变量时不需要加锁。（如果要加锁的话用普通变量就行了，没必要用volatile了）
1.2首先介绍一些乐观锁与悲观锁:
　　悲观锁：总是假设最坏的情况，每次去拿数据的时候都认为别人会修改，所以每次在拿数据的时候都会上锁，这样别人想拿这个数据就会阻塞直到它拿到锁。传统的关系型数据库里边就用到了很多这种锁机制，比如行锁，表锁等，读锁，写锁等，都是在做操作之前先上锁。再比如Java里面的同步原语synchronized关键字的实现也是悲观锁。
　　乐观锁：顾名思义，就是很乐观，每次去拿数据的时候都认为别人不会修改，所以不会上锁，但是在更新的时候会判断一下在此期间别人有没有去更新这个数据，可以使用版本号等机制。乐观锁适用于多读的应用类型，这样可以提高吞吐量，像数据库提供的类似于write_condition机制，其实都是提供的乐观锁。在Java中java.util.concurrent.atomic包下面的原子变量类就是使用了乐观锁的一种实现方式CAS实现的。
 
乐观锁的一种实现方式-CAS(Compare and Swap 比较并交换)：
　　锁存在的问题:
　　　　Java在JDK1.5之前都是靠 synchronized关键字保证同步的，这种通过使用一致的锁定协议来协调对共享状态的访问，可以确保无论哪个线程持有共享变量的锁，都采用独占的方式来访问这些变量。这就是一种独占锁，独占锁其实就是一种悲观锁，所以可以说 synchronized 是悲观锁。
　　　　悲观锁机制存在以下问题：　　
　　　　　　1. 在多线程竞争下，加锁、释放锁会导致比较多的上下文切换和调度延时，引起性能问题。
　　　　　　2. 一个线程持有锁会导致其它所有需要此锁的线程挂起。
　　　　　　3. 如果一个优先级高的线程等待一个优先级低的线程释放锁会导致优先级倒置，引起性能风险。
　　　　对比于悲观锁的这些问题，另一个更加有效的锁就是乐观锁。其实乐观锁就是：每次不加锁而是假设没有并发冲突而去完成某项操作，如果因为并发冲突失败就重试，直到成功为止。
　　乐观锁：
　　　　乐观锁（ Optimistic Locking ）在上文已经说过了，其实就是一种思想。相对悲观锁而言，乐观锁假设认为数据一般情况下不会产生并发冲突，所以在数据进行提交更新的时候，才会正式对数据是否产生并发冲突进行检测，如果发现并发冲突了，则让返回用户错误的信息，让用户决定如何去做。
　　　　上面提到的乐观锁的概念中其实已经阐述了它的具体实现细节：主要就是两个步骤：冲突检测和数据更新。其实现方式有一种比较典型的就是 Compare and Swap ( CAS )。
　　CAS：
　　　　CAS是乐观锁技术，当多个线程尝试使用CAS同时更新同一个变量时，只有其中一个线程能更新变量的值，而其它线程都失败，失败的线程并不会被挂起，而是被告知这次竞争中失败，并可以再次尝试。　　　
　　　　CAS 操作中包含三个操作数 —— 需要读写的内存位置（V）、进行比较的预期原值（A）和拟写入的新值(B)。如果内存位置V的值与预期原值A相匹配，那么处理器会自动将该位置值更新为新值B。否则处理器不做任何操作。无论哪种情况，它都会在 CAS 指令之前返回该位置的值。（在 CAS 的一些特殊情况下将仅返回 CAS 是否成功，而不提取当前值。）CAS 有效地说明了“ 我认为位置 V 应该包含值 A；如果包含该值，则将 B 放到这个位置；否则，不要更改该位置，只告诉我这个位置现在的值即可。 ”这其实和乐观锁的冲突检查+数据更新的原理是一样的。
　　　　这里再强调一下，乐观锁是一种思想。CAS是这种思想的一种实现方式。
　　JAVA对CAS的支持：
　　　　在JDK1.5 中新增 java.util.concurrent (J.U.C)就是建立在CAS之上的。相对于对于 synchronized 这种阻塞算法，CAS是非阻塞算法的一种常见实现。所以J.U.C在性能上有了很大的提升。
　　　　以 java.util.concurrent 中的 AtomicInteger 为例，看一下在不使用锁的情况下是如何保证线程安全的。主要理解 getAndIncrement 方法，该方法的作用相当于 ++i 操作。

  public class AtomicInteger extends Number implements java.io.Serializable {   2     private volatile int value;  3  4     public final int get() {   5         return value;   6     }   7  8     public final int getAndIncrement() {   9         for (;;) {  10             int current = get();  11             int next = current + 1;  12             if (compareAndSet(current, next))  13                 return current;  14         }  15     }  16 17     public final boolean compareAndSet(int expect, int update) {  18         return unsafe.compareAndSwapInt(this, valueOffset, expect, update);  19     }  20 }

　　　　在没有锁的机制下,字段value要借助volatile原语，保证线程间的数据是可见性。这样在获取变量的值的时候才能直接读取。然后来看看 ++i 是怎么做到的。
　　　  getAndIncrement 采用了CAS操作，每次从内存中读取数据然后将此数据和 +1 后的结果进行CAS操作，如果成功就返回结果，否则重试直到成功为止。
　　　  而 compareAndSet 利用JNI（Java Native Interface）来完成CPU指令的操作：
 public final boolean compareAndSet(int expect, int update) {   2     return unsafe.compareAndSwapInt(this, valueOffset, expect, update);3 }　　　
　　　　其中unsafe.compareAndSwapInt(this, valueOffset, expect, update);类似如下逻辑：
 if (this == expect) {2     this = update3     return true;4 } else {5     return false;6 }
　　　　那么比较this == expect，替换this = update，compareAndSwapInt实现这两个步骤的原子性呢？ 参考CAS的原理
　　CAS原理：
　　　　CAS通过调用JNI的代码实现的。而compareAndSwapInt就是借助C来调用CPU底层指令实现的。
　　　　  下面从分析比较常用的CPU（intel x86）来解释CAS的实现原理。
 　　　  下面是sun.misc.Unsafe类的compareAndSwapInt()方法的源代码：
 public final native boolean compareAndSwapInt(Object o, long offset,2                                               int expected,3                                               int x);
　　　　 可以看到这是个本地方法调用。这个本地方法在JDK中依次调用的C++代码为：

 #define LOCK_IF_MP(mp) __asm cmp mp, 0  \ 2                        __asm je L0      \ 3                        __asm _emit 0xF0 \ 4                        __asm L0: 5  6 inline jint     Atomic::cmpxchg    (jint     exchange_value, volatile jint*     dest, jint     compare_value) { 7   // alternative for InterlockedCompareExchange 8   int mp = os::is_MP(); 9   __asm {10     mov edx, dest11     mov ecx, exchange_value12     mov eax, compare_value13     LOCK_IF_MP(mp)14     cmpxchg dword ptr [edx], ecx15   }16 }

　　　　如上面源代码所示，程序会根据当前处理器的类型来决定是否为cmpxchg指令添加lock前缀。如果程序是在多处理器上运行，就为cmpxchg指令加上lock前缀（lock cmpxchg）。反之，如果程序是在单处理器上运行，就省略lock前缀（单处理器自身会维护单处理器内的顺序一致性，不需要lock前缀提供的内存屏障效果）。
 
　　CAS缺点：
　　　　　1. ABA问题：
　　　　　　　比如说一个线程one从内存位置V中取出A，这时候另一个线程two也从内存中取出A，并且two进行了一些操作变成了B，然后two又将V位置的数据变成A，这时候线程one进行CAS操作发现内存中仍然是A，然后one操作成功。尽管线程one的CAS操作成功，但可能存在潜藏的问题。如下所示：
　　　　　　　
　　　　　　　现有一个用单向链表实现的堆栈，栈顶为A，这时线程T1已经知道A.next为B，然后希望用CAS将栈顶替换为B：
　　　　　　　　　　head.compareAndSet(A,B);
　　　　　　　　在T1执行上面这条指令之前，线程T2介入，将A、B出栈，再pushD、C、A，此时堆栈结构如下图，而对象B此时处于游离状态：
　　　　　　　
　　　　　　　此时轮到线程T1执行CAS操作，检测发现栈顶仍为A，所以CAS成功，栈顶变为B，但实际上B.next为null，所以此时的情况变为：
　　　　　　　
　　　　　　　其中堆栈中只有B一个元素，C和D组成的链表不再存在于堆栈中，平白无故就把C、D丢掉了。
　　　　　　　从Java1.5开始JDK的atomic包里提供了一个类AtomicStampedReference来解决ABA问题。这个类的compareAndSet方法作用是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

 public boolean compareAndSet(2                V      expectedReference,//预期引用3 4                V      newReference,//更新后的引用5 6               int    expectedStamp, //预期标志7 8               int    newStamp //更新后的标志9 ) 

　　　　　　　　实际应用代码：
 private static AtomicStampedReference<Integer> atomicStampedRef = new AtomicStampedReference<Integer>(100, 0);2 3 ........4 5 atomicStampedRef.compareAndSet(100, 101, stamp, stamp + 1);
 
 
 　　　　2. 循环时间长开销大：
　　　　　　自旋CAS（不成功，就一直循环执行，直到成功）如果长时间不成功，会给CPU带来非常大的执行开销。如果JVM能支持处理器提供的pause指令那么效率会有一定的提升，pause指令有两个作用，第一它可以延迟流水线执行指令（de-pipeline）,使CPU不会消耗过多的执行资源，延迟的时间取决于具体实现的版本，在一些处理器上延迟时间是零。第二它可以避免在退出循环的时候因内存顺序冲突（memory order violation）而引起CPU流水线被清空（CPU pipeline flush），从而提高CPU的执行效率。
　　　　
　　　　3. 只能保证一个共享变量的原子操作：
　　　　　　当对一个共享变量执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，这个时候就可以用锁，或者有一个取巧的办法，就是把多个共享变量合并成一个共享变量来操作。比如有两个共享变量i＝2,j=a，合并一下ij=2a，然后用CAS来操作ij。从Java1.5开始JDK提供了AtomicReference类来保证引用对象之间的原子性，你可以把多个变量放在一个对象里来进行CAS操作。
 
　　CAS与Synchronized的使用情景：　　　
　　　　1、对于资源竞争较少（线程冲突较轻）的情况，使用synchronized同步锁进行线程阻塞和唤醒切换以及用户态内核态间的切换操作额外浪费消耗cpu资源；而CAS基于硬件实现，不需要进入内核，不需要切换线程，操作自旋几率较少，因此可以获得更高的性能。
　　　 2、对于资源竞争严重（线程冲突严重）的情况，CAS自旋的概率会比较大，从而浪费更多的CPU资源，效率低于synchronized。
　　　补充： synchronized在jdk1.6之后，已经改进优化。synchronized的底层实现主要依靠Lock-Free的队列，基本思路是自旋后阻塞，竞争切换后继续竞争锁，稍微牺牲了公平性，但获得了高吞吐量。在线程冲突较少的情况下，可以获得和CAS类似的性能；而线程冲突严重的情况下，性能远高于CAS。
concurrent包的实现：
　　　　由于java的CAS同时具有 volatile 读和volatile写的内存语义，因此Java线程之间的通信现在有了下面四种方式：
　　　　　　1. A线程写volatile变量，随后B线程读这个volatile变量。
　　　　　　2. A线程写volatile变量，随后B线程用CAS更新这个volatile变量。
　　　　　　3. A线程用CAS更新一个volatile变量，随后B线程用CAS更新这个volatile变量。
　　　　　　4. A线程用CAS更新一个volatile变量，随后B线程读这个volatile变量。
　　　　Java的CAS会使用现代处理器上提供的高效机器级别原子指令，这些原子指令以原子方式对内存执行读-改-写操作，这是在多处理器中实现同步的关键（从本质上来说，能够支持原子性读-改-写指令的计算机器，是顺序计算图灵机的异步等价机器，因此任何现代的多处理器都会去支持某种能对内存执行原子性读-改-写操作的原子指令）。同时，volatile变量的读/写和CAS可以实现线程之间的通信。把这些特性整合在一起，就形成了整个concurrent包得以实现的基石。如果我们仔细分析concurrent包的源代码实现，会发现一个通用化的实现模式：
　　　　　　1. 首先，声明共享变量为volatile；　　
　　　　　　2. 然后，使用CAS的原子条件更新来实现线程之间的同步；
　　　　　　3. 同时，配合以volatile的读/写和CAS所具有的volatile读和写的内存语义来实现线程之间的通信。

最近在看jdk7中java.util.concurrent下面的源码中，发现许多类中使用了Unsafe类中的方法来保证并发的安全性，而java 7 api中并没有这个类的相关介绍，在网上查了许多资料，其中http://ifeve.com/sun-misc-unsafe/这个网站详细的讲解了Unsafe的相关用法，而下面是结合网站中的介绍和具体的AtomicInteger类来讲解一下其相关的用法。
[java] view plain copy
private static final Unsafe unsafe = Unsafe.getUnsafe();  
private static final long valueOffset;  
  
static {  
    try {  
           valueOffset = unsafe.objectFieldOffset  
           (AtomicInteger.class.getDeclaredField("value"));  
    } catch (Exception ex) { throw new Error(ex); }  
}  
  
private volatile int value;  
首先可以看到AtomicInteger类在域中声明了这两个私有变量unsafe和valueOffset。其中unsafe实例采用Unsafe类中静态方法getUnsafe()得到，但是这个方法如果我们写的时候调用会报错，因为这个方法在调用时会判断类加载器，我们的代码是没有“受信任”的，而在jdk源码中调用是没有任何问题的；valueOffset这个是指类中相应字段在该类的偏移量，在这里具体即是指value这个字段在AtomicInteger类的内存中相对于该类首地址的偏移量。
然后可以看一个有一个静态初始化块，这个块的作用即是求出value这个字段的偏移量。具体的方法使用的反射的机制得到value的Field对象，再根据objectFieldOffset这个方法求出value这个变量内存中在该对象中的偏移量。
volatile关键字保证了在多线程中value的值是可见的，任何一个线程修改了value值，会将其立即写回内存当中
[java] view plain copy
1.public final int getAndSet(int newValue) {  
2.    for (;;) {  
3.        int current = get();  
4.        if (compareAndSet(current, newValue))  
5.            return current;  
6.    }  
7.}  
8.  
9.public final boolean compareAndSet(int expect, int update) {  
10.    return unsafe.compareAndSwapInt(this, valueOffset, expect, update);  
11.}  
getAndSet这个方法作用为将value值设置为newValue，并返回修改前的value值
在for循环中，保证每次如果compareAndSet这个方法失败之后，能重新进行尝试，直到成功将value值设置为newValue。
compareAndSet这个方法主要调用unsafe.compareAndSwapInt这个方法，这个方法有四个参数，其中第一个参数为需要改变的对象，第二个为偏移量(即之前求出来的valueOffset的值)，第三个参数为期待的值，第四个为更新后的值。整个方法的作用即为若调用该方法时，value的值与expect这个值相等，那么则将value修改为update这个值，并返回一个true，如果调用该方法时，value的值与expect这个值不相等，那么不做任何操作，并范围一个false。
因此之所以在getAndSet方法中调用一个for循环，即保证如果调用compareAndSet这个方法返回为false时，能再次尝试进行修改value的值，直到修改成功，并返回修改前value的值。
整个代码能保证在多线程时具有线程安全性，并且没有使用java中任何锁的机制，所依靠的便是Unsafe这个类中调用的该方法具有原子性，这个原子性的保证并不是靠java本身保证，而是靠一个更底层的与操作系统相关的特性实现。


（1）从数据库系统的角度来看，锁分为以下三种类型：
独占锁（Exclusive Lock）
独占锁锁定的资源只允许进行锁定操作的程序使用，其它任何对它的操作均不会被接受。执行数据更新命令，即INSERT、 UPDATE 或DELETE 命令时，SQL Server 会自动使用独占锁。但当对象上有其它锁存在时，无法对其加独占锁。独占锁一直到事务结束才能被释放。
共享锁（Shared Lock）
共享锁锁定的资源可以被其它用户读取，但其它用户不能修改它。在SELECT 命令执行时，SQL Server 通常会对对象进行共享锁锁定。通常加共享锁的数据页被读取完毕后，共享锁就会立即被释放。
更新锁（Update Lock）
更新锁是为了防止死锁而设立的。当SQL Server 准备更新数据时，它首先对数据对象作更新锁锁定，这样数据将不能被修改，但可以读取。等到SQL Server 确定要进行更新数据操作时，它会自动将更新锁换为独占锁。但当对象上有其它锁存在时，无法对其作更新锁锁定。

（2）从程序员的角度看，锁分为以下两种类型：
悲观锁（Pessimistic Lock）
悲观锁，正如其名，它指的是对数据被外界（包括本系统当前的其他事务，以及来自外部系统的事务处理）修改持保守态度，因此在整个数据处理过程中，将数据处于锁定状态。悲观锁的实现，往往依靠数据库提供的锁机制（也只有数据库层提供的锁机制才能真正保证数据访问的排他性，否则，即使在本系统中实现了加锁机制，也无法保证外部系统不会修改数据）。
乐观锁（Optimistic Lock）
相对悲观锁而言，乐观锁机制采取了更加宽松的加锁机制。悲观锁大多数情况下依靠数据库的锁机制实现，以保证操作最大程度的独占性。但随之而来的就是数据库性能的大量开销，特别是对长事务而言，这样的开销往往无法承受。
而乐观锁机制在一定程度上解决了这个问题。乐观锁，大多是基于数据版本（ Version ）记录机制实现。何谓数据版本？即为数据增加一个版本标识，在基于数据库表的版本解决方案中，一般是通过为数据库表增加一个 “version” 字段来实现。读取出数据时，将此版本号一同读出，之后更新时，对此版本号加一。此时，将提交数据的版本数据与数据库表对应记录的当前版本信息进行比对，如果提交的数据版本号大于数据库表当前版本号，则予以更新，否则认为是过期数据。

简要说明为什么会发生死锁?解决死锁的主要方法是什么?
若干事务相互等待释放封锁，就陷入无限期等待状态，系统就进入死锁
[cpp] view plain copy 
(1). 测试用的基础数据：  
   
CREATE  TABLE  Lock1(C1  int  default ( 0 ));  
CREATE  TABLE  Lock2(C1  int  default ( 0 ));  
INSERT  INTO  Lock1  VALUES( 1 );  
INSERT  INTO  Lock2  VALUES( 1 );  
   
(2). 开两个查询窗口，分别执行下面两段 sql  
-- Query 1   
Begin  Tran   
  Update Lock1  Set C1 = C1 + 1 ;  
  WaitFor  Delay  ' 00:01:00 ' ;  
  SELECT  *   FROM Lock2  
  Rollback  Tran ;  
   
-- Query 2   
 Begin  Tran   
   Update Lock2  Set C1 = C1 + 1 ;  
   WaitFor  Delay  ' 00:01:00 ' ;  
   SELECT  *   FROM Lock1  
   Rollback  Tran ;  
     
（3）、测试完后删除这两张表。  
   droptable Lock1;  
   droptable Lock2;  


解决死锁的方法应从预防和解除的两个方面着手：
(1)死锁的预防方法：
a、要求每一个事务必须一次封锁所要使用的全部数据（要么全成功，要么全不成功）
b、规定封锁数据的顺序，所有事务必须按这个顺序实行封锁。
(2)允许死锁发生，然后解除它，如果发现死锁，则将其中一个代价较小的事物撤消，回滚这个事务，并释放此事务持有的封锁，使其他事务继续运行。
oracle:
共享锁
[sql] view plaincopy
1.LOCK TABLE 表 IN SHARE MODE ;  

排他锁：
[sql] view plaincopy
1.LOCK TABLE 表 IN EXCLUSIVE MODE ;  

加锁后其它人不可操作，直到加锁用户解锁，用commit或rollback解锁


行排他锁不阻止其他Session申请表共享锁和其他行的排他锁，但阻止申请表排他锁和锁定行的任何锁。
表排他锁阻止其他Session的申请的所有锁。
表共享锁不阻止其他Session申请行排他锁和表共享锁，但阻止申请表排他锁。
