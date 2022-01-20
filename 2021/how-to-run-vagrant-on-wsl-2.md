https://blog.thenets.org/how-to-run-vagrant-on-wsl-2/

wsl2里运行vagrant
virtualbox >6.1.30 安装在windows上

最重要的一点是：

VirtualBox必须安装在 Windows 上。Windows 将处理 VirtualBox 进程，该进程将通过虚拟化类型 2 创建 VM 。在https://www.ibm.com/cloud/learn/hypervisors了解有关虚拟化的更多信息。
Vagrant必须安装在 Linux (WSL 2) 上。需要 Linux 二进制文件，因为 Windows 版本与 WSL 2 不兼容。
