
### 导航
| ID | Problem  | Article | 
| --- | ---   | :--- |
| 000 |CAS是什么 | [解决思路](/docs/CASandABA.md) |
| 001 |CAS底层原理 | [解决思路](/docs/CASandABA.md) |
| 002 |CAS缺点 | [解决思路](/docs/CASandABA.md) |
| 003 |ABA问题 | [解决思路](/docs/CASandABA.md) |
| 004 |AtomicReference原子引用 | [解决思路](/docs/CASandABA.md) |
| 005 |AtomicStampedReference版本号原子引用 | [解决思路](/docs/CASandABA.md) |
| 006 |ABA问题的解决| [解决思路](/docs/CASandABA.md) |

#### 什么是CAS ? 

CAS 是 compare And Swap 的缩写  比较交换 类似于java中的乐观锁 

假如我们要修改一个值 之前的做法就是从内存中去除这个值然后比较 然后更改 但是在咱们现在的CAS操作 只是是一个指令级别 
指令级别 是可以保证原子操作的

 ![整体流程](https://raw.githubusercontent.com/qiurunze123/imageall/master/casunsafe.png)
 
 如果没成功 是在自旋 CAS操作 一直到成功为止 

![整体流程](https://raw.githubusercontent.com/qiurunze123/imageall/master/threadnew10.png)

#### CAS底层原理 

atomicInteger.getAndIncrement() 方法为什么可以避免i++ 问题呢??

![整体流程](https://raw.githubusercontent.com/qiurunze123/imageall/master/casunsafe.png)
 
1.Unsafe类

底层大部分都是有unsafe完成，unsafe自己属于JDK-- sun包下的，在我查看源码发现unsafe是 是CAS的核心类 
由于Java 方法无法直接访问底层 ,需要通过本地(native)方法来访问,UnSafe相当于一个后门,基于该类可以直接操作特定的内存数据.
UnSafe类在于sun.misc包中,其内部方法操作可以向C的指针一样直接操作内存,因为Java中CAS操作的执行依赖于UNSafe类的方法.

`**注意UnSafe类中所有的方法都是native修饰的,也就是说UnSafe类中的方法都是直接调用操作底层资源执行响应的任务**` 
  
  ![整体流程](https://raw.githubusercontent.com/qiurunze123/imageall/master/casunsafe2.png)
  
2.变量valueoff是该变量在内存中的偏移地址 ,unsafe就是根据内存偏移地址来获取数据的！
  变量value 被volatile修饰 保证了多线程的可见性  
  
  ![整体流程](https://raw.githubusercontent.com/qiurunze123/imageall/master/cas1.png)

  CAS的全称compare-And-Swap 它是一条CPU并发原语 它的功能是判断内存某个位置是否为预期值 如果是则更改为新值整个过程是原子的
  
        CAS并发原语提现在Java语言中就是sun.miscUnSaffe类中的各个方法.调用UnSafe类中的CAS方法,
        JVM会帮我实现CAS汇编指令.这是一种完全依赖于硬件 功能,通过它实现了原子操作,再次强调,
        由于CAS是一种系统原语,原语属于操作系统用于范畴,是由若干条指令组成,
        用于完成某个功能的一个过程,并且原语的执行必须是连续的,在执行过程中不允许中断,也即是说CAS是一条原子指令
        ,不会造成所谓的数据不一致的问题. 简单来说是原子操作
  
 举个例子:
 
 
  
   ![整体流程](https://raw.githubusercontent.com/qiurunze123/imageall/master/casunsafe3.png)
   
   此处为jdk实现 
   
       public final int getAndAddInt(Object var1, long var2, int var4) {
              int var5;
              do {
                  //兄弟们告诉我var1 和var2 定位到的内存地址的值是多少
                  var5 = this.getIntVolatile(var1, var2);
              } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
      
              return var5;
          }
          
          1.var1 就是 AtomicInteger new 的本身
          var2 该对象的引用地址 就是偏移量 valueoff
          var4 需要变动的数量
          var5是通过var1和var2找到的主内存中的真实值
          用该对象当前的值与var5比较 如果相同 更新var5+var4并返回true 如果不同继续循环比较直至更新完成！
  
  
  假设线程A和线程B两个线程同时执行getAndAddInt操作(分别在不同的CPU上):
   
  1.AtomicInteger里面的value原始值为3,即主内存中AtomicInteger的value为3,根据JMM模型,线程A和线程B各自持有一份值为3的value的副本分别到各自的工作内存.
   
  2.线程A通过getIntVolatile(var1,var2) 拿到value值3,这是线程A被挂起.
   
  3.线程B也通过getIntVolatile(var1,var2) 拿到value值3,此时刚好线程B没有被挂起并执行compareAndSwapInt方法比较内存中的值也是3 成功修改内存的值为4 线程B打完收工 一切OK.
   
  4.这是线程A恢复,执行compareAndSwapInt方法比较,发现自己手里的数值和内存中的数字4不一致,说明该值已经被其他线程抢先一步修改了,那A线程修改失败,只能重新来一遍了.
   
  5.线程A重新获取value值,因为变量value是volatile修饰,所以其他线程对他的修改,线程A总是能够看到,线程A继续执行compareAndSwapInt方法进行比较替换,直到成功.
  
  
##### 为什么要用CAS而不用synchronize锁??

synchronize一段时间内只允许一个线程访问 虽然一致性得到了保障但是并发性下降 用的

                 do {
                     var5 = this.getIntVolatile(var1, var2);
                    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
                 
                 没有加锁 可以反复的比较 直到比较成功那一刻为止
                                                            
 synchronize 是基于阻塞锁的机制 所以使用的时候就会出现这么几个问题
   
       1.被阻塞的线程优先级很高
       2.拿到锁的线程一直不释放锁怎么办
       3.大量的竞争消耗cpu 同时带来死锁和其他线程安全的问题
       4.粒度较大 在一些方法中适用会显得比较笨拙 比如 计数器

##### 底层汇编  
![整体流程](https://raw.githubusercontent.com/qiurunze123/imageall/master/cas2.png)
  
  在多处理器情况下必须使用lock指令加锁来完成。从这个例子就可以比较清晰的了解CAS的底层实现了，
  当然不同的操作系统和处理器的实现会有所不同，大家可以自行了解。 `计算机并发原语`是叫这个吧哈哈！！
  
#### CAS缺点

      1.类似于自旋锁的循环等待更新循环时间长开销大
      2.只能保证一个共享变量的原子性要是想保证多个只能加锁保证
      3.产生ABA问题

##### 什么是ABA问题

狸猫换太子

![整体流程](https://raw.githubusercontent.com/qiurunze123/imageall/master/cas3.png)


首尾一样 但是中间产生了许多变化  ABA问题 产生了

    CAS算法实现一个比较重要的前提是需要取出内存中某个时刻的数据并在当下的时刻比较并替换 那么这个时间差会导致数据的变化
    比如说一个线程one从内存中位置V取出A 这时候另一个线程two也从内存位置取出A 那么线程
    TWO进行了一些操作将值变为了B 然后TWO又将V位置的数据变为了A 这时候线程one进行CAS操作发现内存仍然是A 然后线程one操作成功
    尽管线程oneCAS操作成功但是不代表这个过程是没有问题的
    
    
##### 如何解决ABA问题

    1.原子引用
    2.增加一种机制 那就是修改版本号（类似于时间戳）
    
    T1 100 1(version)           2019 2 （现在比较值和版本）  
    
    T2 100 1(version)   101 2   100 3 
    
    T1 提交上去的是 2019 2 对不起 你低于3版本 需要重新去拿 版本3的值再去做操作 
    
     
  ABADemo




























//==================================================================================================




CAS算法过程： 

![整体流程](https://raw.githubusercontent.com/qiurunze123/imageall/master/threadnew10.png)

CPU指令：

     /*
        accumulator = AL, AX, or EAX, depending on whether
        a byte, word, or doubleword comparison is being performed
        */
        if(accumulator == Destination) {
        ZF = 1;
        Destination = Source;
        }
        else {
        ZF = 0;
        accumulator = Destination;
        }

目标值和寄存器里的值相等的话，就设置一个跳转标志，并且把原始数据设到目标里面去。如果不等的话，就不设置跳转标志了

无锁类使用：(无锁类要比阻塞效率搞得多毋庸置疑)

AtomicInteger 所有方法

![整体流程](https://raw.githubusercontent.com/qiurunze123/imageall/master/threadnew11.png)
 
 AtomicInteger 继承了 Number类 随便找一个方法解析下 /atomic/AtomicInteger1 
 
 incrementAndGet 底层实现 
 
 ![整体流程](https://raw.githubusercontent.com/qiurunze123/imageall/master/threadnew13.png)
 
 unsafe.compareAndSwapInt方法，他的意思是，对于this这个类上的偏移量为valueOffset的变量值如果与期望值expect相同，那么把这个变量的值设为update。
 其实偏移量为valueOffset的变量就是value
 
 ![整体流程](https://raw.githubusercontent.com/qiurunze123/imageall/master/threadnew14.png)
 
 
 CAS是有可能会失败的，但是失败的代价是很小的，所以一般的实现都是在一个无限循环体内，直到成功为止。
 
     public final int getAndIncrement() {
             for (;;) {
                 int current = get(); // 获取最新的值
                 int next = current + 1;
                 if (compareAndSet(current, next))
                     return current;
             }
         }


**主要接口实现**

Unsafe 的操作事线程非安全的操作 作用：

1.根据偏移量设置值（AtomicInteger中已经看到了这个功能）

2.park()（把这个线程停下来）

3.底层的CAS操作

 ![整体流程](https://raw.githubusercontent.com/qiurunze123/imageall/master/threadnew15.png)
 
 
**ABA 问题**

从AtomicInteger的incrementAndGet我们也可以发现一个问题 

 ![整体流程](https://raw.githubusercontent.com/qiurunze123/imageall/master/threadnew17.png)
 
 此时的我们需要 AtomicStampedReference 

**AtomicStampedReference**

其内部实现一个Pair类来封装值和时间戳

 ![整体流程](https://raw.githubusercontent.com/qiurunze123/imageall/master/threadnew18.png)


 ![整体流程](https://raw.githubusercontent.com/qiurunze123/imageall/master/threadnew19.png)


 ![整体流程](https://raw.githubusercontent.com/qiurunze123/imageall/master/threadnew20.png)


使用AtomicStampedReference来实现时，只会给用户充值一次，因为每次操作使得时间戳+1

**AtomicIntegerArray**

 ![整体流程](https://raw.githubusercontent.com/qiurunze123/imageall/master/threadnew21.png)
 
 它的内部只是封装了一个普通的array
 private final int[] array;
 
 里面有意思的是运用了二进制数的前导零来算数组中的偏移量。
 
 shift = 31 - Integer.numberOfLeadingZeros(scale);
 
 前导零的意思就是比如8位表示12,00001100，那么前导零就是1前面的0的个数，就是4 /atomic/AtomicArray1
 
**AtomicIntegerFieldUpdater** 

1.Updater只能修改它可见范围内的变量。因为Updater使用反射得到这个变量。如果变量不可见，就会出错。
比如如果score申明为private，就是不可行的。


2.为了确保变量被正确的读取，它必须是volatile类型的。如果我们原有代码中未申明这个类型，那么简单得
申明一下就行，这不会引起什么问题。

3.由于CAS操作会通过对象实例中的偏移量直接进行赋值，因此，它不支持static字段（Unsafe.
objectFieldOffset()不支持静态变量） /atomic/AtomicIntegerFieldUpdater1



