# Mininet 指南
本指南给出了大部分 Mininet 命令的使用方法，以及一些和 Wireshark 一起使用的典型例子。

本指南假定你是使用 Mininet 虚拟机镜像（Mininet VM，或者简称为 VM）进行安装的，或者是在 Ubuntu 系统中安装好了所有的 OpenFlow 工具以及 Mininet 本体（通常使用 **install.sh** 命令进行完整安装）。

阅读完所有材料大约需要一个小时。

+ [第一部分：Mininet 的常用技巧](#1)
    + [显示启动选项](#1.1)
    + [启动 Wireshark](#1.2)
    + [与主机和交换机通信](#1.3)
    + [测试主机间的连通性](#1.4)
    + [运行一个 WEB 服务器以及客户端](#1.5)
    + [清理运行文件](#1.6)
+ [第二部分：高级启动选项](#2)
    + [进行回归测试](#2.1)
    + [修改网络拓扑大小和类型](#2.2)
    + [设置网络连接选项](#2.3)
    + [调整信息输出等级](#2.4)
    + [自定义网络拓扑](#2.5)
    + [ID = MAC](#2.6)
    + [用 XTerm 进行显示](#2.7)
    + [其他交换机类型](#2.8)
    + [Mininet 基准测试](#2.9)
    + [组件命名空间中的一切](#2.10)
+ [第三部分：Mininet 命令行界面（CLI）中的命令](#3)
    + [显示所有选项](#3.1)
    + [Python 解释器](#3.2)
    + [联网与断网](#3.3)
    + [用 XTerm 进行显示](#3.4)
+ [第四部分：Python API 例子](#4)
    + [SSH 守护进程](#4.1)
+ [第五部分：看完指南了！](#5)
    + [完全掌握 Mininet 所需要做的事](#5.1)
+ [附录：补充信息](#6)
    + [使用远程控制器](#6.1)
    + [NOX 控制器](#6.2)

*注意：如果你正在使用 Ubuntu Mininet 2.0.0d4 软件包，在命令的语法上会有一些不同，例如：对于 **Topo()** 而言，会有 **add_switch** 和 **addSwitch** 两种形式。如果你从源代码检出 Mininet，你可能会希望检出和   2.0.04 相一致的 2.0.0d4 版本，以便查看源代码（包括 **example** 中的代码）*

<h2 id="1">第一部分：Mininet 的常用技巧</h2>
首先，是对本指南中出现的命令的语法上的说明：

+ **$** 表示在 shell 终端中输入的 Linux 命令

+ **mininet>** 表示在 Mininet CLI 中输入的命令

+ **#** 表示在超级用户下输入的 Linux 命令

在每一个例子中，你都应该只输入提示符右侧的命令（并且按下回车键）。

<h3 id="1.1">1.1 显示启动选项</h3>
让我们从 Mininet 的启动选项开始吧。

输入以下命令来显示所有的启动选项：
```bash
$ sudo mn -h
```
本指南会覆盖其中大部分的选项。

<h3 id="1.2">1.2 启动 Wireshark </h3>
用支持 OpenFlow 的 Wireshark 可以查看控制流中的数据包。首先要在后台打开 Wireshark：

```bash
$ sudo wireshark &
```
在 Wireshark 的过滤器选项中，输入以下过滤条件，并点击**应用**：
```bash
of
```
在 Wireshark 中，点击捕获，然后回到主界面中，然后选择本地环回（**lo**）。

现在，Wireshark 的主窗口中应该就会有 OpenFlow 数据包显示了。

*注意：Wireshark 是在 Mininet VM 中被默认安装好的。如果你所用的系统中没有安装Wireshark 和 OpenFlow 插件，你可以通过像下面这样使用 Mininet 命令：**install.sh**来安装它们*：
```bash
$ cd ~
$ git clone https://github.com/mininet/mininet  # if it's not already there
$ mininet/util/install.sh -w
```
如果 Wireshark 在安装后不能运行（就是你在运行的时候出现了类似于 **\$DISPLAY not set** 之类的错误，请到 FAQ 中寻找解决方法：[https://github.com/mininet/mininet/wiki/FAQ#wiki-x11-forwarding](https://github.com/mininet/mininet/wiki/FAQ#wiki-x11-forwarding)）。

把 X11 设置好后，你就能运行其他 GUI 程序以及 **XTerm** 模拟器了。我们会在接下来的章节中谈到这个问题。

<h3 id="1.3">1.3 与主机和交换机通信</h3>
在 CLI 中输入以下命令来启动一个最小化的网络拓扑结构：

```bash
$ sudo mn
```
默认的拓扑结构是**最小化的**，只包含了2个连接到 OpenFlow 内核的主机，以及 OpenFlow 控制器。这个拓扑也能被用命令显式地调用：**--topo=minimal**。其它的拓扑结构也是开箱即用的，通过查看 **mn -h** 中的 **--topo** 部分可以查看到相关信息。

所有的网络实体（2个主机进程，1个交换机进程以及1个基本控制器）运行在VM中。然而，控制器是可以在虚拟机镜像之外运行的。相关的命令会在指南的末尾进行介绍。

如果没有用参数的形式输入特定的检测指令，Mininet CLI 就会出现了。

在 Wireshark 窗口中，你应该能看到交换机被连接到了控制器上（*译注：产生了控制流数据包*）。

显示所有可用的 Mininet CLI 命令：
```bash
mininet> help
```
显示网络中所有节点：
```bash
mininet> nodes
```
显示网络中的所有连接：
```bash
mininet> net
```
打印所有节点的信息：
```bash
mininet> dump
```
你就能看到1台交换机和2台主机了。

如果在 Mininet CLI 中输入的命令的第一个参数是某一主机、交换机或者控制器的名字，这条命令就会在那个节点上运行。像下面这样在主机进程中运行命令：
```bash
mininet> h1 ifconfig -a
```
你应该能在 Wireshark 的本地还回接口（**lo**）中查看到主机的 **h1-eth0** 网卡所发送的数据包。注意：这块网卡是不能在真实的 Linux 系统中用 **ifconfig** 命令查看到的，因为这只在主机进程的命名空间中有效。

相反，交换机是默认运行在真实系统中的命名空间内的。所以在交换机中运行命令是和在通常的终端中运行命令是没有区别的。
```bash
mininet> s1 ifconfig -a
```
这条命令会显示和交换机的网卡有关的信息，以及 VM 与外部（**eth0**）的链接。

而对于主机与网络相隔离的情况而言（*译注：即主机刚插上网卡，尚未发送 ARP 包来确认自己的网络中的邻居的 MAC，不能与其他主机通信）*，在 **h1** 和 **s1** 上运行 **arp** 和 **route** 命令。

把每一个主机、交换机和控制器放在独立的网络空间中是可能的。但是，这样做并无意义，除非你想要创建一个有多个控制器的网络。Mininet 支持这种操作，用
**--innamesace**。

注意，只有网络是被虚拟的；每一个主机进程都能看到（和自己）同一组（的）进程和目录。例如，在主机界面打印所有主机进程列表：
```bash
mininet> h1 ps -a
```
这样子跟在根命名空间看到的是一样的：
```bash
mininet> s1 ps -a
```
在 Linux 中使用不同的进程空间是能做到的。但目前 Mininet 并不支持这种操作。然而把所有进程都放在根进程的命名空间中是便于 debug 的，因为这样子，你就能用 **ps** 、**kill** 等命令在控制台看到所有进程的信息。

<h3 id="1.4">1.4 测试主机间的连通性</h3>
现在，确认你能从 host 0 ping 通 host 1：

```bash
mininet> h1 ping -c 1 h2
```
如果命令中的参数出现了节点名字，那么这个节点名就会被它的 IP 地址所代替。

你应该能观察到 OpenFlow 的控制流。主机一的 ARP 包查询主机二的 MAC 地址，并同时发送 **packe\_in** 到信息到控制器中。然后控制器就会广播 **packet\_out** 信息到网络中。而主机二在收到这个 ARP 请求后，向对方发送回复。这个回复也会被发送到控制器中，然后控制器就会把这个数据包转发给主机一，并且下发一个流表项（到相关的交换机中）。

现在，主机一已经知道了主机二的 MAC 地址，并且能把它的 ping 数据包通过一个ICMP Echo 请求发送出去。这个请求，以及由主机二所发送的回复，都会被发送控制器，并变成一个流标项下发（以及要发送的实际数据）。

重复上一次ping操作：
```bash
mininet> h1 ping -c 1 h2
```
你应该发现，第二次 **ping** 操作的延迟低了很多（<100us）。很明显，一条包含ICMP **ping** 的流表项目（译注：用于转发双方数据的）已经被下发到交换机中了。所以不会有新的控制流产生，并且这一类的数据包都能直接通过交换机。

一种更方便的方式使用 Mininet CLI 內建的 pingall 命令（实现了所有主机之间的 ping 操作）：
```bash
mininet> pingall
```

<h3 id="1.5">1.5 运行一个WEB服务器以及客户端</h3>

记住，**ping** 操作并不是你在主机上能进行的唯一操作！ Mininet 允许你运行在Linux 系统（或者 VM）以及对应的文件系统中所能获得的所有命令或者程序。你也能输入任何 **bash** 命令，包括作业控制命令（**\$**，**jobs** 等等）。

下一步，尝试着在 **h1** 上运行一个简单的 HTTP 服务器，并从 **h2** 中发送一个 HTTP 请求，然后关闭这个服务器：
```bash
mininet> h1 python -m SimpleHTTPServer 80 &
mininet> h2 wget -O - h1
...
mininet> h1 kill %python
```
最后退出 CLI：
```bash
mininet> exit
```

<h3 id="1.6">1.6 清理运行文件</h3>
如果 Mininet 因为某些原因崩溃了，请用以下命令清理相关的冗余文件：

```bash
$ sudo mn -c
```

<h2 id="2">第二部分：高级启动选项</h2>

<h3 id="2.1">2.1 进行回归测试</h3>
你不必进入到 CLI，Mininet 也能进行自主回归测试：

```bash
$ sudo mn --test pingpair
```
这条命令创建了一个最小化的网络拓扑，开启了 OpenFlow 控制器，运行了一个所有主机之间相互 **ping** 的测试，并关闭了这个网络拓扑以及控制器。

另外一中有用的测试时 **iperf**（给它10秒钟来完成测试）：
```bash
$ sudo mn --test iperf
```
这条命令创建了和上面一样的 Mininet 环境。在某一主机上运行了一台 iperf 服务器，在另外一台主机上运行了一个客户端，并解析了所获得的带宽信息。

<h3 id="2.2">2.2 修改网络拓扑大小和类型</h3>

默认的网络拓扑只有一台交换机，和连接到交换机上的两台主机。你能通过输入 **--topo** 来使用另外一个拓扑，并在后面用参数的形式传入新的拓扑信息。例如，为了测试一个只有一台交换机和三台主机的网络的连通性：

运行一次回归测试：
```bash
$ sudo mn --test pingall --topo single,3
```
另外一个例子，（创建并测试）一个线性拓扑（每一台交换机都连接着一台主机，并且所有交换机都连接在一根总线上）：
```bash
$ sudo mn --test pingall --topo linear,4
```
使用参数化的网络是 Mininet 最有用最强大的特性。

<h3 id="2.3">2.3 设置网络连接选项</h3>
Mininet 2.0 允许你设置连接选项，并且这些选项能够通过命令被自动地设定：

```bash
$ sudo mn --link tc,bw=10,delay=10ms
mininet> iperf
...
mininet> h1 ping -c10 h2
```
如果每一段链路的延迟是 10ms，由于 ICMP 请求走过了两段链路（一段由源主机去往交换机，另外一段去往目的主机），并且 ICMP 回复也走过了同样的链路，那么整个往返时延（RTT）就是 40ms。

*你可以用 [Mininet的Python API](https://github.com/mininet/mininet/wiki/Introduction-to-Mininet) 来自定义每一段链路，但现在，你更需要的做是继续阅读我们的指南。*

<h3 id="2.4">2.4 调整信息输出等级</h3>

默认的信息输出等级是 **info** ，会打印显示所有 Mininet 的动作。相对而言，更加全面的是 **debug** 。用参数 **-v** 来指定输出等级：
```bash
$ sudo mn -v debug
...
mininet> exit
```
然而，这样做，屏幕会显示大量的信息。现在，使用 **output** 来打印 CLI 输出以及一些其他的信息：
```bash
$ sudo mn -v output
mininet> exit
```
除了 CLI，还有其他的等级可以使用，例如 **warning**，能用来在回归测试中隐藏不需要的功能信息。

<h3 id="2.5">2.5 自定义网络拓扑</h3>

如果用 Python API，自定义的网络拓扑是很容易创建的。其中一个例子在 **custom/topo-2sw-2host.py**。这个拓扑把两台交换机直接相连，并给每一台交换机连上了一台主机：
```python
"""Custom topology example

Two directly connected switches plus a host for each switch:

   host --- switch --- switch --- host

Adding the 'topos' dict with a key/value pair to generate our newly defined
topology enables one to pass in '--topo=mytopo' from the command line.
"""

from mininet.topo import Topo

class MyTopo( Topo ):
    "Simple topology example."

    def __init__( self ):
        "Create custom topo."

        # Initialize topology
        Topo.__init__( self )

        # Add hosts and switches
        leftHost = self.addHost( 'h1' )
        rightHost = self.addHost( 'h2' )
        leftSwitch = self.addSwitch( 's3' )
        rightSwitch = self.addSwitch( 's4' )

        # Add links
        self.addLink( leftHost, leftSwitch )
        self.addLink( leftSwitch, rightSwitch )
        self.addLink( rightSwitch, rightHost )


topos = { 'mytopo': ( lambda: MyTopo() ) }
```
当一个自定义的 Mininet（拓扑）文件被提供时，能在命令中添加新的拓扑、
新的交换机种类，以及测试刚刚创建的网络。例如：
```bash
$ sudo mn --custom ~/mininet/custom/topo-2sw-2host.py --topo mytopo --test pingall
```

</h3 id="2.6">2.6 ID = MAC</h3>

默认地，主机在启动的时候会被随机指派 MAC 地址。这样会使 debug 变得更加困难，因为每一次启动 Mininet 时，MAC 地址都会被改变，故某些主机之间的控制流会变得难以传输。

**--mac** 选项是十分有用的，并且能把主机的 MAC 地址和 IP 地址设置成为简单的、唯一的、便于阅读的 ID。

变换之前：
```bash
$ sudo mn
...
mininet> h1 ifconfig
h1-eth0  Link encap:Ethernet  HWaddr f6:9d:5a:7f:41:42  
          inet addr:10.0.0.1  Bcast:10.255.255.255  Mask:255.0.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:6 errors:0 dropped:0 overruns:0 frame:0
          TX packets:6 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:392 (392.0 B)  TX bytes:392 (392.0 B)
mininet> exit
```

变换之后：
```bash
$ sudo mn --mac
...
mininet> h1 ifconfig
h1-eth0  Link encap:Ethernet  HWaddr 00:00:00:00:00:01
          inet addr:10.0.0.1  Bcast:10.255.255.255  Mask:255.0.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
mininet> exit
```

*然而，向 Linux 报告的交换机数据端口仍然是随机的。这是因为你能对一个数据端口用 OpenFlow 来指定端口，正如在 FAQ 中提到的那样。这是一个你可能会忽略的小细节。*

<h3 id="2.7">2.7 用 XTerm 进行显示</h3>
对于复杂的 debug 操作，你能在启动 Mininet 的时候让它启动多个 XTerm 窗口。

要为每一个主机和交换机都启动一个**XTerm**窗口，用 **-x** 选项：
```bash
$ sudo mn -x
```
几秒之后，这些窗口就会出现了，并且会自动地设置正确的窗口名字。

另外，你能打开额外的 XTerm 窗口。

默认地，只有主机是被置于独立的命名空间中；而给每一个交换机都打开一个窗口则是不必要的（如果这样做，那跟只打开一个终端没有什么区别）。然而，这样有助于把交换机的 debug 命令分开（例如流计数器输出）。

例如：

在 XTerm 窗口“switch: s1 (root)”中运行：
```bash
# dpctl dump-flows tcp:127.0.0.1:6634
```
不会有输出信息；交换机也不会增加流表项。要在别的交换机上使用 **dpctl**，请以详细模式启动 Mininet，并且在交换机被被创建的时候查看它们的被动监听端口。

现在，在标题为“host: h1”的 XTerm 窗口中运行：
```bash
# ping 10.0.0.2
```
回到 s1 并打印其中的流表：
```bash
# dpctl dump-flows tcp:127.0.0.1:6634
```
你应该能看到很多个流表项。另外（通常而言，更加方便），你能在无需任何 XTerm 窗口或者手动指定交换机的 IP 和端口的情况下，使用 Mininet CLI 内置的 **dpctl** 命令。

你通过检查 ifconfig 来确认一个 XTerm 窗口是否在跟命名空间中；如果所有的网卡都显示出来了（包括 **eth0**），那么这个窗口就是在根命名空间中。另外，它的标题应该包括“（root）”。

在 Mininet CLI 中关闭 Mininet：
```bash
mininet> exit
```
这些 XTerm 窗口应该会自动地关闭。

<h3 id="2.8">2.8 其他交换机类型</h3>
（除了默认的交换机）有其他的类型的交换即可供使用。例如，运行一个在用户空间的交换机：

```bash
$ sudo mn --switch user --test iperf
```
注意：这样做会使得 iperf 所报告的网络带宽比用内核（默认）交换机的要小。

由于现在数据包需要从内核区跑到用户区，所以，你会在运行 ping 测试的时候察觉到更高的延迟。而且，由于模拟主机的进程会被操作系统调度操作所影响，ping 的延迟时间变化会更大。

另一方面，处于用户空间的交换机会更加便于实现新功能，特别是那些对性能要求不高的功能。

另外一个例子就是预装在 Mininet VM 中的 Open vSwitch（OVS）。由 iperf 所报告的带宽应该和处在内核中 OpenFlow（交换机）模块相类似，甚至更大：
```bash
$ sudo mn --switch ovsk --test iperf
```

<h3 id="2.9">2.9 Mininet 基准测试</h3>
要记录建立和拆除网络拓扑的时间，运行“none”测试：

```bash
$ sudo mn --test none
```

<h3 id="2.10">2.10 组件命名空间中的一切（只涉及到用户空间中的交换机）</h3>

默认地，主机是被放置在他们自己的命名空间中的，而交换机和控制器是被放置在根命名空间中的。而要把交换机放置在它们自己的命名空间中，使用
**--innamespace** 选项：
```bash
$ sudo mn --innamespace --switch user
```
（有自己命名空间的）交换机会单独地通过一条桥接控制链接与控制器相通信，而非使用本地环回（loopback）。然而，就命令本身的功能而言，这并不是一个很有用的命令。但这也提供了一个如何隔离不同的交换机的方法。

注意：这个选项是不会作用于 Open vSwitch 的。
```bash
mininet> exit
```

<h2 id="3">第三部分：Mininet 命令行界面（CLI）中的命令</h2>

<h3 id="3.1">3.1 显示所有选项</h3>
要查看 CLI 中所支持的选项，请开启一个最小化的网络拓扑，并让它运行：

```bash
$ sudo mn
```
显示所有选项：
```bash
mininet> help
```

<h3 id="3.2">3.2 Python 解释器</h3>

如果 Mininet 命令的第一个词是 **py**，那么这条命令就会用 Python 执行。这对于扩展 Mininet 的功能以及探查它的内部结构来说，是很方便的。每一个主机、交换机和控制器都有一个相关的“节点”对象。

在 Mininet CLI 运行如下命令：
```bash
mininet> py 'hello ' + 'world'
```
打印可用的本地变量：
```bash
mininet> py locals()
```
然后，使用 dir 函数来查看节点可用的方法和属性：
```bash
mininet> py dir(s1)
```
你可以通过使用 **help()** 函数来查看节点上可用的函数的在线文档：
```bash
mininet> py help(h1) 
(Press "q" to quit reading the documentation.)
```
你也可以得到变量的值：
```bash
mininet> py h1.IP()
```

<h3 id="3.3">3.3 联网与断网</h3>
对于默认的容错测试而言，能把某段链路单独联网或者断网是非常有用的。

要关闭某一段虚拟以太网链路（双方均关闭）：
```bash
mininet> link s1 h1 down
```
你可以发现，OpenFlow Port Status Change（OpenFlow 端口改变）通知就会被产生。而要开启这段链路：
```bash
mininet> link s1 h1 up
```

<h3 id="3.4">3.4 用 XTerm 进行显示</h3>
要用一个 XTerm 窗口来显示 h1 和 h2（的相关信息）：

```bash
mininet> XTerm h1 h2
```

<h2 id="4">第四部分：Python API 例子</h2>

Mininet 源代码文件夹中有一个[样例文件夹](https://github.com/mininet/mininet/tree/master/examples)，包括了许多如何使用 Mininet 的 Python API 的例子，以及许多能被整合进 Mininet 本体的有用代码。

*注意：正如在本指南开头所提醒的一样，本指南已经假定你正在使用包含了所有东西的 Mininet VM，或者包括了所有相关程序的本地（native）安装。而完整的本地包含用 OpenFlow 实现的控制器 **controller**。如果还没有安装的话，请用 **install.sh -f** 命令进行完整安装。*

<h3 id="4.1">4.1 SSH 守护进程</h3>
一个在每一个主机上都运行一个 SSH 守护进程的例子如下：

```bash
$ sudo ~/mininet/examples/sshd.py
```
在另外一个终端中，你可以用 SSH 登录任意主机，并运行命令：

```bash
$ ssh 10.0.0.1
$ ping 10.0.0.2
...
$ exit
```
退出登录：
```bash
$ exit
```
你会想要在看完 [Mininet简介](https://github.com/mininet/mininet/wiki/Introduction-to-Mininet) 之后再回来看这些介绍Python API的例子。

<h2 id="5">第五部分：看完指南了！</h2>
恭喜！你已经看完 Mininet 指南了。自信地去尝试拓扑、新控制器或者检出最新版本的源代码吧。

<h3 id="5.1">5.1 完全掌握 Mininet 所需要做的事</h3>

如果你觉得还不够，你完全可以去看我们的 [OpenFlow教程](https://github.com/mininet/openflow-tutorial/wiki)。

虽然你已经学到了很多使用 Mininet CLI 的技巧，然而，当你掌握了它的 Python API 之后，Mininet 会变得越来越有用越来越强大。在 [Mininet简介](https://github.com/mininet/mininet/wiki/Introduction-to-Mininet) 提供了对 Mininet 本体和其 Python API 的介绍。

如果你正在想怎么样才能用上一个远程控制器（就是不受 Mininet 控制的控制器），方法就在下面。

<h2 id="6">附录：补充信息</h2>
这些并不是必须要看的内容。然而，当你浏览完之后，也许会有用。

<h3 id="6.1">6.1 使用远程控制器</h3>

*注意：这并不是本指南的默认部分；当你的控制器在不在 VM 中的时候，如在 VM 中的主机，或者在另外一台物理 PC 上，这一部才会变的有用。OpenFlow 教程用 **controller --remote** 来启动一个用 POX、NOX、Beacon 或者 Floodlight 来控制的，简单的，用于学习的交换机。*

当你启动 Mininet 的时候，每一个主机都会被链接到一个远程控制器上。这个控制器可以在 VM 中，可以不在 VM 中，也可以在你的本地机器上，甚至世界上的任何地方。

如果你在你的机子上已经安装好了控制器和开发工具（如 Eclipse），或者你想要测试一个在不同的主机（甚至在云上）的控制器，那么本教程就是十分有用的。

如果你想要尝试（使用远程控制器），把以下命令中的 IP 地址和监听端口换成控制器所在的 IP 地址和端口号：
```bash
$ sudo mn --controller=remote,ip=[controller IP],port=[controller listening port]
```
例如，要运行 POX 自带的交换机，你可以在其中某个窗口这么做：
```bash
$ cd ~/pox
$ ./pox.py forwarding.l2_learning
```
而在另一个窗口，启动 Mininet 并连接到“远程”控制器（实际上是在本地运行的，只不过不受 Mininet 控制）：
```bash
$ sudo mn --controller=remote,ip=127.0.0.1,port=6633
```
注意：这些实际上是默认的 IP 地址和端口号。

如果你进行了某些产生数据流量的操作（如：**h1 ping h2**），你应该能在 POX 控制器的窗口看到，交换机已经连接上了，并且流表项已经下发。

只要你启动这些控制器，并指明 **remote** 选项以及给出正确的 IP 地址和端口号，许多 OpenFlow 控制器就能在Mininet中运用。

<h3 id="6.2">6.2 NOX Classic</h3>

Mininet的默认安装方式（用 **util/install.sh -a** 安装）并不包括 POX 控制器。如果你想要安装它，请用：**sudo ~/mininet/util/install.sh -x**。

注意，NOX 控制器是不建议使用的，Mininet 以后可能不会再支持它。

要在运行了 NOX 程序“pyswitch”的 NOX 控制器上运行回归测试，那么 **NOX_CORE_DIR** 必须设定为包含有 NOX 可执行文件的目录。

首先，确认 NOX 是否在运行：
```bash
$ cd $NOX_CORE_DIR
$ ./nox_core -v -i ptcp:
```
按 Ctrl + C 来结束 NOX 进程，然后测试 NOX 的 pyswitch 是否在运行：
```bash
$ cd
$ sudo -E mn --controller=nox,pyswitch --test pingpair
```
注意，**--controller** 选项有一个便于指明控制器选项的写法（例如在本例中，在 **nox** 运行 **pyswitch**）。

在 NOX 控制器加载文件，以及交换机连接的时候，会有几秒钟的延迟，但在此过程中，已经完成了 **ping** 操作。

注意，现在 **mn** 已经可以被 **sudo -E** 所调用了，不需要 **NOX\_CORE\_DIR** 环境变量。如果你是在远程调用 NOX 控制器，那么参数 **-E** 是不必要的。另外，你可以用 **sudo visudo** 来更改 **/etc/sudoers** 中的第一行，并把其中的 **env_reseet** 改为：
```bash
Defaults !env_reset
```
那么，在运行 **sudo** 的时候，环境变量就不会被改变了。
