---
title: redis源码分析-对象篇
date: 2019-02-20 22:19:16
tags: [算法,C,redis,读书笔记]
toc: true
---

《Redis设计与实现》读书笔记，[推荐去看作者原著](http://redisbook.com/)

## 类型与编码
```c
typedef struct redisObject {
    // 类型
    unsigned type:4;
    // 编码
    unsigned encoding:4;
    // 指向底层实现数据结构的指针
    void *ptr;
    // ...
} robj;
```
<!-- more -->

|类型常量|对象的名称|
|---|---|
|REDIS_STRING|字符串对象|
|REDIS_LIST|列表对象|
|REDIS_HASH|哈希对象|
|REDIS_SET|集合对象|
|REDIS_ZSET|有序集合对象|


|类型|编码|对象|
|---|---|---|
|REDIS_STRING|REDIS_ENCODING_INT|使用整数值实现的字符串对象。|
|REDIS_STRING|REDIS_ENCODING_EMBSTR|使用 embstr 编码的简单动态字符串实现的字符串对象。|
|REDIS_STRING|REDIS_ENCODING_RAW|使用简单动态字符串实现的字符串对象。|
|REDIS_LIST|REDIS_ENCODING_ZIPLIST|使用压缩列表实现的列表对象。|
|REDIS_LIST|REDIS_ENCODING_LINKEDLIST|使用双端链表实现的列表对象。|
|REDIS_HASH|REDIS_ENCODING_ZIPLIST|使用压缩列表实现的哈希对象。|
|REDIS_HASH|REDIS_ENCODING_HT|使用字典实现的哈希对象。|
|REDIS_SET|REDIS_ENCODING_INTSET|使用整数集合实现的集合对象。|
|REDIS_SET|REDIS_ENCODING_HT|使用字典实现的集合对象。|
|REDIS_ZSET|REDIS_ENCODING_ZIPLIST|使用压缩列表实现的有序集合对象。|
|REDIS_ZSET|REDIS_ENCODING_SKIPLIST|使用跳跃表和字典实现的有序集合对象。|

## 字符串对象
字符串支持int、raw、embstr三种编码方式。
int编码存储整形数值
embstr编码存储短字符串
raw存储长字符串

图示如下：
![int](http://redisbook.com/_images/graphviz-c0ba08ec03934562687cc3cb79580e76edef81e3.png)

![raw](http://redisbook.com/_images/graphviz-8731210637d0567af28d3a9d4089d5f864d29950.png)

![embstr](http://redisbook.com/_images/graphviz-9512800b17c43f60ef9568c6b0b4921c90f7f862.png)

### 命令
|命令|int 编码的实现方法|embstr 编码的实现方法|raw 编码的实现方法|
|---|---|---|
|SET|使用 int 编码保存值。|使用 embstr 编码保存值。|使用 raw 编码保存值。|
|GET|拷贝对象所保存的整数值， 将这个拷贝转换成字符串值， 然后向客户端返回这个字符串值。|直接向客户端返回字符串值。|直接向客户端返回字符串值。|
|APPEND|将对象转换成 raw 编码， 然后按 raw 编码的方式执行此操作。|将对象转换成 raw 编码， 然后按 raw 编码的方式执行此操作。|调用 sdscatlen 函数， 将给定字符串追加到现有字符串的末尾。|
|INCRBYFLOAT|取出整数值并将其转换成 long double 类型的浮点数， 对这个浮点数进行加法计算， 然后将得出的浮点数结果保存起来。|取出字符串值并尝试将其转换成 long double 类型的浮点数， 对这个浮点数进行加法计算， 然后将得出的浮点数结果保存起来。 如果字符串值不能被转换成浮点数， 那么向客户端返回一个错误。|取出字符串值并尝试将其转换成 long double 类型的浮点数， 对这个浮点数进行加法计算， 然后将得出的浮点数结果保存起来。 如果字符串值不能被转换成浮点数， 那么向客户端返回一个错误。|
|INCRBY|对整数值进行加法计算， 得出的计算结果会作为整数被保存起来。|embstr 编码不能执行此命令， 向客户端返回一个错误。|raw 编码不能执行此命令， 向客户端返回一个错误。|
|DECRBY|对整数值进行减法计算， 得出的计算结果会作为整数被保存起来。|embstr 编码不能执行此命令， 向客户端返回一个错误。|raw 编码不能执行此命令， 向客户端返回一个错误。|
|STRLEN|拷贝对象所保存的整数值， 将这个拷贝转换成字符串值， 计算并返回这个字符串值的长度。|调用 sdslen 函数， 返回字符串的长度。|调用 sdslen 函数， 返回字符串的长度。|
|SETRANGE|将对象转换成 raw 编码， 然后按 raw 编码的方式执行此命令。|将对象转换成 raw 编码， 然后按 raw 编码的方式执行此命令。|将字符串特定索引上的值设置为给定的字符。|
|GETRANGE|拷贝对象所保存的整数值， 将这个拷贝转换成字符串值， 然后取出并返回字符串指定索引上的字符。|直接取出并返回字符串指定索引上的字符。|直接取出并返回字符串指定索引上的字符。

## 列表对象
列表对象支持ziplist和linkedlist两种编码方式。
![ziplist](http://redisbook.com/_images/graphviz-a8d31075b4c0537f4eb6d84aaba1df928c67c953.png)

![linkedlist](http://redisbook.com/_images/graphviz-84c0d231f30c740a431407c7aaf3851b96399590.png)
### 命令

|命令|ziplist 编码的实现方法|linkedlist 编码的实现方法|
|---|---|---|
|LPUSH|调用 ziplistPush 函数， 将新元素推入到压缩列表的表头。|调用 listAddNodeHead 函数， 将新元素推入到双端链表的表头。|
|RPUSH|调用 ziplistPush 函数， 将新元素推入到压缩列表的表尾。|调用 listAddNodeTail 函数， 将新元素推入到双端链表的表尾。|
|LPOP|调用 ziplistIndex 函数定位压缩列表的表头节点， 在向用户返回节点所保存的元素之后， 调用 ziplistDelete 函数删除表头节点。|调用 listFirst 函数定位双端链表的表头节点， 在向用户返回节点所保存的元素之后， 调用 listDelNode 函数删除表头节点。|
|RPOP|调用 ziplistIndex 函数定位压缩列表的表尾节点， 在向用户返回节点所保存的元素之后， 调用 ziplistDelete 函数删除表尾节点。|调用 listLast 函数定位双端链表的表尾节点， 在向用户返回节点所保存的元素之后， 调用 listDelNode 函数删除表尾节点。|
|LINDEX|调用 ziplistIndex 函数定位压缩列表中的指定节点， 然后返回节点所保存的元素。|调用 listIndex 函数定位双端链表中的指定节点， 然后返回节点所保存的元素。|
|LLEN|调用 ziplistLen 函数返回压缩列表的长度。|调用 listLength 函数返回双端链表的长度。|
|LINSERT|插入新节点到压缩列表的表头或者表尾时， 使用 ziplistPush 函数； 插入新节点到压缩列表的其他位置时， 使用 ziplistInsert 函数。|调用 listInsertNode 函数， 将新节点插入到双端链表的指定位置。|
|LREM|遍历压缩列表节点， 并调用 ziplistDelete 函数删除包含了给定元素的节点。|遍历双端链表节点， 并调用 listDelNode 函数删除包含了给定元素的节点。|
|LTRIM|调用 ziplistDeleteRange 函数， 删除压缩列表中所有不在指定索引范围内的节点。|遍历双端链表节点， 并调用 listDelNode 函数删除链表中所有不在指定索引范围内的节点。|
|LSET|调用 ziplistDelete 函数， 先删除压缩列表指定索引上的现有节点， 然后调用 ziplistInsert 函数， 将一个包含给定元素的新节点插入到相同索引上面。|调用 listIndex 函数， 定位到双端链表指定索引上的节点， 然后通过赋值操作更新节点的值。

## 哈希对象
哈希对象支持ziplist和hashtable两种编码方式。
![ziplist](http://redisbook.com/_images/graphviz-d2524c9fe90fb5d91b5875107b257e0053794a2a.png)

![ziplist](http://redisbook.com/_images/graphviz-7ba8b1f3af17e2e62cdf43608914333bf14d8e91.png)


![hashtable](http://redisbook.com/_images/graphviz-68cb863d265a1cd1ccfb038d44ce6b856ebbbe3a.png)

### 命令

|命令|ziplist 编码实现方法|hashtable 编码的实现方法|
|---|---|----|
|HSET|首先调用 ziplistPush 函数， 将键推入到压缩列表的表尾， 然后再次调用 ziplistPush 函数， 将值推入到压缩列表的表尾。|调用 dictAdd 函数， 将新节点添加到字典里面。|
|HGET|首先调用 ziplistFind 函数， 在压缩列表中查找指定键所对应的节点， 然后调用 ziplistNext 函数， 将指针移动到键节点旁边的值节点， 最后返回值节点。|调用 dictFind 函数， 在字典中查找给定键， 然后调用 dictGetVal 函数， 返回该键所对应的值。|
|HEXISTS|调用 ziplistFind 函数， 在压缩列表中查找指定键所对应的节点， 如果找到的话说明键值对存在， 没找到的话就说明键值对不存在。|调用 dictFind 函数， 在字典中查找给定键， 如果找到的话说明键值对存在， 没找到的话就说明键值对不存在。|
|HDEL|调用 ziplistFind 函数， 在压缩列表中查找指定键所对应的节点， 然后将相应的键节点、 以及键节点旁边的值节点都删除掉。|调用 dictDelete 函数， 将指定键所对应的键值对从字典中删除掉。|
|HLEN|调用 ziplistLen 函数， 取得压缩列表包含节点的总数量， 将这个数量除以 2 ， 得出的结果就是压缩列表保存的键值对的数量。|调用 dictSize 函数， 返回字典包含的键值对数量， 这个数量就是哈希对象包含的键值对数量。|
|HGETALL|遍历整个压缩列表， 用 ziplistGet 函数返回所有键和值（都是节点）。|遍历整个字典， 用 dictGetKey 函数返回字典的键， 用 dictGetVal 函数返回字典的值。

## 集合对象
集合对象支持intset或者hashtable两种编码方式。

![intset](http://redisbook.com/_images/graphviz-fbd8f0e1aaad0bdef314af55d01212f83cba8b59.png)

![hashtable](http://redisbook.com/_images/graphviz-3f77c5cca338422f418d6d11bc02109fc945e790.png)

### 命令

|命令|intset 编码的实现方法|hashtable 编码的实现方法|
|---|---|---|
|SADD|调用 intsetAdd 函数， 将所有新元素添加到整数集合里面。|调用 dictAdd ， 以新元素为键， NULL 为值， 将键值对添加到字典里面。|
|SCARD|调用 intsetLen 函数， 返回整数集合所包含的元素数量， 这个数量就是集合对象所包含的元素数量。|调用 dictSize 函数， 返回字典所包含的键值对数量， 这个数量就是集合对象所包含的元素数量。|
|SISMEMBER|调用 intsetFind 函数， 在整数集合中查找给定的元素， 如果找到了说明元素存在于集合， 没找到则说明元素不存在于集合。|调用 dictFind 函数， 在字典的键中查找给定的元素， 如果找到了说明元素存在于集合， 没找到则说明元素不存在于集合。|
|SMEMBERS|遍历整个整数集合， 使用 intsetGet 函数返回集合元素。|遍历整个字典， 使用 dictGetKey 函数返回字典的键作为集合元素。|
|SRANDMEMBER|调用 intsetRandom 函数， 从整数集合中随机返回一个元素。|调用 dictGetRandomKey 函数， 从字典中随机返回一个字典键。|
|SPOP|调用 intsetRandom 函数， 从整数集合中随机取出一个元素， 在将这个随机元素返回给客户端之后， 调用 intsetRemove 函数， 将随机元素从整数集合中删除掉。|调用 dictGetRandomKey 函数， 从字典中随机取出一个字典键， 在将这个随机字典键的值返回给客户端之后， 调用 dictDelete 函数， 从字典中删除随机字典键所对应的键值对。|
|SREM|调用 intsetRemove 函数， 从整数集合中删除所有给定的元素。|调用 dictDelete 函数， 从字典中删除所有键为给定元素的键值对。|

## 有序集合对象
有序集合对象支持ziplist和skiplist两种编码方式。

![](http://redisbook.com/_images/graphviz-61b04c9bb72915ec0374125ba9455bc6783db4ff.png)

![](http://redisbook.com/_images/graphviz-8d7b7d0e78ad9d445ff14834e9e9618234395d46.png)

![](http://redisbook.com/_images/graphviz-122e7ebdcd23e888fae17c21813be048c2d3f0a8.png)

![](http://redisbook.com/_images/graphviz-75ee561bcc63f8ea960d0339768aec97b1f570f0.png)

### 命令

|命令|ziplist 编码的实现方法|zset 编码的实现方法|
|---|---|---|
|ZADD|调用 ziplistInsert 函数， 将成员和分值作为两个节点分别插入到压缩列表。|先调用 zslInsert 函数， 将新元素添加到跳跃表， 然后调用 dictAdd 函数， 将新元素关联到字典。|
|ZCARD|调用 ziplistLen 函数， 获得压缩列表包含节点的数量， 将这个数量除以 2 得出集合元素的数量。|访问跳跃表数据结构的 length 属性， 直接返回集合元素的数量。|
|ZCOUNT|遍历压缩列表， 统计分值在给定范围内的节点的数量。|遍历跳跃表， 统计分值在给定范围内的节点的数量。|
|ZRANGE|从表头向表尾遍历压缩列表， 返回给定索引范围内的所有元素。|从表头向表尾遍历跳跃表， 返回给定索引范围内的所有元素。|
|ZREVRANGE|从表尾向表头遍历压缩列表， 返回给定索引范围内的所有元素。|从表尾向表头遍历跳跃表， 返回给定索引范围内的所有元素。|
|ZRANK|从表头向表尾遍历压缩列表， 查找给定的成员， 沿途记录经过节点的数量， 当找到给定成员之后， 途经节点的数量就是该成员所对应元素的排名。|从表头向表尾遍历跳跃表， 查找给定的成员， 沿途记录经过节点的数量， 当找到给定成员之后， 途经节点的数量就是该成员所对应元素的排名。|
|ZREVRANK|从表尾向表头遍历压缩列表， 查找给定的成员， 沿途记录经过节点的数量， 当找到给定成员之后， 途经节点的数量就是该成员所对应元素的排名。|从表尾向表头遍历跳跃表， 查找给定的成员， 沿途记录经过节点的数量， 当找到给定成员之后， 途经节点的数量就是该成员所对应元素的排名。|
|ZREM|遍历压缩列表， 删除所有包含给定成员的节点， 以及被删除成员节点旁边的分值节点。|遍历跳跃表， 删除所有包含了给定成员的跳跃表节点。 并在字典中解除被删除元素的成员和分值的关联。|
|ZSCORE|遍历压缩列表， 查找包含了给定成员的节点， 然后取出成员节点旁边的分值节点保存的元素分值。|直接从字典中取出给定成员的分值。

## 类型检查与命令多态



## 内存回收

## 对象共享

## 对象的空转时长

## 重点回顾