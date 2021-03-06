# 什么是JUC

1. 什么是进程

   在计算机中，把一个任务称为一个进程，浏览器就是一个进程，微信也是一个进程；是程序的一次执行过程，是系统运行程序的基本单位，进程包含线程多个或者至少一个线程；

2. 什么是线程

   是比进程更小的执行单位，一个进程在执行过程中可以产生多个线程，同类的线程共享堆，方法区和元空间，每个线程又有自己的计数器，虚拟机栈，因为线程又称轻量级进程；

3. 进程与线程

   进程和线程是包含关系，但是多任务可以多进程来实现，也可让单线程的内部多线程实现；

   和多线程相比，多进程的缺点在于：

   1. 创建进程比创建线程开销大，尤其是在Windows机器上
   2. 进程间通信比线程间通信要慢，因线程间的通信就是读写同一个变量，速度快；

   优点在于：多进程的稳定性高，多进程的情况下，一个进程的崩溃不会影响到其他进程，而在多线程情况下，一个线程崩溃会导致整个进程崩溃

4. 什么是并发和并行

   1. 并发是指计算机在同一时间段内处理多任务的执行，如我和小明同时访问淘宝网站，淘宝服务器就同时处理我和小明的访问请求
   2. 并行是指多任务同时执行，但是任务之间没有任何关系，不共享资源；如一边听歌，一边写作

5. 线程的几种状态

   NEW新建，RUNNABLE可运行，BLOCKED阻塞，WAITTING等待，TIMED_WAITTING计时等待，TERMINATED结束

6. 线程有几种创建方式

   1. 继承Thread类，实现run方法
   2. 实现Runnable接口，实现run方法
   3. 直接 new Thread（（）->{}，ThreadName）；

# 锁之提纲

1. 互斥锁

   - 互斥锁是一种独占锁，比如线程A加锁成功之后，此时互斥锁已经被线程A独占了，只要线程A没有释放手中的锁，线程B加锁就会失败，于是就会释放CPU让给其他线程，**既然线程B释放了CPU，线程B自然加锁的代码会被阻塞；所以互斥锁加锁失败而阻塞的现象是由系统内核实现的**，线程加锁失败时，内核会将该线程的状态从运行状态设置为睡眠状态，把CPU切给其他线程运行；当锁被释放时，之前睡眠状态的线程会变为运行，然后内核会在合适的时间把CPU切换给该线程运行；

   - 这种CPU切换是叫线程上下文切换，当两个线程是属于同一个进程，虚拟内存是共享的，所以在切换时，虚拟内存这些资源就保持不动，切换线程的私有数据，寄存器等不共享的数据

2. 自旋锁

   - 自旋锁是通过CPU提供的CAS函数完成加锁和解锁的操作，不会产生线程上下文切换，相比互斥锁会快一点；
   - 加锁两步骤：1.查看锁的状态，如果锁是空闲的，执行第二步。2.将锁设置为当前线程持有
   - CAS函数是将加锁两步骤合并成一条硬件级指令，形成原子指令
   - 使用自旋锁的时候，当发生多线程竞争锁的情况时，加锁的失败线程会 **忙等待**，直到锁释放，自旋锁在单CPU上无法使用，因为一个自旋的线程永远不会放弃CPU
   - 自旋锁开销少，**因为在多核系统下不会产生上下文线程切换**，适合异步，协程；如果锁住的代码执行时间长，自旋的线程会长时间占用CPU资源，所以自旋的时间和被锁住的代码执行时间是成正比的

3. 读写锁

   读写锁是独占锁共享锁的具体实现，写锁允许一个线程写入，读锁允许多个线程在没有写入时同时读取

4. 悲观锁

   在进入同步方法的时候都会获取当前同步锁对象，直到退出同步方法时才会释放同步锁对象；

5. 乐观锁

   是一种读多写少，遇到并发写的可能性较小，每次去拿数据的时候认为不会修改，所以不会上锁；Java乐观锁都是通过CAS操作实现的

6. 公平锁

   根据线程在队列的优先级获取锁，比如线程优先加入阻塞队列，线程就会优先获得锁

7. 非公平锁

   指在获取锁的时候，每个线程都会去争抢，并且都有机会获取到锁，无关优先级

8. 可重入锁

   一个线程获取到锁之后，如果继续遇到相同锁修饰的方法或者代码块，那么可以继续获取该锁，对synchronized来说，每个锁都有线程持有者和锁计数器，每次线程获取到锁，会记录下改线程，并且锁的计数器+1，当线程退出synchronized代码块的时候，线程计数器就会-1，当锁计数为0的时候，就释放锁；

9. 独占锁

   锁一次只能被一个线程占有使用，其他线程在锁外等待释放，synchronized和ReetrantLock都是独占锁，悲观锁实现的锁

10. 共享锁

    锁可以被多个线程持有，对ReentrantReadWriteLock而言，它的读锁是共享锁，乐观锁实现的锁；写锁是独占锁

11. 偏向锁

    偏向锁会偏向第一个获取它的线程，它适合一个线程争抢资源的情况，当一个线程获取偏向锁之后，如果再次同步代码的时候，只需判断该对象的对象头的Mark Word的偏向锁标识是否指向它

12. 轻量级锁

    偏向锁适用于单线程竞争资源，如果其他线程获取该锁对象升级为轻量级锁，多线程获取同一个锁，但是之间可能没有竞争，它使用CAS操作，无需向系统互斥，可以避免从用户态到内核态

