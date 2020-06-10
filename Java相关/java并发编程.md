1.java线程池大小为何会大多被设置成CPU核心数+1<br>

线程池究竟设成多大是要看你给线程池处理什么样的任务，任务类型不同，线程池大小的设置方式也是不同的。<br>

任务一般可分为：CPU密集型、IO密集型、混合型，对于不同类型的任务需要分配不同大小的线程池。<br>

CPU密集型任务 <br>
尽量使用较小的线程池，一般为CPU核心数+1。 <br>
因为CPU密集型任务使得CPU使用率很高，若开过多的线程数，只能增加上下文切换的次数，因此会带来额外的开销。<br>
IO密集型任务 <br>
可以使用稍大的线程池，一般为2*CPU核心数。<br> 
IO密集型任务CPU使用率并不高，因此可以让CPU在等待IO的时候去处理别的任务，充分利用CPU时间。<br>
混合型任务 <br>
可以将任务分成IO密集型和CPU密集型任务，然后分别用不同的线程池去处理。 <br>
只要分完之后两个任务的执行时间相差不大，那么就会比串行执行来的高效。<br><br> 
因为如果划分之后两个任务执行时间相差甚远，那么先执行完的任务就要等后执行完的任务，最终的时间仍然取决于后执行完的任务，而且还要加上任务拆分与合并的开销，得不偿失

对于计算密集型的程序，线程数应当等于核心数，但是再怎么计算密集，总有一些IO吧，所以再加一个线程来把等待IO的CPU时间利用起来<br>
对于计算密集型的任务，在拥有N个处理器的系统上，当线程池的大小为N+1时，通常能实现最优的效率。(即使当计算密集型的线程偶尔由于缺失故障或者其他原因而暂停时，
这个额外的线程也能确保CPU的时钟周期不会被浪费。)<br>

2.什么是CPU密集型、IO密集型？<br>
CPU密集型（CPU-bound）<br>
CPU密集型也叫计算密集型，指的是系统的硬盘、内存性能相对CPU要好很多，此时，系统运作大部分的状况是CPU Loading 100%，CPU要读/写I/O(硬盘/内存)，I/O在很短的时间就可以完成，而CPU还有许多运算要处理，CPU Loading很高。

在多重程序系统中，大部份时间用来做计算、逻辑判断等CPU动作的程序称之CPU bound。例如一个计算圆周率至小数点一千位以下的程序，在执行的过程当中绝大部份时间用在三角函数和开根号的计算，便是属于CPU bound的程序。

CPU bound的程序一般而言CPU占用率相当高。这可能是因为任务本身不太需要访问I/O设备，也可能是因为程序是多线程实现因此屏蔽掉了等待I/O的时间。

IO密集型（I/O bound）<br>
IO密集型指的是系统的CPU性能相对硬盘、内存要好很多，此时，系统运作，大部分的状况是CPU在等I/O (硬盘/内存) 的读/写操作，此时CPU Loading并不高。<br>

I/O bound的程序一般在达到性能极限时，CPU占用率仍然较低。这可能是因为任务本身需要大量I/O操作，而pipeline做得不是很好，没有充分利用处理器能力。<br><br>

CPU密集型 vs IO密集型<br>
我们可以把任务分为计算密集型和IO密集型。<br>
<br>
计算密集型任务的特点是要进行大量的计算，消耗CPU资源，比如计算圆周率、对视频进行高清解码等等，全靠CPU的运算能力。这种计算密集型任务虽然也可以用多任务完成，但是任务越多，花在任务切换的时间就越多，CPU执行任务的效率就越低，所以，要最高效地利用CPU，计算密集型任务同时进行的数量应当等于CPU的核心数。

计算密集型任务由于主要消耗CPU资源，因此，代码运行效率至关重要。Python这样的脚本语言运行效率很低，完全不适合计算密集型任务。对于计算密集型任务，最好用C语言编写。

