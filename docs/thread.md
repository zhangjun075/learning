# exchanger
Exchanger可以在两个线程之间交换数据，只能是2个线程，他不支持更多的线程之间互换数据。

当线程A调用Exchange对象的exchange()方法后，他会陷入阻塞状态，直到线程B也调用了exchange()方法，然后以线程安全的方式交换数据，之后线程A和B继续运行

```
package com.java.exchanger;

import java.util.ArrayList;
import java.util.List;
import java.util.Random;
import java.util.concurrent.Exchanger;
import java.util.concurrent.TimeUnit;

/**
 * Created by junzhang on 2017/4/7.
 */
public class ExchangerTest {
    public static void main(String[] args) {
        ExchangerTest exchangerTest = new ExchangerTest();
        Exchanger<List<Integer>> exchanger = new Exchanger<>();
        exchangerTest.new E_Consumer(exchanger).start();
        exchangerTest.new E_Producer(exchanger).start();
    }


    class E_Producer extends Thread {
        List<Integer> list = new ArrayList<Integer>();
        Exchanger<List<Integer>> exchanger = null;

        public E_Producer(Exchanger<List<Integer>> exchanger) {
            this.exchanger = exchanger;
        }

        @Override
        public void run() {
            Random rand = new Random();
            for(int i=0; i<10; i++) {
                list.clear();
                list.add(rand.nextInt(10000));
                list.add(rand.nextInt(10000));
                list.add(rand.nextInt(10000));
                list.add(rand.nextInt(10000));
                list.add(rand.nextInt(10000));
                try {
                    list = exchanger.exchange(list);
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
            }

        }
    }

    class E_Consumer extends Thread{
        List<Integer> list = new ArrayList<>();
        Exchanger<List<Integer>> exchanger = null;
        public E_Consumer(Exchanger<List<Integer>> exchanger) {
            super();
            this.exchanger = exchanger;
        }
        @Override
        public void run() {
            for(int i=0; i<10; i++) {
                try {
                    list = exchanger.exchange(list);
                    TimeUnit.SECONDS.sleep(2);
                } catch (InterruptedException e) {
                    // TODO Auto-generated catch block
                    e.printStackTrace();
                }
                System.out.print(list.get(0)+", ");
                System.out.print(list.get(1)+", ");
                System.out.print(list.get(2)+", ");
                System.out.print(list.get(3)+", ");
                System.out.println(list.get(4)+", ");
            }
        }
    }

}

```

# DelayQueue
这是一个无界的BlockingQueue,用于放置实现了Delayed接口的对象。

# PriorityBlockingQueue
这是一个很基础的优先级队列，它具有可阻塞的读取操作。

# 免锁容器
容器是所有变成中的基础工具，这其中自然也包括并发编程。出于这个原因，像vector和Hashtable这类早期容器具有许多synchronized方法，当他们用于非多线程的应用程序中时，便会导致不可接受的开销。java se5添加了一些新的容器，通过使用更灵巧的技术来消除加锁，从而提高线程安全的性能。

这些免锁容器背后的通用策略是：对容器的修改可以与读取操作同时发生，只要读取者只能看到完成修改的结果即可。修改是在容器数据结构的某个部分的一个单独的副本(有时是整个数据结构的副本)上执行的，并且这个副本在修改过程中是不可视的。只有当修改完成时，被修改的结构才会自动地与主数据结构进行交换，之后读取者就可以看到这个修改了。

在**CopyOnWriteArrayList**中，写入将导致创建整个底层数组的副本，而源数组将保留在原地。使得复制的数组在被修改时，读取操作可以安全的执行。当修改完成时，一个原子性的操作将把新的数组换入，使得新的读取操作可以看到这个新的修改。**CopyOnWriteArrayList**的好处之一是当多个迭代器同时遍历和修改这个列表时，不会抛出 **ConcurrentModificationException**,因此你不必编写特殊的代码去防范这种异常，就像你以前必须做的那样。

**CopyOnWriteArraySet**将使用**CopyOnWriteArrayList**来实现其免锁行为。

**ConcurrentHashMap**和**ConcurrentLinkedQueue**使用了类似的技术，允许并发的读取和写入，但是容器中只有部分内容而不是整个容器可以被复制和修改。然而，任何修改在完成之前，读取者仍旧不能看到他们。**ConcurrentHashMap**不会抛出**ConcurrentModificationException**异常。

Java中，List在遍历的时候，如果被修改了会抛出java.util.ConcurrentModificationException错误。

