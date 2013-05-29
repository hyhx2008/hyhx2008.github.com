PeerSim使用教程(5)-Event-drive(事件驱动)模型
=============================================

:date: 2013-04-12 17:46:00
:tags: PeerSim

**1.简介**

本教程使用Event-driven模型来演示一个简单的例子，仍然使用的是gossip-based平均数协议，对消息的发送将进行更细节的建模；通过与cycle-based模型的对比，可以发现本协议存在的问题。

在Event-based模型中，除了时间管理和control传递给protocols的方式以外，其它与cycle-based模型相同。
不可执行的Protocols(只用于存储数据，比如只存储邻居节点的linkable协议，或存储数值的vectors)可以以同样的方式应用和初始化，在peersim.cdsim包之外的controls也都可以使用。
在默认情况下，在cycle-based模型中，controls会的每个周期中调用 ，但在event-based模型中，它们需要进行明确的调度，因为事件驱动模型并不存在周期的概念。 

显然，我们可以编写专用于Event-based模型的controls，即可以给协议发送事件(消息)。在很多情况下，这是必要的，因为系统经常完全或部分地由外部事件如用户的查询来驱动，这能很好地用由生成这些事件，并且驱动仿真执行的controls进行建模。

有些组件是不可用的。例如依赖于静态类peersim.cdsim.CDState(它提供了读取cycle相关的全局状态的接口)的所有组件。
我们的经验是，很多依赖于这个状态的cycle-based 组件可以经过简单的修改并删除这个依赖。

然而，可能有些令人吃惊的是，实现了cycle-based接口的peersim.cdsim.CDProtocol也可以使用于event-based模型，但是必须指出，在大部份的情况下这样做没有什么意义。
然而，这个特性的有用之处在于：它让周期性地调用protocols变得很简单，这是一个几乎对所有与housekeeping，失效检测和sending heartbeat message有关的P2P协议来说典型的特性。

**2.the Protocol**

下面以event-based模型来实现平均数协议：

.. code-block:: java

	package example.edaggregation;

	import peersim.vector.SingleValueHolder;
	import peersim.config.*;
	import peersim.core.*;
	import peersim.transport.Transport;
	import peersim.cdsim.CDProtocol;
	import peersim.edsim.EDProtocol;

	/**
	* Event driven version of epidemic averaging.*/

	public class AverageED extends SingleValueHolder
		implements CDProtocol, EDProtocol {

首先，这里同时实现了EDProtocol和CDProtocol接口，前者能让这个类能处理输入的消息，后者则可能令人困惑，因为它属于Cycle-based模型的接口。
但注意Event-based的协议不是必须实现CDProtocol接口的，然而想要实现一个可以周期性取得control的协议时，可以通过实现CDProtocol接口，并在配置文件中设置一个CDScheduler 来实现。
这样，代码就显得更清晰：以一个单独的组件来管理周期性的执行，而且能单独地进行配置。最后，我们还能简单地将 Cycle-based 的协议移植到 Event-based 模型上。

.. code-block:: java

	/**
	* @param prefix string prefix for config properties*/
	
	public AverageED(String prefix) {
		super(prefix);
	}

这个简单的协议并不读取任何配置参数，现在来关注cycle-based接口的实现，这个方法定义了周期性进行的行为。

.. code-block:: java

	/**
	* This is the standard method the define periodic activity.
	* The frequency of execution of this method is defined by a
	* {@link peersim.edsim.CDScheduler} component in the configuration.*/
	
	public void nextCycle( Node node, int pid ) {
		Linkable linkable =
			(Linkable) node.getProtocol( FastConfig.getLinkable(pid) );

		if (linkable.degree() > 0) {
			Node peern = linkable.getNeighbor(
				CommonState.r.nextInt(linkable.degree()));

			// XXX quick and dirty handling of failures
			// (message would be lost anyway, we save time)

			if(!peern.isUp())
				return;

			AverageED peer = (AverageED) peern.getProtocol(pid);

			((Transport)node.getProtocol(FastConfig.getTransport(pid))).
			send(node,peern,new AverageMessage(value,node),pid);
		}
	}

在这里要观察的是event-based模型中的方法，即如何处理传输层。
首先，FastConfig类让我们能访问为这个协议配置的传输层，通过使用这个传输层，我们可以在其它节点上将消息发送给协议。
一条消息可以是任意的对象 ，由于这个仿真器并不是分布式的，所以不用处理序列化等等问题；而对象则会通过引用来存储。
目标协议是由目标节点peern定义的，协议的标识符则是pid，在这个例子中，我们在一个不同的节点发送消息给同一个协议。
显然，目标协议应当实现EDProtocol接口。

.. code-block:: java

		/**
		* This is the standard method to define to
		* process incoming messages.*/

		public void processEvent( Node node, int pid, Object event ) {
			AverageMessage aem = (AverageMessage)event;

			if( aem.sender!=null )
				((Transport)node.getProtocol(FastConfig.getTransport(pid))).
				send(node,aem.sender,
					new AverageMessage(value,null),pid);

			value = (value + aem.value) / 2;
		}
	}

