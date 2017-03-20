# Mininet指南
本指南给出了大部分Mininet命令的使用方法，以及一些和Wireshark一起使用的典型例子。

本指南假定你是使用Mininet虚拟机镜像进行安装的，或者是在Ubuntu系统中安装好了所有的OpenFlow工具以及Mininet本体（通常使用**install.sh**命令进行完整安装）。

阅读所有材料大约需要一个小时。

+ [第一部分：Mininet的常用技巧](#1)
    + [显示启动选项](#1.1)
    + [启动Wireshark](#1.2)
    + [与主机和交换机通信](#1.3)
    + [测试主机间的连通性](#1.4)
    + [运行一个web服务器以及客户端](#1.5)
    + [清理运行文件](#1.6)
+ [第二部分：高级启动选项](#2)
    + [进行回归测试](#2.1)
    + [修改网络拓扑大小和类型](#2.2)
    + [设置网络连接选项](#2.3)
    + [调整信息输出等级](#2.4)
    + [自定义网络拓扑](#2.5)
    + [ID = MAC](#2.6)
    + [用XTerm进行显示](#2.7)
    + [其他交换机类型](#2.8)
    + [Mininet基准测试](#2.9)
    + [组件命名空间中的一切](#2.10)
+ [第三部分：Mininet命令行界面（CLI）中的命令](#3)
    + [显示设置](#3.1)
    + [Python解释器](#3.2)
    + [联网与断网](#3.3)
    + [用XTerm进行显示](#3.4)
+ [第四部分：Python API 例子](#4)
    + [SSH 虚拟客户端](#4.1)
+ [第五部分：看完指南了！](#5)
    + [完全掌握Mininet所需要做的其他事](#5.1)
+ [附录](#6)
    + [使用远程控制器](#6.1)
    + [NOX控制器](#6.2)

*注意：如果你正在使用Ubuntu Mininet 2.0.0d4软件包，在命令的语法上会有一些不同，例如：对于**Topo()**，会有**add_switch**和**addSwitch**两种形式。如果你从源代码检出Mininet，你可能会希望检出和2.0.04相一致的2.0.0d4版本，以便查看源代码（包括**example**中的代码）*

<h2 id="1">第一部分：Mininet的常用技巧</h2>
首先，是对本指南中出现的命令的语法上的说明：

+ **$** 表示在shell终端中输入的Linux命令

+ **mininet>** 表示在Mininet CLI中输入的命令

+ **#** 表示在超级用户下输入的Linux命令

在每一个例子中，你都应该只输入提示符右侧的命令（并且按下回车键）。

<h3 id="1.1">显示启动选项</h3>
让我们从Mininet的启动选项开始吧。

输入以下命令来显示所有的启动选项：
```bash
$ sudo mn -h
```
本指南会覆盖其中大部分的选项。

<h3 id="1.2">启动Wireshark</h3>
用支持OpenFlow的Wireshark可以查看控制流。首先要在后台打开wiresha：
```bash
$ sudo wireshark &
```
在Wireshark的过滤器选项中，输入以下过滤条件，并点击**应用**：
```bash
of
```
在Wireshark中，点击捕获，然后回到主界面中，然后选择本地环回（**lo**）

现在，Wireshark的主窗口中应该就会有OpenFlow数据包显示了。

*注意：Wireshark是在Mininet虚拟机镜像中被默认安装好的。如果你所用的系统中没有安装Wireshark和OpenFlow插件，你可以通过使用Mininet命令：**install.sh**，来安装他们*：
```bash
$ cd ~
$ git clone https://github.com/mininet/mininet  # if it's not already there
$ mininet/util/install.sh -w
```
如果Wireshark在安装后不能运行（就是你在运行的时候出现了类似于**$DISPLAY not set**之类的错误，请到FAQ中寻找解决方法：[https://github.com/mininet/mininet/wiki/FAQ#wiki-x11-forwarding](https://github.com/mininet/mininet/wiki/FAQ#wiki-x11-forwarding)）。

把X11设置好后，你就能运行其他事GUI程序以及**xterm**模拟器了。我们会在接下来的章节中谈到这个问题。

<h3 id="1.3">与主机和交换机通信</h3>
在CLI中输入以下命令来启动一个最小化的网络拓扑结构：
```bash
$ sudo mn
```
默认的拓扑结构是**最小化的**，只包含了连接到2个的OpenFlow内核，以及OpenFlow控制器。这个拓扑也能被用命令显式地调用：**--topo=minimal**。其它的拓扑结构也是开箱即用的，通过查看**mn -h**中的**--topo**部分可以查看到相关信息。

所有的网络实体（2个主机进程，1个交换机进程以及1个基本的控制器）运行在虚拟机镜像中。然而，控制器是可以在虚拟机镜像之外运行的。相关的命令会在指南的末尾进行介绍。

如果没有用参数的形式输入特定的检测指令，Mininet CLI就会出现了。

在Wireshark窗口中，你应该能看到交换机被连接到了控制器上。

显示所有可用的Mininet CLI命令：
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

如果在Mininet CLI中输入的命令的第一个参数是一台主机、交换机或者控制器的名字，这条命令就会在那个节点上运行。在主机进程中运行命令：
```bash
mininet> h1 ifconfig -a
```
你应该能在Wireshark的本地还回接口（**lo**）中查看到主机的**h1-eth0**网卡所发送的数据包。注意：这块网卡是不能在真实的Linux系统中用**ifconfig**命令查看到的，因为这只在主机进程的命名空间中有效。

相反，交换机是默认运行在真实系统中的命名空间内的，所以在交换机中运行命令是和在通常的终端中运行命令是没有区别的。
```bash
mininet> s1 ifconfig -a
```
这条命令会显示和交换机的网卡有关的信息，以及VM与外部（**eth0**）的链接。

而对于其他主机与网络相隔离的例子而言，在**h1**和**s1**上运行**arp**和**route**命令。

把每一个主机、交换机和控制器放在独立的网络空间中是可能的。但是，这样做并无意义，除非你想要创建一个有多个控制器的网络。Mininet支持这种操作，用**--innamesace**。

注意，只有网络是被虚拟的；每一个主机进程都能看到（和自己）同一组（的）进程和目录。例如，在主机界面打印所有主机进程列表：
```bash
mininet> h1 ps -a
```
这样子跟在跟命名空间看到的是一样的：
```bash
mininet> s1 ps -a
```
在Linux容器中用不同的进程空间是能做到的。但目前Mininet并不支持这种操作。然而把所有进程都放在根进程的命名空间中是便于debug的，因为这样子，你就能用**ps**、**kill**等命令在控制台看到所有进程的信息。

<h3 id="1.4">测试主机间的连通性</h3>
现在，确定你能从host 0 ping 通 host 1：
```bash
mininet> h1 ping -c 1 h2
```
如果命令中的参数出现了节点名字，那么这个节点名就会被他的IP地址所代替。

你应该能观察到OpenFlow的控制流。主机一的ARP包查询主机二的MAC地址，并同时发送**packe\_in**到信息到控制器中。然后控制器就会广播**packet\_out**信息到网络中。而主机二在收到这个ARP请求后，向对方发送回复。这个回复也会被发送到控制器中，然后控制器就会把这个数据包转发给主机一，并且下发一个流表接口。

现在，主机一已经知道了主机二的MAC地址，并且能把它的ping数据包通过一个ICMP Echo请求发送出去。这个请求，以及由主机二所发送的回复，都会被发送控制器，并变成一个流标条目下发（以及要发送的实际数据）。

重复上一次ping操作：
```bash
mininet> h1 ping -c 1 h2
```
你应该发现，第二次**ping**操作的延迟低了很多（<100us）。很明显，一条包含ICMP**ping**流量的流表项目已经被下发到交换机中了。所以不会有新的控制流产生，并且这一类的数据包都能直接通过交换机。

一种更方便的方式使用Mininet CLI內建的pingall命令（实现了所有主机之间的ping操作）：
```bash
mininet> pingall
```

<h3 id="1.5">运行一个web服务器以及客户端</h3>
记住，**ping**操作并不是你在主机上能进行的唯一操作！Mininet允许你在Linux系统（或者VM）以及对应的文件系统中所能获得的所有命令或者程序。你也能输入任何**bash**命令，包括作业控制命令（**$**，**jobs**等等）。

下一步，尝试着在**h1**上运行一个简单的HTTP服务器，并从**h2**中发送一个HTTP请求，然后关闭这个服务器：
```bash
mininet> h1 python -m SimpleHTTPServer 80 &
mininet> h2 wget -O - h1
...
mininet> h1 kill %python
```
最后退出CLI：
```bash
mininet> exit
```

<h3 id="1.6">清理运行文件</h3>
如果Mininet因为某些原因崩溃了，请清理相关的冗余文件：
```bash
$ sudo mn -c
```

<h2 id="2">第二部分：高级启动选项</h2>

<h3 id="2.1">进行回归测试</h3>
你不必进入到CLI，Mininet也能进行自主回归测试：
```bash
$ sudo mn --test pingpair
```
这条命令创建了一个最小化的网络拓扑，开启了OpenFlow控制器，运行了一个所有主机之间相互**ping**的测试，并关闭了这个网络拓扑以及控制器。

另外一中有用的测试时**iperf**（给它10秒钟来完成测试）：
```bash
$ sudo mn --test iperf
```
这条命令创建了和上面一样的Mininet环境，在某一主机上运行了一台iperf服务器，在另外一台主机上运行了一个客户端，并解析了所获得的带宽信息。

<h3 id="2.2">修改网络拓扑大小和类型</h3>
默认的网络拓扑只有一台交换机连接到两台主机。你能通过输入**--topo**来使用另外一个拓扑，并在后面用参数的形式传入新的拓扑信息。例如，为了测试一个只有一台交换机和三台主机的网络的连通性：

运行一次回归测试：
```bash
$ sudo mn --test pingall --topo single,3
```
另外一个例子，（创建并测试）一个线性拓扑（每一台交换机都连接着一台主机，并且所有交换机都连接在一根总线上）：
```bash
$ sudo mn --test pingall --topo linear,4
```
能使用参数化的网络是Mininet最有用最强大的特性。

<h3 id="2.3">设置网络连接选项</h3>
Mininet 2.0允许你设置连接选项，并且这些选项能够命令行中被自动地设定：
```bash
$ sudo mn --link tc,bw=10,delay=10ms
mininet> iperf
...
mininet> h1 ping -c10 h2
```
如果每一段链路的延迟是10ms，由于ICMP请求走过了两段链路（一段去往交换机，另外一段去往目的主机），并且ICMP回复也走过了同样的链路，那么整个往返时延（RTT）就是40ms。

*你可以用[Mininet的Python API](https://github.com/mininet/mininet/wiki/Introduction-to-Mininet)来自定义每一段链路，但现在，你更需要的是继续阅读我们的指南。*

<h3 id="2.4">调整信息输出等级</h3>
默认的信息输出等级是**info**，会打印显示所有Mininet的动作。相对而言，更加全面的是**debug**。用参数**-v**来指定输出等级：
```bash
$ sudo mn -v debug
...
mininet> exit
```
然而，屏幕会显示大量的信息。现在，使用**output**来打赢CLI输出以及一些其他的信息：
```bash
$ sudo mn -v output
mininet> exit
```
除了CLI，还有其他的等级可以使用，例如**warning**，能用来在回归测试中隐藏不需要的功能信息。

<h3 id="2.5">自定义网络拓扑</h3>
如果用Python API，自定义的网络拓扑是很容易创建的。其中一个例子在**custom/topo-2sw-2host.py**。这个拓扑把两台交换机直接相连，并给每一台交换机连上了一台主机：
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
当一个自定义的Mininet（拓扑）文件被提供时，能在命令中添加新的拓扑、
新的交换机种类，以及测试刚刚创建的网络。例如：
```bash
$ sudo mn --custom ~/mininet/custom/topo-2sw-2host.py --topo mytopo --test pingall
```

</h3 id="2.6">ID = MAC</h3>
默认地，主机在启动的时候会被随机指派MAC地址。这样会使debug变得更加困难，因为每一次启动Mininet是，MAC地址都会被改变，故某些主机之间的控制流会变得难以传输。

**--mac**选项是十分有用的，并且能把主机的MAC地址和IP地址设置成为简单的、唯一的、便于阅读的ID。

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

*相反，向Linux报告的交换机数据端口仍然是随机的。这是因为你能对一个数据端口用OpenFlow来指定端口，正如在FAQ中提到的那样。这是一个你可能会忽略的小细节。*

<h3 id="2.7">用XTerm进行显示</h3>
对于复杂的debug操作，你能在启动Mininet的时候让它启动多个xterm窗口。

要为每一个主机和交换机都启动一个**xterm**窗口，用**-x**选项：
```bash
$ sudo mn -x
```
几秒之后，这些窗口就会出现了，并且会自动地设置正确的窗口名字。

另外，你能打开额外的xterm窗口。

默认地，只有主机是被置于独立的命名空间中；而给每一个交换机都打开一个窗口则是不必要的（如果这样做，那跟只打开一个终端没有什么区别）。然而，这样有助于把交换机的debug命令分开（例如流计数器输出）。

例如：

在xterm窗口“switch: s1 (root)”中运行：
```bash
# dpctl dump-flows tcp:127.0.0.1:6634
```
不会有输出信息；交换机也不会增加流表项。要在别的交换机上使用**dpctl**，以详细模式启动Mininet，并且在交换机被被创建的时候查看它们的被动监听端口。

现在，在标题为“host: h1”的xterm窗口中运行：
```bash
# ping 10.0.0.2
```
回到s1并打印其中的流表：
```bash
# dpctl dump-flows tcp:127.0.0.1:6634
```
你应该能看到很多个流标项。另外（通常而言，更加方便），你能在无需任何xterm窗口或者手动指定交换机的IP和端口的情况下，使用Mininet CLI内置的**dpctl**命令。

你通过检查ifconfig来确认一个xterm窗口是否在跟命名空间中；如果所有的网卡都显示出来了（包括**eth0**），那么这个窗口就是在根命名空间中。另外，它的标题应该包括“（root）”。

在Mininet CLI中关闭Mininet：
```bash
mininet> exit
```
这些xterm窗口应该会自动地关闭。

<h3 id="2.8">其他交换机类型</h3>
（除了默认的交换价）有其他的类型的交换即可供使用。例如，运行一个在用户空间的交换机：
```bash
$ sudo mn --switch user --test iperf
```
注意：这样做会使得iperf所报告的网络带宽比用内核（默认）交换机的要小。

由于现在数据包需要从内核区跑到用户区，所以，你会在运行ping测试的时候察觉到更高的延迟。而且，由于模拟主机的进程会被操作系统调度操作所影响，ping的延迟时间变化会更大。

另一方面，处于用户空间的交换机会更加便于实现新功能，特别是那些对性能要求不高的功能。

另外一个例子就是预装在Mininet VM中的Open vSwitch（OVS）。由iperf所报告的带宽应该和处在内核中OpenFlow（交换机）模块相类似，甚至更大：
```bash
$ sudo mn --switch ovsk --test iperf
```

<h3 id="2.9">Mininet基准测试</h3>
要记录建立和拆除网络拓扑的时间，运行“none”测试：
```bash
$ sudo mn --test none
```

<h3 id="2.10">组件命名空间中的一切（只涉及到用户空间中的交换机）</h3>
默认地，主机是被放置在他们自己的命名空间中的，而交换机和控制器是被放置在根命名空间中的。而要把交换机放置在它们自己的命名空间中，使用**--innamespace**选项：
```bash
$ sudo mn --innamespace --switch user
```
（有自己命名空间的）交换机会单独地通过一条桥接控制链接与控制器相通信，而非使用本地环回（loopback）。然而，就命令本身的功能而言，这并不是一个很有用的命令。但这也提供了一个如何隔离不同的交换机的方法。

注意：这个选项是不会作用与Open vSwitch的。
```bash
mininet> exit
```

<h2 id="3">第三部分：Mininet命令行界面（CLI）中的命令</h2>

<h2 id="4">第四部分：Python API 例子</h2>

<h2 id="5">第五部分：看完指南了！</h2>

<h2 id="6">附录</h2>