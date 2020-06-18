---
title: CentOS SUID/SGID/SBIT权限与其在项目协作中的运用
date: 2018/09/05 21:12:09 
categories: 

- OS
- CentOS

tags: 

- CentOS
- 权限管理

permalink: centos-SUID-SGID-SBIT

---

## **一、几个特殊权限**

### **1、SUID**
当s这个标志出现在文件拥有者的x权限上时，例如用户信息文件/usr/bin/passwd的权限状态为：

```bash
-rwsr-xr-x. 1 root root 27832 6月  10 2014 /usr/bin/passwd
```
这样的权限就被称为Set UID ,简称SUID。

<!--more--> 

对于SUID有如下限制和功能：

- SUID 权限仅对二进位程序(binary program)有效
- 执行者对于该程序需要具有x 的可执行权限
- 本权限仅在执行该程序的过程中有效(run-time)
- 执行者将具有该程序拥有者(owner) 的权限

举个例子来说，我们的Linux系统中，所有帐号的密码都记录在/etc/shadow这个文件里面，这个文件的权限为：【---------- 1 root root】，意思是这个文件仅有root可读且仅有root可以强制写入而已。既然这个文件仅有root可以修改，但是我们知道，在centos中一般用户（例如用户dmtsai）是能够用【 passwd 】指令修改自己的密码的，也就是说一般用户在修改密码时也同时改动了/etc/shadow这个文件的内容，这是否和该文件的权限冲突了呢？

当然没有，这就引出了SUID的功能，因为满足之前所列的四点限制和功能：

- passwd是一个二进制程序
- dmtsai用户对passwd具有x权限
- 该权限在执行过程中有效 
- dmtsai会暂时取得passwd拥有者(root)的权限

