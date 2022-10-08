# IO

你要记住最重要的一点，在 Linux 中一切皆文件。不仅普通的文件和目录，就连块设备、套接字、管道等，也都要通过统一的文件系统来管理。为了方便管理，Linux 文件系统为每个文件都分配两个数据结构，索引节点（index node）和目录项（directory entry）。它们主要用来记录文件的元信息和目录结构。

索引节点，简称为 inode，用来记录文件的元数据，比如 inode 编号、文件大小、访问权限、修改日期、数据的位置等。索引节点和文件一一对应，它跟文件内容一样，都会被持久化存储到磁盘中。所以记住，索引节点同样占用磁盘空间。

目录项，简称为 dentry，用来记录文件的名字、索引节点指针以及与其他目录项的关联关系。多个关联的目录项，就构成了文件系统的目录结构。不过，不同于索引节点，目录项是由内核维护的一个内存数据结构，所以通常也被叫做目录项缓存。

换句话说，索引节点是每个文件的唯一标志，而目录项维护的正是文件系统的树状结构。目录项和索引节点的关系是多对一，你可以简单理解为，一个文件可以有多个别名。

第一种，根据是否利用标准库缓存，可以把文件 I/O 分为缓冲 I/O 与非缓冲 I/O。
缓冲 I/O，是指利用标准库缓存来加速文件的访问，而标准库内部再通过系统调度访问文件。非缓冲 I/O，是指直接通过系统调用来访问文件，不再经过标准库缓存。注意，这里所说的“缓冲”，是指标准库内部实现的缓存。比方说，你可能见到过，很多程序遇到换行时才真正输出，而换行前的内容，其实就是被标准库暂时缓存了起来。

第二，根据是否利用操作系统的页缓存，可以把文件 I/O 分为直接 I/O 与非直接 I/O。
想要实现直接 I/O，需要你在系统调用中，指定 O_DIRECT 标志。如果没有设置过，默认的是非直接 I/O。不过要注意，直接 I/O、非直接 I/O，本质上还是和文件系统交互。如果是在数据库等场景中，你还会看到，跳过文件系统读写磁盘的情况，也就是我们通常所说的裸 I/O。

第三，根据应用程序是否阻塞自身运行，可以把文件 I/O 分为阻塞 I/O 和非阻塞 I/O：所谓阻塞 I/O，是指应用程序执行 I/O 操作后，如果没有获得响应，就会阻塞当前线程，自然就不能执行其他任务。所谓非阻塞 I/O，是指应用程序执行 I/O 操作后，不会阻塞当前的线程，可以继续执行其他的任务，随后再通过轮询或者事件通知的形式，获取调用的结果。

第四，根据是否等待响应结果，可以把文件 I/O 分为同步和异步 I/O：所谓同步 I/O，是指应用程序执行 I/O 操作后，要一直等到整个 I/O 完成后，才能获得 I/O 响应。所谓异步 I/O，是指应用程序执行 I/O 操作后，不用等待完成和完成后的响应，而是继续执行就可以。等到这次 I/O 完成后，响应会用事件通知的方式，告诉应用程序。举个例子，在操作文件时，如果你设置了 O_SYNC 或者 O_DSYNC 标志，就代表同步 I/O。如果设置了 O_DSYNC，就要等文件数据写入磁盘后，才能返回；而 O_SYNC，则是在 O_DSYNC 基础上，要求文件元数据也要写入磁盘后，才能返回。再比如，在访问管道或者网络套接字时，设置了 O_ASYNC 选项后，相应的 I/O 就是异步 I/O。这样，内核会再通过 SIGIO 或者 SIGPOLL，来通知进程文件是否可读写。

## df

df 命令加上 -i 参数，查看索引节点的使用情况，如下所示：

索引节点的容量，（也就是 Inode 个数）是在格式化磁盘时设定好的，一般由格式化工具自动生成。当你发现索引节点空间不足，但磁盘空间充足时，很可能就是过多小文件导致的。所以，一般来说，删除这些小文件，或者把它们移动到索引节点充足的其他磁盘中，就可以解决这个问题。

实际上，内核使用 Slab 机制，管理目录项和索引节点的缓存。/proc/meminfo 只给出了 Slab 的整体大小，具体到每一种 Slab 缓存，还要查看 /proc/slabinfo 这个文件。

```bash
cat /proc/slabinfo | grep -E '^#|dentry|inode'
```

这个界面中，dentry 行表示目录项缓存，inode_cache 行，表示 VFS 索引节点缓存，其余的则是各种文件系统的索引节点缓存。

存储系统 I/O 的工作原理。

文件系统层，包括虚拟文件系统和其他各种文件系统的具体实现。它为上层的应用程序，提供标准的文件访问接口；对下会通过通用块层，来存储和管理磁盘数据。

