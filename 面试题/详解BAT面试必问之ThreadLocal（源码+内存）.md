### 深入理解ThreadLocal

#### 用途

我们一般用ThreadLocal来提供线程局部变量。线程局部变量会在每个Thread内拥有一个副本，Thread只能访问自己的那个副本。文字解释总是晦涩的，我们来看个例子。

```
public class Test {

    private static ThreadLocal<String> threadLocal = new ThreadLocal<>();

    public static void main(String[] args) {
        Thread thread1 = new MyThread("lucy");
        Thread thread2 = new MyThread("lily");
        thread1.start();
        thread2.start();
    }

    private static class MyThread extends Thread {

        MyThread(String name) {
            super(name);
        }

        @Override
        public void run() {
            Thread thread = Thread.currentThread();
            threadLocal.set("i am " + thread.getName());
            try {
                //睡眠两秒，确保线程lucy和线程lily都调用了threadLocal的set方法。
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(thread.getName() + " say: " + threadLocal.get());
        }
    }
}

```

这个例子非常简单，就是创建了lucy和lily两个线程。在线程内部，调用threadLocal的set方法存入一字符串，睡眠2秒后输出线程名称和threadLocal中的字符串。我们运行这单代码，看一下输出内容。

```
lucy say: i am lucy
lily say: i am lily

```

#### 原理

上面例子很好的解释了ThreadLocal的作用，接下来我们分析一下这是如何实现的。

我们定位到ThreadLocal的set方法。源码中set方法被拆分为几个方法，为了表述方便笔者将这几个方法进行了整合。

```
public void set(T value) {
    //获取当前线程
    Thread t = Thread.currentThread();
    //获取当前线程的ThreadLocalMap
    ThreadLocalMap map = t.threadLocals;
    if (map != null)
        //将数据放入ThreadLocalMap中，key是当前ThreadLocal对象，值是我们传入的value。
        map.set(this, value);
    else
        //初始化ThreadLocalMap，并以当前ThreadLocal对象为Key，value为值存入map中。
        t.threadLocals = new ThreadLocalMap(this, value);
}

```

通过上面这段代码可以看到，ThreadLocal的set方法主要是通过当前线程的ThreadLocalMap实现的。ThreadLocalMap是一个Map，它的key是ThreadLoacl，value是Object。

TreadLocal的get方法的源码我就不贴出来了，大体上与set方法类似，就是先获取到当前线程的ThreadLocalMap，然后以this为key可以取得value。

到这里我们基本上明白了ThreadLocal的工作原理，我们总结一下

1.  每个Thread实例内部都有一个ThreadLocalMap，ThreadLocalMap是一种Map，它的key是ThreadLocal，value是Object。
2.  ThreadLocal的set方法其实是往当前线程的ThreadLocalMap中存入数据，其key是当前ThreadLocal对象，value是set方法中传入的值。
3.  使用数据时，以当前ThreadLocal为key，从当前线程的ThreadLocalMap中取出数据。

#### ThreadLocalMap

上面我们介绍了ThreadLocal主要是通过线程的ThreadLocalMap实现的。

```
    static class ThreadLocalMap {
        private ThreadLocal.ThreadLocalMap.Entry[] table;

        static class Entry extends WeakReference<ThreadLocal<?>> {
            Object value;

            Entry(ThreadLocal<?> var1, Object var2) {
                super(var1);
                this.value = var2;
            }
        }
    }

```

ThreadLocalMap是一种Map，其内部维护着一个Entry[]。

ThreadLocalMap其实是就是将Key和Value包装成Entry，然后放入Entry数组中。我们看一下它的set方法。

```
        private void set(ThreadLocal<?> key, Object value) {
            Entry[] tab = table;
            int len = tab.length;
            int i = key.threadLocalHashCode & (len-1);

            for (Entry e = tab[i];
                 e != null;
                 e = tab[i = nextIndex(i, len)]) {
                ThreadLocal<?> k = e.get();

                if (k == key) {
                  //如果已经存在，直接替换value
                    e.value = value;
                    return;
                }

                if (k == null) {//如果当前位置的key ThreadLocal为空，替换key和value。下文ThreadLocal内存分析中会提到为什么会有这段代码。
                    replaceStaleEntry(key, value, i);
                    return;
                }
            }

            tab[i] = new Entry(key, value);//该位置没有数据，直接存入。
            int sz = ++size;
            if (!cleanSomeSlots(i, sz) && sz >= threshold) //检查是否扩容
                rehash();
        }

        private static int nextIndex(int i, int len) {
            return ((i + 1 < len) ? i + 1 : 0);
        }

```