第二种任务的类型是IO密集型，涉及到网络、磁盘IO的任务都是IO密集型任务，这类任务的特点是CPU消耗很少，任务的大部分时间都在等待IO操作完成（因为IO的速度远远低于CPU和内存的速度）。对于IO密集型任务，任务越多，CPU效率越高，但也有一个限度。常见的大部分任务都是IO密集型任务，比如Web应用。

IO密集型任务执行期间，99%的时间都花在IO上，花在CPU上的时间很少，因此，用运行速度极快的C语言替换用Python这样运行速度极低的脚本语言，完全无法提升运行效率。对于IO密集型任务，最合适的语言就是开发效率最高（代码量最少）的语言，脚本语言是首选，C语言最差。

总之，计算密集型程序适合C语言多线程，I/O密集型适合脚本语言开发的多线程。<br>

3.线程池中excute与submit方法的区别<br>
3.1. 接收的参数不一样;<br>
execute()方法的入参为一个实现了Runnable接口的类，返回值为void<br>
sumbit()方法，入参可以为实现了Callable<T>接口的类，也可以为实现了Runnable的类，而且方法有返回值Future<T>；<br>
3.2.submit()有返回值，而execute()没有;<br>
例如，有个validation的task，希望该task执行完后告诉我它的执行结果，是成功还是失败，然后继续下面的操作。<br>
3.3. submit()可以进行Exception处理;<br>
例如，如果task里会抛出checked或者unchecked exception，而你又希望外面的调用者能够感知这些exception并做出及时的处理，<br>
那么就需要用到submit，通过对Future.get()将抛出异常的捕获，然后对其进行处理。<br>

4.java并发容器-队列<br>

