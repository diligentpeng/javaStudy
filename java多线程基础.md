
  java多线程基础
====

# 一：概念

并发和并行概念

* 并发：指两个或多个事件在同一个时间段内发生。交替执行多个事件    
在操作系统中，安装了多个程序，并发指的是在一段时间内宏观上有多个程序同时运行，这在单 CPU 系统中，每 一时刻只能有一道程序执行，即微观上这些程序是分时的交替运行，只不过是给人的感觉是同时运行，那是因为分 时交替运行的时间是非常短的
* 并行：指两个或多个事件在同一时刻发生（同时发生）。同时执行多个事件     
而在多个 CPU 系统中，则这些可以并发执行的程序便可以分配到多个处理器上（CPU）,实现多任务并行执行，即利用每个处理器来处理一个可以并发执行的程序，这样多个程序便可以同时执行。

***
进程和线程概念
>* 进程Process：进程是系统进行资源分配的基本单位，有独立的内存空间。（运行的程序就是进程）
>* 线程Thread：线程是CPU调度和分派的基本单位，线程依附于进程存在，每个线程会共享父进程的资源。
>* 协程：协程是一种用户态的轻量级线程，协程的调度完全由用户控制，协程间切换只需要保存任务的上下文，没有内核的开销  
>简而言之：一个程序运行后至少有一个进程，一个进程中可以包含多个线程
>常用的Windows、Linux等操作系统都采用抢占式多任务，如何调度线程完全由操作系统决定，程序不能决定什么时候执行，以及执行多长时间。
>
>Java语言内置了多进程支持，一个Java程序实际上是一个JVM进程，JVM进程用一个主线程来执行main()方法，在main()方法内部，又可以启动多个线程。

进程是程序的一次动态执行过程，它需要经历从代码加载，代码执行到执行完毕的一个完整的过程，这个过程也是进程本身从产生，发展到最终消亡的过程。多进程操作系统能同时达运行多个进程（程序），由于 CPU 具备分时机制，所以每个进程都能循环获得自己的CPU 时间片。由于 CPU 执行速度非常快，使得所有程序好像是在同时运行一样。

线程是比进程更小的执行单位，线程是进程的基础之上进行进一步的划分。所谓多线程是指一个进程在执行过程中可以产生多个更小的程序单元，这些更小的单元称为线程，这些线程可以同时存在，同时运行，一个进程可能包含多个同时执行的线程。

>例子：JVM就是一个进程
>* 在几一个进程中，如果开辟了多个线程，线程的运行由调度器安排调度，调度器是与操作系统密切相关的，先后顺序是不能人为干扰的
>* JVM进程用一个主线程来执行main()方法，为系统的入口，用于执行整个程序。在main()方法内部，又可以启动多个线程。
>* 在程序运行时，即使没有自己创建线程，JVM后台也会有GC垃圾回收线程，主线程等。
>* 对同一份资源操作时，会存在资源抢夺的问题，需要加入并发控制。
>* 线程会带来额外的开销，如CPU调度时间，并发控制开销。

**注意：很多多线程都是模拟出来的，真正的多线程指是指有多个cpu（多核）。如果是模拟出来的多线程，即是在一个cpu的情况下，在同一个时间点，cpu只能执行一个代码，因为切换的很快，所以有了同时执行的错觉。**

# 二：并发编程之多线程基础
## 2.1 线程上下文切换
由于中断处理，多任务处理，用户态切换等原因会导致CPU从一个线程切换到另一个线程，切换过程需要保存当前进程的状态并恢复另一个进程的状态。

上下文切换的代价是高昂的，因为在核心上切换线程会花费很多时间。上下文切换的延迟取决于不同的因素，大概在50到100纳秒之间。考虑到硬件平均在每个核心上每纳秒执行12条指令，那么一次上下文切换可能花费600到1200条指令的延迟时间。实际上，上下文切换占用了大量程序执行指令的时间。

如果存在跨核上下文切换（Cross-Core Context Switch），可能会导致CPU缓存失败（CPU从缓存访问数据的成本大约是3到40个时钟周期，从主存访问数据的成本大约是100到300个时钟周期），这种场景的切换成本会更加昂贵。

## 2.2 线程创建方式
常见的Java线程的4种创建方式分别为：
1. 继承Thread类；
2. 实现Runnable接口；
3. 通过ExcutorService和Callable实现有返回值的线程；
4. 基于线程池。

