# 技术面试必备基础知识-Redis

## 基本概念
- Redis：非关系型（Non-relational database, NoSQL）的远程内存键值数据库，可存储（key）和五种不同类型值（value）之间的映射（Mapping）。
- Redis 特性介绍
	- 将内存中的数据持久化到硬盘中，使用复制来扩展 `读性能`。
		- 快照持久化（Snapshotting）：将存在于某一时刻的所有数据都写入硬盘中。
		- AOF 持久化（Append-only file）：在执行写命令时，把执行写命令复制到硬盘中。
	- 使用客户端 `分片` 来扩展 `写性能`。

		> `分片`：将数据划分为多个部分的方法，数据划分可基于键包含的ID、基于键的散列值或者两者的组合进行。数据分片后，数据可存储于多台服务器中。

## 数据类型
- 键的数据类型：字符串
- 值的数据类型：字符串、列表、集合、散列表、有序集合

	| 数据类型 | 存储的值 | 操作 |
	| :---: | :---: | :--- |
	| String | 字符串 / 整数 / 浮点数 | 对整个字符串或者字符串的其中一部分执行操作<br>对整数和浮点数执行自增 / 自减操作 |
	| List | 列表 | 从两端压入或者弹出元素<br>对单个或者多个元素进行修剪，只保留一个范围内的元素 |
	| Set | 无序集合 | 添加、获取、移除单个元素<br>检查一个元素是否存在于集合中<br>计算交集、并集、差集<br>从集合里面随机获取元素 |
	| Hash | 包含键值对的无序散列表 | 添加、获取、移除单个键值对<br>获取所有键值对<br>检查某个键是否存在 |
	| Zset | 有序集合 | 添加、获取、删除元素<br>根据分值范围或者成员来获取元素<br>计算一个键的排名 |

## 快速上手
### 字符 String

```bash
> set key_name value
OK
> get key_name
"value"
> del key_name
(integer) 1
get key_name
(nil)
```

### 列表 List

```bash
# 向列表推入新元素后会返回列表当前长度
> rpush list_key item1
(integer) 1
> rpush list_key item2
(integer) 2
> rpush list_key item3
(integer) 3
	
# 0 为起始索引，-1 为结束索引，取出指定范围内的所有元素
> lrange list_key 0 -1
1) "item1"
2) "item2"
3) "item3"
	
# 取出指定索引的元素
> lindex list_key 1
"item2"
	
# 从列表中弹出一个元素（出队）
> lpop list_key
"item1"
```

### 集合 Set

```bash
# 向集合中添加一个元素
# 返回 1 表示元素被成功追加，返回 0 表示元素已存在集合中
> sadd set_key item1
(integer) 1
> sadd set_key item2
(integer) 1
> sadd set_key item1
(integer) 0
	
# 获取集合所有元素 --> 序列
> sismember set_key
1) "item1"
2) "item2"
	
# 检查元素是否存在于集合中
> sismember set_key item3
(integer) 0
> sismember set_key item1
(integer) 1
	
# 从集合中移除某元素，成功返回 1，失败返回 0（键不存在）
> srem set_key item2
(integer) 1
> srem set_key item2
(integer) 0
```

### 散列 Hash

```bash
# 向散列表中添加键值对
> hset hash-key hkey1 value1
(integer) 1
> hset hash-key hkey2 value2
(integer) 1
> hset hash-key hkey1 value3
(integer) 0
	
# 获取散列表中所有键值对 --> 字典
> hgetall hash-key
1) "hkey1"
2) "value1"
3) "hkey2"
4) "value2"

# 获取散列表中某个键值
> hget hash-key hkey1
2) "value1"

# 删除键值对，成功返回 1，失败返回 0（键不存在）
> hdel hash-key hkey1
> (integer) 1
> hdel hash-key hkey1
> (integer) 0 
```

### 有序集合 Zset

```bash
# 向有序集合中添加元素，成功返回 1，失败返回 0（键已存在）
# 示例：zadd zset_key 分值 元素（成员）
> zadd zset_key 100 member1
(integer) 1
> zadd zset_key 200 member0
(integer) 1
> zadd zset_key 100 member0
(integer) 0

# 获取有序集合所有元素，多个元素按照分值大小排序
> zrange zset_key 0 -1 withscores
1) "member1"
2) "100"
3) "member0"
4) "200"

# 根据分支筛选有序集合的一部分元素
> zrangebyscore zset_key 150 300 withscores
1) "member0"
2) "200"

# 0 为起始索引，-1 为结束索引，取出指定范围内的所有元素
> zrange zset_key 0 -1 withscores
1) "member1"
2) "100"
3) "member0"
4) "200"

# 移除有序元素中的某个元素，成功返回 1，失败返回 0
> zrem zset_key member1
(integer) 1
> zrem zset_key member1
(integer) 0
```

## 数据结构
### 字典
- dictht 是一个散列表结构，使用拉链法解决哈希冲突。

### 跳跃表

## 核心概念
### 持久化

### 复制

### 处理故障

### 事务

### 事件

## 使用场景

## 参考资料
- [Cyc2018. Redis [OL]. cyc2018](https://cyc2018.xyz/数据库/Redis.html)
- [JosiahL.Carlson. Redis in Action [M]. Manning Publications. 2012](https://redis.com/ebook/redis-in-action/)