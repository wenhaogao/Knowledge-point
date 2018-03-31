###       细说ConcurrentHashMap
#####  同学老款面试中。。。
面试官：你是老款吧！请简单的简绍一下你自己。

老款：我是来自bala bala bala......。

面试官：Java中的集合框架了解过吗？你能说一下集合中的HashMap吗？

老款：HashMap我用的还比较多，它的原理是@#￥%......&*。

面试官：很好。HashMap是线程安全的吗？

老款：HashMap并不是线程安全的。当它在并发插入元素的时候，有可能出现环链表，使下一次操作出现死循环。

面试官：额，那么什么样的哈希数据结构可以保证线程的安全呢？

老款：好像是......ConcurrentHashMap吧！
面试官：那你说说在并发场景下，它是怎么能保证线程的安全的？怎么提高读写性能的？

老款：......!

面试官：......!

在并发编程中，ConcurrentHashMap是一个经常被使用的数据结构，为了使大家不要向老款一样和面试官尬聊，接下来我将带领大家深入了解ConcurrentHashMap的底层结构。

#### 首先给大家抛出几个问题：
a、ConcurrentHashMap在put的时候，key经过几次hash计算？

b、segment 会增大吗？

c、新的值是放在链表的表头还是表尾？

##### 什么是ConcurrentHashMap?为什么要引ConcurrentHashMap？
原因是:

<1>HashMap线程是不安全，它的线程不安全主要发生在put等对HashEntry有直接写操作的地方：

有兴趣的同学可以下去看看源码，HashMap线程不安全主要可能发生在这两个地方：

* key已经存在，需要修改HashEntry对应的value；
* key不存在，在HashEntry中做插入。

<2>Hashtable线程安全，但是效率低下：

Hashtable是用synchronized关键字来保证线程安全的，但synchronized的机制是在同一时刻只能有一个线程操作，其他的线程阻塞或者轮询等待，在线程竞争激烈的情况下，这种方式的效率会非常的低下。

ConcurrentHashMap是线程安全并且高效的HashMap，在并发编程中经常可见它的使用，下面将为小伙伴们进行详细的简述。

##### ConcurrentHashMap为什么高效？
Hashtable效率低的原因是所有访问Hashtable的线程都争夺一把锁。如果容器有很多把锁，每一把锁控制容器中的一部分数据，那么当多个线程访问容器里的不同部分的数据时，线程之前就不会存在锁的竞争，这样就可以有效的提高并发的访问效率。这也正是ConcurrentHashMap使用的分段锁技术。将ConcurrentHashMap容器的数据分段存储，每一段数据分配一个Segment（锁），当线程占用其中一个Segment时，其他线程可正常访问其他段数据。

##### 那么ConcurrentHashMap里为什么分segment呢？
这也是ConcurrentHashMap高明之处，我们都知道锁只在segment中存在，这样就把锁的粒度变小，提高并发，同时还是线程安全的，
ConcurrentHashMap的结构图：
https://github.com/wenhaogao/Knowledge-point/blob/dev/image/%E5%9B%BE%E7%89%871.png
* Segment是可重入锁，它在ConcurrentHashMap中扮演分离锁的角色；
* HashEntry主要存储键值对；
* ConcurrentHashMap包含一个Segment数组，每个Segment包含一个HashEntry数组并且守护它，当修改HashEntry数组数据时，需要先获取它对应的Segment锁；而HashEntry数组采用开链法处理冲突，所以它的每个HashEntry元素又是链表结构的元素。
* 
##### Segment定位
Hash算法
ConcurrentHashMap使用分段锁segment来保护数据，也就是说，在插入和读取元素，需要先通过hash算法定位segment。ConcurrentHashMap使用了变种hash算法对元素的hashCode再散列。

##### 思考：为什么需要再散列？
再散列的目的是为了减少冲突，让元素可以近似均匀的分布在不同的Segment上，从而提升存储效率。如果hash算法不好，最差的情况是所有的元素都在一个Segment中，这时候hash表将退化成链表，查询插入的时间复杂度都会从理想的o(1)退化成o(n^2)，同时，分段锁也会失去存在的意义。
##### ConcurrentHashMap相关操作实现分析
ConcurrentHashMap常用的三个操作：get/put/size的具体实现。
##### get操作
1.为输入的Key做Hash运算，得到hash值。

2.通过hash值，定位到对应的Segment对象

3.再次通过hash值，定位到Segment当中数组的具体位置。

比起Hashtable，ConcurrentHashMap的get操作高效之处在于整个get操作不需要加锁。如果不加锁，ConcurrentHashMap的get操作是如何做到线程安全的呢？原因是volatile，所有的value都定义成了volatile类型，volatile可以保证线程之间的可见性，这也是用volatile替换锁的经典应用场景。
##### put操作
ConcurrentHashMap提供两个方法put和putIfAbsent来完成put操作。
* put()方法做插入时key存在会更新key所对应的value 
* putIfAbsent()方法做插入时key存在不会更新key所对应的value
##### Put方法：
* 1.为输入的Key做Hash运算，得到hash值。
* 2.通过hash值，定位到对应的Segment对象
* 3.获取可重入锁
* 4.再次通过hash值，定位到Segment当中数组的具体位置。
* 5.插入或覆盖HashEntry对象。
* 6.释放锁。
##### size实现
如果要计算ConcurrentHashMap的容量，必须统计所有Segment容量然后求和，Segment提供变量count用于存储当前Segment的容量。但是ConcurrentHashMap为了保证线程安全，并不是直接把所有的Segment的count相加来得到整个容器的大小，下面我们来看看ConcurrentHashMap是怎么来统计容量的。

由于在累加count的操作的过程中之前累加过的count发生变化的几率非常小，所以ConcurrentHashMap先尝试2次不锁住Segment的方式来统计每个Segment的大小，如果在统计的过程中Segment的count发生了变化，这时候再加锁统计Segment的count。
##### ConcurrentHashMap如何判断统计过程中Segment的cout发生了变化？
Segment使用变量modCount来表示Segment大小是否发生变化，在put/remove/clean操作里都会将modCount加1，那么在统计size的前后只需要比较modCount是否发生了变化，如果发生变化，Segment的大小肯定发生了变化。
##### 文章上方的答案：
a、一次    b、不会   c、表头