# 8锁分析

# 谈谈synchronized关键字

**synchronized关键字解决的是多个线程之间访问资源的同步性，synchronized关键字可以保证被他修饰的方法或者代码块在任意时刻只能有一个线程执行**

##### 你是怎么使用synchronized关键字的？

1. **修饰实例方法**：作用于当前对象实例加锁，进入同步代码前需要获得当前对象实例的锁
2. **修饰静态方法**：是给当前类加锁，会作用于类的所有对象实例，因为静态成员不属于任何一个实例对象，是类的成员。如果一个线程A调用一个实例对象的非静态synchronized方法，而线程B需要调用这个实例对象的静态synchronized方法，是允许的不会产生互斥，**因为访问静态synchronized方法占用的是当前类的锁，而非访问非静态synchronized方法占用的锁是当前实例对象的锁**
3. **修饰代码块**：指定加锁对象，对给定对象加锁，进入同步代码库前要获得给定对象的锁

对synchronized关键字修饰的总结：synchronized关键字加到static静态方法和synchronized（class）代码块都是给Class类上锁；synchronized关键字加到实例方法上是给实例对象上锁，**尽量不要使用synchronized（String a)因为JVM中，字符串常量池具有缓存功能**

##### 手写一个双重检验锁方式的单例模式，说说原理

`public class Singleton{

​			private volatile static Singleton uniqueInstance;

​			private Singleton(){}

​			public static Singleton getUniqueInstance(){

​				//判断对象是否已经实例化过，没有实例化进入加锁代码

​				if(uniqueInstance==null){

​					//类对象加锁

​					synchronized(Singleton.class){

​						if(uniqueInstance==null){

​							uniqueInstance = new Singleton();

​						}

​					}

​				}

​				return uniqueInstance;

​			}

}`

##### synchronized关键字底层的一些原理-->属于JVM层面

- synchronized同步语句块的情况：查看Java类中使用了synchronized关键字修饰的方法，编译后查看字节码；**synchronized同步语句块的实现使用的是monitorenter和monitorexit指令，其中monitorenter指向同步代码块的开始位置，monitorexit指向同步代码块的结束位置**，当执行monitorenter指令时，线程试图获取锁也就是monitor（monitor对象存在于每个Java对象的对象头中）持有权，当计数器为0则获取锁 成功，获取之后将计数器设为1也就是+1；相应的执行monitorexit指令之后，将计数器设为0，表明锁被释放；如果获取对象锁失败，那当前线程就要阻塞等待，直到锁被另外一个线程释放为止。
- synchronized修饰方法的情况：synchronized修饰的方法并没有monitorenter和monitorexit指令，取而代之的时ACC_SYNCHRONIZED标识，该标识指明了该方法是一个同步方法，JVM通过该ACC_SYNCHRONIZED访问标识来辨别一个方法是否为同步方法，从而执行相应的同步调用。

##### jdk1.6对synchronized关键字做了哪些优化

- 锁主要有四种状态，**依次升级不可以降级：无锁状态>偏向锁>轻量级锁>重量级锁**

##### synchronized和ReentrantLock的区别

1. 两者都是 **可重入锁**
2. synchronized依赖于JVM，ReeentrantLock依赖于API
   1. synchronized是依赖于JVM实现的，Java虚拟机在JDK1.6时对synchronized做了很多优化，都是在虚拟机层面实现，没有直接暴露给应用程序。
   2. ReentrantLock是JDK层面实现的（API接口），需要lock（），unlock（）方法配合try/finally语句块来完成
3. ReentrantLock比synchronized增加了一些高级功能
   1. **等待可中断**：通过lock.lockInterruptibly()方法实现正在等待的线程可以选择放弃等待，改为处理其他事情。
   2. **实现公平锁**：Reentrant Lock可以指定公平锁和非公平锁；而synchronized关键字只能是非公平锁，所谓的公平锁就是先等待的线程先获得锁。**ReentrantLock默认是非公平锁，可以通过ReentrantLock类ReentrantLock(boolean fair)构造指定是否公平**
   3. **可实现选择性通知**：synchronized关键字与wait()和notify()/notifyAll()方法相结合实现等待/通知机制，ReentrantLock类借助于Condition接口与newCondition()方法可以实现多路通知功能，在一个Lock对象中创建多个Condition实例，**线程对象可以注册在指定的Condition中，从而有选择性的进行线程通知，在调度线程上更加灵活；在使用notify()/notifyAll()方法进行通知时，被通知的线程是由JVM选择的，用Reentrant Lock类结合Condition实例可以实现“选择性通知”**。synchronized关键字就相当于整个lock对象中只有Condition实例，所有的线程注册在它一个身上，执行notifyAll()方法就会通知所有处于等待的线程造成效率问题，而Condition实例的signalAll()方法只会唤醒注册在该Condition实例中的所有线程
4. synchronized会自动释放锁，lock必须手动释放锁
5. synchronized时不可中断，lock可以中断也可不中断
6. 通过lock可以知道线程有没有拿到锁，synchronized不可以
7. synchronized能锁住方法和代码块，lock只能锁代码块
8. lock可以使用读锁提高多线程效率

# 深挖synchronized关键字

# volatile必须吹半个小时

# 谈谈Threadlocal线程本地存储

# 了解并深入AQS，不入虎穴，焉得offer

# Callable接口和异步回调

# 吹吹线程池

# 什么是线程安全，你真的知道吗？

