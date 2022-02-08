* [redis](#redis)
   * [数据结构](#数据结构)
      * [字符串](#字符串)
      * [哈希表](#哈希表)
      * [列表](#列表)
      * [集合](#集合)
      * [有序集合](#有序集合)
      * [其他](#其他)

# redis

## 数据结构
![image](https://user-images.githubusercontent.com/6240382/151688527-56d616a4-0542-40f8-9542-2f6e88350090.png)


### 字符串

- setnx可以作为分布式锁的实现方法；
- 内部编码
```

字符串类型的内部编码有3种：

·int：8个字节的长整型。

·embstr：小于等于39个字节的字符串。

·raw：大于39个字节的字符串。

Redis会根据当前值的类型和长度决定使用哪种内部编码实现。


```


### 哈希表


- 内部编码

```

- ziplist
- hashtable

当field个数比较少且没有大的value时，内部编码为ziplist：
当有value大于64字节，内部编码会由ziplist变为hashtable：
当field个数超过512，内部编码也会由ziplist变为hashtable：



```

- ziplist
![image](https://user-images.githubusercontent.com/6240382/151689421-f802292e-4de9-4fe9-85cf-5dddb1dee9a3.png)

- 场景
    - 复杂api频控 


### 列表

- 内部编码
```


- ziplist
- linkedlist

当元素个数较少且没有大元素时，内部编码为ziplist
当元素个数超过512个，内部编码变为linkedlist：
或者当某个元素超过64字节，内部编码也会变为linkedlist：



```

- 场景
    - lpush+lpop=Stack（栈）
    - lpush+rpop=Queue（队列）
    - lpsh+ltrim=Capped Collection（有限集合）
    - lpush+brpop=Message Queue（消息队列）

### 集合

- 内部编码
```


- intset
- hashtable

当元素个数较少且都为整数时，内部编码为intset：
当元素个数超过512个，内部编码变为hashtable：
当某个元素不为整数时，内部编码也会变为hashtable：
```
- 场景
    - 用户标签
    - spop/srandmember=Random item（生成随机数，比如抽奖）
    - sadd+sinter=Social Graph（社交需求）
    
### 有序集合

- 内部编码

```

ziplist
skiplist

当元素个数较少且每个元素较小时，内部编码为ziplist：
当元素个数超过128个，内部编码变为skiplist：
当某个元素大于64字节时，内部编码也会变为hashtable：


```

- skiplist
![image](https://user-images.githubusercontent.com/6240382/151689444-5cb681b0-cd16-4103-9be3-96dc036bd654.png)
    - 数据结构
        - 表头（head）：负责维护跳跃表的节点指针。
        - 跳跃表节点：保存着元素值，以及多个层。
        - 层：保存着指向其他元素的指针。高层的指针越过的元素数量大于等于低层的指针，为了提高查找的效率，程序总是从高层先开始访问，然后随着元素值范围的缩小，慢慢降低层次。
        - 表尾：全部由 NULL 组成，表示跳跃表的末尾。
    - 跳跃表是一种随机化数据结构，查找、添加、删除操作都可以在对数期望时间下完成。
    - 跳跃表目前在 Redis 的唯一作用，就是作为有序集类型的底层数据结构（之一，另一个构成有序集的结构是字典）。
    - 为了满足自身的需求，Redis 基于 William Pugh 论文中描述的跳跃表进行了修改，包括：
        - score 值可重复。
        - 对比一个元素需要同时检查它的 score 和 memeber 。
        - 每个节点带有高度为 1 层的后退指针，用于从表尾方向向表头方向迭代。


- 场景
    - 排行榜
## 其他
- 纯内存存储、IO多路复用技术、单线程架构是造就Redis高性能的三个因素
- 由于Redis的单线程架构，所以需要每个命令能被快速执行完，否则会存在阻塞Redis的可能，理解Redis单线程命令处理机制是开发和运维Redis的核心之一。
- move、dump+restore、migrate是Redis发展过程中三种迁移键的方式，其中move命令基本废弃，migrate命令用原子性的方式实现了dump+restore，并且支持批量操作，是Redis Cluster实现水平扩容的重要工具。
- scan命令可以解决keys命令可能带来的阻塞问题，同时Redis还提供了hscan、sscan、zscan渐进式地遍历hash、set、zset。








