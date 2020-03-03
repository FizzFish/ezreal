## register_filesystem
注册文件系统
```c
static struct file_system_type *file_systems;//全局file_systems链表指针
int register_filesystem(struct file_system_type * fs)
{
    struct file_system_type ** p = find_filesystem(fs->name, strlen(fs->name));
    //strncmp((*p)->name, name,len) for p in &file_systems
    *p = fs;
}

//fs/ext4/super.c:
static struct file_system_type ext4_fs_type = {
	.owner		= THIS_MODULE,
	.name		= "ext4",
	.mount		= ext4_mount,
	.kill_sb	= kill_block_super,
	.fs_flags	= FS_REQUIRES_DEV,
};
//moudle_init(ext4_init_fs)=>
static int __init ext4_init_fs(void)
{
	err = register_filesystem(&ext4_fs_type);
}
```
