---
title: 非对齐访存与自动向量化 - 我们应该注意什么?
subtitle: Unaligned Memory Access & Auto Vectorization
categories:
  - Compiler
tags:
  - Compiler
mathjax: true
date: 2024-01-21
---

## 前言

本文旨在介绍自动向量化与非对齐内存访问之间的关系，以及一些编译器上的解决方案。
笔者有幸从事某国产体系结构上的自动向量化编译器项目 (clang/llvm)，在 SPEC2017[^spec] 上遇到了不少类似问题，在此也回顾一下中间的技术路线与决策，供读者参考。

[^spec]: [SPEC2017](https://www.spec.org/cpu2017/) 是一个标准化的测试集合，用来衡量计算密集的、CPU 上的程序性能。

## 非对齐内存访问

