---
layout: post
title: 使用Mininet搭建环形拓扑
description: Mininet是一个可以在有限资源的普通电脑上快速建立大规模SDN原型系统的网络仿真工具。
category: blog
---

[Mininet][]是一个可以在有限资源的普通电脑上快速建立大规模SDN原型系统的网络仿真工具。该系统由虚拟的终端节点（End-Host）、OpenFlow交换机、控制器（也支持远程控制器）组成，这使得它可以模拟真实网络，可对各种想法或网络协议等进行开发验证。目前Mininet已经作为官方的演示平台对各个版本的OpenFlow协议进行演示和测试。Mininet是基于Linux Container这一内核虚拟化技术开发出的进程虚拟化平台，主要运用了Linux内核的namespace机制。

![sudo mn](/images/sdn/sudo mn.png)
Mininet在/mininet/node.py中定义了Node、OVSSwitch和Controller等重要的类。在OVSSwitch类中，通过使用命令OVS的命令创建交换机从而得到一个OVS实例。Mininet创建的host，switch等实例实际上是运行在不同namespace下的某个进程。默认情况下Host运行在自己的namespace中，交换节点运行在root namespace中。
mininet作为一个轻量级软定义网络研发和测试平台，其主要特性包括

<ul>
    <li>支持Openflow、OpenvSwitch等软定义网络部件</li>
    <li>方便多人协同开发</a></li>
    <li>支持系统级的还原测试</li>
    <li>支持复杂拓扑、自定义拓扑</li>
    <li>提供python API</li>
    <li>很好的硬件移植性（Linux兼容），结果有更好的说服力</li>
    <li>高扩展性，支持超过4096台主机的网络结构</li>
</ul>

当然他也有缺点：

* 只是一个在计算机上进行网络拓扑搭建仿真的工具

大致介绍到此，对于资源比较短缺又想设计模拟真实网络拓扑的人员来说，Mininet确实是一个比较不错的选择。

---
##要完成的目标
用mininet搭建一个有环网络，使用Pox控制器实现网络中主机互相通信（避免环路影响）。
![item](/images/sdn/item.png)
## 配置和使用Mininet
考虑到mininet虚拟实验只需要在一台电脑上运行即可，所以没有选择[SDN创新实验平台]来做，而是在自己电脑上通过装Vmware Workstation+Ubuntu+Mininet+Pox的方式进行实验，这样也方便修改拓扑结构的源码或自定义拓扑。
安装虚拟机和Ubuntu的过程不是实验重点，不再赘述。安装mininet采用了本地安装Mininet源代码的方式：
    $ git clone git://github.com/mininet/mininet  
   【获取源代码】
    $ mininet/util/install.sh –a  
   【安装所有工具，包括OpenvSwitch、Wireshark、POX】
   由于采用了安装所有工具的方法，因此省去了单独安装POX控制器的步骤.
    $sudo mn  
   默认拓扑测试后验证安装成功。
![sudomn success](/images/sdn/sudomn success.png)
## 搭建自定义拓扑
 mininet提供了python api，可以用来方便的自定义拓扑结构，在mininet/custom目录下给出了一个例子：topo-2sw-2host.py。
 该文件中定义了一个mytopo，则可以通过–topo选项来指定使用这一拓扑。实验时先对官方给的2sw-2host类型的自定义拓扑进行了学习，
 之后在custom文件夹下新建了一个实验拓扑，考虑到是环路拓扑，所以取名MyRingTopo.py。
![custom](/images/sdn/custom.png)
  自定义拓扑MyRingTopo.py的文件代码如下：
  ![topo](/images/sdn/topo.png)
## 开启Mininet和POX控制器进行测试
实验要求实现网络中主机互相通信（避免环路影响），在做实验前经过查资料了解到环路广播风暴，如果有环路，数据帧将会在环路中来回传递，大量增生数据帧，形成广播风暴，从而导致正常的通信过程不能继续。在SDN中部分控制器可以抑制环路广播风暴的产生，如Floodlight，而像POX控制器，不能很好的解决广播风暴的问题，但对于本次实验的简单拓扑依然可以解决。
环路通信常常是在现实环境下出于对通信稳定的保障而设置的备用连线，确有其实用价值，在物理拓扑上必不可少，逻辑上则要采取控制来抑制由于环路导致的可能出现的广播风暴等问题。而mininet本身只是一个虚拟网络仿真工具，并不能对出现的环路广播风暴等问题进行控制，因此需要相应的控制器添加处理规则等方式来确保环路通信的正常运转。
### 1.开启自定义拓扑：