实现一个线程安全的队列有两种实现方式：一种是使用阻塞算法，阻塞队列就是通过使用加锁的阻塞算法实现的；
另一种非阻塞的实现方式则可以使用循环CAS（比较并交换）的方式来实现<br>
ConcurrentLinkedQueue是一个基于链表实现的无界线程安全队列，它采用先进先出的规则对节点进行排序，当我们添加一个元素的时候，它会添加到队列的尾部，
当我们获取一个元素时，它会返回队列头部的元素。<br>
一：入队<br>
![](https://images2015.cnblogs.com/blog/1018541/201703/1018541-20170313214020416-738374759.jpg)<br>
入队主要做两件事情：<br>
1.定位尾节点：每次入队都必须通过tail节点来找到尾节点。尾节点可能是tail节点，也可能是tail节点的next节点<br>
2.使用CAS算法将入队节点设置成尾节点的next节点，如果tail节点为尾节点，则设置tail节点的next节点为入队节点<br>
如果tail节点的next节点为尾节点，表示有其他线程更新了尾节点，则需要重新获取当前队列的尾节点。<br>
二：出队<br>
![](https://images2015.cnblogs.com/blog/1018541/201703/1018541-20170313211713838-48054664.jpg)<br>
首先获取头结点的元素，然后判断头节点元素是否为空，如果为空，表示另外一个线程已经进行了一次出队操作将该节点的元素取走，如果不为空，则使用
CAS的方式将头节点的引用设置成null，如果CAS成功，则直接返回头节点的元素，如果不成功，表示另外一个线程已经进行了一次出队操作更新了头结点
需要重新获取头结点。<br>
三：非阻塞却线程安全的原因<br>
ConcurrentLinkedQueue的线程安全是通过其插入、删除时采取CAS操作来保证的<br>
如果有其他任何一个线程成功执行了插入、删除都会改变tail/head结点，那么当前线程的插入和删除操作就会失败，<br>
则通过循环再次定位tail、head结点位置进行插入、删除，直到成功为止。<br>
四：非阻塞队列中的几个主要方法：<br>

　　add(E e):将元素e插入到队列末尾，如果插入成功，则返回true；如果插入失败（即队列已满），则会抛出异常；<br>

　　remove()：移除队首元素，若移除成功，则返回true；如果移除失败（队列为空），则会抛出异常；<br>

　　offer(E e)：将元素e插入到队列末尾，如果插入成功，则返回true；如果插入失败（即队列已满），则返回false；<br>

　　poll()：移除并获取队首元素，若成功，则返回队首元素；否则返回null；<br>

　　peek()：获取队首元素，若成功，则返回队首元素；否则返回null<br>
　　对于非阻塞队列，一般情况下建议使用offer、poll和peek三个方法，不建议使用add和remove方法。因为使用offer、poll和peek三个方法可以通过返回值判断操作成功与否，
  而使用add和remove方法却不能达到这样的效果。<br>

阻塞队列<br>
阻塞队列(BlockingQueue)是一个支持两个附加操作的队列。这两个附加的操作是：在队列为空时，获取元素的线程会等待队列变为非空。当队列满时，存储元素的线程会等待队列可用。<br>
阻塞队列常用于生产者和消费者的场景，生产者是往队列里添加元素的线程，消费者是从队列里拿元素的线程。<br>
阻塞队列包括了非阻塞队列中的大部分方法，上面列举的5个方法在阻塞队列中都存在，但是要注意这5个方法在阻塞队列中都进行了同步措施。除此之外，阻塞队列提供了另外4个非常有用的方法：<br>
JDK7提供了7个阻塞队列。分别是<br>
ArrayBlockingQueue ：一个由数组结构组成的有界阻塞队列。<br>
LinkedBlockingQueue ：一个由链表结构组成的有界阻塞队列。<br>
PriorityBlockingQueue ：一个支持优先级排序的无界阻塞队列。<br>
DelayQueue：一个使用优先级队列实现的无界阻塞队列。<br>
SynchronousQueue：一个不存储元素的阻塞队列。<br>
LinkedTransferQueue：一个由链表结构组成的无界阻塞队列。<br>
LinkedBlockingDeque：一个由链表结构组成的双向阻塞队列。<br>
　　put(E e)<br>

　　take()<br>

　　offer(E e,long timeout, TimeUnit unit)<br>

　　poll(long timeout, TimeUnit unit)<br>

　　put方法用来向队尾存入元素，如果队列满，则等待；<br>

　　take方法用来从队首取元素，如果队列为空，则等待；<br>

　　offer方法用来向队尾存入元素，如果队列满，则等待一定的时间，当时间期限达到时，如果还没有插入成功，则返回false；否则返回true；<br>

　　poll方法用来从队首取元素，如果队列空，则等待一定的时间，当时间期限达到时，如果取到，则返回null；否则返回取得的元素；<br>
  以ArrayBlockingQueue为例<br>
  ArrayBlockingQueue中用来存储元素的实际上是一个数组<br>
  然后看它的两个关键方法的实现：put()和take()：<br>
  public void put(E e) throws InterruptedException {<br>
    >if (e == null) throw new NullPointerException();<br>
    >final E[] items = this.items;<br>
    >final ReentrantLock lock = this.lock;<br>
    >lock.lockInterruptibly();<br>
    >try {<br>
       >> try {<br>
            >>>while (count == items.length)<br>
                >>>>notFull.await();<br>
        >>} catch (InterruptedException ie) {<br>
            >>>>notFull.signal(); // propagate to non-interrupted thread<br>
            >>>>throw ie;<br>
       >> }<br>
       >>>> insert(e);<br>
    >>} finally {<br>
        >>>>lock.unlock();<br>
    >>}<br>
>}<br>
从put方法的实现可以看出，它先获取了锁，并且获取的是可中断锁（ReentrantLock保证公平性，阻塞的线程可以按照阻塞的先后顺序访问队列），<br>
然后判断当前元素个数是否等于数组的长度，如果相等，则调用notFull.await()进行等待，如果捕获到中断异常，则唤醒线程并抛出异常。<br>
当被其他线程唤醒时，通过insert(e)方法插入元素，最后解锁。<br>
下面是take()方法的实现<br>
public E take() throws InterruptedException {<br>
    >final ReentrantLock lock = this.lock;<br>
    >lock.lockInterruptibly();<br>
    >try {<br>
        >>try {<br>
           >>> while (count == 0)<br>
                >>>>notEmpty.await();<br>
        >>} catch (InterruptedException ie) {<br>
            >>>>notEmpty.signal(); // propagate to non-interrupted thread<br>
            >>>>throw ie;<br>
        >>}<br>
        >>>>E x = extract();<br>
        >>>>return x;<br>
    >>} finally {<br>
        >>>>lock.unlock();<br>
    >>}<br>
>}<br>
从put方法的实现可以看出，它先获取了锁，并且获取的是可中断锁（ReentrantLock保证公平性，阻塞的线程可以按照阻塞的先后顺序访问队列），<br>
然后判断当前元素个数是否等于0，如果相等，则调用notEmpty.await()进行等待，如果捕获到中断异常，则唤醒线程并抛出异常。<br>
当被其他线程唤醒时，通过extract()方法获取元素，最后解锁。<br>

ConcurrentLinkedQueue入队和出队操作均利用CAS（compare and set）更新，这样允许多个线程并发执行，并且不会因为加锁而阻塞线程，使得并发性能更好。

4.Thread.interrupt()
https://blog.csdn.net/tianyuxingxuan/article/details/76222935
https://www.cnblogs.com/skywang12345/p/3479949.html
采用supend,stop在调用之后不会释放线程已经占有的资源，比如锁，而是占着资源进入睡眠状态，这样容易引发死锁
4.1通过中断的方式终止处于“阻塞状态”的线程
当线程由于被调用了sleep(), wait(), join()等方法而进入阻塞状态；若此时调用线程的interrupt()将线程的中断标记设为true。由于处于阻塞状态，中断标记会被清除，同时产生一个InterruptedException异常。将InterruptedException放在适当的为止就能终止线程，形式如下：
public void run() {
    try {
        while (true) {
            // 执行任务...
        }
    } catch (InterruptedException ie) {  
        // 由于产生InterruptedException异常，退出while(true)循环，线程终止！
    }
}
说明：在while(true)中不断的执行任务，当线程处于阻塞状态时，调用线程的interrupt()产生InterruptedException中断。中断的捕获在while(true)之外，这样就退出了while(true)循环！

4.2终止处于"运行状态"下的线程
  (01) 通过“中断标记”终止线程。
  public void run() {
    while (!isInterrupted()) {
        // 执行任务...
     }
  }
  说明：isInterrupted()是判断线程的中断标记是不是为true。当线程处于运行状态，并且我们需要终止它时；可以调用线程的interrupt()方法，使用线程的中断标   记为true，即isInterrupted()会返回true。此时，就会退出while循环。
  
  (02) 通过“额外添加标记”
   private volatile boolean flag= true;
   protected void stopTask() {
      flag = false;
   }

   @Override
   public void run() {
     while (flag) {
        // 执行任务...
     }
  }
 说明：线程中有一个flag标记，它的默认值是true；并且我们提供stopTask()来设置flag标记。当我们需要终止该线程时，调用该线程的stopTask()方法就可以让线  程退出while循环。
 
5.ThreadLocal
https://www.cnblogs.com/dolphin0520/p/3920407.html
https://www.cnblogs.com/aspirant/p/8991010.html

6.ReentranLock
https://blog.csdn.net/u011521203/article/details/80186741

7.再对缓存进行增删改的时候必须与缓存的同步操作进行synchronized隔离，避免增删改与同步操作同时进行，就比如说在添加实例的时候，先添加到资源那边，然后我们就同步了缓存，最后再去添加实例到缓存，这个时候在缓存就会出现相同的实例，所以添加操作要与同步操作隔离开，要么添加完了再去资源同步缓存，或者要么同步完缓存再去执行添加操作。

缓存的使用应该是先去缓存中去查数据，如果没有的话再去数据库查询，数据库查到了的话再添加到缓存中。

什么情况下才会用到缓存呢？有些数据长时间不会发生变动，但是用户又访问特别频繁

8.condition的实现分析
http://www.importnew.com/30150.html

9.CountDownLatch典型用法1：某一线程在开始运行前等待n个线程执行完毕。将CountDownLatch的计数器初始化为n new CountDownLatch(n) ，每当一个任务线程执行完毕，就将计数器减1 countdownlatch.countDown()，当计数器的值变为0时，在CountDownLatch上 await() 的线程就会被唤醒。一个典型应用场景就是启动一个服务时，主线程需要等待多个组件加载完毕，之后再继续执行。

实现原理：new CountDownLatch(n)初始化计数器为n,
         CountDownLatch.countDown()调用AQS的tryReleaseShared将采用CAS方式将计数器进行减一
         CountDownLatch.await()调用AQS的tryAcquireShared方法判断计数器的值是否减为了0，如果为0则退出死循环，否则则一直循环下去，从而达到阻塞当前
         线程的效果
         
10.volatile没有原子性举例：AtomicInteger自增
例如你让一个volatile的integer自增（i++），其实要分成3步：1）读取volatile变量值到local； 2）增加变量的值；3）把local的值写回，让其它的线程可见。这3步的jvm指令为：
mov   0xc(%r10),%r8d ; Load
inc    %r8d           ; Increment
mov    %r8d,0xc(%r10) ; Store
lock addl $0x0,(%rsp) ; StoreLoad Barrier
注意最后一步是内存屏障。    

什么是内存屏障（Memory Barrier）？
内存屏障（memory barrier）是一个CPU指令。基本上，它是这样一条指令： a) 确保一些特定操作执行的顺序； b) 影响一些数据的可见性(可能是某些指令执行后的结果)。编译器和CPU可以在保证输出结果一样的情况下对指令重排序，使性能得到优化。插入一个内存屏障，相当于告诉CPU和编译器先于这个命令的必须先执行，后于这个命令的必须后执行。内存屏障另一个作用是强制更新一次不同CPU的缓存。例如，一个写屏障会把这个屏障前写入的数据刷新到缓存，这样任何试图读取该数据的线程将得到最新值，而不用考虑到底是被哪个cpu核心或者哪颗CPU执行的。

