---
title: "Go"
date: 2023-04-10T18:00:53+08:00
draft: false
tags:
- interview 
- go
---

## GMP

M: machine,对应系统核心线程
P: processor, 绑定m和G
G: gonrutine go协程



### go scheduler
发生调度的时机

1. `go` 新建goroutine
2. `GC` GC会新建一个goroutine
3. `系统调用` 系统调用时 会堵塞M,所有该goroutine会被调度走
4. `同步访问` atomic,mutex,chanal操作是goroutine堵塞.

### 工作窃取
当P2的本地队列没有G,优先从全局的G队列拿,如果还没有就会


## CSP
`Do not communicate by sharing memory; instead, share memory by communicating.`

CSP 全称是 “Communicating Sequential Processes”，这也是 Tony Hoare 在 1978 年发表在 ACM 的一篇论文。论文里指出一门编程语言应该重视 input 和 output 的原语，尤其是并发编程的代码。

原则: 尽量使用 channel；把 goroutine 当作免费的资源，随便用。

link [CSP](https://golang.design/go-questions/channel/csp/)


## GC
两类GC
1. 追踪式GC
   1. 标记清楚: 
   2. 标记整理: 解决内存碎片的问题
   3. 增量式: 将标记与清扫的过程分批执行，每次执行很小的部分，从而增量的推进垃圾回收，达到近似实时、几乎无停顿的目的
   4. 增量整理
   5. 分代
2. 引用计数: 根据对象自己的引用计数俩回收,引用为0立即回收

不整理,并发的 三色标记清除算法


1. 对象整理的优势是解决内存碎片问题以及“允许”使用顺序内存分配器。但 Go 运行时的分配算法基于 tcmalloc，基本上没有碎片问题。 并且顺序内存分配器在多线程的场景下并不适用。Go 使用的是基于 tcmalloc 的现代内存分配算法，对对象进行整理不会带来实质性的性能提升。
2. 分代 GC 依赖分代假设，即 GC 将主要的回收目标放在新创建的对象上（存活时间短，更倾向于被回收），而非频繁检查所有对象。但 Go 的编译器会通过逃逸分析将大部分新生对象存储在栈上（栈直接被回收），只有那些需要长期存在的对象才会被分配到需要进行垃圾回收的堆中。也就是说，分代 GC 回收的那些存活时间短的对象在 Go 中是直接被分配到栈上，当 goroutine 死亡后栈也会被直接回收，不需要 GC 的参与，进而分代假设并没有带来直接优势。并且 Go 的垃圾回收器与用户代码并发执行，使得 STW 的时间与对象的代际、对象的 size 没有关系。Go 团队更关注于如何更好地让 GC 与用户代码并发执行（使用适当的 CPU 来执行垃圾回收），而非减少停顿时间这一单一目标上。

## 内存泄露的主要可能
1. 全局变量
2. goroutine