* 第一种：继承Thread
Thread类实现了Runnable接口，并定义了操作线程的一些方法。创建一个类并继承Thread接口，然后实例化线程对象并调用start方法启动线程。结果是两个线程并发的运行:当前线程（main线程）和另一个线程（创建的新线程，执行其run方法）。start方法为一个native方法，通过在操作系统上启动一个新线程，并最终执行run方法来启动一个线程。
```java

//step1:通过继承Thread类创建NewThread线程(覆盖run()方法））)
public class NewThread extends Thread {
    public void run(){
        //run方法线程体
        for（int i=0 ; i<20 ; i++）{
            System.out.println("新线程 extends thread");
        }
        
    }
    public static void main(String[] args){
        //step2:实例化一个NewThread线程对象
        NewThread newThread = new NewThread();
        //step3:调用start方法启动NewThread线程
        newThread.start();
        //新线程和主线程抢夺cpu时间片
      
        //主线程
        for（int i=0 ; i<2000 ; i++）{
            System.out.println("主线程 main thread");
        }
      
    }
}

```
> **过程解释：JVM执行main方法，栈OS开辟一条main方法通向cpu的路径，这个路径叫main线程，主线程，cpu通过这个线程可以执行main方法。 newThread.start();开会开辟一条新的通往cpu的路径（开辟了一个新的栈空间在新的栈空间中执行run方法）。对cpu来说：就有了两条执行的路径，两个线程，一个main线程，一个新线程一起抢夺cpu的执行权（执行时间），谁抢到就执行谁的代码**

* 第二种：实现Runnable接口
**注意开启线程的方式和继承Thread类的方式不一样**
* 继承Thread类：子类对象.strat();
* 实现Thread接口：Thread对象.strat(实现类对象)
* 实现Thread接口的好处：避免了单继承的局限性，灵活方便，方便同一个对象被多个线程使用
```java
class MyThread implements Runnable{ // 实现Runnable接口，作为线程的实现类 
     private String name ;       // 表示线程的名称 
     public MyThread(String name){ 
         this.name = name ;      // 通过构造方法配置name属性 
     } 
     public void run(){  // 覆写run()方法，作为线程 的操作主体 
         for(int i=0;i<10;i++){ 
             System.out.println(name + "运行，i = " + i) ; 
         } 
     } 
}; 
public class RunnableDemo01{ 
    public static void main(String args[]){ 
        MyThread mt1 = new MyThread("线程A ") ;    // 实例化对象 
        MyThread mt2 = new MyThread("线程B ") ;    // 实例化对象 
        Thread t1 = new Thread(mt1) ;       // 实例化Thread类对象 
        Thread t2 = new Thread(mt2) ;       // 实例化Thread类对象 
        t1.start() ;    // 启动多线程 
        t2.start() ;    // 启动多线程 
    } 
};

```
* 第三种：通过ExcutorService和Callable实现有返回值的线程；
Callable可以实现有返回值的线程（了解即可） 

* 第四种：使用线程池
```
线程池：其实就是一个容纳多个线程的容器，其中的线程可以反复使用，省去了频繁创建线程对象的操作，
        无需反复创建线程而消耗过多资源。（一般使用LinkedList <Thread> ,HashMap）

当程序第一次启动的时候，创建多个线程，保存到一个集合中，当我们想要使用线程的时候，就可以
从集合中取出线程使用Thread t = linked.removeFirst();
当我们使用完毕线程，需要把线程归还到线程池：list.add(t);  linked.addList(t);

在JDK1.5之后，JDK内置了线程池，我们可以直接使用

线程池的使用

    1: java.util.concurrent.Executors:用来生成线程池
    2：方法：public static ExecutorService newFixedThreadPool(int nThreads) ：
        返回线程池对象。(创建的是有界线程池,也就是池中的线程个数可以指定最大数量)
        参数：int nThreads：创建线程池中包含的线程数量
        返回值： ExecutorService接口，返回的是ExecutorService接口的实现类对象，可以使用ExecutorService接口来接收

    3：java.util.concurrent.Executors：线程池接口,用来从线程池中获取线程，调用start方法，执行线程任务
        public Future<?> submit(Runnable task) :获取线程池中的某一个线程对象，并执行
        void shutDown（）：关闭/销毁线程池的方法

    4：使用步骤
        1：使用线程池工厂类Executors里面的静态方法newFixedThreadPool(int nThreads)，生产一个指定线程数量的线程池
        2：创建一个类，实现Runnable接口，重写run方法，设置线程任务
        3：调用ExecutorService中的方法submit，传递线程任务（实现类），开启线程，执行run方法
        4：调用ExecutorService中的方法shutDown（）销毁线程池
```
```java
 // 2：创建一个类，实现Runnable接口，重写run方法，设置线程任务
class RunnableImpl3 implements Runnable{
     @Override
     public void run() {
         System.out.println(Thread.currentThread().getName()+"创建了一个新的线程执行");
     }
 }
public class xianchengPool {
    public static void main(String[] args) {
        // 1：使用线程池工厂类Executors里面的静态方法newFixedThreadPool(int nThreads)，生产一个指定线程数量的线程池
        ExecutorService es = Executors.newFixedThreadPool(2);

        //3：调用ExecutorService中的方法submit，传递线程任务（实现类），开启线程，执行run方法
        es.submit(new RunnableImpl3());
        es.submit(new RunnableImpl3());
        es.submit(new RunnableImpl3());
        //线程池会一直开启，使用完了线程，回自动把线程归还到线程池，线程可以继续使用
        es.submit(new RunnableImpl3());

        //关闭线程池
        es.shutdown();
    }
}
```
## 用户线程与守护线程
java分两种线程：用户线程和守护线程。  
> 正常的线程都是用户线程（main线程，自己开启的线程等）
> 守护线程是指程序运行时在后台提供的一种通用服务的线程，比如垃圾回收GC，这种线程并不属于程序不可或缺的部分。
> 
> 当Java程序启动后会开启一个JVM进程，JVM进程中有两类线程，用户线程和守护线程，当用户线程执行结束JVM进程就停止了（停止可能还会需要一点时间，这段时间类守护线程还会执行一下），不管守护线程是否执行完毕(虚拟机不用等待守护线程执行完毕)。
> 
> 比如GC，GC作为守护进程在JVM进程中一直存在，当用户线程运行完毕后，剩下GC等守护进程，JVM也会退出。
>
> **要设置线程为守护线程，需要在start()前调用setDaemon(true)**
```java
public static void main(String [] args){
    Thread t1 = new Thread(()->{
        for(int i=0; i<100; i++){
            try{
                Thread.sleep(200);
                System.out.println("t1输出："+ i);
            }catch(InterruptedException e){
                e.printStack();
            }
        }
    });
    t1.setDaemon(true);
    t1.start();
    Thread t2 = new Thread(()->{
        for(int i=0; i<100; i++){
            try{
                Thread.sleep(50);
                System.out.println("t2输出："+ i);
            }catch(InterruptedException e){
                e.printStack();
            }
        }
    });
    t2.start()
}
```

