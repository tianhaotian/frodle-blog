---
layout: 'title:'
title: gdb调试运行中进程
author: Frodle Tim
date: 2019-03-02 16:57:24
tags:
---


> 安装Python调试环境: https://wiki.python.org/moin/DebuggingWithGdb

## 常用GDB命令
 
 1. gdb原生命令
    1. `bt` 显示当前线程C部分调用栈
    2. `list` 示当前线程的C代码执行位置
    3. `info threads` 显示当前进程中的所有线程
    4. `thread apply {ID} {command}` 在线程{ID}上执行命令, 默认命令在1号线程执行
    5. `print` 打印C变量值
    6. `up` 返回C部分调用者
    7. `down` 进入C部分函数调用
 2. Python扩展命令 
    1. `py-bt` 显示当前线程Python部分调用栈
    2. `py-list` 显示当前线程的Python代码执行位置
    3. `py-locals` 获取当前局部变量的值
    4. `py-print` 打印某个变量的值
    5. `py-up` 返回Python部分调用者
    6. `py-down` 进入Python部分函数调用

## 常用Linux命令

1. `strace -p {PID}` 跟踪指定进程的系统调用情况
2. `lslocks` 列出当前系统中的文件锁情况
3. `fuser {file}` 列出打开该文件的进程ID
