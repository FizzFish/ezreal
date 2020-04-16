---
layout: default
---
# linux\_binprm
```c
struct linux_binprm {
	char buf[BINPRM_BUF_SIZE];
#ifdef CONFIG_MMU
	struct vm_area_struct *vma;
	unsigned long vma_pages;
#else
# define MAX_ARG_PAGES	32
	struct page *page[MAX_ARG_PAGES];
#endif
	struct mm_struct *mm;
	unsigned long p; /* current top of mem */
	unsigned int
		cred_prepared:1,/* true if creds already prepared (multiple
				 * preps happen for interpreters) */
		cap_effective:1;/* true if has elevated effective capabilities,
				 * false if not; except for init which inherits
				 * its parent's caps anyway */
	unsigned int recursion_depth;
	struct file * file;
	struct cred *cred;	/* new credentials */
	int unsafe;		/* how unsafe this exec is (mask of LSM_UNSAFE_*) */
	unsigned int per_clear;	/* bits to clear in current->personality */
	int argc, envc;
	const char * filename;	/* Name of binary as seen by procps */
	const char * interp;	/* Name of the binary really executed. Most
				   of the time same as filename, but could be
				   different for binfmt_{misc,script} */
	unsigned interp_flags;
	unsigned interp_data;
	unsigned long loader, exec;
	char tcomm[TASK_COMM_LEN];
};
```

# execve
```c
SYSCALL_DEFINE3(execve, filename, argv, envp)
	do_execve(getname(filename), argv, envp)
	    do_execve_common(filename, argv, envp);

static int do_execve_common(filename, argv, envp)
    bprm = kzalloc(sizeof(*bprm), GFP_KERNEL);
	retval = prepare_bprm_creds(bprm);
	file = do_open_exec(filename);
	sched_exec();
	bprm->file = file;
	bprm->filename = bprm->interp = filename->name;
	retval = bprm_mm_init(bprm);

	bprm->argc = count(argv, MAX_ARG_STRINGS);
	bprm->envc = count(envp, MAX_ARG_STRINGS);
	retval = prepare_binprm(bprm);
	retval = copy_strings_kernel(1, &bprm->filename, bprm);
	bprm->exec = bprm->p;
	retval = copy_strings(bprm->envc, envp, bprm);//将envp和argc拷贝到栈空间
	retval = copy_strings(bprm->argc, argv, bprm);
	retval = search_binary_handler(bprm);//从注册的handler中找到合适的处理函数
```

#load\_elf\_binary

```c
load_elf_binary(struct linux_binprm *bprm)
struct {struct elfhdr elf_ex, interp_elf_ex;} *loc;
loc = kmalloc(...)
loc->elf_ex = *bprm->buf;
//check magic 
//check etype == ET_EXEC | ET_DYN
elf_phdata = kmalloc(sizeof(struct elf_hdr) * e_phnum)
kernel_read(bprm->file, loc->elf_ex.e_phoff, elf_phdata, size)
for elf_ppn in elf_phdata:
    if (elf_ppnt->p_type == PT_INTERP)
        kernel_read(bprm->file, elf_ppnt->p_offset, elf_interpreter) //ls.so
        interpreter = open_exec(elf_interpreter);
        kernel_read(interpreter, 0, bprm->buf) //将interpreter的elfhdr读入buf
        loc->interp_elf_ex = *((struct elfhdr *)bprm->buf);
    if (elf_ppnt->p_type == PT_GNU_STACK)
        executable_stack = elf_ppnt->p_flags & PF_X //检查栈可执行权限
setup_new_exec(bprm);
setup_arg_pages(bprm, randomize_stack_top(STACK_TOP), executable_stack);
current->mm->start_stack = bprm->p;
for elf_ppn in elf_phdata:
    set elf_prot based on elf_ppn->pflags
    vaddr = elf_ppn->p_vaddr
    if loc->elf_ex.e_type == ET_DYN: //DYN需要随机化
        if elf_interpreter:
            load_bias = ELF_ET_DYN_BASE; //TASK_SIZE * 3 /2; TASK_SIZE=PAGE_OFFSET=0xffff8800000...
                if (current->flags & PF_RANDOMIZE)
                    load_bias += arch_mmap_rnd();
        total_size = total_mapping_size(elf_phdata,
                            loc->elf_ex.e_phnum); //CVE-2017-1000253 patch
        elf_map(bprm->file, load_bias + vaddr, elf_ppnt,
                elf_prot, elf_flags, total_size); //将ELF中的segment映射进用户虚拟地址空间
        if !load_addr_set:
            load_addr_set = 1;
            load_addr = (elf_ppnt->p_vaddr - elf_ppnt->p_offset);
            load_addr += load_bias if e_type == ET_DYN
//load_bias相当于映射的整体偏移，取决于ELF_ET_DYN_BASE+arch_mmap_rnd()
loc->elf_ex.e_entry += load_bias;
elf_bss += load_bias;
elf_brk += load_bias;
start_code += load_bias;
end_code += load_bias;
start_data += load_bias;
end_data += load_bias;

set_brk(elf_bss, elf_brk, bss_prot);
install_exec_creds(bprm);
create_elf_tables(bprm, &loc->elf_ex,
              load_addr, interp_load_addr);
current->mm->end_code = end_code;
current->mm->start_code = start_code;
current->mm->start_data = start_data;
current->mm->end_data = end_data;
current->mm->start_stack = bprm->p;
start_thread(regs, elf_entry, bprm->p);
```
