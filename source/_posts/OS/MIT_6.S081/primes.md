---
title: xv6-util-primes
date: 2022/6/20 17:47:19
categories:
  - OS
  - MIT_6.S081
tags:
  - OS
---
# Primes

这个实验要求设计一个素数发生器。[基于管道实现埃氏筛](https://swtch.com/~rsc/thread/)。

```C
#include "user.h"
void produce(int backward_pipe_fd) {
  int prime;
  read(backward_pipe_fd, &prime, sizeof(int));
  printf("prime %d\n", prime);
  if (prime >= 31) {
    return;
  }
  int pp[2];
  pipe(pp);
  int pid = fork();
  if (pid == 0) {
    // child
    close(pp[1]);
    produce(pp[0]);
    close(pp[0]);
  } else {
    int n;
    close(pp[0]); //  close a ----- b ----read--- c
    for (; read(backward_pipe_fd, &n, sizeof(int));) {
      if (n % prime)
        write(pp[1], &n, sizeof(int));
    }
    close(pp[1]);            // close a ---------- b ---write-- c
    close(backward_pipe_fd); // close a ---read--- b ---------- c
    wait(0);
  }
  exit(0);
}

int main() {
  printf("prime 2\n");
  int pp[2];
  pipe(pp);
  int pid = fork();
  if (pid == 0) {
    // child
    close(pp[1]);
    produce(pp[0]);
  } else {
    close(pp[0]);
    for (int i = 3; i < 32; i += 2) {   // filter 2-factors
      write(pp[1], &i, sizeof(int));
    }
    close(pp[1]);
    wait(0);
  }
  exit(0);
}
```

[最开始的实验版本](https://github.com/inclyc/xv6-labs-2021/commit/96d992511da523395b7984a78709315bb66f348a)我在`fork`之后才调用`pipe`，导致出现了逻辑上的问题。这个程序中如果不及时关闭pipe中不需要的那个`fd`，会导致阻塞/资源用尽。