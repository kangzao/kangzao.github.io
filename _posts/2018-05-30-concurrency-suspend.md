---
layout: post
title:  "多线程基础-bad suspend"
categories: concurrency
tags:  多线程基础 
author: 恩来小平
---
suspend被弃用的原因是因为它会造成死锁。suspend不会释放锁，相反它会一直保持对锁的占有，一直到其他的线程调用resume方法，它才能继续向下执行。如果加锁发生在resume之前，则死锁发生。

```java
public class BadSuspend extends Thread {
    private static Object object = new Object();

    public BadSuspend(String name) {
        super(name);
    }

    public static void main(String[] args) throws InterruptedException {
        BadSuspend t1 = new BadSuspend("t1");
        BadSuspend t2 = new BadSuspend("t2");
        t1.start();
        Thread.sleep(100);
        t2.start();
        t1.resume();
        t2.resume(); 
        t1.join();
        t2.join(); // t2永远等不到结束
    }

    @Override
    public void run() {
        synchronized (object) {
            System.out.println("in " + this.getName());
            this.suspend(); // suspend时，线程仍然是Runnable状态
        }
    }
}
```

注意这句：t2.resume();由于t1 resume之后，还有一个t1释放锁的过程，导致t2 resume在t2 supend之前，这样实际上resume时什么都没做，t2 supend之后，再也不会resume。jps找到对应的进程，使用jstack分析线程状态：

![concurrency-suspend](/images/posts/concurrency/concurrency-suspend.png)

我们发现，t2仍然在suspend，所以t2永远不会结束。