## 2.3：线程的优先级

线程的切换是由线程调度器控制的，程序员无法通过代码干涉，但是可以通过提高线程的优先级来最大程度的改善线程获取时间片的几率。

线程的优先级被划分为10级，值分别i为1-10，其中1最低，10最高。线程提供了三个常量来表示最低、最高和默认优先级：
 * Thread.MIN_PRIORITY 1
 * Thread.MAX_PRIORITY 10
 * Thread.NORM_PRIORITY 5
 * 不在1-10之间会抛出throw new IllegalArgumentException（）;
 
**优先级的设置建议在start（）调度前,一般都是默认优先级5来公平竞争（main（）线程优先级也是5）**  
**线程对象通过setPriority()来设置线程优先级。其值默认是5，提高线程的优先级是改善线程获取时间片的几率，优先级低意味着获取调度的概率低，并不是优先级低就不会被调用了，这都是看cpu的调度**

### 线程常用API

|常用线程api方法||
|--|---|
|start()|启动线程|
|getID()|获取当前线程ID Thread-编号 该编号从0开始|
|getName()|获取当前线程的名称|
|Stop()|停止线程（已废弃可以自己覆盖重写）|
|getPriority()/setPriority()|返回线程的优先级|
|boolean isAlive()|	测试线程是否处于活动状态|
|isDaemon() / setDaemon()|测试线程是否为守护线程|
|isInterrupted()|测试线程是否已经中断,没有调用interrupt()返回false，否则返回true|
|interrupt()|	线程中断，代替stop()|
|Thread.currentThread()|**静态方法** 获取当前线程对象|
|Thread对象.getState()|State为Thread的内部成员枚举类，获取线程的状态,对应线程的生命周期，可以使用Thread.Stata 对象名 接受当前线程对象返回的Stata对象|
|**常用线程构造函数**||
|Thread()|分配一个新的Thread对象|
|Thread(String name)|分配一个新的Thread对象，具有指定的name正如其名|
|Thread(Runnable r)|分配一个新的Thread对象|
|Thread(Runnable r,String name)|分配一个新的Thread对象|


