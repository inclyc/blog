---
title: 启动xv6
date: 2022/6/20 04:10:19
categories:
  - OS
  - MIT_6.S081
tags:
  - OS
---
# 启动xv6(easy)

## 前言

我的实验环境是一台PC, 运行在我的寝室，作为工作站运行。

```
[I] lyc@lyc-workstation ~/w/学/C/c/MIT6.S081 (master)> neofetch
         -/oyddmdhs+:.                lyc@lyc-workstation
     -odNMMMMMMMMNNmhy+-`             -------------------
   -yNMMMMMMMMMMMNNNmmdhy+-           OS: Gentoo Base System release 2.8 x86_64
 `omMMMMMMMMMMMMNmdmmmmddhhy/`        Host: MS-7D42 1.0
 omMMMMMMMMMMMNhhyyyohmdddhhhdo`      Kernel: 5.18.5-gentoo-lyc
.ydMMMMMMMMMMdhs++so/smdddhhhhdm+`    Uptime: 22 hours, 29 mins
 oyhdmNMMMMMMMNdyooydmddddhhhhyhNd.   Packages: 1473 (emerge)
  :oyhhdNNMMMMMMMNNNmmdddhhhhhyymMh   Shell: bash 5.1.16
    .:+sydNMMMMMNNNmmmdddhhhhhhmMmy   Resolution: 3840x2160
       /mMMMMMMNNNmmmdddhhhhhmMNhs:   DE: Plasma 5.24.5
    `oNMMMMMMMNNNmmmddddhhdmMNhs+`    WM: KWin
  `sNMMMMMMMMNNNmmmdddddmNMmhs/.      WM Theme: Breeze 微风
 /NMMMMMMMMNNNNmmmdddmNMNdso:`        Theme: Breeze Light [Plasma], Breeze [GTK2/3]
+MMMMMMMNNNNNmmmmdmNMNdso/-           Icons: Fluent [Plasma], Fluent [GTK2/3]
yMMNNNNNNNmmmmmNNMmhs+/-`             Terminal: Konsole
/hMMNNNNNNNNMNdhs++/-`                CPU: 12th Gen Intel i7-12700 (20) @ 4.900GHz
`/ohdmmddhys+++/:.`                   GPU: Intel AlderLake-S GT1
  `-//////:--.                        Memory: 36679MiB / 128619MiB




```


## RISC-V 64运行和交叉编译环境

### cross-compilers

要运行老师给的`xv6`操作系统代码，必须先有交叉编译工具。[[1]](https://pdos.csail.mit.edu/6.828/2021/tools.html)

用gentoo的crossdev工具[[2]](https://wiki.gentoo.org/wiki/Cross_build_environment)安装跨指令集的编译工具，包括`GNU binutils`, `gcc`, `g++`。

```bash
sudo crossdev --stable -t riscv64-unknown-linux-gnu
```

### Emulator(QEMU)

安装打开了riscv64对应的编译开关的`qemu`，我的实验主机是`Gentoo Linux`操作系统，在`Portage`中加入`USE`

```
write-use app-emulation qemu qemu_user_targets_riscv64 qemu_softmmu_targets_riscv64
```

其中`write-use`是一个自己写的辅助脚本

```bash
lyc@lyc-workstation ~> cat $(which write-use)
#!/bin/fish

set portage_use_dir "/etc/portage/package.use"
echo "$argv[1]/$argv[2] $argv[3..]" | sudo tee -a "$portage_use_dir/$argv[1]"
```

然后，重新编译qemu

```bash
sudo emerge -av qemu
```


## 获取实验源代码

### clone mit xv6仓库

```bash
git clone git://g.csail.mit.edu/xv6-labs-2021
```


### 选到util分支

```bash
cd xv6-labs-2021 && git checkout util
```




## 编译和进入xv6系统


```
[I] lyc@lyc-workstation ~/w/学/C/c/M/xv6-labs-2021 (util)> make qemu
qemu-system-riscv64 -machine virt -bios none -kernel kernel/kernel -m 128M -smp 3 -nographic -drive file=fs.img,if=none,format=raw,id=x0 -device virtio-blk-device,drive=x0,bus=virtio-mmio-bus.0

xv6 kernel is booting

hart 1 starting
hart 2 starting
init: starting sh
$

```



`Ctrl-a x`退出系统。