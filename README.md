# ThreadPoolTree
线程池源码技术研究


线程池的状态

![](https://i.imgur.com/Pnj5gHo.png)

<pre>
Running：
           创建线程池的时候，线程池的初始状态为Running，接着就可以进行提交任务执行了。

ShutDown:
           当在Running状态调用ShutDown()时，线程池的状态会被改为shutdown，这时候，submit任务
      会被拒绝。可以使用多种拒绝策略。

           对于正在执行中的线程，会继续执行，同时会把阻塞队列中的任务也一并执行完毕，等到全部任务
      执行完毕，线程池会进入Tidying状态，等执行钩子方法teriminated()之后，整个线程池完全终止。

Stop:
           当在Running状态调用ShutDownNow()时，线程池状态会被改为Stop()，这时候，submit任务会被拒绝，
      并中断各个工作线程，所以任务会如何执行要看任务做的是什么事情，有没有处理中断异常。而阻塞队列
      如果有任务，这些任务不会再执行，ShutDownNow()执行后，将会返回阻塞队列中的未执行的任务列表。

Tidying
           Tidying只是一个过渡状态，当所有工作线程都停止后，线程池的状态会进入Tidying状态，然后执行一个
      terminated()方法，最后线程池会进入Terminated状态。

Terminated
           线程池的终止状态。
</pre>


ThreadPoolExcutor

![](https://i.imgur.com/2oKhIrM.png)

<pre>
workQueue: 阻塞队列
         ArrayBlockingQueue: 构造函数需要传大小，固定容量
         LinkedBlockingQueue: 构造函数不传大小时，默认为65535，容易造成内存耗尽。
         SynchronousQueue：同步队列，一个没有存储空间的阻塞队列，将任务同步交给工作线程
         PriorityBlockingQueue:优先级队列
         Distruptor：

Handler: 饱和策略
         AbortPolicy：默认， 直接抛弃
         CallerRunsPolicy：用调用者的线程执行任务。
         DiscardOldestPolicy：抛弃队列中最久的任务
         DiscardPolicy：抛弃当前任务


执行逻辑：
        1）当有任务提交的时候，会创建核心线程去执行任务（即使有核心线程空闲仍会创建）；
        2）当核心线程数达到corePoolSize时，后续提交的都会进BlockingQueue中排队；
        3）当BlockingQueue满了(offer失败)，就会创建临时线程(临时线程空闲超过一定时间后，会被销毁)；
        5）当线程总数达到maximumPoolSize时，后续提交的任务都会被RejectedExecutionHandler拒绝
</pre>