```
import java.util.ArrayList;  
import java.util.List;  
  
public class Resource3 {  
  
    public static void main(String[] args) throws InterruptedException {  
        List<String> a = new ArrayList<String>();  
        a.add("a");  
        a.add("b");  
        a.add("c");  
        final ArrayList<String> list = new ArrayList<String>(  
                a);  
        Thread t = new Thread(new Runnable() {  
            int count = -1;  
  
            @Override  
            public void run() {  
                while (true) {  
                    list.add(count++ + "");  
                }  
            }  
        });  
        t.setDaemon(true);  
        t.start();  
        Thread.currentThread().sleep(3);  
        for (String s : list) {  
            System.out.println(s);  
        }  
    }  
} 
```

```
Exception in thread "main" java.util.ConcurrentModificationException
    at java.util.ArrayList$Itr.checkForComodification(ArrayList.java:901)
    at java.util.ArrayList$Itr.next(ArrayList.java:851)
    at com.java.copyonwritelist.Resource.main(Resource.java:30)
    at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
    at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
    at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
    at java.lang.reflect.Method.invoke(Method.java:498)
    at com.intellij.rt.execution.application.AppMain.main(AppMain.java:147)
```

这段代码运行的时候就会抛出java.util.ConcurrentModificationException错误。这是因为主线程在遍历list的时候，子线程在向list中添加元素。
那么有没有办法在遍历一个list的时候，还向list中添加元素呢？办法是有的。就是Java concurrent包中的CopyOnWriteArrayList。

先解释下CopyOnWriteArrayList类。

CopyOnWriteArrayList类最大的特点就是，在对其实例进行修改操作（add/remove等）会新建一个数据并修改，修改完毕之后，再将原来的引用指向新的数组。这样，修改过程没有修改原来的数组。也就没有了ConcurrentModificationException错误。

```
import java.util.ArrayList;  
import java.util.List;  
import java.util.concurrent.CopyOnWriteArrayList;  
  
public class Resource3 {  
  
    public static void main(String[] args) throws InterruptedException {  
        List<String> a = new ArrayList<String>();  
        a.add("a");  
        a.add("b");  
        a.add("c");  
        final CopyOnWriteArrayList<String> list = new CopyOnWriteArrayList<String>(a);  
        Thread t = new Thread(new Runnable() {  
            int count = -1;  
  
            @Override  
            public void run() {  
                while (true) {  
                    list.add(count++ + "");  
                }  
            }  
        });  
        t.setDaemon(true);  
        t.start();  
        Thread.currentThread().sleep(3);  
        for (String s : list) {  
            System.out.println(list.hashCode());  
            System.out.println(s);  
        }  
    }  
} 
```

这段代码在for循环中遍历list的时候，同时会输出list的hashcode来看看list是不是同一个list了

部分输出结果如下：

```
669661746  
a  
2119153548  
b  
471684173  
c  
550648901  
-1  
-76447331  
0  
1638154873  
1  
921225916  
2  
1618672031  
3  
1404182932  
4  
950140076  
5  
-610377050  
6  
-610377050  
7  
-610377050  
8  
-610377050  
9  
-610377050  
10  
-610377050  
11  
-610377050  
12 
```

从上面的结果很容易就看出来，hashcode变化了多次，说明了list已经不是原来的list对象了。这说明了CopyOnWriteArrayList类的add函数在执行的时候确实是修改了list的数组对象。

看add函数的代码：

```
/** 
     * Appends the specified element to the end of this list. 
     * 
     * @param e element to be appended to this list 
     * @return <tt>true</tt> (as specified by {@link Collection#add}) 
     */  
    public boolean add(E e) {  
    final ReentrantLock lock = this.lock;  
    lock.lock();  
    try {  
        Object[] elements = getArray();  
        int len = elements.length;  
        Object[] newElements = Arrays.copyOf(elements, len + 1);  
        newElements[len] = e;  
        setArray(newElements);  
        return true;  
    } finally {  
        lock.unlock();  
    }  
    }  
```

add函数中拷贝了原来的数组并在最后加上了新元素。然后调用setArray函数将引用链接到新数组：

```
/** 
    * Sets the array. 
    */  
   final void setArray(Object[] a) {  
       array = a;  
   } 
```

# ReentrantReadWriteLock

Lock比传统线程模型中的synchronized方式更加面向对象，与生活中的锁类似，锁本身也应该是一个对象。两个线程执行的代码片段要实现同步互斥的效果，它们必须用同一个Lock对象。

