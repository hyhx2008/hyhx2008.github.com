PeerSim使用教程(1)-入门
=========================

:date: 2013-03-20 13:40
:tags: PeerSim, P2P

最近一年所谓的研究几乎没有什么大的收获，研究的题目也一直局限在P2P，除了水文一篇，最大的收获就是学会使用PeerSim这个P2P模拟器。在这里总结一下PeerSim的使用方法，以此来结束对P2P的研究。

**1.PeerSim简介**

PeerSim是一个用java编写的P2P overlay Network的模拟器，可以模拟结构化和非结构化的P2P网络。根据模拟的方式不同，在4GB的内存情况下，支持模拟十万到千万个节点级别。其本身并没有实现任何具体的协议，但是提供了很好的扩展性，研究人员已经在其基础上实现了各种流行的P2P协议，官方网站上也提供下载。

**2.PeerSim 下载与运行**

PeerSim实际上是一个java的工程，我们可以在其官方网站上下载源代码，官网链接为:
`PeerSim: A Peer-to-Peer Simulator <http://peersim.sourceforge.net/>`_
，下载并解压得到一个名为
**peersim-1.0.5的文件夹**
。

由于PeerSim是用java编写的，学习如何时需要经常阅读源代码，所以推荐使用Eclipse来运行(当然高手也可以直接通过命令行来运行)。
这里也只说明如何在Eclipse环境下运行PeerSim。

1> 新建java工程，命名例如PeerSim_t。

2> 将peersim-1.0.5中的src文件夹内的全部内容(包括两个包，peersim和example)拖入工程的src文件夹下。

3> 将peersim-1.0.5中的三个jar包(peersim-1.0.5.jar, djep-1.0.0.jar, jep-2.3.0.jar)拖入工程并build path添加到工程内，如下图所示:

.. figure:: ../statics/pics/peersim_tutorial_1.png
	:alt: figure

(文件夹内还有一个jar包peersim-doclet.jar，添加后运行时会出错，不知道这个jar有何用处)

4> peersim-1.0.5中的example文件夹内有4个配置文件的样例，以config-example1.txt为例，将其拖入工程根目录下。

5> 找到工程中的peersim包，鼠标右键包内的Simulator.java，点击Run As -> Run Configurations，在Arguments中填入配置文件的文件名(config-example1.txt)，点击Run，就可以看到模拟器运行样例了。

至此，就可以将一个peersim模拟成功运行起来了。



