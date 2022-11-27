---
title: xv6-util-sleep
date: 2022/6/20 04:10:19
categories:
  - OS
  - MIT_6.S081
tags:
  - OS
---
# sleep

实现一个UNIX`sleep`程序，要求使用`sleep`系统调用。

* [xv6 book](https://pdos.csail.mit.edu/6.828/2021/xv6/book-riscv-rev2.pdf) 第一章
* `user/`下有不少可以参考的程序
* 如果用户没有参数，则打印错误信息
* 用系统调用
* 系统调用源代码在`kernel/sysproc.c`中可见
* `main`函数应该使用`exit(0)`来正常推出
* 将程序写入到`Makefile`的UPROGS中。


## 构建系统添加sleep

```Makefile
UPROGS=\
	$U/_cat\
	$U/_echo\
	$U/_forktest\
	$U/_grep\
	$U/_init\
	$U/_kill\
	$U/_ln\
	$U/_ls\
	$U/_mkdir\
	$U/_rm\
	$U/_sh\
	$U/_stressfs\
	$U/_usertests\
	$U/_grind\
	$U/_wc\
	$U/_zombie\
	$U/_sleep\
```


## sleep syscall的实现

```C
// kernel/sysproc.c

uint64
sys_sleep(void)
{
  int n;
  uint ticks0;

  if(argint(0, &n) < 0)
    return -1;
  acquire(&tickslock);
  ticks0 = ticks;
  while(ticks - ticks0 < n){
    if(myproc()->killed){
      release(&tickslock);
      return -1;
    }
    sleep(&ticks, &tickslock);
  }
  release(&tickslock);
  return 0;
}
```

其中，这里的`sleep`函数是一个在`proc.c`中实现的函数

```C
// kernel/proc.c

// Atomically release lock and sleep on chan.
// Reacquires lock when awakened.
void
sleep(void *chan, struct spinlock *lk)
{
  struct proc *p = myproc();

  // Must acquire p->lock in order to
  // change p->state and then call sched.
  // Once we hold p->lock, we can be
  // guaranteed that we won't miss any wakeup
  // (wakeup locks p->lock),
  // so it's okay to release lk.

  acquire(&p->lock);  //DOC: sleeplock1
  release(lk);

  // Go to sleep.
  p->chan = chan;
  p->state = SLEEPING;

  sched();

  // Tidy up.
  p->chan = 0;

  // Reacquire original lock.
  release(&p->lock);
  acquire(lk);
}

```

调度器、状态转移，这些概念应该在之后的课程中会涉及到。


## 自己实现的sleep.c

```C
// user/sleep.c
#include "user.h"

int main(int argc, char *argv[]) {
  if (argc != 2) {
    printf("Usage: sleep <seconds>\n");
    exit(-1);
  }
  int ticks = atoi(argv[1]);
  sleep(ticks);
  exit(0);
}
```

其中，他这里的`uint`还会出现一些编译错误

```
In file included from user/sleep.c:1:
user/user.h:36:1: 错误：unknown type name ‘uint’; did you mean ‘int’?
   36 | uint strlen(const char*);
      | ^~~~
      | int
user/user.h:37:26: 错误：unknown type name ‘uint’; did you mean ‘int’?
   37 | void* memset(void*, int, uint);
      |                          ^~~~
      |                          int
user/user.h:38:1: 错误：函数声明中出现形参名却未指定类型 [-Werror]
   38 | void* malloc(uint);
      | ^~~~
user/user.h:41:40: 错误：unknown type name ‘uint’; did you mean ‘int’?
   41 | int memcmp(const void *, const void *, uint);
      |                                        ^~~~
      |                                        int
user/user.h:42:36: 错误：unknown type name ‘uint’; did you mean ‘int’?
   42 | void *memcpy(void *, const void *, uint);
      |                                    ^~~~
      |                                    int
cc1：所有的警告都被当作是错误
```


我在`user/user.h`中加了一个类型定义，来解决这个编译错误


然后再次`make qemu`就可以了。