到这里，如果你了解HashMap，应该可以看出ThreadLocalMap就是一种HashMap。不过它并没有采用java.util.HashMap中数组+链表的方式解决Hash冲突，而是采用index后移的方式。

我们简单分析一下这段代码：

1.  通过ThreadLocal的threadLocalHashCode与当前Map的长度计算出数组下标 i。

2.  从i开始遍历Entry数组，这会有三种情况：

    *   Entry的key就是我们要set的ThreadLocal，直接替换Entry中的value。

    *   Entry的key为空，直接替换key和value。

    *   发生了Hash冲突，当前位置已经有了数据，查找下一个可用空间。

3.  找到没有数据的位置，将key和value放入。

4.  检查是否扩容。

我们知道，HashMap是一种get、set都非常高效的集合，它的时间复杂度只有O(1)。但是如果存在严重的Hash冲突，那HashMap的效率就会降低很多。我们通过上段代码知道，ThreadLocalMap是通过 key.threadLocalHashCode & (len-1)计算Entry存放index的。len是当前Entry[]的长度，这没什么好说的。那看来秘密就在threadLocalHashCode中了。我们来看一下threadLocalHashCode是如何产生的。

```
public class ThreadLocal<T> {

    private final int threadLocalHashCode = nextHashCode();

    private static AtomicInteger nextHashCode = new AtomicInteger();

    private static final int HASH_INCREMENT = 0x61c88647;

    private static int nextHashCode() {
        return nextHashCode.getAndAdd(HASH_INCREMENT);
    }
}

```

这段代码非常简单。有个全局的计数器nextHashCode，每有一个ThreadLocal产生这个计数器就会加0x61c88647，然后把当前值赋给threadLocalHashCode。关于0x61c88647这个神奇的常量，可以点[这里](https://www.jianshu.com/p/529c03d9b67e)。

#### ThreadLocal内存分析

不知从何时起，网上开始流传ThreadLocal有内存泄漏的问题。下面我们从ThreadLocal的内存入手，分析一下这种说法是否正确。话不多说直接上图。

![image](//upload-images.jianshu.io/upload_images/3161762-29cb153de3b48c3d.png?imageMogr2/auto-orient/strip|imageView2/2/w/823/format/webp)

现在，我们假设ThreadLocal完成了自己的使命，与ThreadLocalRef断开了引用关系。此时内存图变成了这样。

![image](//upload-images.jianshu.io/upload_images/3161762-4862f7027163f006.png?imageMogr2/auto-orient/strip|imageView2/2/w/815/format/webp)

系统GC发生时，由于Heap中的ThreadLocal只有来自key的弱引用，因此ThreadLocal内存会被回收到。

![image](//upload-images.jianshu.io/upload_images/3161762-144ad54f368c9bbc.png?imageMogr2/auto-orient/strip|imageView2/2/w/824/format/webp)

到这里，value被留在了Heap中，而我们没办法通过引用访问它。value这块内存将会持续到线程结束。如果不想依赖线程的生命周期,那就调用remove方法来释放value的内存吧。个人认为，这种设计应该也是JDK开发大佬的无奈之举。我们从源码中来感受一下这些大佬为了尽可能降低内存泄漏风险作出的努力。

1.  ThreadLocalMap.Entry软引用ThreadLocal，避免了ThreadLocal的内存泄漏。

2.  还记得ThreadLocalMap set方法中的这段代码吗？

    ```
    private void set(ThreadLocal<?> key, Object value) {

        for (Entry e = tab[i];
             e != null;
             e = tab[i = nextIndex(i, len)]) {
            ThreadLocal<?> k = e.get();

               ...

            if (k == null) {
                replaceStaleEntry(key, value, i);
                return;
            }
        }
    }

    ```

3.  ThreadLocal get方法获取时，有一段如果Entry的key为null，移除Entry和Entry.value的代码。

    ```
    private int expungeStaleEntry(int staleSlot) {
        Entry[] tab = table;
        int len = tab.length;

        // expunge entry at staleSlot
        tab[staleSlot].value = null;
        tab[staleSlot] = null;
        size--;

        // Rehash until we encounter null
        Entry e;
        int i;
        for (i = nextIndex(staleSlot, len);
             (e = tab[i]) != null;
             i = nextIndex(i, len)) {
            ThreadLocal<?> k = e.get();
          //
            if (k == null) {
                e.value = null;
                tab[i] = null;
                size--;
            }
          ...
        }
        return i;
    }
    ```

作者：penicillin丶
链接：https://www.jianshu.com/p/c89f7a1f317f