通用块层，包括块设备 I/O 队列和 I/O 调度器。它会对文件系统的 I/O 请求进行排队，再通过重新排序和请求合并，然后才要发送给下一级的设备层。

设备层，包括存储设备和相应的驱动程序，负责最终物理设备的 I/O 操作。

存储系统的 I/O ，通常是整个系统中最慢的一环。所以， Linux 通过多种缓存机制来优化 I/O 效率。比方说，为了优化文件访问的性能，会使用页缓存、索引节点缓存、目录项缓存等多种缓存机制，以减少对下层块设备的直接调用。同样，为了优化块设备的访问效率，会使用缓冲区，来缓存块设备的数据。

## 磁盘性能指标

说到磁盘性能的衡量标准，必须要提到五个常见指标，也就是我们经常用到的，使用率、饱和度、IOPS、吞吐量以及响应时间等。这五个指标，是衡量磁盘性能的基本指标。

使用率，是指磁盘处理 I/O 的时间百分比。过高的使用率（比如超过 80%），通常意味着磁盘 I/O 存在性能瓶颈。

饱和度，是指磁盘处理 I/O 的繁忙程度。过高的饱和度，意味着磁盘存在严重的性能瓶颈。当饱和度为 100% 时，磁盘无法接受新的 I/O 请求。

IOPS（Input/Output Per Second），是指每秒的 I/O 请求数。

吞吐量，是指每秒的 I/O 请求大小。

响应时间，是指 I/O 请求从发出到收到响应的间隔时间。

这里要注意的是，使用率只考虑有没有 I/O，而不考虑 I/O 的大小。换句话说，当使用率是 100% 的时候，磁盘依然有可能接受新的 I/O 请求。

举个例子，在数据库、大量小文件等这类随机读写比较多的场景中，IOPS 更能反映系统的整体性能；而在多媒体等顺序读写较多的场景中，吞吐量才更能反映系统的整体性能。

## 磁盘 I/O 观测

iostat 是最常用的磁盘 I/O 性能观测工具，它提供了每个磁盘的使用率、IOPS、吞吐量等各种常见的性能指标，当然，这些指标实际上来自 /proc/diskstats。iostat 的输出界面如下。

```bash
# -d -x表示显示所有磁盘I/O的指标
$ iostat -d -x 1
```

![iostat.webp](iostat.webp)

这些指标中，要注意：

%util ，就是我们前面提到的磁盘 I/O 使用率；

r/s+ w/s ，就是 IOPS；

rkB/s+wkB/s ，就是吞吐量；

r_await+w_await ，就是响应时间。

## pidstat -d 1

从 pidstat 的输出你能看到，它可以实时查看每个进程的 I/O 情况，包括下面这些内容。
用户 ID（UID）和进程 ID（PID） 。
每秒读取的数据大小（kB_rd/s） ，单位是 KB。
每秒发出的写请求数据大小（kB_wr/s） ，单位是 KB。
每秒取消的写请求数据大小（kB_ccwr/s） ，单位是 KB。
块 I/O 延迟（iodelay），包括等待同步块 I/O 和换入块 I/O 结束的时间，单位是时钟周期。

## iotop

从这个输出，你可以看到，前两行分别表示，进程的磁盘读写大小总数和磁盘真实的读写大小总数。因为缓存、缓冲区、I/O 合并等因素的影响，它们可能并不相等。

剩下的部分，则是从各个角度来分别表示进程的 I/O 情况，包括线程 ID、I/O 优先级、每秒读磁盘的大小、每秒写磁盘的大小、换入和等待 I/O 的时钟百分比等。

## lsof

用来查看进程打开文件列表

```bash
lsof -p <pid>
```

## filetop (bcc tool)

```bash
# -C 选项表示输出新内容时不清空屏幕 
/usr/share/bcc/tools/filetop -C
```

## 查询线程属于那个进程

ps -efT | grep <pid>

## opensnoop 。它同属于 bcc 软件包，可以动态跟踪内核中的 open 系统调用。

要分析进程的数据读取，当然还要靠上一节用到过的 strace+ lsof 组合。接下来，还是在终端一中，执行 strace 命令，并且指定 mysqld 的进程号 27458。我们知道，MySQL 是一个多线程的数据库应用，为了不漏掉这些线程的数据读取情况，你要记得在执行 stace 命令时，加上 -f 参数：

strace -f -p 27458

## mysql 问题定位

show full processlist 命令 来查看当前正在执行的 SQL 语句。

这个输出中，db 表示数据库的名字；Command 表示 SQL 类型；Time 表示执行时间；State 表示状态；而 Info 则包含了完整的 SQL 语句。