![重写线程的stop方法](https://github.com/diligentpeng/javaStudy/blob/master/images/stopThread.JPG)


## 2.4：线程的状态

![线程状态图](https://github.com/diligentpeng/javaStudy/blob/master/images/ThreadStatus.PNG)

**注意图中的就绪态和运行态都属于Runnable(可运行)**

**Thread源码中枚举类定义线程有六种状态（生命周期）：NEW(新建)，Runnable(可运行)，Blocked(锁阻塞)，Waiting(无限等待)，Timed Waiting(计时等待)，Teminated(被终止)**

 * NEW new Thread()，尚未执行；
 * RUNNABLE start()就绪后抢夺时间片，正在执行run()
 * BLOCKED 程序加锁，等待获取锁的线程的状态
 * WAITING 调用wait()线程的状态，调用notify()唤醒
 * TIMED_WAITING 调用sleep()方法状态
 * TERMINATED run()执行完毕，进入销毁状态    
 **死亡之后的线程不能再次启动（同一个线程不能启动两次）**
 ![线程状态转换](https://github.com/diligentpeng/javaStudy/blob/master/images/Thread_state3.jpg)
 
## 2.5：线程常用API
## 2.5.1 sleep()方法
让线程休眠一段时间，让其他线程执行任务，但不会释放锁，如果有synchronized同步代码块，其他线程并不能访问这个共享数据。

## 2.5.2 yield()方法
让出本次线程占有cpu时间片，参与下一次竞争，只会给相同或更高优先级的线程以运行机会。

**sleep和yield方法的区别**
> 1. sleep()方法给其他线程运行机会时不考虑线程的优先级，因此会给低优先级的线程以运行机会；
> 2. yield()方法只会给相同优先级或更高优先级的线程以运行机会；
> 3. 线程执行sleep()方法后转入阻塞（blocked）状态，而执行yield()方法后会转入就绪（ready）状态；
> 4. **sleep()方法声明会抛出InterruptedException**，而yield()方法没有声明任何异常；
> 5. sleep()方法比yield()方法具有更好的移植性(跟操作系统cpu调度相关)；

## 2.5.3 join()方法
Thread的join方法：线程等待  
线程调用了join方法，那么就要一直运行到该线程运行结束，才会运行其他线程，这样可以控制线程执行顺序。 
join方法可以传递参数，join(10)表示main线程会等待t1线程10毫秒，10毫秒过去后，ain线程和t1线程之间执行顺序由串行执行变为普通的并行执行

```
A{

  B.join()；//让b线程先执行

  --> 等待b线程执行完毕再接着执行

}
```
面试题：现在有t1、t2、t3三个线程，你怎么保证t2再t1执行完后再执行，t3在t2执行完后再执行。
```java

public class JoinTestSync {
 
	public static void main(String[] args) throws InterruptedException {
		// TODO Auto-generated method stub
		ThreadJoinTest1 t1 = new ThreadJoinTest1("今天");
		ThreadJoinTest1 t2 = new ThreadJoinTest1("明天");
		ThreadJoinTest1 t3 = new ThreadJoinTest1("后天");
		/*
		 * 通过join方法来确保t1、t2、t3的执行顺序
		 * */
		t1.start();
		t1.join();	
		t2.start();
		t2.join();
		t3.start();
		t3.join();
	}
 
}
class ThreadJoinTest1 extends Thread{
    public ThreadJoinTest1(String name){
        super(name);
    }
    @Override
    public void run(){
        for(int i=0;i<5;i++){
            System.out.println(this.getName() + ":" + i);
        }
    }
}
```
**主要使用的join方法实现，直到当前线程执行完才会唤醒其他线程继续执行  **  

## 2.5.4 中断线程
* 1：使用退出标志
当run方法执行完后，线程就会退出。但有时run方法是永远不会结束的，如在服务端程序中使用线程进行监听客户端请求，或是其他的需要循环处理的任务。

在这种情况下，一般是将这些任务放在一个循环中，如while循环。如果想使while循环在某一特定条件下退出，最直接的方法就是设一个boolean类型的标志，并通过设置这个标志为true或false来控制while循环是否退出。  
```java
public class test1 {

    public static volatile boolean exit =false;  //退出标志
    
    public static void main(String[] args) {
        new Thread() {
            public void run() {
                System.out.println("线程启动了");
                while (!exit) {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                System.out.println("线程结束了");
            }
        }.start();
        
        try {
            Thread.sleep(1000 * 5);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        exit = true;//5秒后更改退出标志的值,没有这段代码，线程就一直不能停止
    }
}

```
* 2：interrupted() 和 isInterrupted()来中断线程　

Thread.interrupt()方法: 作用是中断线程。将会设置该线程的中断状态位，即设置为true，中断的结果线程是死亡、还是等待新的任务或是继续运行至下一步，就取决于这个程序本身。线程会不时地检测这个中断标示位，以判断线程是否应该被中断（中断标示值是否为true）。它并不像stop方法那样会中断一个正在运行的线程  　
isInterrupted():测试线程是否已经中断，但是不能清除状态标识

**interrupt()方法只是改变中断状态，不会中断一个正在运行的线程。**需要用户自己去监视线程的状态为并做处理。支持线程中断的方法（也就是线程中断后会抛出interruptedException的方法）就是在**监视线程的中断状态，一旦线程的中断状态被置为“中断状态”，就会抛出中断异常。**这一方法实际完成的是，给受阻塞的线程发出一个中断信号，这样受阻线程检查到中断标识，就得以退出阻塞的状态。

更确切的说，如果某个线程被Object.wait, Thread.join和Thread.sleep三种方法之一阻塞，此时调用该线程的interrupt()方法，那么被阻塞该线程将抛出一个 InterruptedException中断异常（该线程必须事先预备好处理此异常），从而提早地终结被阻塞状态。如果线程没有被阻塞，这时调用 interrupt()将不起作用，直到执行到wait(),sleep(),join()时,才马上会抛出 InterruptedException。
其他线程对目标线程调用interrupt()方法，目标线程需反复检查自身状态，如果isInterrupt则结束执行。
```java
public class Main{
    public static void main(String[] args){
        Thread t = new Thread(()->{
            int n = 0;
            while(!isInterrupt()){ //调用 interrupt 之后为true
                n++;
                System.out.println(n);
            }
        });
        t.start();
        Thread.sleep(1);
        t.interrupt();	//中断t线程
        //t.join()会让main线程处于等待状态，如果此时对main线程调用interrupt()，  join()方法就会立刻抛出InterruptedException。
        //因此，目标线程只要捕获到join抛出的InterruptedException就说明有其他线程对其调用了interrupt()方法，通常情况下该线程就应该立刻结束。
        t.join();	//等待t线程结束
        System.out.print("end");
    }
}
```

join()抛出InterruptedException处理

```java
public class Main{
    public static void main(String[] args){
        Thread t = new MyThread();
        t.start();
        Thread.sleep(1000);
        t.interrupt();	//中断
        t.join();		//等待t线程结束
        System.out.println("end");
    }
}
class MyThread extends Thread{
    public void run(){
        Thread hello = new HelloThread();
        hello.start();
        try{
            hello.join();    //当 t.interrupt()被中断时，t线程中的阻塞会抛出异常
        }catch(InterruptException e){
            System.out.println("interrupted!");
        }
        hello.interrupt();//中断线程
    }
}
class HelloThread extends Thread{
    public void run(){
        int n =0;
        while(!isInterrupted()){   //hello.interrupt()被中断后，线程hello的isInterrupted()为true
            n++;
            System.out.println(n);
            try{
                Thread.sleep(100);
            }catch(InterruptException e){
                break;
            }
        }
    }
}
```

# 三：并发编程之线程安全
## 3.1线程安全问题：
```
出现原因：
当我们使用多个线程访问同一资源的时候，且多个线程中对资源有写的操作，就容易出现线程安全问题。
```
## 3.2 解决线程安全问题
### 3.2.1 使用同步代码块或同步方法synchronized
**synchronized的基本原理**
sychronized是JVM底层帮助实现的，JVM是通过进入、退出对象监视器（Monitor）来实现对方法、同步代码块的同步的。

具体实现是在编译之后在同步方法调用前加入一个monitor.enter指令，在退出方法和异常处插入monitor.exit的指令。

其本质就是对一个对象监视器（Monitor）进行获取，而这个获取过程具有排他性从而达到了同一时刻只能一个线程访问的目的。

而对于没有获取到锁的线程将会阻塞到方法入口处，直到获取锁的线程monitor.exit之后才能尝试继续获取锁
```
使用同步代码块:
    格式：synchronized（锁对象）{
        可能出现线程安全问题的代码（访问共享数据的代码）
    }
    注意：
        1：通过代码的锁对象，可以使用任意的对象
        2：但是必须保证多个线程使用的锁对象是同一个
        3：锁对象的作用：把同步代码块锁住，只让一个线程在同步代码块中执行
        4：在任何时候,最多允许一个线程拥有同步锁（锁对象）,谁拿到锁就进入代码块,其他的线程只能在外等着
        5：同步中的线程，没有执行完毕不会释放锁（归还锁对象），而同步外的线程没有锁就会进入堵塞状态，等待获得锁对象，就进入不了同步代码块
        6：缺点：频繁的判断锁，获得锁，释放锁，程序的效率会变低
```

例子：
```java
    class Station implements Runnable {
        //定义一个多个线程共享的数据
        private int ticket = 10;

        //创建一个锁对象
        Object obj = new Object();

        //设置线程任务：买票

        @Override
        public void run() {
            //窗口 永远开启
            while (true) {
                //同步代码块
                synchronized (obj) {
                    if (ticket > 0) {//有票 可以卖
                        // 出票操作
                        try {
                            Thread.sleep(1000); // 使用sleep模拟一下出票时间
                        } catch (InterruptedException e) {
                            //TODO Auto‐generated catch block
                            e.printStackTrace();
                        }
                        //获取当前线程对象的名字
                        String name = Thread.currentThread().getName();
                        System.out.println(name + "正在卖:" + ticket--);
                    }
                    else return;
                }
            }
        }
    }

    public class xianchengSafe {
        public static void main(String[] args) {
            //创建RunnableImpl接口的实现类对象
            Station run =new Station();

            //创建Thread类对象，构造方法中传递Runnable接口的实现类对象
            Thread t0 = new Thread(run,"窗口一");
            Thread t1 = new Thread(run,"窗口二");
            Thread t2 = new Thread(run,"窗口三");

            //调用start方法开启多线程
            t0.start();
            t1.start();
            t2.start();
        }
    }

```
```
使用synchronized修饰的方法,就叫做同步方法,保证A线程执行该方法的时候,其他线程只能在方法外 等着
使用步骤：
    1：把访问了共享数据的代码抽取出来，放到一个方法中
    2：在方法上添加synchronized修饰符
       格式：public synchronized void method(){ 可能会产生线程安全问题的代码 }
    3：同步方法也会把方法内部的代码锁住，只让一个线程执行：
       此时同步方法的锁对象就是实现类对象 new RunnableImpl（）：也就是this

    4；对于非static方法,同步锁就是this。
       对于static方法（优先于new 对象）,我们使用当前方法所在类的字节码对象(类名.class)。
```
```java
    class RunnableImpl2 implements Runnable {
        //定义一个多个线程共享的数据
        private int ticket = 10;

        //定义一个同步方法
        public synchronized void payTicked(){
            if (ticket > 0) {//有票 可以卖
                // 出票操作
                try {
                    Thread.sleep(1000); // 使用sleep模拟一下出票时间
                } catch (InterruptedException e) {
                    //TODO Auto‐generated catch block
                    e.printStackTrace();
                }
                //获取当前线程对象的名字
                String name = Thread.currentThread().getName();
                System.out.println(name + "正在卖:" + ticket--);
            }
            else return;
        }

        //设置线程任务：买票
        @Override
        public void run() {
            //窗口 永远开启
            while (true) {
                payTicked();
            }
        }
    }

    public class xianchengSafe {
        public static void main(String[] args) {
            //创建RunnableImpl接口的实现类对象
            RunnableImpl2 run =new RunnableImpl2();

            //创建Thread类对象，构造方法中传递Runnable接口的实现类对象
            Thread t0 = new Thread(run);
            Thread t1 = new Thread(run);
            Thread t2 = new Thread(run);

            //调用start方法开启多线程
            t0.start();
            t1.start();
            t2.start();
        }
    }
```

```java
static class A{
    public  static Object o1 = new Object();
    public static void main(String[] args){
        new A().fun1();
        A.fun2();
    }
    public synchronized void fun1(){
        //当前对象this作为锁对象
        //同步代码块
    }
    public synchronized static void fun2(){
        //A.Class对象作为锁对象
        //同步代码块
    }
    
    //fun3和fun4用同一个锁对象才不能被同时调用
    void fun3(){
        sychronized(o1){
            //do something
        }
    }
    
    void fun4(){
        sychronized(o1){
            //do something
        }
    }
}
```
**线程死锁**

![产生死锁的四个必要条件](https://github.com/diligentpeng/javaStudy/blob/master/images/deadLock4.JPG)
```java
public class Main{
    public static void main(String[] args){
        Object o1 = new Object();//锁1
        Object o2 = new Object();//锁2
        new Thread(()->{
            synchronized(o1){
                try{
                    Thread.sleep(1000);
                    synchronized(o2){
                        System.out.print("线程1执行")
                    }
                }catch(InterruptedException e){
                    e.printStackTrace();
                }
            }
        }).start();
        new Thread(()->{
            synchronized(o2){
                try{
                    Thread.sleep(1000);
                    synchronized(o1){
                        System.out.print("线程2执行")
                    }
                }catch(InterruptedException e){
                    e.printStackTrace();
                }
            }
        }).start();
    }
}
```
### 3.2.2 使用Lock锁(ReentrantLock类：可重用锁)
```
我们知道Java语言直接提供了synchronized关键字用于加锁，但这种锁一是很重，二是获取时必须一直等待，没有额外的尝试机制。
引入了一个高级的处理并发的java.util.concurrent包，它提供了大量更高级的并发功能，能大大简化多线程程序的编写。
 java.util.concurrent.locks包提供的ReentrantLock(可重入锁)用于替代synchronized加锁
 java.util.concurrents.Locks.Lock :接口提供了比synchronized代码块和synchronized方法更广泛的锁定操作

 Lock 接口中的方法：
      public void lock() :加同步锁。
      public void unlock() :释放同步锁
  java.util.concurrents.Locks.ReentrantLock implements Lock接口

  使用步骤：
      1：在成员位置创建一个ReentrantLock对象（可使用多态）
      2：在可能出现安全问题的代码前调用Lock接口的方法lock获得锁
      3：在可能出现安全问题的代码后调用Lock接口的方法unlock释放锁
```
```java
//范式
Lock lock = new ReentrantLock();
lock.lock();
try{
    //doSomething...
}finally{
    lock.unlock();
}
```
```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

class RunnableImpl2 implements Runnable {
    //定义一个多个线程共享的数据
    private int ticket = 10;

    //1：在成员位置创建一个ReentrantLock对象（可使用多态）
    Lock l = new ReentrantLock();
    //定义一个同步方法

    //设置线程任务：买票
    @Override
    public void run() {
        //窗口 永远开启
        while (true) {
            // 2：在可能出现安全问题的代码前调用Lock接口的方法lock获得锁
            l.lock();
            if (ticket > 0) {//有票 可以卖
                // 出票操作
                try {
                    Thread.sleep(1000); // 使用sleep模拟一下出票时间
                    //获取当前线程对象的名字
                    String name = Thread.currentThread().getName();
                    System.out.println(name + "正在卖:" + ticket--);
                } catch (InterruptedException e) {
                    //TODO Auto‐generated catch block
                    e.printStackTrace();
                }finally {
                    //3：在可能出现安全问题的代码后调用Lock接口的方法unlock释
                    l.unlock(); //无论代码是否出现异常，都释放锁
                }

            } else {l.unlock();return;}
        }

    }
}

public class ThreadSafe {
    public static void main(String[] args) {
        //创建RunnableImpl接口的实现类对象
        RunnableImpl2 run = new RunnableImpl2();

        //创建Thread类对象，构造方法中传递Runnable接口的实现类对象
        Thread t0 = new Thread(run);
        Thread t1 = new Thread(run);
        Thread t2 = new Thread(run);

        //调用start方法开启多线程
        t0.start();
        t1.start();
        t2.start();
    }
}
```
*  ReentrantLock尝试获取锁lock.tryLock()
因为synchronized是Java语言层面提供的语法，所以我们不需要考虑异常，而ReentrantLock是Java代码实现的锁，我们就必须先获取锁，然后在finally中正确释放锁。

顾名思义，ReentrantLock是可重入锁，它和synchronized一样，一个线程可以多次获取同一个锁。

和synchronized不同的是，ReentrantLock可以尝试获取锁：
```java
if(lock.tryLock(1,TimeUnit.SECONDS)){
    try{
        ...
    }finally{
        lock.unlock;
    }
}
```
上述代码在尝试获取锁的时候，最多等待1秒。如果1秒后仍未获取到锁，tryLock()返回false，程序就可以做一些额外处理，而不是无限等待下去。

所以，使用ReentrantLock比直接使用synchronized更安全，线程在tryLock()失败的时候不会导致死锁。

 **读写锁**
 ReentrantLock和synchonrized只允许一个线程进入临界代码区域，但是有时候这样效率过低。

**当读数据时候可以有多个线程同时获取读锁，当有写操作时只能一个线程获取锁。**

|	|读	|写|
|--|--|--|
|读	|允许	|不允许|
|	写|	不允许|不允许|

使用ReadWriteLock可以解决这个问题，它保证：

* 只允许一个线程写入（其他线程既不能写入也不能读取）；
* 没有写入时，多个线程允许同时读（提高性能）。

用ReadWriteLock实现这个功能十分容易。我们需要创建一个ReadWriteLock实例，然后分别获取读锁和写锁：
```java
public Class Counter{
    private final ReadWriteLock rwlock = new ReentrantReadWriteLock();
    private final Lock rlock = rwlock.readLock();
    private final Lock wlock = rwlock.writeLock();
    private int[] counts = new int[10];
    
    //写操作
    public void inc(int index){
        wlock.lock();	//加写锁
        try{
            counts[index] += 1;
        }finally{
            wlock.unlock();	//释放写锁
        }
    }
    //读操作
    public int[] get(){
        rlock.lock();	//加读锁
        try{
            return Arrays.copyOf(counts,counts.length);
        }finally{
            rlock.unlock();//释放读锁
        }
    }
} 
```
把读写操作分别用读锁和写锁来加锁，在读取时，多个线程可以同时获得读锁，这样就大大提高了并发读的执行效率。
使用ReadWriteLock时，适用条件是同一个数据，有大量线程读取，但仅有少数线程修改。

例如，一个论坛的帖子，回复可以看做写入操作，它是不频繁的，但是，浏览可以看做读取操作，是非常频繁的，这种情况就可以使用ReadWriteLock。

**Lock与synchronized区别**
Lock是一个接口，而synchronied时Java的一个关键字，synchronized时内置的语言实现；

1. synchronized关键字可以直接修饰方法，也可以修饰代码块，而lock只能修饰代码块。

2. synchronized在发生异常时，会自动释放线程占有的锁，一次不会导致死锁现象发生；而lock在发生异常时，如果没有主动通过unLock()去释放锁，则很可能会造成死锁现象，一次使用Lock时需要在finally块中释放锁。

3. Lock可以让等待的线程响应中断，而synchronized却不行，使用synchronized时，等待线程会一直等待下去，不能够响应中断。

4. 通过Lock可以知道有没有成功获取锁，而synchronized却无法办到。（提供tryLock）

5. Lock可以提高多个线程进行读操作的效率（提供读写锁）；

**从性能上说，如果竞争资源不激烈，两者的性能时差不多的，而当竞争资源非常激烈时（既有大量线程同时竞争），此时Lock的性能要远远由于synchronized。所以说，在具体使用时要根据情况选择**

## 线程间的协作（wait/notify）
什么是线程间通信

多个线程在处理同一个资源，并且任务不同时，需要线程通信来帮助解决线程之间对同一个变量的使用或操作；

于时Java提出了等待唤醒机制：Object类的固有方法：wait()/notify()或notifyAll()

wait()、notify()、notifyAll()是三个定义在object类中的方法，可以用来控制线程的状态，这三个方法最终调用的是jvm级的native方法（有jvm的C代码实现）。随着jvm运行平台的不同可能有些差异。

如果对象调用了wait()方法就会使持有该对象的线程把该对象的控制权交出去，然后处于等待状态。

如果对象调用了notify方法就会通知某个正在等待这个对象的控制权的线程可以继续运行（唤醒哪个有一定的随机性），notifyAll会唤醒所有等待该对象控制权的线程。

wait方法必须在synchronized方法或者代码块中调用。
```
笔记：等待唤醒机制
多个线程并发执行时, 在默认情况下CPU是随机切换线程的，当我们需要多个线程来共同完成一件任务，并且我们希望他们有规律的执行,
那么多线程之间需要一些协调通信，以此来帮我们达到多线程共同操作一份数据。

多个线程在处理同一个资源，并且任务不同时，需要线程通信来帮助解决线程之间对同一个变量的使用或操作。 就 是多个线程在操作同一份数据时，
避免对同一共享变量的争夺。也就是我们需要通过一定的手段使各个线程能有效的利用资源。而这种手段即—— 等待唤醒机制。

1. wait：线程不再活动，不再参与调度，进入 wait set 中，因此不会浪费 CPU 资源，也不会去竞争锁了，
        这时 的线程状态即是 WAITING。它还要等着别的线程执行一个特别的动作，也即是“通知（notify）”
        在这个对象 上等待的线程从wait set 中释放出来，重新进入到调度队列（ready queue）中
2. notify：则选取所通知对象的 wait set 中的一个线程释放；例如，餐馆有空位置后，等候就餐最久的顾客最先入座。
3. notifyAll：则释放所通知对象的 wait set 上的全部线程。
   注意： 哪怕只通知了一个等待的线程，被通知线程也不能立即恢复执行，因为它当初中断的地方是在同步块内，
         而此刻它已经不持有锁，所以她需要再次尝试去获取锁（很可能面临其它线程的竞争），成功后才能在当初调 用 wait 方法之后的地方恢复执行。

总结如下： 如果能获取锁，线程就从 WAITING 状态变成 RUNNABLE 状态；
            否则，从 wait set 出来，又进入 entry set，线程就从 WAITING 状态又变成 BLOCKED 状态

```

**面试题：使用多线程实现生产消费者模式**
1. 管程法：生产者+消费者+产品+缓冲区
生产者消费者模式是并发，多线程编程中经典的设计模式，生产者和消费者通过分离的执行工作解耦，简化了开发模式，生产者和消费者可以以不同的速度生产和消费数据。

生产者消费者模式的好处：

1. 它简化了开发，你可以独立的或并发的编写消费者和生产者，它仅仅只需知道共享对象是谁；
2. 生产者不需要知道谁是消费者或者有多少消费者，对消费者来说也是一样；
3. 生产者和消费者可以以不同的速度执行；
4. 分离的消费者和生产者在功能上能写出更简洁，可读，易维护的代码。

生产的数据放入的中间容器（Map，List，Queue等）中供消费者取用

经典题目：生产消费者面试提（wait/notify实现）
* 生产线加工生产汽车；
* 4s店存放生产的汽车，最多存放4量汽车，如果达到4量就停止生产；
* 消费者在4s店购买汽车，如果4s店没有库存，则消费者需等待购买；
public class Main{
    private static int CAR_NAME = 1;
    
    public static void main(String[]args){
        Car4s car4s = new Car4s();
        new Thread(new Provider(car4s)).start;
        new Thread(new Customer(car4s)).start;
    }
    //消费者
    static class Customer implements Runnable{
        private Car4s car4s;
        public Customer(Car4s car4s){
            this.car4s = car4s;
        }
        @override
        public void run(){
            while(true){
                synchronized(car4s){
                    if(car4s.getCarQueue().size() > 0){
                        //消费
                        Car car = car4s.getCarQueue().poll(car);
                        System.out.println("消费汽车===>"car.getName());
                        //无库存开始唤醒生产者
                        car4s.notifyAll();
                    }else{
                        //停止消费
                        try{
                            car4s.wait();
                        }catch(InterruptedException e){
                            e.printStackTrace();
                        }
                    }
                }
            }
        }
    }
    //生产者
    static class Provider implements Runnable{
        private Car4s car4s;
        public Provider(Car4s car4s){
            this.car4s = car4s;
        }
        @override
        public void run(){
            while(true){
                synchronized(car4s){
                    if(car4s.getCarQueue().size() < 4){
                        //生产
                        Car car = new Car();
                        car.setName(CAR_NAME+"");
                        car4s.getCarQueue().push(car);
                        CAR_NAME++;
                        System.out.println("生产汽车===>"car.getName());
                        //有库存开始唤醒消费者
                        car4s.notifyAll();
                    }else{
                        //停止生产
                        try{
                            car4s.wait();
                        }catch(InterruptedException e){
                            e.printStackTrace();
                        }
                    }
                }
            }
        }
    }
    
    //4s店
    static class Car4s{
        private Queue<Car> carQueue = new LinkedList<>();
        //getter/setter
    }
    //车
    static class Car{
        private String name;
        //getter/setter
    }
}
