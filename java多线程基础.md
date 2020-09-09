
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

## 用户线程与守护线程
java分两种线程：用户线程和守护线程。

> 守护线程是指程序运行时在后台提供的一种通用服务的线程，比如垃圾回收GC，这种线程并不属于程序不可或缺的部分。
> 
> 当Java程序启动后会开启一个JVM进程，JVM进程中有两类线程，用户线程和守护线程，当用户线程执行结束JVM进程也就结束了，不管守护线程是否执行完毕。
> 
> 比如GC，GC作为守护进程在JVM进程中一直存在，当用户线程运行完毕后，剩下GC等守护进程，JVM也会退出。
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
 
**线程对象通过setPriority()来设置线程优先级。其值默认是5**

### 线程常用API

|常用线程api方法||
|--|--|
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
|Thread.getState()|**静态方法** 	State为Thread的内部成员枚举类，获取线程的状态,对应线程的生命周期，可以使用Thread.Stata 对象名 接受返回的Stata对象|
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
 
 
 ![线程状态转换](https://github.com/diligentpeng/javaStudy/blob/master/images/Thread_state3.jpg)
 
 
 
 ![重写线程的stop方法](https://github.com/diligentpeng/javaStudy/blob/master/images/stopThread.JPG)

