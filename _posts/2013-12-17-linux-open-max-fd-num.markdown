---
date: 2013-12-17 14:36 AM
title: linux最大文件描述符设置
source: 
layout: post
category: linux
---

在linux中，所有都是文件，也就是说一个连接也是一个文件。每打开一个文件，内核会分配一个文件描述符，因此一个进程可打开的最大文件描述符决定了可支持的最大连接数。这是一个重要的参数。
	linux有两个限制：
	1、系统级限制。限制了整个系统可打开的最大文件描述符
	2、每个进程的限制。一个应用可打开的最大文件描述符

***对于系统限制，可以通过这个查看：
11:32 [logwatch@a07-api-3g-b28-youku]$ cat /proc/sys/fs/file-max
131072
tty:[0] jobs:[0] cwd:[~]
11:32 [logwatch@a07-api-3g-b28-youku]$ cat /proc/sys/fs/file-nr 
510     0       131072

其中，file-nr返回结果意思是：
	510，已打开的文件句柄
	0，已分配但是没有用的文件句柄
	131072，系统支持最大的文件句柄数

其中，系统支持的最大的文件句柄数还可以用cat /proc/sys/fs/file-max（或者sysctl fs.file-max）来查看。

修改系统限制，可以这样修改：

修改文件/etc/sysctl.conf 中的
fs.file-max = 100000
一行，保存后，让用户重新登录一下，即可永久生效；或者执行sysctl -p命令。

***对于用户级限制，可以通过下面命令查看：
12:44 [logwatch@a07-api-3g-b28-youku]$ ulimit -Hn
65536
tty:[2] jobs:[0] cwd:[~]
12:45 [logwatch@a07-api-3g-b28-youku]$ ulimit -Sn
20240

其中，-H是硬限制，-S是软限制。区别是软限制，任何进程都可以修改，但是硬限制，只允许root权限用户修改。

如果要修改默认的用户级限制，可以编辑文件/etc/security/limits.conf
比如：
	# End of file
	* soft nproc 20240
	* hard nproc 16384
	* soft nofile 20240
	* hard nofile 65536
解释如下：
第一列的意思是设置针对哪些用户生效，* 表示针对所有用户，但是，它不包含root，因此，如果要对root用户有效，需要增加root用户，也就是后面两行。
第二列，要么是soft，要么是hard，hard只允许root修改，soft允许普通用户修改，最大是hard设定的值。
第三列，包含了要被限制的资源的类型。nofile表示打开的文件数，nproc表示进程数。
第四列，表示设置的值。

临时设置：

如果不想永久设置，还可以使用下面命令临时设置：
ulimit -n 65536

这样，只影响当前登录用户，下次重新登录后失效




