---
layout: default
---

## dlopen
dlopen()函数用来打开一个动态库，并将其加载到进程地址空间中。
```c
dlopen(file, mode) => __dlopen(file, mode) =>
    _dlfcn_hook->dlopen (file, mode, DL_CALLER);

_dl_start (void *arg) => _dl_start_final (arg) => dl_main => dlmopen_doit (void *a)
_dl_open(file, mode, *caller_dlopen, nsid, argc, *argv[], *env[])
=> dl_open_worker(struct dl_open_args *args)
  call_map = GL(dl_ns)[LM_ID_BASE]._ns_loaded;
  /* Load the named object.  */
  args->map = new = _dl_map_object (call_map, file, lt_loaded, 0,
				    mode | __RTLD_CALLMAP, args->nsid);
=> _dl_map_object
    /* Look for this name among those already loaded.  */
    for (l = GL(dl_ns)[nsid]._ns_loaded; l; l = l->l_next)
    {
	  if (strcmp (name, soname) != 0)
        return l;
    }
    fd = open_path (name, namelen, mode & __RTLD_SECURE,
            &main_map->l_rpath_dirs,
            &realname, &fb, loader ?: main_map, LA_SER_RUNPATH,
            &found_other_class);

# dl_debug
```c
dl_main(*phdr, phnum, *user_entry, *auxv) => process_envvars(&mode)

  while ((envline = _dl_next_ld_env_entry (&runp)) != NULL)//DL_(envline)
      case DEBUG:
	      process_dl_debug (&envline[6]);
```

## process\_dl\_debug
```c
process_dl_debug(*dl_debug)
    while *dl_debug != '\0':
        len = searchNext[' ',',',':']
        if dl_debug[:len] not in [libs, reloc, symbols, versions, scopes, all ...]:
            //这里可能会分配很多次空间，如果DL_DEBUG=AAAA BBBB CCCCC
            char *copy = strndupa(dl_debug, len)
            printf("warning: debug option `%s' unknown; try LD_DEBUG=help\n", copy);
```

## elf\_get\_dynamic\_info
```c
elf_get_dynamic_info (struct link_map *l, ElfW(Dyn) *temp)
    ElfW(Dyn) *dyn = l->l_ld;
    ElfW(Dyn) **info;
    info = l->l_info;
    while (dyn->d_tag != DT_NULL)
        info[dyn->d_tag] = dyn;
        dyn++;
    if (l->l_addr != 0):
        #define ADJUST_DYN_INFO(tag) info[tag]->d_un.d_ptr += l_addr;
        ADJUST_DYN_INFO (DT_HASH);
        ADJUST_DYN_INFO (DT_PLTGOT);
        ADJUST_DYN_INFO (DT_STRTAB);
        ADJUST_DYN_INFO (DT_SYMTAB);
```

## \_dl\_map\_object\_from\_fd
```c
_dl_map_object_from_fd (const char *name, int fd, struct filebuf *fbp,
			char *realname, struct link_map *loader, int l_type,
			int mode, void **stack_endp, Lmid_t nsid)
    l = _dl_new_object (realname, name, l_type, loader, mode, nsid);
    l->l_entry = header->e_entry;
    type = header->e_type;
    l->l_phnum = header->e_phnum;

    for (ph = phdr; ph < &phdr[l->l_phnum]; ++ph)
        switch (ph->p_type)
        case PT_DYNAMIC:
            l->l_ld = (void *) ph->p_vaddr;
            l->l_ldnum = ph->p_memsz / sizeof (ElfW(Dyn));
            break;
        case PT_PHDR:
            l->l_phdr = (void *) ph->p_vaddr;
            break;
        case PT_LOAD:
            c = &loadcmds[nloadcmds++];
            c->mapstart = ph->p_vaddr & ~(GLRO(dl_pagesize) - 1);
            c->mapend = ((ph->p_vaddr + ph->p_filesz + GLRO(dl_pagesize) - 1)
            & ~(GLRO(dl_pagesize) - 1));
            c->dataend = ph->p_vaddr + ph->p_filesz;
            c->allocend = ph->p_vaddr + ph->p_memsz;
            c->mapoff = ph->p_offset & ~(GLRO(dl_pagesize) - 1);
            //set c->prot
        case PT_GNU_STACK:
            stack_flags = ph->p_flags;
            break;
    l->l_ld = (ElfW(Dyn) *) ((ElfW(Addr)) l->l_ld + l->l_addr);
    elf_get_dynamic_info (l, NULL);
```
