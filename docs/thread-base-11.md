
**生产者消费者**


    准确说应该是“生产者-消费者-仓储”模型，离开了仓储，生产者消费者模型就显得没有说服力了
    对于此模型，应该明确一下几点：
    1、生产者仅仅在仓储未满时候生产，仓满则停止生产
    2、消费者仅仅在仓储有产品时候才能消费，仓空则等待
    3、当消费者发现仓储没产品可消费时候会通知生产者生产
    4、生产者在生产出可消费产品时候，应该通知等待的消费者去消费
    
最重要的仓储就是阻塞队列：

BlockingQueue

![整体流程](https://raw.githubusercontent.com/qiurunze123/imageall/master/threadnew46.png)

行为解释：

1（异常）

如果试图的操作无法立即执行，抛一个异常。

2（特定值）
 
如果试图的操作无法立即执行，返回一个特定的值(常常是 true / false)。

3(阻塞)
 
如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行。

4（超时）
 
如果试图的操作无法立即执行，该方法调用将会发生阻塞，直到能够执行，但等待时间不会超过给定值。返回一个特定值以告知该操作是否成功(典型的是 true / false)。

_BlockingQueue 特点_
 
BlockingQueue ：不接受 null 元素。试图 add、put 或 offer 一个 null 元素时，某些实现会抛出 NullPointerException。
null 被用作指示 poll 操作失败的警戒值。

BlockingQueue： 可以是限定容量的。它在任意给定时间都可以有一个 remainingCapacity，超出此容量，便无法无阻塞地 put 附加元素。没有任何内部容量约束的 BlockingQueue 总是报告Integer.MAX_VALUE 的剩余容量。

BlockingQueue ：实现主要用于生产者-使用者队列，但它另外还支持 Collection 接口。因此，举例来说，使用 remove(x) 从队列中移除任意一个元素是有可能的。然而，这种操作通常不 会有效执行，只能有计划地偶尔使用，比如在取消排队信息时。

BlockingQueue ：实现是线程安全的。所有排队方法都可以使用内部锁或其他形式的并发控制来自动达到它们的目的。然而，大量的 Collection 操作（addAll、containsAll、retainAll 和removeAll）没有 必要自动执行，除非在实现中特别说明。因此，举例来说，在只添加了 c 中的一些元素后，addAll(c) 有可能失败（抛出一个异常）。


**BlockingQueue的用法**


一个线程将会持续生产新对象并将其插入到队列之中，直到队列达到它所能容纳的临界点。也就是说，它是有限的。
如果该阻塞队列到达了其临界点，负责生产的线程将会在往里边插入新对象时发生阻塞。它会一直处于阻塞之中，直到负责消费的线程从队列中拿走一个对象。

 负责消费的线程将会一直从该阻塞队列中拿出对象。如果消费线程尝试去从一个空的队列中提取对象的话，
 这个消费线程将会处于阻塞之中，直到一个生产线程把一个对象丢进队列

![整体流程](https://raw.githubusercontent.com/qiurunze123/imageall/master/threadnew47.png)

**BlockingQueue的实现类和继承接口**

1.数组阻塞队列 ArrayBlockingQueue 

![整体流程](https://raw.githubusercontent.com/qiurunze123/imageall/master/threadnew48.png)

对应线程池队列：

有界的任务队列可以使用ArrayBlockingQueue实现。当使用有界队列时，若有新的任务需要执行，如果线程池的实际线程数小于corePoolSize，则会优先创建新的线程，若大于corePoolSize，则会将新任务假如等待队列。若等待队列已满，无法加入，在总线程数，不大于maximumPoolSize的前提下，创建新的进程执行任务。若大于maximumPoolSize，则执行拒绝策略。


2.延迟队列DelayQueue

![整体流程](https://raw.githubusercontent.com/qiurunze123/imageall/master/threadnew49.png)

3.链阻塞队列 LinkedBlockingQueue

![整体流程](https://raw.githubusercontent.com/qiurunze123/imageall/master/threadnew50.png)

对应线程池队列：

无界的任务队列可以通过LinkedBlockingQueue类实现。与有界队列相反，除非系统资源耗尽，否则无界的任务队列不存在任务入队失败的情况。
当有新的任务到来，系统的线程数小于corePoolSize时，线程池会产生新的线程执行任务，但当系统的线程数达到corePoolSize后，就会继续增加。
若后续仍有新的任务假如，而又没有空闲的线程资源，则任务直接进入对列等待。若任务创建和处理的速度差异很大，无界队列会保持快速增长，直到耗尽系统内存。

3.同步队列 SynchronousQueue 

![整体流程](https://raw.githubusercontent.com/qiurunze123/imageall/master/threadnew51.png)

SynchronousQueue经常用来,一端或者双端严格遵守"单工"(单工作者)模式的场景,队列的两个操作端分别是productor和consumer.常用于一个productor多个consumer的场景。

在ThreadPoolExecutor中,通过Executors创建的cachedThreadPool就是使用此类型队列.已确保,如果现有线程无法接收任务(offer失败),将会创建新的线程来执行.

对应线程池队列：

直接提交的队列： 该功能由 SynchronousQueue对象提供。SynchronousQueue 是一个特殊的BlocingQueue。 它没有容量，每一个插入操作都要等待一个相应的删除操作，反之，每一个删除操作都要等待对应的插入操作。如皋市使用SynchronousQueue，提交的任务不会被真实的保存，而总是将新任务提交给线程执行，如果没有空闲的进程，则尝试创建新的进程，如果进程的数量已达到最大值，则执行拒绝策略。

4.优先级 PriorityBlockingQueue

![整体流程](https://raw.githubusercontent.com/qiurunze123/imageall/master/threadnew52.png)

对应线程池队列：

优先任务队列是带有执行优先级的队列，它通过PriorityBlockingQueue实现，可以控制任务的只想你个先后顺序。它是一个特殊的无界队列。

