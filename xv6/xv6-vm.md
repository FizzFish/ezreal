---
layout: default
---

# xv6内存管理
- 全局数据结构 kmem, kmem是一个链表，kmem.freelist保存当前空闲页面地址
- 用户态
    - 用户态malloc可以从缓存中申请连续内存块
    - 当缓存不够时，通过brk系统调用增加缓存大小
        - sys_sbrk->growproc(n)->allocuvm->kmalloc for i in range(n/PGSIZE)
            - mappages(pgdir, la, PGSIZE, pa, perm)
- 核心态
    - kalloc 直接从kmem-\>freelist中获取一个页面
    - kvmalloc 在mian.c中初始化内核页表
        ```c
        static struct kmap {
          void *p;
          void *e;
          int perm;
        } kmap[] = {
          {(void*)USERTOP,    (void*)0x100000, PTE_W},  // I/O space
          {(void*)0x100000,   data,            0    },  // kernel text, rodata
          {data,              (void*)PHYSTOP,  PTE_W},  // kernel data, memory
          {(void*)0xFE000000, 0,               PTE_W},  // device mappings
        };
        ```    

