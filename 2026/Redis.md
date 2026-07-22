# 快速知识点
    1、全拼 Remote Dictionary Service远程字典表服务
    2、内存操作所以读写快，而且数据结构都很简单，复杂度O(1)
    3、解决高并发场景
    4、单线程执行，是线程安全的（原子性操作，Append等）
    5、复杂要求（比如要求检测某个key的value是否包含某个值，如果包含则Append）没有这种命令，那么如何保证原子性？使用lua脚本
    6、数据结构String、List、Hash、set、ZSet(Sorted Set)、Bitmap(使用String实现)等
    7、string类型其实是个动态字符串（SDS动态字符串类型），而C#中string类型是不可变的，不支持修改某个位置的单个字符，只能通过方法（Substring/Replace/Insert等）创建一个新的字符串
# 基础操作命令：
1. 通用键操作（Key Operations）
    SET/GET‌：设置/获取键值对
‌    DEL‌：删除一个或多个键
‌    EXISTS‌：判断键是否存在
‌    TYPE‌：获取键存储的数据类型
‌    EXPIRE/PEXPIRE‌：设置键的过期时间（秒/毫秒）
‌    TTL/PTTL‌：查看键剩余生存时间
‌    KEYS/SCAN‌：查找符合模式的键（生产环境慎用 KEYS）
‌    RENAME‌：重命名键
2. 字符串（String）
‌    SET/GET‌：设值/取值
‌    MSET/MGET‌：批量设值/取值
‌    INCR/DECR‌：数值自增/自减 1
‌    INCRBY/DECRBY‌：数值增加/减少指定整数
‌    APPEND‌：追加字符串到末尾
‌    STRLEN‌：获取值长度
3. 哈希（Hash）
‌    HSET/HGET‌：设置/获取指定字段值
‌    HMSET/HMGET‌：批量设置/获取字段值
‌    HGETALL‌：获取所有字段及值
‌    HDEL‌：删除指定字段
‌    HEXISTS‌：判断字段是否存在
‌    HLEN‌：获取字段数量
‌    HINCRBY‌：字段数值递增
4. 列表（List）
‌    LPUSH/RPUSH‌：头部/尾部插入元素
‌    LPOP/RPOP‌：头部/尾部弹出并返回元素
‌    LRANGE‌：获取指定范围元素（如 0 -1 获取全部）
‌    LLEN‌：获取列表长度
‌    LINDEX‌：通过索引获取元素
‌    LREM‌：删除指定数量的匹配元素
‌    LTRIM‌：修剪列表保留指定范围
5. 集合（Set）
‌    SADD‌：添加成员（自动去重）
‌    SMEMBERS‌：获取所有成员
‌    SREM‌：移除成员
‌    SISMEMBER‌：判断成员是否存在
‌    SCARD‌：获取成员数量
‌    SPOP‌：随机弹出成员
‌    SINTER/SUNION/SDIFF‌：求交集/并集/差集
6. 有序集合（ZSet/Sorted Set）
‌    ZADD‌：添加成员及分数
‌    ZRANGE/ZREVRANGE‌：按排名范围获取成员（可带分数）
‌    ZREM‌：移除成员
‌    ZSCORE‌：获取成员分数
‌    ZRANK/ZREVRANK‌：获取成员排名
‌    ZCARD‌：获取成员数量
‌    ZCOUNT‌：统计指定分数范围内成员数
7. 连接与服务管理
‌    PING‌：测试连接连通性
‌    SELECT‌：切换数据库（默认 0-15）
‌    INFO‌：获取服务器统计信息
‌    FLUSHDB/FLUSHALL‌：清空当前/所有数据库 

# SDS动态字符串类型的实现
    struct sdshdr
    {
        //记录buf数组中已使用字符的数量，等于SDS保存字符串的长度
        int len;
        //记录buf数组中未使用的字符数量
        int free;
        //字符数组，用于保存字符串
        char buf[];
    }
  ## 扩容策略
    1 默认分配是512Byte
    2 当字符串所占空间小于1MB时，Redis对字符串存储空间的扩容是以成倍的方式增加的（*2）
    3 而当所占空间超过1MB时，每次扩容只增加1MB
    4 Redis字符串允许的最大值字节数是512MB

# 对象存取需求
  ## 字符串类型的缺陷 
    面临对象存储的时候string类型有两种方法：
    1、序列化。优点：读取方便，缺点：修改单个属性麻烦（先读取，反序列化，再修改，再序列化写入）
    2、按属性存多个键值。优点：修改方便，缺点：读取麻烦
  ## 解决方法:Hash散列
    一个key对应一个hash表field（也是一个key-value集合）
    hash类型是Redis常用数据类型之一，其底层存储结构有两种实现方式：
    1. 当存储的数据量较少的时，hash采用ziplist作为底层存储结构，此时要求符合以下
        两个条件：（ziplist是个双向链表，紧密摆放-压缩的-很节约空间）
        -哈希对象保存的每一对键值对（键和值）的字符串长度之和均小于64个字节。
        哈希对象保存的键值对数量要小于512个。
        （还记得string默认是分配512Byte一其实特别简单的字符串，还不如用Hash节约空间）
    2. 当无法满足上述条件时，hash就会采用第二种方式来存储数据，也就是dict（字典
        结构）底层是数组和链表相结合的方式存储数据。查找性能非常高效，其时间复杂度为
        O(1)。

# Set结构 
    Set是一个无序的、自动去重的集合数据类型，能做交叉并补操作，Set底层有两种数据结构存储：一个是hashtable，一个是inset

    满足2个条件，就用Inset(就是个速度)：1、元素个数不小于默认值512 ；2、元素可以用整型表示（值必须是整形）

    一般是用Hashtable，这个table跟普通hashtable不一样，只有field没有value
 ## 应用场景
    无序性：随机抽奖、随机歌单推荐
    自动去重：点赞数（一个账号只能点一次）、IP统计（一个ip只统计一次）

# ZSet结构
    ZSet是有序的，同时也是自动去重的
  ## 排序规则：
        优先score排序，score相同则元素字典序
  ## 存储规则： 
        1、满足两个条件采用ziplist存储：
            1> 有序集合保存的元素数量小于默认值128个
            2> 有序集合保存的所有元素的长度小于默认值64字节 
          优点：存储紧凑  缺点：内存移动成本高
        2、 否则使用 字典（dict） + 跳表（skipList）来存储 ，时间复杂度O(logN)
  ## 应用场景
        主播打赏排行榜  
    