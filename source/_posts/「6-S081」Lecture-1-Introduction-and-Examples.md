---
title: 「6.S081」Lecture 1 - Introduction and Examples
date: 2022-04-20 22:20:24
tags:
  - 6.S081
  - 操作系统
categories:
  - 操作系统
  - 6.S081
---

## 课程目标

1. 理解操作系统的设计和实现。
2. 通过一个小型操作系统获得实践经验。

## OS purpose

+ 抽象硬件（Abstract H/W）
+ 多路复用（Multiplex）
+ 隔离（Isolation）
+ 共享（Sharing）
+ 权限系统（Security）
+ 性能（Performance）
+ 用途广泛（Range of Uses）

## 设计思想

User space -> Kernal -> H/W

## 系统调用（System Call）

### 系统调用（System Call）和函数调用（Function Call）有什么区别？

内核拥有特权，可以访问硬件资源；而用户不允许直接访问硬件，只能通过系统调用。

## Why Hard/Interesting？（指构建OS）

+ 困难的编程环境
+ 解决矛盾（Tensions）
  + 效率（Efficient） - 抽象（Abstract）
  + 强大（Powerful）- 简单（Simple）
  + 灵活（Flexible）  - 安全（Secure）
+ 内部实现交互错综复杂

## 课程结构

### 分数

70% - 实验

20% - 实验会议

10% - 作业

## 课程环境

xv6 ----> run on RISC-V ----> QEMU Simulator
