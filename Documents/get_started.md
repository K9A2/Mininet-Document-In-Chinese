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
这种方法适合本地虚拟机、远程EC2（注：亚马逊云服务）以及本地安装。
