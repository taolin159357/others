* redis常见数据结构以及使用场景分析
  * String，常规计数场景：微博数，粉丝数，页面访问次数等，涉及命令：incr、decr、incrby、decrby。
  * List，列表场景：微博的关注列表，粉丝列表，消息列表，涉及指令：lrange（进行分页查询）
  * Hash，存储对象信息：用户信息，商品信息。
  * Set，求交集场景：共同关注、共同粉丝、共同喜好，涉及指令：sinterstore。
  * ZSet，排行榜场景：网易云音乐歌曲排行榜、在线用户列表，直播弹幕消息。

* redis设置过期时间了解吗
  * redis中有个设置时间过期的功能，即对存储在 redis 数据库中的key可以设置一个过期时间。作为一个缓存数据库，这是非常实用的。如我们一般项目中的token或者一些登录信息以及短信验证码都是有时间限制的，按照传统的数据库处理方式，一般都是自己判断过期，这样无疑会严重影响项目性能。使用redis的expire指令可以很好的完成这个需求。

* redis是怎么对过期的key进行删除的？
  * 定期删除+惰性删除。
  * 定期删除：redis默认是每隔 100ms 就随机抽取一些设置了过期时间的key，检查其是否过期，如果过期就删除。注意这里是随机抽取的。为什么要随机呢？你想一想假如 redis 存了几十万个 key ，每隔100ms就遍历所有的设置过期时间的 key 的话，就会给 CPU 带来很大的负载！
  * 惰性删除 ：定期删除可能会导致很多过期 key 到了时间并没有被删除掉。所以就有了惰性删除。假如你的过期 key，靠定期删除没有被删除掉，还停留在内存里，除非你的系统去查一下那个 key，才会被redis给删除掉。这就是所谓的惰性删除，也是够懒的哈！

* redis 持久化机制RDB、AOF。
  * RDB方式：创建数据库快照来持久化redis服务器的数据，将数据保存到rdb文件中。
  * AOF方式：开启AOF持久化后每执行一条会更改Redis中的数据的命令，Redis就会将该命令写入硬盘中的AOF文件，从而实现redis的持久化，并且还会定时的重写AOF文件，以免文件过大。
    * AOF 重写：AOF重写机制并不是对原AOF文件进行重写，而是创建一个临时的AOF文件，然后根据redis的键值对来实现数据持久化，最后再覆盖原AOF文件。重写后的AOF文件和原AOF文件存储的数据一样，但是大小会更轻。

* zSet的底层实现：zSet的底层有两种：ziplist和skiplist。使用ziplist的两个条件：元素个数小于128；所有value的长度小于64字节。否则使用skiplist来实现zset。

  * ziplist介绍：ziplist编码的有序集合使用紧挨在一起的压缩列表节点来保存，第一个节点保存member，第二个保存score。ziplist内的集合元素按score从小到大排序，score较小的排在表头位置。

  * skiplist介绍：跳表(skip List)是一种随机化的数据结构，基于并联的链表，实现简单，插入、删除、查找的复杂度均为O(logN)。简单说来跳表也是链表的一种，只不过它在链表的基础上增加了跳跃功能，正是这个跳跃的功能，使得在查找元素时，跳表能够提供O(logN)的时间复杂度。

    * skiplist的搜索和插入过程

      搜索：首先在最高层单链表开始找，如果找不到就往下一层单链表找，直到找到为止。

      插入：插入之前需要先经历搜索过程，找到了插入的位置之后，随机在一层中分配这个新节点。