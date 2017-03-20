> 都大三了，实验室的小伙伴们还没有考过四六级诶

# 工作流示例

Mininet 能让你便捷地对一个 SDN 网络进行创建、交互和共享操作，并且让你方便地在硬件上运行此网络。本文阐明了基本的 Mininet 的工作流示例，并且很多额外的资料会被放在 [Mininet指南](/Documents/walkthrough.md)、[OpenFlow 教程](https://github.com/mininet/openflow-tutorial/wiki) 以及 Mininet[文档](https://github.com/mininet/mininet/wiki/Documentation)。

## 创建一个网络

你可以用以下命令来创建一个网络：
```bash
sudo mn --switch ovs --controller ref --topo tree,depth=2,fanout=8 --test pingall
```
刚刚的命令启动了一个用 OVS 交换机，每层有 8 个节点，共 3 层的树形拓扑结构（也就是一共 64 台主机连接在 9 个交换机上。8 台交换机直接连接主机，1 台交换机负责连接其余的交换机）的网络拓扑，并且在启动的时候执行 **pingall** 命令来检测网络连通性。

## 与网络相通信

Mininet 的命令行界面能让你在一个命令行窗口就能控制、管理你所创建的整个虚拟网络。
```bash
mininet> h2 ping h3
```
这个命令让主机 h2 去 ping 主机 h3 的 IP 地址。任何可用的 Linux 命令或者程序都能在都能在任意一个虚拟主机上运行。你能在某一个主机上轻松地启动一个 WEB 服务器，并且发送一个 HTTP 请求：
```bash
mininet> h2 python -m SimpleHTTPServer 80 >& /tmp/http.log &
mininet> h3 wget -O - h2
```

## 定制网络

Mininet 提供的 API 能让你用几行 Python 代码来创建一个自定义的网络。就像下面这样：
```python
from mininet.net import Mininet
from mininet.topolib import TreeTopo
tree4 = TreeTopo(depth=2,fanout=2)
net = Mininet(topo=tree4)
net.start()
h1, h4  = net.hosts[0], net.hosts[3]
print h1.cmd('ping -c1 %s' % h4.IP())
net.stop()
```
上面的命令创建了一个小型网络（4 台主机，3 台交换机），并且主机之间相互执行 ping 操作。

Mininet 发行版包括了一些命令行程序和界面程序，我们希望他们能帮助你完成操作，并对你在你的网络上创作有趣的应用程序有所启发。

## 共享你所创建的网络

Mininet VM 中已经包含所有依赖项以及预装软件，能在常见的虚拟化软件，如：VMware，Xen 和 VirtualBox 中运行。这样，就为共享你创建的网络提供了一个方便的载体。在网络原型被创建出来之后，载有它的虚拟机镜像就能让其他人运行，测试和修改。一个完整并压缩的 Mininet 镜像大约有 1GB。（Mininet 也能被安装到 Ubuntu 中，用：**apt-get install mininet**）。如果你正在阅读一篇来自 SIGCOMM（或者来自其他地方）的有关 SDN 的文章，你就不想点击一下，下载并运行一个活生生的系统？如果是的话，考虑一下在你的系统中创建一个新的 Mininet VM 并与他人共享。

## 在硬件上运行
在 Mininet 上结束网络的设计工作之后，这个网络就能被部署到真实的硬件中，以便进行测试以及评估。

为了能成功地登陆到硬件中，这些在 Mininet 中模拟的部件，必须在对应真实硬件上能以同样的方式运行。虚拟拓扑应该和真实的一样；虚拟以太网连接必须用真实的以太网连接代替。用进程模拟的主机，必须用真实主机的镜像来代替。另外，每一个模拟的 OpenFlow 交换机都应该被一台配置好并连接到控制器的交换机所代替。淡当然，控制器是不需要改变的。当 Mininet 在运行的时候，控制器所看到的就是“真实的”交换机。
