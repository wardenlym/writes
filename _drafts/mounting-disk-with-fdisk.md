---
title: "使用fdisk挂载磁盘"
date: 2021-03-29T16:18:52+08:00
draft: true
---

测试用的机器上有一块磁盘需要手动挂载。

通过查看：

fdisk -l

ll /dev/disk/by-path

确定需要挂载的应该叫/dev/sda，执行

fdisk -l /dev/sda

我想把他挂到/data上

mount /dev/sda /data

查看 df -h 
完毕。