　　读写锁：分为读锁和写锁，多个读锁不互斥，读锁与写锁互斥，这是由jvm自己控制的，你只要上好相应的锁即可。如果你的代码只读数据，可以很多人同时读，但不能同时写，那就上读锁；如果你的代码修改数据，只能有一个人在写，且不能同时读取，那就上写锁。总之，读的时候上读锁，写的时候上写锁！

　　ReentrantReadWriteLock会使用两把锁来解决问题，一个读锁，一个写锁

线程进入读锁的前提条件：
    
    没有其他线程的写锁，
    没有写请求或者有写请求，但调用线程和持有锁的线程是同一个

线程进入写锁的前提条件：
    
    没有其他线程的读锁
    没有其他线程的写锁

到ReentrantReadWriteLock，首先要做的是与ReentrantLock划清界限。它和后者都是单独的实现，彼此之间没有继承或实现的关系。然后就是总结这个锁机制的特性了： 

    (a).重入方面其内部的WriteLock可以获取ReadLock，但是反过来ReadLock想要获得WriteLock则永远都不要想。 
     (b).WriteLock可以降级为ReadLock，顺序是：先获得WriteLock再获得ReadLock，然后释放WriteLock，这时候线程将保持Readlock的持有。反过来ReadLock想要升级为WriteLock则不可能，为什么？参看(a)，呵呵. 
     (c).ReadLock可以被多个线程持有并且在作用时排斥任何的WriteLock，而WriteLock则是完全的互斥。这一特性最为重要，因为对于高读取频率而相对较低写入的数据结构，使用此类锁同步机制则可以提高并发量。 
     (d).不管是ReadLock还是WriteLock都支持Interrupt，语义与ReentrantLock一致。 
     (e).WriteLock支持Condition并且与ReentrantLock语义一致，而ReadLock则不能使用Condition，否则抛出UnsupportedOperationException异常。

下面看一个读写锁的例子：
```
package com.java;

/**
 * Created by junzhang on 2017/4/10.
 */
public class ReadWriteLockTest {
    public static void main(String[] args) {
        final Queue3 q3 = new Queue3();
        for(int i=0;i<3;i++)
        {
            new Thread(){
                public void run(){
                    while(true){
                        q3.get();
                    }
                }

            }.start();
        }
        for(int i=0;i<3;i++)
        {
            new Thread(){
                public void run(){
                    while(true){
                        q3.put(new Random().nextInt(10000));
                    }
                }

            }.start();
        }
    }
}
class Queue3{
    private Object data = null;//共享数据，只能有一个线程能写该数据，但可以有多个线程同时读该数据。
    private ReentrantReadWriteLock rwl = new ReentrantReadWriteLock();
    public void get(){
        rwl.readLock().lock();//上读锁，其他线程只能读不能写
        System.out.println(Thread.currentThread().getName() + " be ready to read data!");
        try {
            Thread.sleep((long)(Math.random()*1000));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "have read data :" + data);
        rwl.readLock().unlock(); //释放读锁，最好放在finnaly里面
    }

    public void put(Object data){

        rwl.writeLock().lock();//上写锁，不允许其他线程读也不允许写
        System.out.println(Thread.currentThread().getName() + " be ready to write data!");
        try {
            Thread.sleep((long)(Math.random()*1000));
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        this.data = data;
        System.out.println(Thread.currentThread().getName() + " have write data: " + data);

        rwl.writeLock().unlock();//释放写锁    
    }
}
```

```
Thread-0 be ready to read data!
Thread-1 be ready to read data!
Thread-2 be ready to read data!
Thread-0have read data :null
Thread-2have read data :null
Thread-1have read data :null
Thread-5 be ready to write data!
Thread-5 have write data: 6934
Thread-5 be ready to write data!
Thread-5 have write data: 8987
Thread-5 be ready to write data!
Thread-5 have write data: 8496
```