我们知道，MySQL 的慢查询问题，很可能是没有利用好索引导致的，那这条查询语句是不是这样呢？我们又该怎么确认，查询语句是否利用了索引呢？其实，MySQL 内置的 explain 命令，就可以帮你解决这个问题。继续在 MySQL 终端中，运行下面的 explain 命令：

```sql
explain select * from products where productName='geektime';
```

观察这次的输出。这个界面中，有几个比较重要的字段需要你注意，我就以这个输出为例，分别解释一下：select_type 表示查询类型，而这里的 SIMPLE 表示此查询不包括 UNION 查询或者子查询；table 表示数据表的名字，这里是 products；type 表示查询类型，这里的 ALL 表示全表查询，但索引查询应该是 index 类型才对；possible_keys 表示可能选用的索引，这里是 NULL；key 表示确切会使用的索引，这里也是 NULL；rows 表示查询扫描的行数，这里是 10000。根据这些信息，我们可以确定，这条查询语句压根儿没有使用索引，所以查询时，会扫描全表，并且扫描行数高达 10000 行。响应速度那么慢也就难怪了。

接下来，我们就可以给 productName 建立索引了，也就是执行下面的 CREATE INDEX 命令：mysql> CREATE INDEX products_index ON products (productName);ERROR 1170 (42000): BLOB/TEXT column 'productName' used in key specification without a key length不过，醒目的 ERROR 告诉我们，这条命令运行失败了。根据错误信息，productName 是一个 BLOB/TEXT 类型，需要设置一个长度。所以，想要创建索引，就必须为 productName 指定一个前缀长度。那前缀长度设置为多大比较合适呢？这里其实有专门的算法，即通过计算前缀长度的选择性，来确定索引的长度。不过，我们可以稍微简化一下，直接使用一个固定数值（比如 64），执行下面的命令创建索引：mysql> CREATE INDEX products_index ON products (productName(64));Query OK, 10000 rows affected (14.45 sec)Records: 10000 Duplicates: 0 Warnings: 0

## redis 响应延迟问题

## fio 工具

```bash

# 随机读
fio -name=randread -direct=1 -iodepth=64 -rw=randread -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=/dev/sdb

# 随机写
fio -name=randwrite -direct=1 -iodepth=64 -rw=randwrite -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=/dev/sdb

# 顺序读
fio -name=read -direct=1 -iodepth=64 -rw=read -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=/dev/sdb

# 顺序写
fio -name=write -direct=1 -iodepth=64 -rw=write -ioengine=libaio -bs=4k -size=1G -numjobs=1 -runtime=1000 -group_reporting -filename=/dev/sdb 
```

这其中，有几个参数需要你重点关注一下。

direct，表示是否跳过系统缓存。上面示例中，我设置的 1 ，就表示跳过系统缓存。

iodepth，表示使用异步 I/O（asynchronous I/O，简称 AIO）时，同时发出的 I/O 请求上限。在上面的示例中，我设置的是 64。

rw，表示 I/O 模式。我的示例中， read/write 分别表示顺序读 / 写，而 randread/randwrite 则分别表示随机读 / 写。

ioengine，表示 I/O 引擎，它支持同步（sync）、异步（libaio）、内存映射（mmap）、网络（net）等各种 I/O 引擎。上面示例中，我设置的 libaio 表示使用异步 I/O。

bs，表示 I/O 的大小。示例中，我设置成了 4K（这也是默认值）。

filename，表示文件路径，当然，它可以是磁盘路径（测试磁盘性能），也可以是文件路径（测试文件系统性能）。示例中，我把它设置成了磁盘 /dev/sdb。不过注意，用磁盘路径测试写，会破坏这个磁盘中的文件系统，所以在使用前，你一定要事先做好数据备份。

这个报告中，需要我们重点关注的是， slat、clat、lat ，以及 bw 和 iops 这几行。

先来看刚刚提到的前三个参数。事实上，slat、clat、lat 都是指 I/O 延迟（latency）。不同之处在于：

slat ，是指从 I/O 提交到实际执行 I/O 的时长（Submission latency）；

clat ，是指从 I/O 提交到 I/O 完成的时长（Completion latency）；

而 lat ，指的是从 fio 创建 I/O 到 I/O 完成的总时长。

这里需要注意的是，对同步 I/O 来说，由于 I/O 提交和 I/O 完成是一个动作，所以 slat 实际上就是 I/O 完成的时间，而 clat 是 0。而从示例可以看到，使用异步 I/O（libaio）时，lat 近似等于 slat + clat 之和。

再来看 bw ，它代表吞吐量。在我上面的示例中，你可以看到，平均吞吐量大约是 16 MB（17005 KiB/1024）。最后的 iops ，其实就是每秒 I/O 的次数，上面示例中的平均 IOPS 为 4250。