上面实现的是EDSimulator中的方法，它用于处理进入的消息
。本例中，消息只有一种类型，我们只需要检查sender是否为null，因为如果为null，
则我们无需应答消息(当然，实际上这已经是一种“应答”)，而如果需要应答时则是通过传输层来处理。

.. code-block:: java

	/**
	* The type of a message. It contains a value of type double
	* and the sender node of type {@link peersim.core.Node}.*/
	class AverageMessage {
		final double value;

		/** If not null,
		/*  this has to be answered, otherwise this is the answer.*/
		final Node sender;

		public AverageMessage( double value, Node sender ) {
			this.value = value;
			this.sender = sender;
		}
	}

这个私有类是协议所使用的消息类型，说其私有是因为没有任何其它组件需要处理这种类型的消息。

**3.配置文件**

配置文件与Cycle-based的配置相似，只有很少地方不同，但这些区别很重要。

.. code-block:: c

	# network size
	SIZE 1000
	# parameters of periodic execution
	CYCLES 100
	CYCLE SIZE*10000
	# parameters of message transfer
	# delay values here are relative to cycle length, in percentage,
	# eg 50 means half the cycle length, 200 twice the cycle length, etc.
	MINDELAY 0
	MAXDELAY 0
	# drop is a probability, 0<=DROP<=1
	DROP 0

这里定义了一些常量以让配置文件更清晰，同时也更易于在命令行中修改，例如，CYCLE定义了一个周期的长度。

.. code-block:: c

	random.seed 1234567890
	network.size SIZE
	simulation.endtime CYCLE*CYCLES
	simulation.logtime CYCLE

在这里，simulation.endtime 是最关键的参数，它通知仿真器何时终止。
PeerSim用一个64位的long 长整型来表示时间，在启动时它为0，并且由消息的延迟来推进。
在事件队列为空，或者队列中所有的事件的调度时间都迟于终止时间时，仿真将会终止。

仿真器将在标准错误窗口打印时间的进度，simulation.logtime 指定了打印这些消息的频率。

.. code-block:: c

	################### protocols ===========================
	protocol.link peersim.core.IdleProtocol
	protocol.avg example.edaggregation.AverageED
	protocol.avg.linkable link
	protocol.avg.step CYCLE
	protocol.avg.transport tr
	protocol.urt UniformRandomTransport
	protocol.urt.mindelay (CYCLE*MINDELAY)/100
	protocol.urt.maxdelay (CYCLE*MAXDELAY)/100
	protocol.tr UnreliableTransport
	protocol.tr.transport urt
	protocol.tr.drop DROP

在这里我们配置了协议(avg)，指定了覆盖网络(link)和传输层(tr)，同时也需要指定step参数，这与cycle-based模型是相似的。
这是由于我们实现了cycle-based接口，所以我们需要指定一个周期的长度以使用它。

覆盖网络只是一个links的容器，在仿真过程中会保持不变，它会按如下初始化：
传输层也被视为一个协议进行配置的，它对随机延迟和消息丢失进行了建模。
首先我们定义了一个拥有随机延迟（urt）的传输层，然后将它封装在一个通用的传输层包装器中，并以给定的概率tr来丢弃消息。
传输层被定义在peersim.transport包中，和其它组件一样，它也是模块化的，用户可以简单地开发和使用一个新的传输层协议。

.. code-block:: c

	################ control ==============================
	control.0 SingleValueObserver
	control.0.protocol avg
	control.0.step CYCLE

注意和协议avg一样，我们也需要指定step参数，它指定了这个control多久会调用一次，
否则controls能像cycle-based模型那样进行调度，只是没有默认的step，因为这里没有周期。

**4.运行协议**

如果我们用上面的配置文件运行仿真，我们将在标准错误窗口中得到输出：