由此，dmtsai获得root的权限后便可以修改/etc/shadow中自己的密码信息，整个过程如下图所示：
![](https://img-blog.csdn.net/20180905165216900?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L01TRE5fdGFuZw==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

不过如果dmtsai 使用cat 去读取/etc/shadow 时，由于cat 没有SUID权限，所以无法读出其内容。

另外，SUID仅可用在binary program上，不能够用在shell script上面！这是因为shell script只是将很多的binary执行文件叫进来执行而已！所以SUID的权限部分，还是得要看shell script呼叫进来的程序的设定，而不是shell script本身。当然，SUID对于目录也是无效的。

----------
### **2、SGID**
##### **a）应用于文件：**
当s 标志在档案拥有者的x 项目为SUID，那s 在群组的x 时则称为Set GID，简称SGID，对程序和文件来说，它的限制和功能如下：

- SGID 对二进位程序有用
- 程序执行者对于该程式来说，需具备x 的权限
- 执行者在执行的过程中将会获得该程序群组的支援

举例来说，locate 是centos中一个查询文件位置的命令，它是通过读取和检索/var/lib/mlocate/mlocate.db中的内容来进行快速搜索的，我们来看看两个文件的权限：

```bash
-rwx--s--x. 1 root slocate /usr/bin/locate
-rw-r-----. 1 root slocate /var/lib/mlocate/mlocate.db
```
可以看到mlocate.db 除了拥有者和群组支援外其他人是不能读取的，但是由于locate具有SGID权限，所以普通用户在执行locate之后会获得locate所在群组slocate的支援，而mlocate.db属于slocate群组，所以这时便能够读取mlocate.db文件。

##### **b）应用于文件夹**
除了binary program 之外，事实上SGID 也能够用在目录上，这也是非常常见的一种用途！当一个目录设定了SGID 的权限后，他将具有如下的功能：

- 使用者若对于此目录具有r 与x 的权限时，该使用者能够进入此目录；
- 使用者在此目录下的有效群组(effective group)将会变成该目录的群组；

也就是说，如果使用者在此目录下具有w 的权限，则使用者所建立的新文件的群组与此目录的群组相同。这就给在该目录下工作的不同用户提供了互相访问对方文件的可能，这对于协同工作尤为重要。

具体的例子见下文的项目协作部分

----------


### **3、SBIT**
SBIT 即 Sticky Bit，t 标志在其他成员x权限的位置。目前只针对目录有效，对于文件已经没有效果了。SBIT 对于目录的作用是：

- 当使用者对于此目录具有w, x 权限，亦即具有写入的权限时
- 当使用者在该目录下建立档案或目录时，仅有自己与root 才有权力删除该档案

举例来说就是：当甲这个使用者于A目录是具有群组或其他人的身份，并且拥有该目录w的权限，这表示【甲使用者对该目录内任何人建立的目录或文件均可进行"删除/更名/搬移"等动作】。不过，如果将A目录加上了SBIT的权限项目时，则甲只能够针对自己建立的文件或目录进行删除/更名/移动等动作，而无法删除他人的档案。


### **以上三种权限的设置**
这里只介绍常用的数字型设定方式
以上三种权限的数字位于常规三个权限数字之前，其中：

- 4 为SUID
- 2 为SGID
- 1 为SBIT

例如设置file文件为SUID与SGID可以这样写：

```
chmod 6774 file

其权限变为:
-rws rws r-- 

```

有时候，设置file文件为：

```
chmod 7666 file

其权限变为：
-rwS rwS rwT
```
为什么此处的S和T变为大写了呢？
当 s 与 t 所在位置的 x 权限为【空】时，s与t会显示为大写字母

----------
## **二、项目协作**
有时候在进行一些项目开发时，需要多人合作在同一环境下进行作业，我们一般会选择将项目组成员放在一个群组里，规定一个目录作为工作空间，但是这存在一个问题：尽管是在同一工作空间同一群组，但是成员之间可能仍旧无法互相访问对方的文件，比如像下面这种情况：

A与B都是项目组的成员，系统管理员将他们都放到project这个群组里，同时给他们分配了一个工作空间为/srv/wplace，并设置该工作空间的权限为 : drwx rwx ---  root  project 。操作如下：

先制作两人的帐号资料
```bash
[root@tang ~]# groupadd project                <==增加新的群组 
[root@tang ~]# useradd -G project alex         <==建立alex帐号，且支援project 
[root@tang ~]# useradd -G project arod         <==建立arod帐号，且支援project 
[root@tang ~]# id manA                         <==查阅alex帐号的属性 
uid=1001(manA) gid=1002(agroup) groups=1002(agroup), 1001(project ) 
[root@tang ~]# id manB
uid=1002(manB) gid=1003(bgroup) groups=1003(bgroup), 1001(project)  
```
再建立项目工作空间：

```bash
[root@tang ~]# mkdir /srv/wplace
[root@tang ~]# ll -d /srv/wplace
drwxr-xr-x. 2 root root 6 Jun 17 00:22 /srv/wplace
```

修改工作空间权限

```bash
[root@tang ~]# chgrp project /srv/wplace 
[root@tang ~]# chmod 770 /srv/wplace 
[root@tang ~]# ll -d /srv/wplace 
drwx rwx ---. 2 root project 6 Jun 17 00:22 /srv/wplace
```

从上面可以看出A B两人已经在同一群组里，且该群组可以为他们提供wplace目录的群组权限。这时候貌似没有什么问题了，但是大多数时候A与B的有效群组并非都为project，如A的有效群组为agroup，B的有效群组为bgroup，这时我们来测试看看两个使用者的工作情况：

```bash
[root@tang ~]# su - manA        <==先切换身份成为manA来处理 
[manA@www ~]$ cd /srv/wplace    <==切换到群组的工作目录去 
[manA@www ahome]$ touch abcd  <==建立一个空的档案出来！
[manA@www ahome]$ exit         <==离开manA的身份

[root@tang ~]# su - manB 
[manB@www ~]$ cd /srv/wplace 
[manB@www wplace]$ ll abcd 
-rw-rw-r--. 1 manA agroup 0 Jun 17 00:23 abcd
#仔细看一下上面的档案，由于群组是agroup ，manB并不支援！
#因此对于abcd这个档案来说， manB只是属于其他人范围，只有r的权限而
[arod@www ahome]$ exit
```

由上面的结果我们可以知道，若单纯使用传统的rwx，则对刚刚A建立的abcd这个档案来说， B可以删除他，但是却不能编辑他，然而我们需要的是A B之间能够互相访问编辑彼此的文件，这时候我们便可以引入SGID权限了

```bash

[root@tang ~]# chmod 2770 /srv/wplace 
[root@tang ~]# ll -d /srv/wplace
drwx rws ---. 2 root project 17 Jun 17 00:23 /srv/wplace

测试：使用manA去建立一个档案，并且查阅档案权限看看： 
[root@tang ~]# su - manA
[manA@www ~]$ cd /srv/wplace 
[manA@www wplace]$ touch 1234 
[manA@www wplace]$ ll 1234 
-rw- rw- r--. 1 manA project 0 Jun 17 00:25 1234
```
可以看到，当wplace拥有SGID权限时，在其中建立的文件的群组会强制变为project，这样A与B都在群组project的支援里，故而能够互相访问和编辑彼此的文件，从而达到项目协作的目的。