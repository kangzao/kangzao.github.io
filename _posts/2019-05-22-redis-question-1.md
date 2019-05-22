---
title: redis都有哪些数据类型？分别在哪些场景下使用比较合适？
published: true
layout: post
tag: redis
categories: interview redis
date: 2019-05-22
author: 恩来小平
---
* content
{:toc}

#### (1) string
这是最基本的类型了，就是普通的set和get，做简单的kv缓存。

    SET key value  
设置指定 key 的值

    GET key  
获取指定 key 的值

    SETNX key value 
只有在 key 不存在时设置 key 的值。

#### (2)hash
这个是类似map的一种结构，这个一般就是可以将结构化的数据，比如一个对象（前提是这个对象没嵌套其他的对象）给缓存在redis里，然后每次读写缓存的时候，可以就操作hash里的某个字段。
Redis hash 是一个string类型的field和value的映射表，hash特别适合用于存储对象。
Redis 中每个 hash 可以存储 
```math
2^{32} 
```
 键值对（40多亿）。
 
    HMSET key field1 value1 [field2 value2 ] 
同时将多个 field-value (域-值)对设置到哈希表key中

    HGETALL key 
获取在哈希表中指定 key 的所有字段和值

    HGET key field 
获取存储在哈希表中指定字段的值。

![1A363F72-6209-45F0-BEFC-EADA6F5ABA38.png](/images/posts/java-interview/1A363F72-6209-45F0-BEFC-EADA6F5ABA38.png)

#### (3)list
Redis列表是简单的字符串列表，按照插入顺序排序。你可以添加一个元素到列表的头部（左边）或者尾部（右边）
一个列表最多可以包含 2 32 - 1 个元素 (4294967295, 每个列表超过40亿个元素)。

列表的特点：
列表中的元素是有序的，可以通过索引下标来获取某个元素或者某个范围内的元素列表
列表中的元素是可以重复的

    LPUSH key value1 [value2] 
将一个或多个值插入到列表头部

    LRANGE key start stop 

获取列表指定范围内的元素

返回列表中指定区间内的元素，区间以偏移量 START 和 END 指定。 

其中 0 表示列表的第一个元素， 1 表示列表的第二个元素，以此类推。 

你也可以使用负数下标，以 -1 表示列表的最后一个元素， -2 表示列表的倒数第二个元素，以此类推。

微博，某个大v的粉丝，就可以以list的格式放在redis里去缓存。
通过lrange命令，可以基于list实现简单的高性能分页，可以做类似微博那种下拉不断分页的功能。
简单队列，从list头部放入，从尾部取出。

```shell
127.0.0.1:6379> LPUSH runoobkey redis mongodb mysql

(integer) 3

顺序是 mysql mongodb redis

127.0.0.1:6379> LRANGE runoobkey -2 -1

1) "mongodb"

2) "redis"
```


#### (4)set
无序集合，自动去重
Redis 的 Set 是 String 类型的无序集合。
集合成员是唯一的，这就意味着集合中不能出现重复的数据。
Redis 中集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是 O(1)。
集合中最大的成员数为 232 - 1 (4294967295, 每个集合可存储40多亿个成员)。
可以基于set交集、并集、差集的操作，比如交集，可以把两个人的粉丝列表整一个交集，看看俩人的共同好友。

```shell
127.0.0.1:6379> SADD myset redis mongodb mysql

(integer) 3

127.0.0.1:6379> SMEMBERS MYSET

(empty list or set)

127.0.0.1:6379> SMEMBERS myset

1) "mongodb"

2) "redis"

3) "mysql"
```


#### (5) sorted set
Redis 有序集合和集合一样也是string类型元素的集合,且不允许重复的成员。

不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序。

有序集合的成员是唯一的,但分数(score)却可以重复。

集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)

排序的set，去重但是可以排序，写进去的时候给一个分数，自动根据分数排序，最大的特点是有个分数可以自定义排序规则。
Redis中的SortedSet根据一个名为score的64位双精度浮点数的参数实现排序. 但是在实际应用中推荐将score当做64位长整型来使用. 原因很简单: long的取值范围要大于double.

排行榜：将每个用户以及其对应的什么分数写入进去，zadd board score username，接着zrevrange board 0 99，就可以获取排名前100的用户；zrank board username，可以看到用户在排行榜里的排名

ZADD key score1 member1 [score2 member2] 
向有序集合添加一个或多个成员，或者更新已存在成员的分数

ZRANGE key start stop [WITHSCORES] 

Returns the specified range of elements in the sorted set stored at key. The elements are considered to be ordered from the lowest to the highest score. Lexicographical order is used for elements with equal score.

It is possible to pass the WITHSCORES option in order to return the scores of the elements together with the elements. The returned list will contain value1,score1,...,valueN,scoreN instead of value1,...,valueN. 

If start is larger than the largest index in the sorted set, or start > stop, an empty list is returned. If stop is larger than the end of the sorted set Redis will treat it like it is the last element of the sorted set.

通过索引区间返回有序集合成指定区间内的成员


```shell
127.0.0.1:6379> ZADD  myzset 1 "one" 2 "two" 3 "three"

(integer) 3

127.0.0.1:6379> ZRANGE myzset 0 -1

1) "one"

2) "two"

3) "three"

127.0.0.1:6379> ZRANGE myzset -1 -2

(empty list or set)

127.0.0.1:6379> ZRANGE myzset 2,3

(error) ERR wrong number of arguments for 'zrange' command

127.0.0.1:6379> ZRANGE myzset 2 3

1) "three"

127.0.0.1:6379> zrange myzset -2 -1

1) "two"

2) "three"


127.0.0.1:6379> zrange myzset 0 1 withscores

1) "one"

2) "1"

3) "two"

4) "2"

Zrank 返回有序集中指定成员的排名。其中有序集成员按分数值递增(从小到大)顺序排列。

127.0.0.1:6379> ZRANK myzset "one"

(integer) 0
```




