![starttopo](/images/sdn/starttopo.png)
 
指定远端控制器ip为127.0.0.1，端口号为6666。生成的拓扑结果如下：
![ettopo](/images/sdn/ettopo.png)
打印拓扑节点信息：
![printtopo](/images/sdn/printtopo.png)
所有的网络接口：
![intfs](/images/sdn/intfs.png)
注：实验时先在没有添加任何抑制广播风暴的规则下进行测试，发现环路不能ping通：
![failconnect](/images/sdn/failconnect.png)
且pox控制台会不停输出错误信息：
![errinfo](/images/sdn/errinfo.png)
通过查资料了解到POX中的openflow.spanning_tree组件可以解决这个问题:该组件使用discovery组件来创建网络拓扑的视图，构造一棵生成树，然后使不在生成树中的交换机端口的洪泛功能失效，使得网络中不存在洪泛回路。虽然提到了生成树，但是该组件同生成树协议几乎没有关系，只是有相似的目的。要使用的两个选项的意义如下：
    $--no-flood
   只要交换机连接上了就使该交换机的所有端口洪泛失效，对于某些端口，稍后将使能。
    $--hold-down
   防止洪泛控制在一个完整的发现回路完成前被改变
因此输入以下命令使用该组件能在一定程度上抑制广播风暴。
    $ openflow.spanning_tree --no-flood --hold-down 
### 2.开启POX：
![startpox](/images/sdn/startpox.png)
注释：<ul>
      <li>l2_learning属于forwarding组件包，使得OpenFlow交换机的行为如同一个二层的具有自学习功能的交换机该组件，实现 L2链路层上的地址学习。</li>
      <li>openflow.of_01组件的功能是是同openflow 1.0协议版本的交换机进行通讯，默认启动。
        $--port=6666指定该POX端口为6666。</li>
      <li>spanning_tree组件依赖discovery组件，，同属OpenFlow组件包。</li>
   </ul>
   
###3.进行测试：
 在配置完POX的相应组件和规则后，对原有的自定义拓扑进行了测试:
所有节点之间互ping结果：
![ping](/images/sdn/ping.png)
pingpair测试结果：
![pingpair](/images/sdn/pingpair.png)
h1和h4带宽测试结果：
![iperf](/images/sdn/iperf.png)
h1和h4指定带宽udp测试结果：
![iperfudp](/images/sdn/iperfudp.png)
关闭Mininet时mininet控制台输出：
![exit](/images/sdn/exit.png)
关闭Mininet时POX控制台输出：
![exitpox](/images/sdn/exitpox.png)

---
##结语
在解决环状拓扑广播风暴的问题上，刚开始只是知道广播风暴形成的原因，却不知道怎么解决。后来对POX的组件和功能进行了仔细的了解，找出了相应的解决办法。不过POX的操作比起Mininet的操作相对麻烦些，期间好多次操作错误后一遍又一遍的重启控制器。
环状拓扑仿真对网络拓扑的理解要求更高，并且利用到了Mininet和POX控制器。
操作期间参考了黄韬等老师编著的[《软件定义网络-核心原理与应用实践》]、[Milestone]和[SDNLAB]上的资料，在此表示感谢。
如果你跟着这篇不那么详尽的教程，成功搭建了Mininet环境并进行了环形网络拓扑仿真，恭喜你！剩下的就是保持热情去研究吧。
[X-Flowing]: http://xff2016.club  "X-Flowing"
[Mininet]:   http://mininet.org/ "Mininet"
[《软件定义网络-核心原理与应用实践》]:http://book.douban.com/subject/26184169/ "《软件定义网络-核心原理与应用实践》"
[Milestone]:  http://www.muzixing.com "Milestone"
[SDNLAB]:  http://www.sdnlab.com/ "SDNLAB"
[SDN创新实验平台]:http://fnlab.org/"SDN创新实验平台"
[Github Pages]: http://pages.github.com/ "Github Pages"
[Godaddy]:  http://www.godaddy.com/ "Godaddy"
[Jekyll]:   https://github.com/mojombo/jekyll "Jekyll"
[DNSPod]:   https://www.dnspod.cn/ "DNSPod"
[Disqus]: http://disqus.com/
[多说]: http://duoshuo.com/
[1]:    {{ page.url}}  ({{ page.title }})

