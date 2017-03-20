# 下载安装 Mininet

下载安装 Mininet 最简单的方式就是 **下载一个已经封装好的Mininet/Ubuntu虚拟机镜像（Mininet VM）**。VM 包含了 Mininet 自身、所有 Openflow 二进制文件以及预先安装好的各种工具，和已经为支持大型 Mininet 网络而修改过的 Linux 内核。

+ 方法一：通过 Mininet 虚拟机镜像安装（十分方便，强烈推荐）
+ 方法二：通过源代码来进行本地安装
+ 方法三：通过软件包的方式进行安装
+ 方法四：升级现有的 Mininet

---

## 方法一：通过 Mininet 虚拟机镜像安装（十分方便，强烈推荐）

通过虚拟机镜像进行安装是最容易和最傻瓜的 Mininet 的安装方式，而这也是我们推荐这种方法的原因。

通过以下步骤来进行安装虚拟机镜像。

1. 下载[虚拟机镜像](https://github.com/mininet/mininet/wiki/Mininet-VM-Images)。
2. 下载并安装一种虚拟机软件。我们推荐 [VirtualBox](http://www.virtualbox.org/wiki/Downloads)（免费，遵循 GPL 协议），因为它是免费的，并且能在 OS X、Windows 和 Linux 上运行（即使它在我们的测试中比 VMware 稍微慢了一点）。你也可以在任意平台上用 [Qemu](http://qemu.org/)、在 Windows 和 Linux 上用 [VMware Workstation](http://www.vmware.com/products/workstation/)、[VMware Fusion](http://www.vmware.com/products/fusion)，或者在Linux上用 [KVM](http://www.linux-kvm.org/)。
3. 在 [Mininet讨论区](https://mailman.stanford.edu/mailman/listinfo/mininet-discuss) 中注册。这是 Mininet 相关**支持和讨论**的来源，并且是一个友善的Mininet社区。
4. 按照 [VM Setup Notes](http://mininet.org/vm-setup-notes) 的提示执行 Mininet 并登入其中，并按照自己的要求来定制相关选项。
5. 参照 [Mininet指南](http://mininet.org/walkthrough) 来熟悉 Mininet 的命令以及典型应用。

（另外，除了以上资源，我们还特意准备了一些[常见问题的解答](http://mininet.org/faq)，以及可供你随时查看的[Mininet 文档](http://mininet.org/docs)）

在你完成了 [Mininet 指南](http://mininet.org/walkthrough)上所有的教程之后，你应该能很清楚地指导 Mininet 是什么以及有什么用了。如果你对 OpenFlow 和软件定义网络（SDN）感兴趣，你也可以一并完成 [OpenFlow 教程](https://github.com/mininet/openflow-tutorial/wiki)。

---

## 方法二：通过源代码来进行本地安装

这种方法适合本地虚拟机、远程 EC2（注：亚马逊云服务）以及本地安装。本方法假定使用者正在使用一个新版的 Ubuntu 系统（或者 Fedora）。（如果你准备从旧版 Mininet 或者 OVS（Open vSwitch）进行升级，参照下面的提示来卸载旧版软件。）

我们强烈建议使用最新版的 Ubuntu 发行版，因为最新版的 Ubuntu 系统提供了对新版 Open vSwitch 的支持。（Fedora 也提供了对最新 OVS 的支持。）

为了从进行本地安装，我们首先所需要做的事就是拿到源代码：
```bash
git clone git://github.com/mininet/mininet
```

注意，上面的 git 命令会检出最新最好的 Mininet 版本（这也是我们所推荐的版本）。如果你想要运行最近被标记或发行的 Mininet，抑或其他任意的版本，你可以像这样显式检出你所需要的版本：
```bash
cd mininet
git tag  # 此命令获取版本列表
git checkout -b 2.2.1 2.2.1  # 或者其他你想要的版本号
cd ..
```

在你获得了 Mininet 的源代码之后，你可以通过如下命令来安装 Mininet：
```bash
mininet/util/install.sh [options]
```

通常地，**install.sh** 的参数有：
+ **-a**：安装所有 Mininet 虚拟机镜像所包含的软件，包括例如 Open vSwitch 之类的依赖项和支持 OpenFlow 的 Wireshark 以及 POX 控制器。这些软件会被安装到你的根目录中。
+ **-nfv**：安装 Mininet、OpenFlow 引用和 Open vSwitch。
+ **-s mydir**：通过在其他参数之前添加这个参数来指定安装地点。

所以，你可能会用到下面的一条或者几条命令：
```bash
要在根目录安装所有内容：install.sh -a
要在指定的文件夹中安装所有内容：install.sh -s mydir -a
要安装 Mininet + 用户自定的交换机 + OVS（使用根目录）：install.sh -nfv
要安装 Mininet + 用户自定的交换机 + OVS（使用其他目录）：install.sh -s mydir -nfv
```

你能通过如下命令找到其他有用的选项：
```bash
install.sh -h
```

在安装完成后，通过以下命令来检测是否成功安装：
```bash
sudo mn --test pingall
```

然后进行方法一中的第3-5步。如果你遇到了任何问题，首先到以下链接中 [FAQ](http://mininet.org/faq)、[Documentation](http://mininet.org/docs) 以及 [邮件列表](https://mailman.stanford.edu/pipermail/mininet-discuss/) 看看是否有类似的问题，以及相应的解决方案。如果这些都没有用并且你不能自行解决此问题（即使包括来自 Google 的帮助），你可以在 [mininet-discuss](https://mailman.stanford.edu/mailman/listinfo/mininet-discuss) 中要求协助。

---

## 方法三：通过软件包的方式进行安装
如果你正在使用较新版本的 Ubuntu 发行版，你可以通过软件包的形式来安装 Mininet。注意，即使这种方式可能会在你的电脑上安装一个非最新版本的 Mininet，但这也是一种非常方便的方法。

首先，如果你正在或者已经从一个较久的版本（如1.0）升级，或者使用一个不在 **/usr/local** 目录中编译安装的 Open vSwitch，你确保在升级前就把旧版的 Mininet 和 Open vSwitch 从 **/usr/local** 都卸载了：
```bash
sudo rm -rf /usr/local/bin/mn /usr/local/bin/mnexec \
    /usr/local/lib/python*/*/*mininet* \
    /usr/local/bin/ovs-* /usr/local/sbin/ovs-*
```

然后，确认你的操作系统的版本：
```bash
lsb_release -a
```

再者，对应你的操作系统的版本号，只用下面的命令中的其中之一来安装 Mininet 基本软件包：
```bash
Mininet 2.1.0 on Ubuntu 14.10: sudo apt-get install mininet
Mininet 2.1.0 on Ubuntu 14.04: sudo apt-get install mininet
Mininet 2.0.0 on Ubuntu 12.04: sudo apt-get install mininet/precise-backports
```

然后用如下命令来测试 Mininet 是否成功安装：
```bash
sudo mn --test pingall
```

如果 Mininet 报告 Open vSwitch 出现问题，你就可能需要重新编译它的内核：
```bash
sudo dpkg-reconfigure openvswitch-datapath-dkms
sudo service openflow-switch restart
```

如果你想跟随指南来学习，你会需要安装一些其他软件：
```bash
git clone git://github.com/mininet/mininet
mininet/util/install.sh -fw
```
以上命令会安装 OpenFlow 推荐的软交换机、控制器以及 Wireshark 分析器。

## 方法四：升级现有的 Mininet
升级方式多种多样。如果你不曾改动 Mininet，你通常可以这么做：
```bash
cd mininet
git fetch
git checkout master   # 或者一个具体的版本号，如2.2.1
git pull
sudo make install
```

你可以用 **sudo make develop** 来代替 **sudo make install**，以便你能从 **/usr/python/...** 建立符号链接到你的源代码文件夹中。

注意，这种方法只会升级 Mininet 本体。其他组件，如 Open vSwitch 等，则可以根据需要分开升级。