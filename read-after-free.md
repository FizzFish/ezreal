---
layout: default
---

# CVE-2020-8428
漏洞点在may_create_in_sticky函数中对于参数dir的解引用，指针dir可能已经不存在了。
```c
static int may_create_in_sticky(struct dentry * const dir,
				struct inode * const inode)
    dereference(dir->d_inode->i_mode);
    //dereference may dangerous, because the dir reference maybe dropped.
    ...
}
```


