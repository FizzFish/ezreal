---
layout: default
---

# Interrupt vs Trap vs Syscall
* interrupt: asynchronous, timer interrupt
* trap: synchronous, divided zero trap
* syscall: special trap, trapnum = 64, syscall num decided by eax

# Global variables
struct gatedesc idt[256];//Interrupt descriptor table (shared by all CPUS)
uint vectors[];
```c
struct gatedesc {
  uint off_15_0 : 16;   // low 16 bits of offset in segment
  uint cs : 16;         // code segment selector
  uint args : 5;        // # args, 0 for interrupt/trap gates
  uint rsv1 : 3;        // reserved(should be zero I guess)
  uint type : 4;        // type(STS_{TG,IG32,TG32})
  uint s : 1;           // must be 0 (system)
  uint dpl : 2;         // descriptor(meaning new) privilege level
  uint p : 1;           // Present
  uint off_31_16 : 16;  // high bits of offset in segment
};

.globl alltraps
.globl vector*x*
vector0:
  pushl $0
  pushl $*x*
  jmp alltraps
```

# Init

```c
void tvinit(void)
{
  for(i = 0; i < 256; i++)
    SETGATE(idt[i], 0, SEG_KCODE<<3, vectors[i], 0);
  SETGATE(idt[T_SYSCALL], 1, SEG_KCODE<<3, vectors[T_SYSCALL], DPL_USER);
}
#define SETGATE(gate, istrap, sel, off, dpl) {...}
void idtinit(void) {
    lidt(idt, sizeof(idt));
}
//lidt将中断向量表的相关信息装载到中断向量寄存器中
void lidt(struct gatedesc *p, int size) {
    ushort pd[3] = {size-1, p, p>>16};
    asm volatitle("lidt (%0)" : : "r" (pd));
}
```

# Handle
```c
void trap(struct trapframe * tf) {
    if(tf->trapno == T_SYSCALL) {//handle syscall
        proc->tf = tf;
        syscall()
        return;
    }
    switch(tf->trapno){
    case T_IRQ0 + IRQ_TIMER:
    if(cpu->id == 0){
      acquire(&tickslock);
      ticks++;
      wakeup(&ticks);
      release(&tickslock);
    }
    lapiceoi();//lapic
    break;
    //handle other interrupt
    //including IRQ_IDE, IRQ_KBD, IRQ_COM1
    //case IRQ_IDE: ideintr()
    //case IRQ_KBD: kbdintr()
    //case IRQ_COM1: uartintr()
    }
    
  if(proc && proc->state == RUNNING && tf->trapno == T_IRQ0+IRQ_TIMER) //时钟中断时，运行的进程会将时间片交给scheduler调度
    yield();
}
```

# Syscall

```c
void syscall(void)
{
  num = proc->tf->eax;
  if(num > 0 && num < NELEM(syscalls) && syscalls[num] != NULL) {
    proc->tf->eax = syscalls[num]();
  } 
}
static int (*syscalls[])(void) = {
[SYS_chdir]   sys_chdir,
[SYS_fork]    sys_fork,
[SYS_open]    sys_open,
[SYS_read]    sys_read,
[SYS_wait]    sys_wait,
...
};
```

        
