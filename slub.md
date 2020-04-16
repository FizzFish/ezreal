---
layout: default
---

## OO
caculate_sizes(kmem_cache* s, int forced_order)
    order = caculate_order(size, s->reserved)
        order = slab_order
        //we try to keep the page order as low as possible
        //break if (rem <= slab_size / fract_leftover)

## sock->ioctl

SYS_ioctl(fp, cmd,arg)
    do_vfs_ioctl(fp, cmd, arg)
        vfs_ioctl(fp, cmd, arg)
            filp->f_op->unlocked_ioctl(filp, cmd, arg); => sock_ioctl
                sock_do_ioctl
                    inet6_ioctl
struct proto_ops inet_stream_ops {
    .ioctl = inet_ioctl;
}
static struct inet_protosw inetsw_array[] =
{
    {
        .type =       SOCK_STREAM,
        .protocol =   IPPROTO_TCP,
        .prot =       &tcp_prot,
        .ops =        &inet_stream_ops,
    },
}

const struct file_operations socket_file_ops = {
    .unlocked_ioctl = sock_ioctl;
}


#0  inet6_ioctl (sock=0xffff88003cd51b80, cmd=21531, arg=0) at net/ipv6/af_inet6.c:489
#1  0xffffffff81605480 in sock_do_ioctl (net=net@entry=0xffffffff81cfe180 <init_net>, sock=<optimized out>, cmd=cmd@entry=21531, arg=0) at net/socket.c:1060
#2  0xffffffff81605b85 in sock_ioctl (file=<optimized out>, cmd=21531, arg=<optimized out>) at net/socket.c:1145
#3  0xffffffff81232b1c in vfs_ioctl (arg=0, cmd=1025595392, filp=0xffff88003d215800) at fs/ioctl.c:43
#4  do_vfs_ioctl (filp=filp@entry=0xffff88003d215800, fd=fd@entry=60, cmd=cmd@entry=21531, arg=arg@entry=0) at fs/ioctl.c:631
#5  0xffffffff81232e10 in SYSC_ioctl (arg=0, cmd=21531, fd=60) at fs/ioctl.c:646
#6  SyS_ioctl (fd=60, cmd=21531, arg=0) at fs/ioctl.c:637


[   50.438556] BUG: unable to handle kernel paging request at 0000559be7d7f8f8
[   50.442367] IP: [<0000559be7d7f8f8>] 0x559be7d7f8f8
[   50.443059] PGD 800000003be4d067 PUD 3be02067 PMD 3be05067 PTE 3c61a025
[   50.444089] Oops: 0011 [#1] SMP 
[   50.444590] Modules linked in:
[   50.445021] CPU: 0 PID: 73 Comm: exploit Not tainted 3.10.0+ #4
[   50.445848] Hardware name: QEMU Standard PC (i440FX + PIIX, 1996), BIOS rel-1.12.1-0-ga5cab58e9a3f-prebuilt.qemu.org 04/01/2014
[   50.447562] task: ffff88003bd31fa0 ti: ffff88003be34000 task.ti: ffff88003be34000
[   50.448610] RIP: 0010:[<0000559be7d7f8f8>]  [<0000559be7d7f8f8>] 0x559be7d7f8f8
[   50.449655] RSP: 0018:ffff88003be37e28  EFLAGS: 00010302
[   50.450379] RAX: 0000559be7d7f8f8 RBX: 000000000000541b RCX: 00007f33788c6000
[   50.451345] RDX: 0000000000000000 RSI: 000000000000541b RDI: 00007f33788c6000
[   50.452324] RBP: ffff88003be37e30 R08: 0000000000000000 R09: 0000000000000000
[   50.453298] R10: 0000000000000000 R11: 0000000000000010 R12: 0000000000000000
[   50.454274] R13: ffffffff81cfe180 R14: 0000000000000001 R15: ffff88003d20d900
[   50.455261] FS:  00007f33788c7740(0000) GS:ffff88003f600000(0000) knlGS:0000000000000000
[   50.456393] CS:  0010 DS: 0000 ES: 0000 CR0: 000000008005003b
[   50.457185] CR2: 0000559be7d7f8f8 CR3: 000000003be5e000 CR4: 00000000000006f0
[   50.458177] DR0: 0000000000000000 DR1: 0000000000000000 DR2: 0000000000000000
[   50.459164] DR3: 0000000000000000 DR6: 00000000fffe0ff0 DR7: 0000000000000400
[   50.460125] Call Trace:
[   50.460474]  [<ffffffff816e1342>] ? inet6_ioctl+0x92/0xa0
[   50.461385]  [<ffffffff81605480>] sock_do_ioctl+0x20/0x50
[   50.462258]  [<ffffffff81605b85>] sock_ioctl+0x1d5/0x2a0
[   50.462984]  [<ffffffff81232a4c>] do_vfs_ioctl+0x33c/0x590
[   50.463758]  [<ffffffff812d631f>] ? file_has_perm+0x6f/0xc0
[   50.464573]  [<ffffffff81232d40>] SyS_ioctl+0xa0/0xc0
[   50.465286]  [<ffffffff81753d15>] ? system_call_after_swapgs+0xa2/0x146
[   50.466203]  [<ffffffff81753dcf>] system_call_fastpath+0x16/0x1b
[   50.467021]  [<ffffffff81753d21>] ? system_call_after_swapgs+0xae/0x146

error_code 17: PF_PROT | PF_INSTR
