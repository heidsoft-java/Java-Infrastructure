##Java中Volatile关键字详解

- 1、Java 内存模型中的可见性、原子性和有序性

   - 1、可见性：

   ```
   可见性是一种复杂的属性，因为可见性中的错误总是会违背我们的直觉。通常，我们无法确保执行读操作的线程能适时地看到其他线程写入的值，有时甚至是根本不可能的事情。为了确保多个线程之间对内存写入操作的可见性，必须使用同步机制。
     
   可见性，是指线程之间的可见性，一个线程修改的状态对另一个线程是可见的。也就是一个线程修改的结果。另一个线程马上就能看到。比如：用volatile修饰的变量，就会具有可见性。volatile修饰的变量不允许线程内部缓存和重排序，即直接修改内存。所以对其他线程是可见的。但是这里需要注意一个问题，volatile只能让被他修饰内容具有可见性，但不能保证它具有原子性。比如 volatile int a = 0；之后有一个操作 a++；这个变量a具有可见性，但是a++ 依然是一个非原子操作，也就是这个操作同样存在线程安全问题。
     
   在 Java 中 volatile、synchronized 和 final 实现可见性。
   ```
   
   ```
    原子性：原子是世界上的最小单位，具有不可分割性。比如 a=0；（a非long和double类型） 这个操作是不可分割的，那么我们说这个操作时原子操作。再比如：a++； 这个操作实际是a = a + 1；是可分割的，所以他不是一个原子操作。非原子操作都会存在线程安全问题，需要我们使用同步技术（sychronized）来让它变成一个原子操作。一个操作是原子操作，那么我们称它具有原子性。java的concurrent包下提供了一些原子类，我们可以通过阅读API来了解这些原子类的用法。比如：AtomicInteger、AtomicLong、AtomicReference等。
    
    在 Java 中 synchronized 和在 lock、unlock 中操作保证原子性。
   ```
   
   ```
有序性：Java 语言提供了 volatile 和 synchronized 两个关键字来保证线程之间操作的有序性，volatile 是因为其本身包含“禁止指令重排序”的语义，synchronized 是由“一个变量在同一个时刻只允许一条线程对其进行 lock 操作”这条规则获得的，此规则决定了持有同一个对象锁的两个同步块只能串行执行。
   ```

##2、Volatile原理  

- Java语言提供了一种稍弱的同步机制，即volatile变量，用来确保将变量的更新操作通知到其他线程。当把变量声明为volatile类型后，编译器与运行时都会注意到这个变量是共享的，因此不会将该变量上的操作与其他内存操作一起重排序。volatile变量不会被缓存在寄存器或者对其他处理器不可见的地方，因此在读取volatile类型的变量时总会返回最新写入的值。

