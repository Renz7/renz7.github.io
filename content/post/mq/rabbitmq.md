---
title: "Rabbitmq"
date: 2023-04-07T15:43:09+08:00
draft: false
summary: rabbitmq interview
tags:
- intetview
---

## Rabitmq

### Why MQ
1. 解耦
2. 异步
3. 削峰

###  Disadvantage
1. 可用性降低
2. 复杂度提升
3. 数据一致性问题

### Exchange 类型
1. Direct
2. Fanout
3. topic  
    匹配规则 
   - Routingkey 和Bindngkey 以`.`为分隔符 `custom.msg.push`
   - BindingKey 使用`#/*`来匹配 `*`一个`#`多个或者0
4. headers(一般使用topic)  
   根据消息头的headers属性来匹配


### 数据一致性问题
#### 重复消费
消费者消费之后,由于网络原因没有发送ack,其他消费者重复消费  
*关键词*: 唯一性和幂等性
1. 修改业务逻辑,确保多次消费不影响结果
2. 基于其他约束,确保单次消费.比如数据库主键约束
3. 记录关键key,消费之前进行确认. 数据库unique index, redis set

#### 消息丢失
1. 生产者消息丢失
   - rabbitmq 事务机制  
     发送异常可以回滚.会因为同步堵塞问题引起吞吐量下降
   - confirm模式  
    confirm模式下消息会被分配id.消息被投递到匹配的queue之后会发送ack,否则会发送nack.优点: 异步的,可以实现ack,nack回调进一步处理
2. broker 消息丢失
   - 开启持久化
   - 镜像队列
3. 消费者消息丢失  
   一般是因为自动confirm机制导致的.auto confirm 会在收到消息之后发生ack,这是队列消息被删除
   - 解决方案: 手动confirm.消费异常不confirm
#### 消息有序性
1. 单一消费者对应单一线程的queue
2. 多个线程场景下,开启重试机制,需要重试的msg压入重试队列,等待前面的消息消费结束

#### 其他
1. 消息基于channal传输,复用TCP链接
2. 延迟队列实现: 通过消息TTL+死信队列DLX
3. 至少需要一个磁盘节点.如果磁盘节点宕机,不能新建exchange.queue.但是可以继续使用