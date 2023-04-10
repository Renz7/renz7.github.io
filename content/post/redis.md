---
title: "Redis"
date: 2023-04-10T16:29:33+08:00
draft: false
summary: redis interview
tags:
- intetview
---

### Redis数据类型
1. str
4. list
2. set
3. zset  
   实现原理: dict+zskiplst dict存贮数据,zskip用于排序和range操作
4. hash
   hash冲突解决方案: 链式寻址法
5. stream


### redis队列的实现
1. list的lpush rpop
2. pub sub问题
3. stream

存在的问题
1. 没有ack机制
2. 消息丢失,pubsub不能确保送达, strean存在sizexianzhi
3. 不能重复消费


### 集群以及高可用
1. 主从 
   1. 主数据库负责写数据
2. 哨兵  
    1. 单哨兵 解决主从模式的选主问题,存在单点故障.
    2. 多哨兵 通过订阅主数据库`_sentinel_:hello`发现其他哨兵
    3. 问题
       1. 中心化方案,只有主节点处理请求
       2. 全量副本,浪费空间
3. cluster模式(3.0之后引入)
   1. 是一种sharding的技术,采用多主多从的方案(三主三从6节点).节点相互彼此互联
   2. 采用hash槽来确定key的为主
   3. 16384个hash槽, crc16 mod 16384 计算key的位置
      1. 大数量心跳包大
      2. 支持的节点够用
      3. 主节点的槽点使用bitmap存贮,槽位少压缩比高

### 数据持久化
1. RDB 快照
   定期全量存储
   1. 使用save 会堵塞服务进程
   2. dgsave用于主从同步的初始化,会fork一个子线程,不会堵塞
2. AOF
    记录日志
    1. 数据完整,秒级丢失
    2. 存贮空间大
    3. 采用fsync ,速度慢

#### 一致性hash
1. 环形空间
2. 数据倾斜问题解决方案,可以吧实体节点映射为虚拟节点

#### hash冲突的解决办法
1. 开放寻址法,在当前的地址链上寻找下一个可用的槽 `python的 dict`
2. 再hash法, 使用不同的hash函数多次进行hash运算
3. 拉链法, 使用链表将冲突的key串起来 `java HashMap(会进一步使用红黑树) redis hash`