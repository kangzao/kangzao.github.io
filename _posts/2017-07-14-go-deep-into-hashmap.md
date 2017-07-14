---
title: 深入理解hashmap
published: true
layout: post
tag: core java
---

### 一、hashmap的数据结构

从下图中我们看出，hashmap具备了数组(寻址容易，插入和删除困难)和链表(寻址困难，插入和删除容易)的优点，每一个数组中都是一个链表。
null key总是存放在Entry[]数组的第一个元素。

![java-util-hashmap-internalcore](/images/posts/corejava/java-util-hashmap-internals.png)


### 二、面试问题


* **1、HashMap 默认数组多大**
* **2、HashMap 什么时候新建数组占用内存**
* **3、HashMap 何时扩容**
* **4、new HashMap<>(20)，hashmap的数组多大**
* **5、JDK8中 HashMap 和之前 HashMap 有什么不同**



***以下是答案：***

* **1、16**

```java
 /**
  * The default initial capacity - MUST be a power of two.
  */
 static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```
* **2、HashMap 在 new 后并不会立即分配数组，而是第一次put时初始化**

构造函数只是初始化了loadFactor(哈希表的负载因子)和threshold(阈值,决定能放入的数据量，一般情况下等于 Capacity * LoadFactor)

```java
 /**
 * Constructs an empty <tt>HashMap</tt> with the specified initial
 * capacity and load factor.
 *
 * @param  initialCapacity the initial capacity
 * @param  loadFactor      the load factor
 * @throws IllegalArgumentException if the initial capacity is negative
 *         or the load factor is nonpositive
 */
 public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
```

这两个参数的解释如下：

>* Initial Capacity The capacity is the number of buckets in the hash table, The initial capacity is simply the capacity at the time the hash table is created.
>* Load factor The load factor is a measure of how full the hash table is allowed to get before its capacity is automatically increased.

* **3、put的元素达到容量乘负载因子的时候，默认16*0.75**

* **4、HashMap的数组大小一定是2的幂，如果入参不是2的幂，实际容量会是最接近指定容量的2的幂，比如 new HashMap<>(20)，比20大且最接近的2的幂是32，实际容量就是32.值得注意的是，rehash会带来比较严重的性能损耗**

```java
/**
 * Rehashes the contents of this map into a new array with a
 * larger capacity.  This method is called automatically when the
 * number of keys in this map reaches its threshold.
 *
 * If current capacity is MAXIMUM_CAPACITY, this method does not
 * resize the map, but sets threshold to Integer.MAX_VALUE.
 * This has the effect of preventing future calls.
 *
 * @param newCapacity the new capacity, MUST be a power of two;
 *        must be greater than current capacity unless current
 *        capacity is MAXIMUM_CAPACITY (in which case value
 *        is irrelevant).
 */
 void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }

    Entry[] newTable = new Entry[newCapacity];
    transfer(newTable, initHashSeedAsNeeded(newCapacity));
    table = newTable;
    threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}
```
* **5、JDK8处于提升性能的考虑，在哈希碰撞的链表长度达到TREEIFY_THRESHOLD（默认8)后，会把该链表转变成树结构。另外，在 resize 的时候，通过巧妙的设计，减少了 rehash 的性能消耗。
具体参考：[http://www.importnew.com/14417.html](http://www.importnew.com/14417.html)**


### 三、HashMap并发死循环问题
具体参考：[http://coolshell.cn/articles/9606.html](http://coolshell.cn/articles/9606.html)


### 四、记一次hashmap的Infinite Loop问题


```shell
top

top - 14:06:23 up 70 days, 16:44,  2 users,  load average: 5.25, 5.32, 5.35
Tasks: 206 total,   1 running, 205 sleeping,   0 stopped,   0 zombie
Cpu(s):  75.9%us,0.0%st

```
首先查看%st(Steal time)，Steal 值比较高的话，服务器上的另一个虚拟机可能拥有更大更多的 CPU 时间片，你可能需要申请升级以与之竞争。另外，高steal值可能意味着主机供应商在服务器上过量地出售虚拟机。

%us — 用户空间占用CPU的百分比。%us高说明，主要是自身应用的消耗。

![hashmap-top.png](/images/posts/corejava/hashmap-top.png)

```shell
ps -mp 16384 -o THREAD,tid,time 或者 top -Hp pid
```



列出pid包含的所有线程信息及本地线程ID (tid),找到最耗时的线程：

![hashmap-thread.png](/images/posts/corejava/hashmap-thread.png)

将需要的线程ID转换为16进制格式 

```shell
printf "%x\n" tid
```
```shell
jstack 16384|grep 7ea3 -A 30
``` 

![hashmap-thread-detail.png](/images/posts/corejava/hashmap-thread-detail.png)

查看线程执行情况，程序经常占了100%的CPU，查看堆栈，你会发现程序都Hang在了HashMap.get()这个方法上了