- **在访问volatile变量时不会执行加锁操作，因此也就不会使执行线程阻塞，因此volatile变量是一种比sychronized关键字更轻量级的同步机制。**

 ![](https://images2015.cnblogs.com/blog/731716/201607/731716-20160708224602686-2141387366.png)
 
- 当对非 volatile 变量进行读写的时候，每个线程先从内存拷贝变量到CPU缓存中。如果计算机有多个CPU，每个线程可能在不同的CPU上被处理，这意味着每个线程可以拷贝到不同的 CPU cache 中。

- 而声明变量是 volatile 的，JVM 保证了每次读变量都从内存中读，跳过 CPU cache 这一步。
 
  一个变量定义为 volatile 之后，将具备两种特性：
 
   - 1.保证此变量对所有的线程的可见性。
      - 这里的“可见性”，如本文开头所述，当一个线程修改了这个变量的值，volatile 保证了新值能立即同步到主内存，以及每次使用前立即从主内存刷新。但普通变量做不到这点，普通变量的值在线程间传递均需要通过主内存（详见：Java内存模型）来完成。

   - 2.禁止指令重排序优化。
      - 有volatile修饰的变量，赋值后多执行了一个“load addl $0x0, (%esp)”操作，这个操作相当于一个内存屏障（指令重排序时不能把后面的指令重排序到内存屏障之前的位置），只有一个CPU访问内存时，并不需要内存屏障；（什么是指令重排序：是指CPU采用了允许将多条指令不按程序规定的顺序分开发送给各相应电路单元处理）。

   - volatile 性能：
      - volatile 的读性能消耗与普通变量几乎相同，但是写操作稍慢，因为它需要在本地代码中插入许多内存屏障指令来保证处理器不发生乱序执行。
 
##3、内存相关
- 1、volatile的特性
  - 可见性：对一个volatile变量的读，总能获取其他任意线程对该变量最后的写入。
  
  - 有序性：JMM会限制volatile变量相关的编译器重排序和处理器重排序。

- 2、内存语义的的实现
  
  - 1.可见性的实现基于volatile的读取，写入两个操作的内存语义。
  
      - volatile写的内存语义：当写入一个volatile变量的时候，JMM将线程工作内存中的该变量的值刷新到主内存。
      
      - volatile读的内存语义：当读取一个volatile变量的时候，JMM首先将该线程工作内存中的这个变量设置为无效，迫使该线程重新从主内存获取最新的有效值。
      - 两者结合起来，就实现了，volatile变量的可见性，因为一个线程去读取volatile变量的时候获取的肯定是最新的值。

  - 2、有序性的实现基于JMM针对编译器制定的volatile重排序表
     ![](https://upload-images.jianshu.io/upload_images/325120-82e98b3ac99c6c47.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/600)

  - 3、最核心的部分还是内存屏障的使用，内存屏障实现了volatile读写的内存语义，也实现了重排序表

    - 理解JMM如何实现volatile的两层内存语义的关键是==内存屏障==。volatile的两层内存语义都是使用内存屏障来实现的。
     
    - 首先，对4中内存屏障的介绍： 内存屏障用于控制特定条件下的重排序和内存可见性问题。

          - LoadLoad屏障：对于这样的语句Load1; LoadLoad; Load2，在Load2及后续读取操作要读取的数据被访问前，保证Load1要读取的数据被读取完毕。
        
          - StoreStore屏障：对于这样的语句Store1; StoreStore; Store2，在Store2及后续写入操作执行前，保证Store1的写入操作对其它处理器可见。
        
          - LoadStore屏障：对于这样的语句Load1; LoadStore; Store2，在Store2及后续写入操作被刷出前，保证Load1要读取的数据被读取完毕。
        
          - StoreLoad屏障：对于这样的语句Store1; StoreLoad; Load2，在Load2及后续所有读取操作执行前，保证Store1的写入对所有处理器可见。它的开销是四种屏障中最大的。 在大多数处理器的实现中，这个屏障是个万能屏障，兼具其它三种内存屏障的功能。

    - 再看看JMM内存屏障的插入策略
     
       - volatile写操作前面插入一个StoreStore屏障

         对比StoreStore屏障的定义，这里的volatile写是那个Store2,该屏障保证在volatile写执行之前，Store1的写入操作对其他处理器可见，那么可以得出，该屏障不仅保证了Store1已经执行完毕（有序性），也保证了可见性。
     
       - 每个volatile写操作的后面插入一个StoreLoad屏障

         对比StoreLoad屏障的定义，这里的volatile写是那个Store1,该屏障也保证了有序性和可见性。其他都是类似的。
         
       - 每个volatile读操作后面插入一个LoadLoad屏障。
       - 每个volatile读操作后面插入一个LoadStore屏障。

    - 4、从CPU的角度，看看可见性如何实现 

     ```
    //假设有个volatile变量instance,对其进行写操作
     instance = new Singleton();
     ```

      在X86处理器下看看其对应的汇编代码的一部分：
   
     ```
     lock add1 $0*0
     ```
     
     lock前缀的指令在多核处理器下引发了两件事：
     
        - 当前处理器缓存行的数据写回系统内存
        - 使其他处理器缓存行中缓存的该内存地址的数据无效
    

