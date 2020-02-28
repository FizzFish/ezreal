# 万物皆文件
* ext4, FAT, proc, net
* 块设备、字符设备、管道、套接字、终端
* VFS虚拟文件系统是一个通用模型，但它并不是所有文件的基类，相反可以看作是所有文件的超集，也即对于某一种具体文件系统，VFS的每个成员并不是都有具体含义。

# inode
* inode是OS对于“文件”的视图。每个文件/目录对应一个独一无二的inode结构体，包含此文件的文件状态和文件内容地址。
* 对于层级的文件目录，inode也表现的像一颗树。
* 软链接vs硬链接：软链接中每个符号都使用了一个inode，而硬链接中两个符号共享同一个inode。
* inode对应一个计数器，当计数器为0时，才可以完全删除。

# VFS结构
![Octocat](./vfs.png)

```c
struct inode {
    //权限
	umode_t			i_mode;
	unsigned short		i_opflags;
	kuid_t			i_uid;
	kgid_t			i_gid;
	unsigned int		i_flags;
    //inode操作
	const struct inode_operations	*i_op;
    //super block
	struct super_block	*i_sb;
    //address_space,保存inode<=>file对应关系
	struct address_space	*i_mapping;
    //i_lru将同种状态的inode保存在一个链表上
    //基本有未被使用、使用、脏三种状态
	struct list_head	i_lru;		/* inode LRU list */
    //super block也保存这inode的链表
	struct list_head	i_sb_list;
	const struct file_operations	*i_fop;	/* former ->i_op->default_file_ops */
	struct file_lock_context	*i_flctx;
	struct address_space	i_data;
	struct list_head	i_devices;
	union {
		struct pipe_inode_info	*i_pipe;
		struct block_device	*i_bdev;
		struct cdev		*i_cdev;
		char			*i_link;
	};
	void			*i_private; /* fs or device private pointer */
};
```

