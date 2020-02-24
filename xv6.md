# MLFQ
最近基于xv6实现了一个MLFQ调度算法，中间踩了好多坑。
## 原理
MLFQ是根据优先级的Round-robin调度算法，满足以下条件：
1. 多任务进程按优先级分为N个队列
2. 高优先级进程可抢占低优先级进程，发生在每次中断处理时(xv6更像是发生在时钟中断)
3. 当统一优先级队列存在多个进程，那么他们之间的调用采用Round-robin调度算法
4. 不同优先级的Round-robin时间片长度不同，优先级高的队列RR-slice较小，优先级低的队列RR-slice较大
5. 不同优先级的进程时间片不同，高优先级时间片较小，这个slice与RR-slice不是一回事~~，优先级为0的队列退化为一个FIFO调度算法~~
6. 进程消耗完当前队列的时间片后，会自动进入下一个优先级队列，并获得相应的时间片
7. 为了避免饥饿，当进程的等待时间超过对应优先级RR-slice * 10(10是经验值，与系统平均运行的进程数有关)时，进入高一级别的优先级队列
## 设计
1. struct proc
```c
struct proc {
    int priority;
    int state;
    int ticks[prinum];
    int wait_ticks[prinum];
}
```
2. scheduler
```c
void scheduler(void) {
   struct proc * best = getbest();//获取第一个best进程
    while(1) {//scheduler由0号进程调用，永不会退出
        sti();//开启当前CPU中断
        do {
            acquire(&ptable.lock);//hold the ptable lock
            best = selectbest(best, &round_slice);//基于当前最优进程和RR-slice，选出下一个最优进程进行调度
            assert(best->state == RUNNABLE);//存在所有进程都不可运行的情况
            //Switch to the best process
            proc = best;
            switchuvm(best);
            best->state = RUNNING;
            swtch(&cpu->scheduler, proc->context);
            swtichkvm();
            //return to scheduler
            round_slice--;
            best->ticks[best->priority]++;
            //如果ticks超过约定上限，best->priority    
            release(&ptable.lock);
        while(round_slice);
        proc = 0;
    }
}
```
3. selectbest
```python
def selectbest(privious, rr_slice):
    best = getbest()
    next = getnext(privious)
    return best or next or privious
```
4. timer
xv6的调度是通过0号进程运行scheduler，每次选择最合适的RUNNABLE进程进行调用（如果没有RUNNABLE进程，则在始终在scheduler循环）;
当找到合适的进程时，切换到此进程的上下文，并将scheduler的上下文保存在cpu-\>scheduler处；
当timer interrupt到来是，current(在xv6里是proc)会调用yield主动让出CPU；
yield函数将current状态设为RUNNABLE，并调用sched函数切换回scheduler函数，重新选择合适的进程。
5. ptable.lock
ptable.lock是维护进程列表一致性的锁。每个进程最开始的eip设置为forkret，forkret第一条指令就是release(&ptable.lock)；
scheduler里每个调度周期里首先会acquire(&ptable.lock)，选择并执行进程时，进程或者处于初始状态，或者处于yield函数中（下一条指令是release(&ptable.lock))，因此ptable.lock是自洽的
## 系统调用
为了证实MLFQ的有效性，为系统添加了两个系统调用，分别可以获取系统进程表信息和主动提升进程优先级。
1. kernel/sysfunc.h: int sys\_boostproc(void)
2. kernel/proc.c: int sys\_boostproc(void) {...}
3. kernel/syscall.c: [SYS\_boostproc] sys\_boostproc,
4. include/syscall.h: #define SYS\_boostproc 23
5. user/usys.S: SYSCALL(boostproc)
6. user/user.h: int boostproc(void); 
    

