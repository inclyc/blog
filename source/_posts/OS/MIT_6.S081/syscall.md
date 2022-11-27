---
title: xv6-syscall
date: 2022/6/21 17:47:19
categories:
  - OS
  - MIT_6.S081
tags:
  - OS
---

# syscall

## 系统调用简介

系统调用`syscall`可以看作一次特殊的`函数调用`。程序按照`Calling Convention`将参数放在寄存器、堆栈中，然后CPU硬件修改程序计数器，权限状态等信息，跳转到内核提前设定好的一个位置（`stvec`）。

之后，处理器开始执行内核代码。内核代码执行完后，恢复发生中断时的现场，继续运行程序。

## syscall lab 概述

这个[lab](https://pdos.csail.mit.edu/6.828/2021/labs/syscall.html)一共有两个实验，分别是`trace`和`sysinfo`，前者要求我们在之后系统调用时，打印系统调用的情况，后者需要我们提供内存使用和进程的使用信息。

我的所有实验代码都可以在[github](https://github.com/inclyc/xv6-labs-2021/tree/syscall)找到。

### trace

这个实验重点是对于每个`proc`结构体保存一个flag，表示他要不要进行trace。

然后，在系统调用返回时，设计成可以打印结果:

```C
  num = p->trapframe->a7;
  if(num > 0 && num < NELEM(syscalls) && syscalls[num]) {
    int ret = syscalls[num]();
    p->trapframe->a0 = ret;
    if (p->tsysflag & (1 << num)) printf("%d: %s -> %d\n", p->pid, syscallnames[num], ret);
  } else {
    printf("%d %s: unknown sys call %d\n",
            p->pid, p->name, num);
```

`tsysflag`是需要新写到结构体里的一个成员。

`syscallnames`，每个系统调用的字符串名字。

```C
static char *syscallnames[] = {
[SYS_fork]    "fork",
[SYS_exit]    "exit",
[SYS_wait]    "wait",
[SYS_pipe]    "pipe",
[SYS_read]    "read",
[SYS_kill]    "kill",
[SYS_exec]    "exec",
[SYS_fstat]   "fstat",
[SYS_chdir]   "chdir",
[SYS_dup]     "dup",
[SYS_getpid]  "getpid",
[SYS_sbrk]    "sbrk",
[SYS_sleep]   "sleep",
[SYS_uptime]  "uptime",
[SYS_open]    "open",
[SYS_write]   "write",
[SYS_mknod]   "mknod",
[SYS_unlink]  "unlink",
[SYS_link]    "link",
[SYS_mkdir]   "mkdir",
[SYS_close]   "close",
[SYS_trace]   "trace",
};
```

`trace`实验的所有代码参考[这里](https://github.com/inclyc/xv6-labs-2021/commit/082a7358cb69e5c0cef12117142c37e78a54d4b6?diff=split`)。


### sysinfo

主要是实现两个模块，进程和内存。

内存方面，内核用一个`kfreelist`来保存现在空闲的内存信息，这是一个链表，统计时只需要遍历链表即可。内存都是按照页分配的，所以结果就是`PGSIZE * free_num`。

```C
// kernel/kalloc.c
int kfreepages_count(){
  int n = 0;
  struct run *current = kmem.freelist;

  for(;current;current = current->next) ++n;
  return n;
}
```


进程计数，直接遍历进程数组，看他们中哪些不是`UNUSED`。

```C
// kernel/proc.c
int nproc(){
  struct proc *p;
  int ret = 0;
  for(p = proc;p < proc + NPROC; p++){
    acquire(&p->lock);
    if(p->state != UNUSED) ret++;
    release(&p->lock);
  }
  return ret;
}
```

最后，整理出系统调用的结果，把他们拷贝到用户态去。

```C
// kernel/sysinfo.c
uint64 sys_sysinfo(void)
{
  uint64 va_info;
  if(argaddr(0, &va_info)){
    return -1;
  }
  struct sysinfo info;
  struct proc *p = myproc();
  info.freemem = kfreepages_count() * PGSIZE;
  info.nproc = nproc();
  if(copyout(p->pagetable, va_info, (void*)&info, sizeof(info))){
    return -1;
  }
  return 0;
}
``` 