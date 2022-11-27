---
title: 延迟加载conda环境
categories:
  - Misc
tags:
  - Misc
  - Zsh
date: 2022-10-06 11:00:19
---


# 前言

每次开机或重启后，新开一个终端，总会有漫长的conda加载时间。

`conda init` 会在你的shell中写入 `conda initialize` 相关的内容，保证 conda 环境被初始化。

```bash
# >>> conda initialize >>>
__conda_setup="$("$__conda_prefix/bin/conda" 'shell.bash' 'hook' 2> /dev/null)"
if [ $? -eq 0 ]; then
    eval "$__conda_setup"
else
    if [ -f "$__conda_prefix/etc/profile.d/conda.sh" ]; then
        . "$__conda_prefix/etc/profile.d/conda.sh"
    else
        export PATH="$__conda_prefix/bin:$PATH"
    fi
fi
unset __conda_setup
# <<< conda initialize <<<
```

然而，不知道为什么，这个东西会严重拖慢第一次打开终端时进入正常交互式shell的速度。

# 解决方案

在这里我的解决方案是，添加一个延迟加载选项。我的日常环境中不是所有的终端都会用上conda，甚至大概率不用。所以按需要加载环境就非常重要。

或许可以设计一个`load_conda`的函数，然后需要用conda的时候加载它！

然而，这样好像太蠢了！有没有一个办法能够在我输入conda指令时，虽然引发了一个错误但自动加载conda环境，然后执行加载后的conda? （类似于虚拟内存与缺页中断）

我把激活conda相关的变量改成了这样：

```bash
# https://www.reddit.com/r/zsh/comments/qmd25q/lazy_loading_conda/
# Add any commands which depend on conda here
lazy_conda_aliases=('python' 'conda')

load_conda() {
  for lazy_conda_alias in $lazy_conda_aliases; do
    unalias $lazy_conda_alias
  done

  __conda_prefix="$HOME/anaconda3" # Set your conda Location

  # >>> conda initialize >>>
  __conda_setup="$("$__conda_prefix/bin/conda" 'shell.bash' 'hook' 2> /dev/null)"
  if [ $? -eq 0 ]; then
      eval "$__conda_setup"
  else
      if [ -f "$__conda_prefix/etc/profile.d/conda.sh" ]; then
          . "$__conda_prefix/etc/profile.d/conda.sh"
      else
          export PATH="$__conda_prefix/bin:$PATH"
      fi
  fi
  unset __conda_setup
  # <<< conda initialize <<<

  unset __conda_prefix
  unfunction load_conda
}

for lazy_conda_alias in $lazy_conda_aliases; do
  alias $lazy_conda_alias="load_conda && $lazy_conda_alias"
done
```

这个方案实在是太巧了！很符合我对未来的想象（

当你在shell输入conda时，没有加载conda时会触发load_conda，然后自动加载conda。之后又会被`alias`到conda指令本身，看起来就像无缝加载了一个变量一样！
