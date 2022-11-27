---
title: xv6-util-pingpong
date: 2022/6/20 17:40:19
categories:
  - OS
  - MIT_6.S081
tags:
  - OS
---
# Pingpong

继续Lab1的实验，这个程序要求我们用一个管道实现父子进程通信，即开一个pipe，然后父进程发一个ping，子进程表示自己收到了ping，然后再翻过来。

```C
#include "user.h"

int main(int argc, char *argv[]) {
  int pipes[2];
  int n = pipe(pipes);
  if (n < 0) {
    printf("pipe error\n");
    exit(1);
  }
  int pid = fork();
  if (pid < 0) {
    printf("fork error\n");
    exit(1);
  }
  if (pid == 0) {
    char rd;
    read(pipes[0], &rd, sizeof(char));
    printf("%d: received ping\n", getpid());
    write(pipes[1], &rd, sizeof(char));
  } else {
    // parent process
    char wr = 'p';
    write(pipes[1], &wr, sizeof(char));
    read(pipes[0], &wr, sizeof(char));
    printf("%d: received pong\n", getpid());
  }
  exit(0);
}
```