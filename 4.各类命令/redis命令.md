云服务器中操作

* redis-server /usr/local/redis/redis-single/redis.conf 
* ps axu|grep redis
* redis-cli shutdown #没有设置密码，运行此行代码
* redis-cli -a 123456 shutdown #设置密码，运行此行
* redis-cli -h 127.0.0.1 -p 6379 -a 123456 #登录

# 常用命令

### string

* `set,get,strlen,del,exists,dect,incr`
* `mset,mget`
* `expire,setex,ttl`

### list

* `rpush,rpop,lpop,lpush,lrange,llen`

### hash

* `hset,hmset,hexists,hget,hgetall,hkeys,hvals`

### set

* `sadd,spop,smembers,sismemeber,scard,sinterstore,sunion`

### zset

* `zdd,zcard,zscore,zrange,zrevrange,zrem`

### 清除

`flushall`清空所有库，所有key

`flushdb` 清空当前库，所有key

### 查看

`info`可以查看所有库的key的数量

`dbsize`则是当前库key的数量

`keys *` 统计当前有效的key，和dubsize不同（有效和未被销毁不一样）**，不推荐使用**

# String

**普通字符串的基本操作：**

```bash
127.0.0.1:6379> set key val #设置 key-value 类型的值
OK
127.0.0.1:6379> get key # 根据 key 获得对应的值
"val"
127.0.0.1:6379> exists key  # 判断某个 key 是否存在,存在返回1，不存在返回0
(integer) 1
127.0.0.1:6379> strlen key # 返回 key 所储存的字符串值的长度。
(integer) 3
127.0.0.1:6379> del key # 删除某个 key 对应的值
(integer) 1
127.0.0.1:6379> get key
(nil)
```

**批量设置** :

``` bash
127.0.0.1:6379> mset key1 val1 key2 val2 # 批量设置 key-value 类型的值
OK
127.0.0.1:6379> mget key1 key2 # 批量获取多个 key 对应的 value
1) "val1"
2) "val2"
```

**计数器（字符串的内容为整数的时候可以使用）：**

``` bash
127.0.0.1:6379> set number 5
OK
127.0.0.1:6379> incr number # 将 key 中储存的数字值增一
(integer) 6
127.0.0.1:6379> get number
"6"
127.0.0.1:6379> decr number # 将 key 中储存的数字值减一
(integer) 5
127.0.0.1:6379> get number
"5"
```

**过期**：

``` bash
127.0.0.1:6379> expire key  100 # 数据 100s 后过期
(integer) 1
127.0.0.1:6379> setex key 100 value # set数据并且设置过期时间是100s
OK
127.0.0.1:6379> ttl key # 查看数据还有多久过期，返回值为-2代表不存在，-1表示没有设置过期时间
(integer) 56
```

# list

**通过 `rpush/lpop` 实现队列：**

``` bash
127.0.0.1:6379> rpush myList value1 # 向 list 的头部（右边）添加元素
(integer) 1
127.0.0.1:6379> rpush myList value2 value3 # 向list的头部（最右边）添加多个元素
(integer) 3
127.0.0.1:6379> lpop myList # 将 list的尾部(最左边)元素取出
"value1"
127.0.0.1:6379> lrange myList 0 1 # 查看对应下标的list列表， 0 为 start,1为 end
1) "value2"
2) "value3"
127.0.0.1:6379> lrange myList 0 -1 # 查看列表中的所有元素，-1表示倒数第一
1) "value2"
2) "value3"
```

**通过 `rpush/rpop` 实现栈：**

``` bash
127.0.0.1:6379> rpush myList2 value1 value2 value3
(integer) 3
127.0.0.1:6379> rpop myList2 # 将 list的头部(最右边)元素取出
"value3"
```

我专门花了一个图方便小伙伴们来理解：

![redis list](https://gitee.com/stiwen/images_bed/raw/master/img/redis-list2.png)

**通过 `lrange` 查看对应下标范围的列表元素：**

``` bash
127.0.0.1:6379> rpush myList value1 value2 value3
(integer) 3
127.0.0.1:6379> lrange myList 0 1 # 查看对应下标的list列表， 0 为 start,1为 end
1) "value1"
2) "value2"
127.0.0.1:6379> lrange myList 0 -1 # 查看列表中的所有元素，-1表示倒数第一
1) "value1"
2) "value2"
3) "value3"
```

通过 `lrange` 命令，你可以基于 list 实现分页查询，性能非常高！

**通过 `llen` 查看链表长度：**

``` bash
127.0.0.1:6379> llen myList
(integer) 3
```

# hash

``` bash
127.0.0.1:6379> hset userInfoKey name "guide" description "dev" age "24"
OK
127.0.0.1:6379> hexists userInfoKey name # 查看 key 对应的 value中指定的字段是否存在。
(integer) 1
127.0.0.1:6379> hget userInfoKey name # 获取存储在哈希表中指定字段的值。
"guide"
127.0.0.1:6379> hget userInfoKey age
"24"
127.0.0.1:6379> hgetall userInfoKey # 获取在哈希表中指定 key 的所有字段和值
1) "name"
2) "guide"
3) "description"
4) "dev"
5) "age"
6) "24"
127.0.0.1:6379> hkeys userInfoKey # 获取 key 列表
1) "name"
2) "description"
3) "age"
127.0.0.1:6379> hvals userInfoKey # 获取 value 列表
1) "guide"
2) "dev"
3) "24"
127.0.0.1:6379> hset userInfoKey name "GuideGeGe" # 修改某个字段对应的值
127.0.0.1:6379> hget userInfoKey name
"GuideGeGe"
```

# set

``` bash
127.0.0.1:6379> sadd mySet value1 value2 # 添加元素进去
(integer) 2
127.0.0.1:6379> sadd mySet value1 # 不允许有重复元素
(integer) 0
127.0.0.1:6379> smembers mySet # 查看 set 中所有的元素
1) "value1"
2) "value2"
127.0.0.1:6379> scard mySet # 查看 set 的长度
(integer) 2
127.0.0.1:6379> sismember mySet value1 # 检查某个元素是否存在set 中，只能接收单个元素
(integer) 1
127.0.0.1:6379> sadd mySet2 value2 value3
(integer) 2
127.0.0.1:6379> sinterstore mySet3 mySet mySet2 # 获取 mySet 和 mySet2 的交集并存放在 mySet3 中
(integer) 1
127.0.0.1:6379> smembers mySet3
1) "value2"
```

# zset

``` bash
127.0.0.1:6379> zadd myZset 3.0 value1 # 添加元素到 sorted set 中 3.0 为权重
(integer) 1
127.0.0.1:6379> zadd myZset 2.0 value2 1.0 value3 # 一次添加多个元素
(integer) 2
127.0.0.1:6379> zcard myZset # 查看 sorted set 中的元素数量
(integer) 3
127.0.0.1:6379> zscore myZset value1 # 查看某个 value 的权重
"3"
127.0.0.1:6379> zrange  myZset 0 -1 # 顺序输出某个范围区间的元素，0 -1 表示输出所有元素
1) "value3"
2) "value2"
3) "value1"
127.0.0.1:6379> zrange  myZset 0 1 # 顺序输出某个范围区间的元素，0 为 start  1 为 stop
1) "value3"
2) "value2"
127.0.0.1:6379> zrevrange  myZset 0 1 # 逆序输出某个范围区间的元素，0 为 start  1 为 stop
1) "value1"
2) "value2"
```
