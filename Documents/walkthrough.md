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
    + [调整信息等级](#2.4)
    + [自定义网络拓扑](#2.5)
    + [ID = MAC](#2.6)
    + [用XTerm进行显示](#2.7)
    + [其他交换机类型](#2.8)
    + [Mininet Benchmark](#2.9)
    + [Namespace中的一切](#2.10)
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



<h2 id="3">第三部分：Mininet命令行界面（CLI）中的命令</h2>

<h2 id="4">第四部分：Python API 例子</h2>

<h2 id="5">第五部分：看完指南了！</h2>

<h2 id="6">附录</h2>