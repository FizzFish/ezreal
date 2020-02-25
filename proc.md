---
layout: default
---

# Proc文件系统
* proc 文件系统(proc filesystem),它使得内核可以生成与系统的状态和配置有关的信息。
* 一种虚拟文件系统，信息不从块设备中读取，只有在读取文件时才动态生成内容信息。
* proc文件系统提供sysctl机制导出的所有选项。
* 包括：
    - 特定进程数据
        - cmdline
        - maps
        - status
    - 一般性系统信息
        - /proc/iomem 设备内存信息
        - /proc/ioports 设备端口信息
        - /proc/buddyinfo 伙伴系统信息
        - /proc/slabinfo slab信息
        - /proc/meminfo 系统内存使用情况
        - /proc/vmstat 内存管理信息
        - /proc/kallsyms 符号表
        - /proc/kcore 动态内核
        - /proc/interrupts 内核中断信息
    - /proc/net 网络信息
    - /proc/sys 系统行为控制，与sysctl系统调用等价

# 数据结构
proc作为一种文件系统，集成在内核的VFS抽象层中。
```c
struct proc_dir_entry {
	const struct inode_operations *proc_iops;//虛擬文件系統接口
	const struct file_operations *proc_fops;
	struct proc_dir_entry *parent;
	char name[];
}
```
proc\_inode可以理解為inode的字类。
```c
struct proc_inode {
	struct pid *pid;
	int fd;
	union proc_op op;
	struct proc_dir_entry *pde;
	struct ctl_table_header *sysctl;
	struct ctl_table *sysctl_entry;
	const struct proc_ns_operations *ns_ops;
	struct inode vfs_inode;
};
```

# 初始化
proc\_root\_init

# 装载proc文件系统

# 读写proc文件数据流

# proc文件系统处理进程信息

# sysctl控制机制    
