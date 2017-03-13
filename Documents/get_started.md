# 下载安装Mininet

下载安装Mininet最简单的方式就是**下载一个已经封装好的Mininet/Ubuntu虚拟机镜像**。虚拟机镜像包含了Mininet自身、所有Openflow二进制文件以及预先安装好的各种工具，和已经为支持大型Mininet网络而修改过的Linux内核。

+ 方法一：通过Mininet虚拟机镜像安装（十分方便，强烈推荐）
+ 方法二：通过源代码来进行本地安装
+ 方法三：通过软件包的方式进行安装
+ 方法四：升级现有的Mininet

---

## 方法一：通过Mininet虚拟机镜像安装（十分方便，强烈推荐）
通过虚拟机镜像进行安装是最容易和最傻瓜的安装Mininet的方式，而这也是我们推荐这种方法的原因。

通过以下步骤来进行安装虚拟机镜像。

1. 下载[虚拟机镜像](https://github.com/mininet/mininet/wiki/Mininet-VM-Images)。
2. 下载并安装一种虚拟机软件。我们推荐[VirtualBox](http://www.virtualbox.org/wiki/Downloads)（免费，遵循GPL协议），因为它是免费的，并且能在OS X、Windows和Linux上运行（即使它在我们的测试中比VMware稍微慢了一点）。你也可以在任意平台上用[Qemu](http://qemu.org/)、在Windows和Linux上用[VMware Workstation](http://www.vmware.com/products/workstation/)、在[VMware Fusion](http://www.vmware.com/products/fusion)，或者在Linux上用[KVM](http://www.linux-kvm.org/)。
3. 在[Mininet讨论区](https://mailman.stanford.edu/mailman/listinfo/mininet-discuss)中注册。这是Mininet相关**支持和讨论**的来源，并且是一个友善的Mininet社区。
4. 按照[VM Setup Notes](http://mininet.org/vm-setup-notes)的提示执行Mininet并登入其中，并按照自己的要求来定制相关选项。
5. 参照[Mininet指南](http://mininet.org/walkthrough)来熟悉Mininet的命令以及典型应用。

（另外，除了以上资源，我们还特意准备了一些[常见问题的解答](http://mininet.org/faq)，以及可供你随时查看的[Mininet文档](http://mininet.org/docs)）

在你完成了[Mininet指南](http://mininet.org/walkthrough)上所有的教程之后，你应该能很清楚地指导Mininet是什么以及有什么用了。如果你对OpenFlow和软件定义网络（SDN）感兴趣，你也可以一并完成[OpenFlow教程](https://github.com/mininet/openflow-tutorial/wiki)。

---

## 方法二：通过源代码来进行本地安装
这种方法适合本地虚拟机、远程EC2（注：亚马逊云服务）以及本地安装。本方法嘉定使用者正在使用一个新版的Ubuntu系统（或者Fedora）。（如果你准备从旧版Mininet或者OVS（Open vSwitch）进行升级，参照下面的提示来卸载旧版软件。）

我们强烈建议使用最新版的Ubuntu发行版，因为最新版的Ubuntu系统提供了对新版Open vSwitch的支持。（Fedora也提供了对最新OVS的支持。）

为了从进行本地安装，我们首先所需要做的事就是拿到源代码：
```bash
git clone git://github.com/mininet/mininet
```

注意，上面的git命令会检出最新最好的Mininet版本（这也是我们所推荐的版本）。如果你想要运行最近被标记或发行的Mininet，抑或其他任意的版本，你可以像这样显式检出你所需要的版本：
```bash
cd mininet
git tag  # 此命令获取版本列表
git checkout -b 2.2.1 2.2.1  # 或者其他你想要的版本号
cd ..
```

在你获得了Mininet的源代码之后，你可以通过如下命令来安装Mininet：
```bash
mininet/util/install.sh [options]
```

通常地，**install.sh**的参数有：
+ **-a**：安装所有Mininet虚拟机镜像所包含的软件，包括例如Open vSwitch之类的依赖项和支持OpenFlow的wireshark以及POX控制器。这些软件会被安装到你的根目录中。
+ **-nfv**：安装Mininet、OpenFlow引用和Open vSwitch。
+ **-s mydir**：通过在其他参数之前添加这个参数来指定安装地点。

所以，你可能会用到下面的一条或者几条命令：
```bash
要在根目录安装所有内容：install.sh -a
要在指定的文件夹中安装所有内容：install.sh -s mydir -a
要安装Mininet + 用户自定的交换机 + OVS（使用根目录）：install.sh -nfv
要安装Mininet + 用户自定的交换机 + OVS（使用其他目录）：install.sh -s mydir -nfv
```

你能通过如下命令找到其他有用的选项：
```bash
install.sh -h
```

在安装完成后，通过以下命令来检测是否成功安装：
```bash
sudo mn --test pingall
```

然后进行方法一中的第3-5步。如果你遇到了任何问题，首先到以下链接中[FAQ](http://mininet.org/faq)、[Documentation](http://mininet.org/docs)以及[邮件列表](https://mailman.stanford.edu/pipermail/mininet-discuss/)看看是否有类似的问题，以及相应的解决方案。如果这些都没有用并且你不能自行解决此问题（即使包括来自Google的帮助），你可以在[mininet-discuss](https://mailman.stanford.edu/mailman/listinfo/mininet-discuss)中要求协助。

---

## 方法三：通过软件包的方式进行安装
