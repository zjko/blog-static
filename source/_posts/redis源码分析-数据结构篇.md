---
title: Redis设计与实现-数据结构篇
date: 2019-02-19 22:08:59
tags: [算法,C,Redis,读书笔记]
toc: true
---

《Redis设计与实现》读书笔记，[推荐去看作者原著](http://redisbook.com/)

## SDS
sds.h/sdshdr
```c
struct sdshdr {
    // 记录 buf 数组中已使用字节的数量
    // 等于 SDS 所保存字符串的长度
    int len;
    // 记录 buf 数组中未使用字节的数量
    int free;
    // 字节数组，用于保存字符串
    char buf[];
};
```
<!-- more -->
![图示](http://redisbook.com/_images/graphviz-72760f6945c3742eca0df91a91cc379168eda82d.png)

### SDS与C字符串的区别
* 不以\0为休止符，而是通过len来确定字符串结束

|C字符串 | SDS|
|---|---|
|获取字符串长度的复杂度为 O(N) 。|获取字符串长度的复杂度为 O(1) 。|
|API 是不安全的，可能会造成缓冲区溢出。|API 是安全的，不会造成缓冲区溢出。|
|修改字符串长度 N 次必然需要执行 N 次内存重分配。|修改字符串长度 N 次最多需要执行 N 次内存重分配。|
|只能保存文本数据。|可以保存文本或者二进制数据。|
|可以使用所有 <string.h> 库中的函数。|可以使用一部分 <string.h> 库中的函数。|

### SDS主要API
|函数|作用|时间复杂度|
|---|---|---|
|sdsnew|创建一个包含给定 C 字符串的 SDS 。|O(N) ， N 为给定 C 字符串的长度。|
|sdsempty|创建一个不包含任何内容的空 SDS 。|O(1)|
|sdsfree|释放给定的 SDS 。|O(1)|
|sdslen|返回 SDS 的已使用空间字节数。|这个值可以通过读取 SDS 的 len 属性来直接获得， 复杂度为 O(1) 。|
|sdsavail|返回 SDS 的未使用空间字节数。|这个值可以通过读取 SDS 的 free 属性来直接获得， 复杂度为 O(1) 。|
|sdsdup|创建一个给定 SDS 的副本（copy）。|O(N) ， N 为给定 SDS 的长度。|
|sdsclear|清空 SDS 保存的字符串内容。|因为惰性空间释放策略，复杂度为 O(1) 。|
|sdscat|将给定 C 字符串拼接到 SDS 字符串的末尾。|O(N) ， N 为被拼接 C 字符串的长度。|
|sdscatsds|将给定 SDS 字符串拼接到另一个 SDS 字符串的末尾。|O(N) ， N 为被拼接 SDS 字符串的长度。|
|sdscpy|将给定的 C 字符串复制到 SDS 里面， 覆盖 SDS 原有的字符串。|O(N) ， N 为被复制 C 字符串的长度。|
|sdsgrowzero|用空字符将 SDS 扩展至给定长度。|O(N) ， N 为扩展新增的字节数。|
|sdsrange|保留 SDS 给定区间内的数据， 不在区间内的数据会被覆盖或清除。|O(N) ， N 为被保留数据的字节数。|
|sdstrim|接受一个 SDS 和一个 C 字符串作为参数， 从 SDS 左右两端分别移除所有在 C 字符串中出现过的字符。|O(M*N) ， M 为 SDS |的长度， N 为给定 C 字符串的长度。|
|sdscmp|对比两个 SDS 字符串是否相同。|O(N) ， N 为两个 SDS 中较短的那个 SDS 的长度。|

## 链表

adlist.h/listNode
adlist.h/list
```c
typedef struct listNode {
    // 前置节点
    struct listNode *prev;
    // 后置节点
    struct listNode *next;
    // 节点的值
    void *value;
} listNode;

typedef struct list {
    // 表头节点
    listNode *head;
    // 表尾节点
    listNode *tail;
    // 链表所包含的节点数量
    unsigned long len;
    // 节点值复制函数
    void *(*dup)(void *ptr);
    // 节点值释放函数
    void (*free)(void *ptr);
    // 节点值对比函数
    int (*match)(void *ptr, void *key);
} list;
```
![图示](http://redisbook.com/_images/graphviz-5f4d8b6177061ac52d0ae05ef357fceb52e9cb90.png)

### Redis链表特点
* 双端： 链表节点带有 prev 和 next 指针， 获取某个节点的前置节点和后置节点的复杂度都是 O(1) 。
* 无环： 表头节点的 prev 指针和表尾节点的 next 指针都指向 NULL ， 对链表的访问以 NULL 为终点。
* 带表头指针和表尾指针： 通过 list 结构的 head 指针和 tail 指针， 程序获取链表的表头节点和表尾节点的复杂度为 O(1) 。
* 带链表长度计数器： 程序使用 list 结构的 len 属性来对 list 持有的链表节点进行计数， 程序获取链表中节点数量的复杂度为 O(1) 。
* 多态： 链表节点使用 void* 指针来保存节点值， 并且可以通过 list 结构的 dup 、 free 、 match 三个属性为节点值设置类型特定函数， 所以链表可以用于保存各种不同类型的值。

### 链表和链表节点的主要API
|函数|作用|时间复杂度|
|---|---|---|
|listSetDupMethod|将给定的函数设置为链表的节点值复制函数。|O(1) 。|
|listGetDupMethod|返回链表当前正在使用的节点值复制函数。|复制函数可以通过链表的 dup 属性直接获得， O(1)|
|listSetFreeMethod|将给定的函数设置为链表的节点值释放函数。|O(1) 。|
|listGetFree|返回链表当前正在使用的节点值释放函数。|释放函数可以通过链表的 free 属性直接获得， O(1)|
|listSetMatchMethod|将给定的函数设置为链表的节点值对比函数。|O(1)|
|listGetMatchMethod|返回链表当前正在使用的节点值对比函数。|对比函数可以通过链表的 match 属性直接获得， O(1)|
|listLength|返回链表的长度（包含了多少个节点）。|链表长度可以通过链表的 len 属性直接获得， O(1) 。|
|listFirst|返回链表的表头节点。|表头节点可以通过链表的 head 属性直接获得， O(1) 。|
|listLast|返回链表的表尾节点。|表尾节点可以通过链表的 tail 属性直接获得， O(1) 。|
|listPrevNode|返回给定节点的前置节点。|前置节点可以通过节点的 prev 属性直接获得， O(1) 。|
|listNextNode|返回给定节点的后置节点。|后置节点可以通过节点的 next 属性直接获得， O(1) 。|
|listNodeValue|返回给定节点目前正在保存的值。|节点值可以通过节点的 value 属性直接获得， O(1) 。|
|listCreate|创建一个不包含任何节点的新链表。|O(1)|
|listAddNodeHead|将一个包含给定值的新节点添加到给定链表的表头。|O(1)|
|listAddNodeTail|将一个包含给定值的新节点添加到给定链表的表尾。|O(1)|
|listInsertNode|将一个包含给定值的新节点添加到给定节点的之前或者之后。|O(1)|
|listSearchKey|查找并返回链表中包含给定值的节点。|O(N) ， N 为链表长度。|
|listIndex|返回链表在给定索引上的节点。|O(N) ， N 为链表长度。|
|listDelNode|从链表中删除给定节点。|O(1) 。|
|listRotate|将链表的表尾节点弹出，然后将被弹出的节点插入到链表的表头， 成为新的表头节点。|O(1)|
|listDup|复制一个给定链表的副本。|O(N) ， N 为链表长度。|
|listRelease|释放给定链表，以及链表中的所有节点。|O(N) ， N 为链表长度。|

## 字典

哈希表 dict.h/dictht
哈希表节点 dict.h/dictEntry
字典 dict.h/dict
```c
typedef struct dictht {
    // 哈希表数组
    dictEntry **table;
    // 哈希表大小
    unsigned long size;
    // 哈希表大小掩码，用于计算索引值
    // 总是等于 size - 1
    unsigned long sizemask;
    // 该哈希表已有节点的数量
    unsigned long used;
} dictht;

typedef struct dictEntry {
    // 键
    void *key;
    // 值
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
    } v;
    // 指向下个哈希表节点，形成链表
    struct dictEntry *next;
} dictEntry;

typedef struct dict {
    // 类型特定函数
    dictType *type;
    // 私有数据
    void *privdata;
    // 哈希表
    dictht ht[2];
    // ht[0] 用来存储数据 当字典进行伸缩等操作时 用ht[1]用来过渡中间期 具体参考rehash过程
    // rehash 索引
    // 当 rehash 不在进行时，值为 -1
    int rehashidx; /* rehashing not in progress if rehashidx == -1 */
} dict;
```
![图示](http://redisbook.com/_images/graphviz-e73003b166b90094c8c4b7abbc8d59f691f91e27.png)

### 主要知识点
* redis采用MurmurHash2 算法来计算hash值，特点是能够让有规律的输入呈现很好的随机分布性，并且速度较快
* hash会根据以下条件进行伸缩：1,服务器目前没有在执行 BGSAVE 命令或者 BGREWRITEAOF 命令， 并且哈希表的负载因子大于等于 1；2、服务器目前正在执行 BGSAVE 命令或者 BGREWRITEAOF 命令， 并且哈希表的负载因子大于等于 5 ；其中负载因子 = 哈希表已保存节点数量 / 哈希表大小
* 渐进式rehash hash表在进行伸缩时会重新对每一个元素进行hash,在这个过程中新hash的元素会放入ht[1],此时若对字典进行CRUD操作会同时作用在ht[0]和ht[1]上，直到所有元素都rehash后，ht[0]将取代ht[1]


### 字典主要API
|函数|作用|时间复杂度|
|---|---|---|
|dictCreate|创建一个新的字典。|O(1)|
|dictAdd|将给定的键值对添加到字典里面。|O(1)|
|dictReplace|将给定的键值对添加到字典里面， 如果键已经存在于字典，那么用新值取代原有的值。|O(1)|
|dictFetchValue|返回给定键的值。|O(1)|
|dictGetRandomKey|从字典中随机返回一个键值对。|O(1)|
|dictDelete|从字典中删除给定键所对应的键值对。|O(1)|
|dictRelease|释放给定字典，以及字典中包含的所有键值对。|O(N) ， N 为字典包含的键值对数量。|
