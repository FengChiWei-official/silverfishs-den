---
tags:
  - type/lit
  - topic/learning
  - status/archive
source: 《卡片盒笔记法》 by Sönke Ahrens
---

## Text
1. CPU utility to CPU spare time(I/O waiting time)
2. I/O waiting time,  process run time -> finish time 
3. How to use muti-process to minimize download time.
4. defference between muti-process and muti-thread(mem share)
5. fork behavious?  fork with all thread? single thread process
6. 时钟中断， yield
7. 文件服务器
8. 用户态 服务器
9. 同步： run thread by order
10. thread's global variable(crete_globe)
11. user mode thread' condiction of implement v.s. asynic interrupt
12. 轮换法/peterson/
13. priority reverse ? in user mode thread
14. stack (structure) of process/thread
15. productor-comsumer
16. hu chi
17. how to implement 信号量 in `可以屏蔽中断的操作系统`
18. 调度算法和优先级反转
19. 最短平均响应时间
20. 调度算法
21. 系统的可调度性
22. 

## Thoughts
1. 性能评价与计算 (1, 2, 19, 20, 21) 

- **CPU 利用率
- **调度算法：** **FCFS**（先来先服务）、**RR**（轮转法）、**Priority**（优先级）。**SJF/SRTN**（最短作业/剩余时间优先）能提供理论上**最短的平均周转/响应时间**。
- **可调度性：** 在实时系统中，指所有任务能否在其截止时间内完成。 

2. 进程与线程的本质 (4, 5, 14) 

- **进程 (Process)：** 资源分配的单位，拥有独立的地址空间、PCB 和 **栈**。
- **线程 (Thread)：** 调度的基本单位，共享所属进程的内存（全局变量、堆、文件描述符），但拥有独立的 **寄存器集合和栈**。
- **Fork 行为：** 现代 Linux 的 `fork()` 只复制调用线程（Single-threaded child）。如果父进程是多线程，子进程中其他线程会悄然消失。 

3. I/O 与并发优化 (3, 7, 8) 

- **并发下载：** 多个下载进程能利用 CPU 闲置时间（I/O Wait），通过建立多个 TCP 连接突破服务器单连接限速。
- **用户态服务器：** 如微内核架构，文件系统作为独立进程运行。优点是可靠性高（一个挂掉不影响内核），缺点是频繁的上下文切换导致开销大。 

4. 进程同步与互斥 (9, 12, 15, 16, 17) 

- **核心挑战：** 解决**竞态条件 (Race Condition)**。
- **互斥 (Mutual Exclusion)：** 通过 **Peterson 算法**（软件模拟）、**屏蔽中断**（仅限单核内核态）或 **信号量 (Semaphore)** 实现。
- **实现信号量：** 在可屏蔽中断的 OS 中，通过“关中断 -> 操作信号量 -> 开中断”来保证原子性。
- **生产者-消费者模型：** 解决同步（缓冲区满/空）与互斥（操作缓冲区）的经典案例。 

5. 线程高级课题 (10, 11, 13, 18) 

- **线程全局变量：** 线程间通过全局变量通信，但需注意互斥；或使用 **Thread-Local Storage (TLS)** 实现线程私有的全局变量。
- **用户态线程：** 内核感知不到用户态线程，若一个线程被 I/O 阻塞，整个进程都会挂起。它们无法直接处理硬件异步中断，需内核协助。
- **优先级反转 (Priority Inversion)：** 低优先级线程持有高优先级线程所需的锁，导致高优先级线程被迫等待。**解决方案：** 优先级继承协议（低优先级暂时提升到高优先级水平）。 

6. 底层运行机制 (6) 

- **时钟中断：** 硬件定时触发，是 OS 夺回 CPU 控制权的唯一手段，也是实现 **抢占式调度** 和 `yield`（主动放弃 CPU）的基础。 

关键复习建议： 

1. **理解“栈”：** 记住**每个线程都有自己的私有栈**，这是面试常考点。
2. **区分同步与互斥：** 互斥是“我进你不能进”，同步是“我做完你才能做”。
3. **优先级反转：** 重点掌握“优先级继承”的概念，它是实时系统稳定性的关键。 

这些知识点涵盖了操作系统最底层的运作逻辑。如果你正在准备 2026 年的考研（408）或大厂后端面试，深入理解这些概念之间的**因果关系**（例如：为什么需要时钟中断？ -> 为了实现抢占式调度算法）至关重要。