---
title: Linux 从入门到重装——文件与目录权限
date: 2017-06-08 15:32:07
categories: 技术
tags: Linux
---
## 查看权限
通过命令 `ls -l` 或者 `ll`，可查看文件或目录的权限
```
[root@localhost tmp]# ll /
total 24
-rw-r--r--   1 root root    0 Jun  8 00:43 abc
lrwxrwxrwx   1 root root    7 Jun  7 16:37 bin -> usr/bin
dr-xr-xr-x   4 root root 4096 Jun  7 08:53 boot
drwxr-xr-x  20 root root 3340 Jun  7 16:52 dev
drwxr-xr-x 138 root root 8192 Jun  8 00:36 etc
```
其中第一列代表代表这个文件的类型和权限，即 `-rw-r--r--`。

## 权限的表示方法
权限由10个字符组成，第一个字符代表这个文件是“目录、文件或链接文件等”。
* [d] 表示目录
* [-] 表示文件
* [l] 表示链接文件
* [b] 表示设备接口
* [c] 表示串行端口设备

接下来的字符中，均为 “rwx” 或 “-” 的 3字组合，其中 [r] 表示可读（read），[w] 表示可写（write），[x] 表示可执行（execute），注意这 3个权限的位置是不会变的，如果没有权限，就用 [-] 表示。
* 第一组为 “文件所有者的权限”，以文件 “abc” 为例，该文件的所有者权限为 "rw-"，即可读、可写。
* 第二组为 “同用户组的权限”。
* 第三组为 “其他非本用户组的权限”。

## 修改权限
常用的用户组、所有者、各种身份的权限的修改命令如下：
* `chgrp` 改变文件所属用户组。
* `chown` 改变文件所有者。
* `chmod` 改变文件的权限。

权限除了用字母表示外，还可以用数字来表示
* [r]: 4
* [w]: 2
* [x]: 1
* 每种身份（owner、group、others）各自的三个权限（r、w、x）的分数需要累加的，以 `-rwxrwx---` 为例：
* owner = rwx = 4+2+1 = 7
* group = rwx = 4+2+1 = 7
* other = --- = 0+0+0 = 0
* 所以当通过 `chmod` 设置权限时，该文件的权限数字就是 770，如果要将这个文件的所有权限启用，那么就执行：
```
[root@localhost test]# ll
total 0
-rwxrwx--- 1 root root 0 Jun  8 01:07 a
[root@localhost test]# chmod 777 a
[root@localhost test]# ll
total 0
-rwxrwxrwx 1 root root 0 Jun  8 01:07 a

```

## 默认权限：umask
umask 指 “目前用户在新建文件或目录时的权限默认值”
```
[root@localhost ~]# umask
0022
[root@localhost ~]# umask -S
u=rwx,g=rx,o=rx
```
* umask 返回的 4位数字的第一位，代表特殊权限，一会再介绍
* “文件” 默认没有可执行（x）权限，最大值为 666
* ”目录“ 的执行（x）权限代表是否可进入，最大值为 777
* umask 返回的默认值是 ”要被减掉的权限“
* 666 - 022 = 644 = rw-r--r--
* 777 - 022 = 755 = rwxr-xr-x

更改默认权限：
```
[root@localhost test]# umask
0022
[root@localhost test]# touch abc
[root@localhost test]# ll
total 0
-rw-r--r--. 1 root root 0 Jun  8 01:19 abc
[root@localhost test]# umask 002
[root@localhost test]# touch 123
[root@localhost test]# ll
total 0
-rw-rw-r--. 1 root root 0 Jun  8 01:19 123
-rw-r--r--. 1 root root 0 Jun  8 01:19 abc
```

## 特殊权限：SUID，SGID，SBIT
除了 r、w、x 之外，还有其他特殊权限（s、t）
```
[root@localhost /]# ls -ld /tmp ; ls -l /usr/bin/passwd
drwxrwxrwt. 17 root root 4096 Jun  8 01:19 /tmp
-rwsr-xr-x. 1 root root 27832 Jun  9  2014 /usr/bin/passwd
```

### Set UID
* 当 s 出现在 owner 的 x 权限位置时，例如 `-rwsr-xr-x`，称为 Set UID，简称 SUID 的特殊权限。
* 具有 x 权限。
* SUID 仅对二进制程序有效。
* 本权限仅在程序执行时（run-time）有效。
* 执行者将具有该程序所有者的权限。

也就是说，当我们执行 `passwd` 时，任何人都暂时获得了该程序的所有者权限，也就是 `rwx`。

### Set GID
* 当 s 出现在用户组的 x 权限位置时，例如 `-rwx--s--x`，称为 Set GID，简称 SGID 的特殊权限。
* 具备 x 权限。
* 可以对目录使用。
* 对二进制程序有用。
* 执行者在执行过程中将会获得该程序用户组的支持。
* 当目录被设置 SGID 权限时，若用户在此目录下具有写入权限，则用户所创建的新文件的用户组与此目录的用户组相同。

### Sticky Bit
* SBIT 仅对目录有效。
* 当用户对于此目录具有 w，x 权限时，用户在该目录下创建文件或目录时，只有自己与 root 才有权利删除该文件。

### SUID，SGID，SBIT 权限设置
特殊权限也可以用数字表示，数字形态的权限的那 “三个数字” 的组合，在之前再加一个数字，就代表这几个特殊权限了。
* SUID 为 4
* SGID 为 2
* SBIT 为 1

```
[root@localhost tmp]# touch test
[root@localhost tmp]# chmod 4755 test; ls -l test
-rwsr-xr-x. 1 root root 0 Jun  8 01:46 test
[root@localhost tmp]# chmod 6755 test; ls -l test
-rwsr-sr-x. 1 root root 0 Jun  8 01:46 test
[root@localhost tmp]# chmod 6755 test; ls -l test
-rwsr-sr-x. 1 root root 0 Jun  8 01:46 test
[root@localhost tmp]# chmod 1755 test; ls -l test
-rwxr-xr-t. 1 root root 0 Jun  8 01:46 test
[root@localhost tmp]# chmod 7666 test; ls -l test
-rwSrwSrwT. 1 root root 0 Jun  8 01:46 test
```
注意大写的 S、T，代表空的权限，由于 7666 不具备 x 权限，S、T 的基本条件无法满足，所以当然是 “空的” 权限。
