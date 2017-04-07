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
