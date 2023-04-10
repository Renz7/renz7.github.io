---
title: "Python"
date: 2023-04-10T17:01:47+08:00
draft: false
summary: pyton basic
tags:
- interview
- python
---

### python collections

1. list
   1. 尽量使用append(),O(1)复杂度
   2. 避免使用 `[1,2]+[3,4]`,使用`extend()`原地修改
2. dict
   hash冲突解决方案:开放寻址
3. set
4. tuple

### 可迭代对象,迭代器,生成器
1. 可迭代对象 
   1. 实现`__iter__`
   2. ` isinstance([1,2],Iterable)==True`
2. 迭代器
   1. 同时实现了`__iter__``next()`
   2. 构建一个迭代器 `iter([1,2,4])`
3. 生成器
   1. 生产器函数
    ```python
    func iterN(n):
        for i in range(n):
            yield i

    ```
   3. 生成表达式 
      1. 列表生成式 `[ i for i in range(10) ]`
      1. 生成器生成式 `( i for i in range(10) )`


### python 协程
1. 可等待对象(3种)
   1. 协程
   2. Task
      1. `task = asyncio.create_task(nested())` ,task是可以取消的
   3. Future
      1. 低层级的可等待对象
      2. examle: `async.gather()`

2. 实现原理
   1. 基于事件循环`eventloop`,回调实现
   2. 基于生成器的设计 从 `yeild -> yeild from -> asyncio -> sync/awiat`
   3. [理解Python协程的本质](https://zhuanlan.zhihu.com/p/330549526)
   4. [How the heck does async/await work in Python 3.5?](https://snarky.ca/how-the-heck-does-async-await-work-in-python-3-5/)
   5. summary `Basically async and await are fancy generators that we call coroutines and there is some extra support for things called awaitable objects and turning plain generators in to coroutines`