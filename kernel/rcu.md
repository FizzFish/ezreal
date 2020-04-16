---
layout: default
---

## 引入
1. spinlock、读写信号量、mutex都使用了原子操作指令，多CPU争用会降低cache一致性；此外，以读写信号量为例，不允许多个写者同时存在。
2. RCU的设计希望读线程之间没有同步开销（锁、原子操作、内存屏障）；写线程等待所有读线程完成后才消除旧数据。
3. RCU常用于链表

## API
* rcu_read_lock()/rcu_read_unlock()组成一个RCU读临界
* rcu_deregerence()获取RCU保护的指针
* rcu_assign_pointer()发布写数据
* synchronize_rcu()等待所有读操作结束
* call_rcu()注册读访问结束后的回调函数

## 经典RCU vs Tree RCU
* 宽限期(Grace Period, GP):所有读线程离开了临界区