内存屏障（memory barrier）和volatile什么关系？上面的虚拟机指令里面有提到，如果你的字段是volatile，Java内存模型将在写操作后插入一个写屏障指令，在读操作前插入一个读屏障指令。这意味着如果你对一个volatile字段进行写操作，你必须知道：1、一旦你完成写入，任何访问这个字段的线程将会得到最新的值。2、在你写入前，会保证所有之前发生的事已经发生，并且任何更新过的数据值也是可见的，因为内存屏障会把之前的写入值都刷新到缓存。

volatile为什么没有原子性?
明白了内存屏障（memory barrier）这个CPU指令，回到前面的JVM指令：从Load到store到内存屏障，一共4步，其中最后一步jvm让这个最新的变量的值在所有线程可见，也就是最后一步让所有的CPU内核都获得了最新的值，但中间的几步（从Load到Store）是不安全的，中间如果其他的CPU修改了值将会丢失。下面的测试代码可以实际测试voaltile的自增没有原子性：

 private static volatile long _longVal = 0;
     
    private static class LoopVolatileimplements Runnable {
        public void run() {
            long val = 0;
            while (val < 10000000L) {
                _longVal++;
                val++;
            }
        }
    }
     
    private static class LoopVolatile2implements Runnable {
        public void run() {
            long val = 0;
            while (val < 10000000L) {
                _longVal++;
                val++;
            }
        }
    }
     
    private  void testVolatile(){
        Thread t1 = new Thread(new LoopVolatile());
        t1.start();
         
        Thread t2 = new Thread(new LoopVolatile2());
        t2.start();
         
        while (t1.isAlive() || t2.isAlive()) {
        }
 
        System.out.println("final val is: " + _longVal);
    }
 
Output:-------------
     
final val is: 11223828
final val is: 17567127
final val is: 12912109
