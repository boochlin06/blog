# Naming threads and thread pools of ExecutorService

> 原文網址：https://boochlin.com/?p=115
> 發布日期：2016-02-16
> 文章編號：115

---

如何替 ExecutorService 所管理的 thread 設定一個自己看得爽的名子

其實這是一件很重要的事情，根據 clean code 裡面所提到的，一個好的名子可以幫助你理解程式碼並 debug，通常配合 thread 觀測工具就可以找出到底有多少莫名其妙的 thread 在消耗你的資源。

以 android ddms 為例

![ddms_thread](../../assets/images/螢幕快照-2016-02-16-下午2.52.27.png)

[https://boochlin.com/wp-content/uploads/2016/02/螢幕快照-2016-02-16-下午2.52.27.png](https://boochlin.com/wp-content/uploads/2016/02/螢幕快照-2016-02-16-下午2.52.27.png)

看得這邊你可能會覺得這篇很廢，不過是替 thread 取個名子，原生就有啦

```
Thread thread = new Thread(Runnable);
thread.setName("Load bitmap task");
```

但是～～～

如果是使用 ExecutorService 做為 thread 排程工具的話，就會發現居然沒有 setName() 可以用，這不是當然的嗎！就算可以取名子也會變成取 ExecutorService 的名子吧。而預設在 thread pool裡的 thread 一律是這格式（如上圖）

```
String namePrefix = "pool-" + poolNumber.getAndIncrement() + "-thread-" + thredNumber.getAndIncrement();
```

基本上這命名有跟沒有差不多，只不過多知道這是由 pool 建出來的，為了解決這個問題，先提供最簡單的土砲法-直接在runnable 把 thread 抓出來命名

```
new Runnable () {
    public void run() {
        Thread.currentThread().setName("Your name");
        doStuff();
    }
}
```

但是這真的太土砲了，實在沒有美感，也代表者有跑到這個 runnable 的 thread 才有機會被命名，而且如果任務不同的話，一個thread 更有可能會常常變換名稱。

最好的方法是利用工廠模式，在創建 thread pool 時把創造 thread 的工廠方法也放進去

```
ExecutorService mPool;
public ExecutorService getExecutorPool() {
        if (mPool == null) {
            mPool = Executors.newFixedThreadPool(POOL_THREAD_NUM, new NameThreadFactory(TAG));
        }
        return mPool;
    }
}
// 另外寫的 thread factory, 簡單只多加命名的工作
public class NameThreadFactory implements ThreadFactory {
    private static final AtomicInteger poolNumber = new AtomicInteger(1);
    private final ThreadGroup group;
    private final AtomicInteger threadNumber = new AtomicInteger(1);
    private final String namePrefix;

    public NameThreadFactory(String namePrefix) {
        SecurityManager s = System.getSecurityManager();
        group = (s != null) ? s.getThreadGroup() :
                Thread.currentThread().getThreadGroup();
        this.namePrefix = namePrefix + "-" +
                poolNumber.getAndIncrement() +
                "-thread-";
    }

    public Thread newThread(Runnable r) {
        Thread t = new Thread(group, r,
                namePrefix + threadNumber.getAndIncrement(),
                0);
        if (t.isDaemon())
            t.setDaemon(false);
        if (t.getPriority() != Thread.NORM_PRIORITY)
            t.setPriority(Thread.NORM_PRIORITY);
        return t;
    }
}
```

或者更簡單的使用已有的車輪 guava , [google common lib](http://search.maven.org/#artifactdetails%7Ccom.google.guava%7Cguava%7C18.0%7Cbundle)

先必須在 gradle 加上

```
compile 'com.google.guava:guava:18.0'
```

接者～

```
ThreadFactory namedThreadFactory = new ThreadFactoryBuilder()
                                       .setNameFormat("workerthread-%d").build()
```

也或者 apache 的這一套 [lib](http://mvnrepository.com/artifact/org.apache.commons/commons-lang3/3.0) ，請先抓jar下來

```
BasicThreadFactory factory = new BasicThreadFactory.Builder()
     .namingPattern("workerthread-%d")
     .daemon(true)
     .build();
```

感謝收看

reference:

[http://stackoverflow.com/questions/6113746/naming-threads-and-thread-pools-of-executorservice](http://stackoverflow.com/questions/6113746/naming-threads-and-thread-pools-of-executorservice)
