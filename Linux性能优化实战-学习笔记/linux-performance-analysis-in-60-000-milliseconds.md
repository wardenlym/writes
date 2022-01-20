# 一分钟Linux性能分析

定位问题的第一个60秒应该做的事：

```bash
uptime
dmesg | tail
vmstat 1
mpstat -P ALL 1
pidstat 1
iostat -xz 1
free -m
sar -n DEV 1
sar -n TCP,ETCP 1
top
```

通过上面这些指令大致了解系统的资源情况。
In 60 seconds you can get a high level idea of system resource usage and running processes by running the following ten commands.

1. uptime

This gives a high level idea of resource load (or demand)

2. dmesg | tail

This views the last 10 system messages, if there are any. Look for errors that can cause performance issues.

3. vmstat 1

Short for virtual memory stat

要检查的列：
r：在 CPU 上运行并等待轮换的进程数。这为确定 CPU 饱和度提供了比负载平均值更好的信号，因为它不包括 I/O。解释：大于 CPU 计数的“r”值是饱和。

free：以千字节为单位的可用内存。如果要数的位数太多，则您有足够的可用内存。包含在命令 7 中的“free -m”命令更好地解释了空闲内存的状态。

si, so：换入和换出。如果这些不为零，则说明您内存不足。

us, sy, id, wa, st：这些是 CPU 时间的细分，平均跨所有 CPU。它们是用户时间、系统时间（内核）、空闲、等待 I/O 和被盗时间（由其他来宾或 Xen，来宾自己的隔离驱动程序域）。

4. mpstat -P ALL 1

每个 CPU 的 CPU 时间细分

5. pidstat 1

进程的cpu占用情况

6.iostat -xz 1

了解块设备（磁盘）、应用的工作负载和由此产生的性能的一个很好的工具

r/s, w/s, rkB/s, wkB/s：这些是每秒传送到设备的读取、写入、读取千字节和写入千字节。使用这些来表征工作负载。性能问题可能仅仅是由于施加了过多的负载。

await：I/O 的平均时间（以毫秒为单位）。这是应用程序遭受的时间，因为它包括排队时间和服务时间。大于预期的平均时间可能是设备饱和或设备问题的指标。

avgqu-sz：向设备发出的平均请求数。大于 1 的值可能是饱和的证据（尽管设备通常可以并行处理请求，尤其是前端多个后端磁盘的虚拟设备。）

%util：设备利用率。这确实是一个繁忙百分比，显示设备每秒工作的时间。大于 60% 的值通常会导致性能不佳（应在 await 中看到），尽管这取决于设备。接近 100% 的值通常表示饱和。

7. free -m

buffers：用于缓冲区缓存，用于块设备 I/O。
cached：用于页面缓存，由文件系统使用。

我们只想检查这些大小是否接近于零，这会导致更高的磁盘 I/O（使用 iostat 确认）和更差的性能。

8. sar -n DEV 1

使用此工具检查网络接口吞吐量：rxkB/s 和 txkB/s，作为工作量的衡量标准，并检查是否已达到任何限制。

9. sar -n TCP,ETCP 1

这是一些关键 TCP 指标的总结视图。这些包括：

active/s：每秒本地发起的 TCP 连接数（例如，通过 connect()）。
Passive/s：每秒远程发起的 TCP 连接数（例如，通过 accept()）。
retrans/s：每秒 TCP 重传次数。

active and passive 计数通常可用于粗略衡量服务器负载：新接受的连接数（passive）和下游连接数（active）。将active视为出站，将passive视为入站可能会有所帮助，但这并不完全正确（例如，考虑 localhost 到 localhost 的连接）。

The active and passive counts are often useful as a rough measure of server load: number of new accepted connections (passive), and number of downstream connections (active). It might help to think of active as outbound, and passive as inbound, but this isn’t strictly true (e.g., consider a localhost to localhost connection).

Retransmits are a sign of a network or server issue; it may be an unreliable network (e.g., the public Internet), or it may be due a server being overloaded and dropping packets. 

retrans重传是网络或服务器问题的标志；它可能是一个不可靠的网络（例如，公共互联网），或者可能是由于服务器过载并丢弃数据包。

10. top

top 命令包括我们之前检查的许多指标。运行它可以很方便地查看是否有任何东西与之前的命令有很大不同，这表明负载是可变的。

top 的一个缺点是随着时间的推移更难看到模式，这在提供滚动输出的 vmstat 和 pidstat 等工具中可能更清楚。如果您没有足够快地暂停输出（Ctrl-S 暂停，Ctrl-Q 继续），则间歇性问题的证据也可能丢失，并且屏幕会清除。

后续分析
您可以应用更多命令和方法来进行更深入的钻取。请参阅来自 Velocity 2015 的 Brendan 的Linux 性能工具教程（https://netflixtechblog.com/netflix-at-velocity-2015-linux-performance-tools-51964ddb81cf），该教程使用 40 多个命令，涵盖可观察性、基准测试、调优、静态性能调优、分析和跟踪。
