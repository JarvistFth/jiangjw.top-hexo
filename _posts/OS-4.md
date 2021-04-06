---
title: 操作系统 - 文件系统
date: 2021-02-27 18:44:28
categories: 
- OS
- 文件系统
tags: [OS,文件系统]
keywords: [OS,文件系统]
---


linux文件系统总结
<!---more--->

## 文件系统

VFS，虚拟文件系统
对不同持久存储系统的抽象；通过抽象文件系统，提供共同接口给上层访问和使用。

用户空间write() -> 系统调用 VFS:sys_write() -> 文件系统的写方法 -> 物理介质

## Linux中VFS对象及其数据结构

### SuperBlock

文件系统的控制块，内容包括inode的总量，block的总量，剩余block等。

### inode

存储文件的权限和属性等。一个文件档案一个inode。

### Dentry

linux中一个目录也作为一个文件来对待，但没有真正的磁盘数据结构。VFS在每次目录项缓存中缓存每次访问过的路径名；如果缓存中没找到，则需要对文件系统里面的每个路径分量进行寻找和解析，再将这个Dentry对象存入cache中。


### file

文件对象表示进程已经打开的文件。类似于目录对象，在磁盘中其实是没有对应的磁盘数据的。里面内容包括文件指针、文件操作表、文件一些属性等。

### file_struct

每个进程都有自己的一组打开的文件，用file_struct / fs_struct / namespace来表示。

- file_struct里面保存着fdtable，也就是进程打开fd的表格，如果大于64个fd，会重新动态申请数组。 
系统最大打开文件描述符数：/proc/sys/fs/file-max
永久设置：修改/etc/sysctl.conf文件，增加fs.file-max = 1000000
进程最大打开文件描述符数：
对于进程使用的fd，使用ulimit -n查看当前设置。使用ulimit -n 1000000进行临时性设置。

- fs_struct则包括了pwd和根目录。

- namespace使得每个进程在系统中看到了唯一的安装文件系统。