.. code-block:: console

	如果我们用上面的配置文件运行仿真，我们将在标准错误窗口中得到输出：

	Simulator: loading configuration
	ConfigProperties: File config-edexample.txt loaded.
	Simulator: starting experiment 0 invoking peersim.edsim.EDSimulator
	Random seed: 1234567890
	EDSimulator: resetting
	Network: no node defined, using GeneralNode
	EDSimulator: running initializers
	- Running initializer init.rndlink: class peersim.dynamics.WireKOut
	- Running initializer init.sch: class peersim.edsim.CDScheduler
	- Running initializer init.vals: class peersim.vector.LinearDistribution
	EDSimulator: loaded controls [control.0]
	Current time: 0
	Current time: 10000000
	Current time: 20000000
	Current time: 30000000
	Current time: 40000000
	Current time: 50000000
	.
	.
	.
	Current time: 980000000
	Current time: 990000000
	EDSimulator: queue is empty, quitting at time 999980413

标准输出窗口的输出如下 :

.. code-block:: console

	control.0: 1.0 1000.0 1000 500.5 83416.66666666667 1 1
	control.0: 37.5 919.0 1000 500.5 25724.159091250687 1 1
	control.0: 206.7109375 767.890625 1000 500.5 8096.807036889389 1 1
	control.0: 352.373046875 695.453125 1000 500.5 2578.022573176135 1 1
	control.0: 412.430419921875 625.474609375 1000 500.5 801.1082179446831 1 1
	control.0: 436.43787479400635 570.459858417511 1000 500.5 243.53994072762902 1 1
	control.0: 470.7608990445733 527.0359845032217 1000 500.49999999999994 74.13788674564383 1 2
	control.0: 483.6040476858616 518.0301055684686 1000 500.49999999999903 23.428974301677556 1 1
	control.0: 490.5196089811798 512.0301471857779 1000 500.4999999999993 7.285566419597019 1 1
	control.0: 494.97216907397836 506.0375954180854 1000 500.4999999999999 2.1798299307442246 1 1
	control.0: 497.18190345272336 503.5837144460532 1000 500.5000000000001 0.6073148838336206 1 1
	control.0: 498.54320551492475 502.3533156558903 1000 500.5 0.1786794435445898 1 2
	control.0: 499.4023441821402 501.4962048486104 1000 500.49999999999966 0.055257607540637785 1 1
	control.0: 500.0032071191514 501.09832936709677 1000 500.4999999999995 0.017914865984002482 1 1
	.
	.
	.
	control.0: 500.5 500.5 1000 500.5 0.0 1000 1000

这些值分别代表最小值，最大值，样本的数量，平均数，方差，最小值实例的数量，最大值实例的数量。
这个输出代表已经找到正确的平均数500.5，方差为 0，即所有的节点都拥有相同的正确的平均值。

这看上去很不错，我们可以添加一些延迟并观察一下会发生什么(在默认的配置文件中delay是0)，
如果在命令行中添加MINDELAY=10和MAXDELAY=10（即表示所有的消息都会恰好延迟10%的周期时间），我们将会得到：

.. code-block:: console

	.
	.
	.
	control.0: 499.126081326076 499.126081326076 1000 499.12608132608807 0.0 1000 1000
	control.0: 499.126081326076 499.126081326076 1000 499.12608132608807 0.0 1000 1000
	control.0: 499.126081326076 499.126081326076 1000 499.12608132608807 0.0 1000 1000

也就是说，简单的延迟方案已经破坏了这个协议的良好属性：尽管是收敛的，但结果不正确，还可以验证不同的随机种子将会导致不同的结果，而改变延迟的和丢失率也对性能有影响。

那么，将延迟和丢失率保持为0能保证得到正确的行为吗？不一定。
下面用更短的周期长度来进行实验，例如，CYCLE=SIZE，这意味着在同一个时间点一般有更多的事件调度发生，
在这样的情况下，PeerSim 以随机的顺序来执行这些事件，我们将得到：

.. code-block:: console

	.
	.
	.
	control.0: 500.4835099381911 500.4835099381911 1000 500.48350993818605 7.807634196601234E-9 1000 1000
	control.0: 500.4835099381911 500.4835099381911 1000 500.48350993818605 7.807634196601234E-9 1000 1000
	control.0: 500.4835099381911 500.4835099381911 1000 500.48350993818605 7.807634196601234E-9 1000 1000

结论是什么？必须指出，这个错误的结果是由于时间分辨率(time resolution)不足导致的。
如果消息的确是零延迟的，那么执行时间的微小差别也会导致不重叠的成对交换，很明显，在连续的时间上没有事件会在相同的时间点执行。
然而，在消息传送时微小的随机延迟让结果变得很有意义，因为顺序的不确定性是确实存在的。

大体而言，使用event-based模型能看到很多在cycle-based模型中看不到的问题，然而，这也会引入一些假象。

**5.声明**

这个例子仅是为了入门，推荐进一步学习相关文档，如peersim.edsim, peersim.transport等。


