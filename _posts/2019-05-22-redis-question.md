---
title: redis的过期策略都有哪些？内存淘汰机制都有哪些？LRU代码实现？
published: true
layout: post
tag: redis
categories: interview redis
date: 2019-05-22
author: 恩来小平
---
* content
{:toc}


### redis删除key的策略
#### 定期删除
所谓定期删除，指的是redis默认是每隔100ms就随机抽取一些设置了过期时间的key，检查其是否过期，如果过期就删除。
#### 惰性删除
在获取某个key的时候，如果过期了此时就会删除，不会返回任何东西。

通过上述两种手段结合起来，保证过期的key一定会被干掉。

如果定期删除漏掉了很多过期key，然后也没及时去查，也就没走惰性删除。如果大量过期key堆积在内存里，可能导致redis内存块耗尽，此时会触发内存淘汰机制。

#### 内存淘汰策略
如果redis的内存占用过多的时候，此时会进行内存淘汰，有如下一些策略：

1）noeviction：当内存不足以容纳新写入数据时，新写入操作会报错，这个一般没人用。

2）allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key(这个是最常用的)。

3）allkeys-random：当内存不足以容纳新写入数据时，在键空间中，随机移除某个key，这个一般没人用。

4）volatile-lru：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的key（这个一般不太合适）

5）volatile-random：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个key。

6）volatile-ttl：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的key优先移除。

#### lrucache

```java

    public class LRUCache<K, V> extends LinkedHashMap<K, V> {
    
    private final int CACHE_SIZE;
    // 这里就是传递进来最多能缓存多少数据
    public LRUCache(int cacheSize) {
        super((int) Math.ceil(cacheSize / 0.75) + 1, 0.75f, true); // 这块就是设置一个hashmap的初始大小，同时最后一个true指的是让linkedhashmap按照访问顺序来进行排序，最近访问的放在头，最老访问的就在尾
        CACHE_SIZE = cacheSize;
    }

    @Override
    protected boolean removeEldestEntry(Map.Entry eldest) {
        return size() > CACHE_SIZE; // 这个意思就是说当map中的数据量大于指定的缓存个数的时候，就自动删除最老的数据
    }
```