下面使用读写锁模拟一个缓存器：
```
package com.thread;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.ReadWriteLock;
import java.util.concurrent.locks.ReentrantReadWriteLock;

public class CacheDemo {
    private Map<String, Object> map = new HashMap<String, Object>();//缓存器
    private ReadWriteLock rwl = new ReentrantReadWriteLock();
    public static void main(String[] args) {
        
    }
    public Object get(String id){
        Object value = null;
        rwl.readLock().lock();//首先开启读锁，从缓存中去取
        try{
            value = map.get(id); 
            if(value == null){  //如果缓存中没有释放读锁，上写锁
                rwl.readLock().unlock();
                rwl.writeLock().lock();
                try{
                    if(value == null){
                        value = "aaa";  //此时可以去数据库中查找，这里简单的模拟一下
                    }
                }finally{
                    rwl.writeLock().unlock(); //释放写锁
                }
                rwl.readLock().lock(); //然后再上读锁
            }
        }finally{
            rwl.readLock().unlock(); //最后释放读锁
        }
        return value;
    }

}
```

# StampedLock
该类是一个读写锁的改进，它的思想是读写锁中读不仅不阻塞读，同时也不应该阻塞写。

读不阻塞写的实现思路：
        
        在读的时候如果发生了写，则应当重读而不是在读的时候直接阻塞写！
        因为在读线程非常多而写线程比较少的情况下，写线程可能发生饥饿现象，也就是因为大量的读线程存在并且读线程都阻塞写线程，

因此写线程可能几乎很少被调度成功！当读执行的时候另一个线程执行了写，则读线程发现数据不一致则执行重读即可。所以读写都存在的情况下，
使用StampedLock就可以实现一种无障碍操作，即读写之间不会阻塞对方，但是写和写之间还是阻塞的！

程序举例：
```
public  class  Point {
           //一个点的x，y坐标
           private   double   x,y;
           /**Stamped类似一个时间戳的作用，每次写的时候对其+1来改变被操作对象的Stamped值
            * 这样其它线程读的时候发现目标对象的Stamped改变，则执行重读*/
           private final   StampedLock  stampedLock   =  new    StampedLock();
  
           // an exclusively locked method
           void move(doubledeltaX,doubledeltaY) {
                   /**stampedLock调用writeLock和unlockWrite时候都会导致stampedLock的stamp值的变化
                  * 即每次+1，直到加到最大值，然后从0重新开始 */
                  longstamp =stampedLock.writeLock(); //写锁
                  try {
                         x +=deltaX;
                         y +=deltaY;
                  } finally {
                         stampedLock.unlockWrite(stamp);//释放写锁
                  }
           }
  
         double distanceFromOrigin() {    // A read-only method
                 /**tryOptimisticRead是一个乐观的读，使用这种锁的读不阻塞写
                 * 每次读的时候得到一个当前的stamp值（类似时间戳的作用）*/
                longstamp =stampedLock.tryOptimisticRead();
     
                //这里就是读操作，读取x和y，因为读取x时，y可能被写了新的值，所以下面需要判断
                double    currentX =x,   currentY =y;
     
                /**如果读取的时候发生了写，则stampedLock的stamp属性值会变化，此时需要重读，
                * 再重读的时候需要加读锁（并且重读时使用的应当是悲观的读锁，即阻塞写的读锁）
                 * 当然重读的时候还可以使用tryOptimisticRead，此时需要结合循环了，即类似CAS方式
                 * 读锁又重新返回一个stampe值*/
                if (!stampedLock.validate(stamp)) {
                        stamp =stampedLock.readLock(); //读锁
                        try {
                              currentX =x;
                              currentY =y;
                        }finally{
                              stampedLock.unlockRead(stamp);//释放读锁
                       }
                }
               //读锁验证成功后才执行计算，即读的时候没有发生写
               return Math.sqrt(currentX *currentX + currentY *currentY);
          }
}
```

StampedLock的实现思想:

在StampedLock中使用了CLH自旋锁，如果发生了读失败，不立刻把读线程挂起，锁当中维护了一个等待线程队列。

所有申请锁但是没有成功的线程都会记录到这个队列中，每一个节点（一个节点表示一个线程）保存一个标记位（locked），用于判断当前线程是否已经释放锁。当一个未标记到队列中的线程试图获得锁时，会取得当前等待队列尾部的节点作为其前序节点，并使用类似如下代码（一个空的死循环）判断前序节点是否已经成功的释放了锁：
        
while(pred.locked){  }   
        
解释：pred表示当前试图获取锁的线程的前序节点，如果前序节点没有释放锁，则当前线程就执行该空循环并不断判断前序节点的锁释放，即类似一个自旋锁的效果，避免被系统挂起。当循环一定次数后，前序节点还没有释放锁，则当前线程就被挂起而不再自旋，因为空的死循环执行太多次比挂起更消耗资源。

[JDK8对并发的新支持](https://my.oschina.net/hosee/blog/615927)
