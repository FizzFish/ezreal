---
layout: default
---

## LD
```c
static void
dl_main (const ElfW(Phdr) *phdr,
	 ElfW(Word) phnum,
	 ElfW(Addr) *user_entry,
	 ElfW(auxv_t) *auxv)
      security_init ();
}
static void
security_init (void)
{
  /* Set up the stack checker's canary.  */
  uintptr_t stack_chk_guard = _dl_setup_stack_chk_guard (_dl_random);
#ifdef THREAD_SET_STACK_GUARD
  THREAD_SET_STACK_GUARD (stack_chk_guard);
#else
  __stack_chk_guard = stack_chk_guard;
#endif

# define THREAD_SET_STACK_GUARD(value) \
    THREAD_SETMEM (THREAD_SELF, header.stack_guard, value)

# define THREAD_SETMEM(descr, member, value) \
  ({ if (sizeof (descr->member) == 1)					      \
       asm volatile ("movb %b0,%%fs:%P1" :				      \
		     : "iq" (value),					      \
		       "i" (offsetof (struct pthread, member)));	      \
     else if (sizeof (descr->member) == 4)				      \
       asm volatile ("movl %0,%%fs:%P1" :				      \
		     : IMM_MODE (value),				      \
		       "i" (offsetof (struct pthread, member)));	      \

typedef struct
{
  void *tcb;		/* Pointer to the TCB.  Not necessarily the
			   thread descriptor used by libpthread.  */
  dtv_t *dtv;
  void *self;		/* Pointer to the thread descriptor.  */
  int multiple_threads;
  int gscope_flag;
  uintptr_t sysinfo;
  uintptr_t stack_guard;
  uintptr_t pointer_guard;
  unsigned long int vgetcpu_cache[2];
  /* Bit 0: X86_FEATURE_1_IBT.
     Bit 1: X86_FEATURE_1_SHSTK.
   */
  unsigned int feature_1;
  int __glibc_unused1;
  /* Reservation of some values for the TM ABI.  */
  void *__private_tm[4];
  /* GCC split stack support.  */
  void *__private_ss;
  /* The lowest address of shadow stack,  */
  unsigned long long int ssp_base;
  /* Must be kept even if it is no longer used by glibc since programs,
     like AddressSanitizer, depend on the size of tcbhead_t.  */
  __128bits __glibc_unused2[8][4] __attribute__ ((aligned (32)));

  void *__padding[8];
} tcbhead_t;
```
