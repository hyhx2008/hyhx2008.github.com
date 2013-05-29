PeerSim使用教程(3)-编写一个新协议
=================================

:date: 2013-03-24 19:35
:tags: PeerSim

**1.编写一个新协议**

我们将要实现的协议是在PeerSim中基于Cycle-based模型的一个简单的负载均衡算法。节点的状态有两种值：本地负载(local load)和配额(quota)，其中配额是指节点在每个周期中允许传输的“负载”的大小。配额是必要的，是一个时间单元中能传输的负载上限。每个节点与和它距离最远的邻居节点交换配额值，这里“距离最远”是指与当前节点的负载差异最大。经过对比距离，协议将在负载均衡时选用push或pull的方式。

在每个周期之后，配额值将会被存储。协议并不关心拓扑管理，它依赖于其它组件来访问邻居节点(例如，Newscast，或者由IdleProtocol实现的静态拓扑)。

**1.1 必要的组件**

一般来说只编写一个协议类是不足够的，还需要一些附加的组件。例如，为了在每个周期结束时为每个节点存储配额值，需要一个特定的Control对象。基本上来说，PeerSim是一个可替换的组件集合，所以在开发时需要注意模块化以让代码尽可能重用，出于这样的目的，我们这样设计下面的类：

*protocol*

	它基于peersim.vector.SimpleValueHolder，这是一个简单的基类，用于访问一个浮点变量。Aggreation协议也使用了同样的基类。

*ResetQuota*

	用于在每个周期结束时存储每个节点配额的Control。

*QuotaObserver*

	一个control，用于监视quota参数，即覆盖网中交换的流量大小。

*initialization*

	这是在aggregation示例中的初始化器，这里也可以直接使用，因为它们实现了同样的接口SingleValueHolder。注意在example包中提供的初始化器是轻量级的，开发者应当更多地使用peersim.vector.*包中的初始化器。

*observers*

	可以使用aggreagation.AverageObserver，因为这些组件实现了相同接口。

下面将根据源代码来解释这一过程：

**1.2 负载均衡的核心类**

.. code-block:: java

	package example.loadbalance;

	import peersim.config.Configuration;
	import peersim.config.FastConfig;
	import peersim.core.*;
	import peersim.vector.SingleValueHolder;
	import peersim.cdsim.CDProtocol;

	public class BasicBalance extends SingleValueHolder implements CDProtocol {

		// ------------------------------------------------------------------------
		// Parameters
		// ------------------------------------------------------------------------	

		protected static final String PAR_QUOTA = "quota";

		// ------------------------------------------------------------------------
		// Fields
		// ------------------------------------------------------------------------	

		// Quota amount. Obtained from config property {@link #PAR_QUOTA}. 
		private final double quota_value;

		protected double quota; // current cycle quota 

		// ------------------------------------------------------------------------
		// Initialization
		// ------------------------------------------------------------------------	
	
		public BasicBalance(String prefix) {
			super(prefix);
			// get quota value from the config file. Default 1.
			quota_value = (Configuration.getInt(prefix + "." + PAR_QUOTA, 1));
			quota = quota_value;
		}

这个类需要实现peersim.cdsim.CDProtocol接口，其中最重要的是实现nextCycle()方法，这个方法包含了协议的算法。顾名思义，协议每个周期内的行为就在这个方法中描述。而且，这个协议继承了SingleValueHolder类（它是SingleValue接口的实现），为内部变量提供getter和setter方法，允许Control以可重用的方式来操作这些数据，在这个例子中，变量保存了节点的实际负载。

在构造方法中的String参数是配置组件的全名，例如，对于LoadBanlance协议来说是protocol.lb。

.. code-block:: java

		// Resets the quota.
		protected void resetQuota() {
			this.quota = quota_value;
		}

resetQuota()方法在每个周期结束时被一个Control对象调用，显然地，一个恰当的control条目应该在配置文件中出现，这里是loadbalance.ResetQu。

.. code-block:: java

		public void nextCycle(Node node, int protocolID) {
			int linkableID = FastConfig.getLinkable(protocolID);
			Linkable linkable = (Linkable) node.getProtocol(linkableID);
			if (this.quota == 0) {
				return; // quota is exceeded
			}
		// this takes the most distant neighbor based on local load
			BasicBalance neighbor = null;
			double maxdiff = 0;
			for (int i = 0; i < linkable.degree(); ++i) {
				Node peer = linkable.getNeighbor(i);
				// The selected peer could be inactive
				if (!peer.isUp())
					continue;
				BasicBalance n = (BasicBalance)peer.getProtocol(protocolID);
				if (n.quota == 0.0)
					continue; 
				
				double d = Math.abs(value - n.value);
				if (d > maxdiff) {
					neighbor = n;
					maxdiff = d;
				}
			}
	
			if (neighbor == null) {
				return;
			}
			doTransfer(neighbor);
		}
	
这个方法是由CDProtocol接口声明的，它定义了协议的行为。这里的参数代表了一个对节点自身的引用（即仿真器调用其nextCycle方法的那个节点）和正在运行的协议的protocol ID。首先我们要取得实现了Linkable接口的协议的protocol ID来访问节点的邻居节点，这可以由下面的代码来完成：

.. code-block:: java

	int linkableID = FastConfig.getLinkable(protocolID);
	Linkable linkable = (Linkable)node.getProtocol(linkableID);

使用静态类peersim.config.FastConfig我们可以取得为正在执行的协议而配置的linkable协议的protocol ID。

如果本地的配额是0，代表着这个节点已经使用完网络流量，所以直接return。

为了取得与本地节点距离最远的节点，我们循环遍历所有邻居节点的负载值；邻居节点的数量等于节点的度（这可以通过linkable接口来访问），通过linkable接口来取得节点的代码如下：

.. code-block:: java

	Node peer = linkable.getNeighbout(i);

而从这个节点就可以取得BasicBanlance协议：

.. code-block:: java

	BasicBalance n = (BasicBalance)peer.getProtocol(protocolID);

当协议寻找到一个合适的邻居节点后，调用doTransfer()方法来进行负载均衡。

.. code-block:: java

	protected void doTransfer(BasicBalance neighbor) {
		double a1 = this.value;
		double a2 = neighbor.value;
		double maxTrans = Math.abs((a1 - a2) / 2);
		double trans = Math.min(maxTrans, quota);
		trans = Math.min(trans, neighbor.quota);
	
		if (a1 <= a2)  {// PULL phase
			a1 += trans;
			a2 -= trans;
		} else{  // PUSH phase
			a1 -= trans;
			a2 += trans;
		}
	
		this.value = a1;
		this.quota -= trans;
		neighbor.value = a2;
		neighbor.quota -= trans
	}

doTransfer()方法将会在当前节点和由参数指定的邻居节点间进行实际的负载交换，它决定了在负载均衡时是用pull还是push方法：在Push的情况下，本地值增加而其它节点的值减少，在push情况下则反之。maxTrans变量是两个涉及的节点需要达到平衡而传输的负载的绝对值；由于配额(quota)是每个周期中传输的上限，这个算法将会选择quota和maxTrans中的最小值，最后两个节点都会减去相同数量的负载值。

**1.3 负载均衡的Control类代码**

.. code-block:: java

	package example.loadbalance;
	import peersim.config.*;
	import peersim.core.*;

	public class ResetQuota implements Control {
		//参数
		private static final String PAR_PROT = "protocol";

		// Value obtained from config property {@link #PAR_PROT}. 
		private final int protocolID;

		// 初始化
		public ResetQuota(String prefix) {
			protocolID = Configuration.getPid(prefix + "." + PAR_PROT);
		}

		public boolean execute() {
			for (int i = 0; i < Network.size(); ++i) {
				((BasicBalance) Network.get(i).getProtocol(protocolID)).resetQuota();
			}
			return false;
		}
	}

这段代码很简洁，部份原因是Control接口本身是很简单的，它只有一个execute方法。
构造方法利用配置文件来进行初始化。execute方法会在所有的协议上调用resetQuota方法，它通过Network类来访问协议，Network是一个只拥有静态数据域的静态类，你可将它视为是一个节点的数组。

**2.扩展协议**

这是对前面的协议的扩展。核心部份是相同的，但是算法在决定将要发送或接收多少负载时，使用了全局的负载平均而不是根据距离最远的邻居节点的负载值。
为了计算全局的负载平均值，这里有一个小技巧，虽然本来可以通过聚集来求取平均数，但我们可以通过运行一个拥有全局信息的静态方法来仿真aggregation协议，这个方法为所有节点初始化了一个全局变量，这样我们就能提升性能的同时又不损失太多的真实性。

这个协议也是为了利用newscast协议：当一个节点到达了全局负载值（平均），它将会将它的fail-state转变为DOWN，然后这个节点会从覆盖网退出，因为newscast协议会删除它。这样的影响是有多少节点达到平均负载则会减少多少个节点。

.. code-block:: java

	package example.loadbalance;

	import peersim.core.*;
	import peersim.config.FastConfig;

	public class AvgBalance extends BasicBalance {

		// The overall system average load. It is computed once by {@link #calculateAVG(int)} method.
	
		public static double average = 0.0;
	
		// This flag indicates if the average value computation has been performed
		// or not. Default is NO.	

		public static boolean avg_done = false;

		// 初始化
		public AvgBalance(String prefix) {
			super(prefix); // calls the BasicBalance constructor.
		}

		// Calculates the system average load. It is run once by the first node scheduled.

		private static void calculateAVG(int protocolID) {
			int len = Network.size();
			double sum = 0.0;
			for (int i = 0; i < len; i++) {
				AvgBalance protocol = (AvgBalance) Network.get(i).getProtocol(protocolID);
				double value = protocol.getValue();
				sum += value;
			}

			average = sum / len;
			avg_done = true;
		}

第一部份是很简单的，定义了两个全局变量，average和avg_done，其中avg_done是个用来确定不进行超过一次计算的标志。注意，虽然看起来在构造方法中定义一个计算平均值的方法是一个更优雅的方案，但这种方案是错误的！因为在构造方法运行时，并不能保证负载的分布已经定义了：那样的话全局的平均数是未定义的。

.. code-block:: java

	protected static void suspend(Node node) {
		node.setFailState(Fallible.DOWN);
	}

这个功能函数用于让节点从覆盖网中退出，这里只是简单地在Fallible接口中设置节点的状态。

.. code-block:: java

	public void nextCycle(Node node, int protocolID) {
	//  只运行一次:
		if (avg_done == false) {
			calculateAVG(protocolID);
			System.out.println("AVG only once " + average);
		}

		if (Math.abs(value - average) < 1) {
			AvgBalance.suspend(node); // switch off node
			return;
		}

		if (quota == 0)
			return; // skip this node if it has no quota
		Node n = null;

		if (value < average) {
			n = getOverloadedPeer(node, protocolID);
			if (n != null) {
				doTransfer((AvgBalance) n.getProtocol(protocolID));
			}
		} else {
			n = getUnderloadedPeer(node, protocolID);
			if (n != null) {
				doTransfer((AvgBalance) n.getProtocol(protocolID));
			}
		}

		if (Math.abs(value - average) < 1)
			AvgBalance.suspend(node);

		if (n != null) {
			if (Math.abs(((AvgBalance) n.getProtocol(protocolID)).value- average) < 1)
				AvgBalance.suspend(n);
		}
	}

nextCycle方法是核心的协议算法，它首先检查平均数，如果标志没有设置就会进行计算。
如果当前负载和平均负载的差别小于1（每个周期中固定的配额值），那么节点将会根据newcast协议从覆盖网退出；进一步地，如果配额已经使用完，将会直接return。然后，协议会检查本地的负载值是小于还是大小平均值，并分别查找负载最大和最小的邻居，最后进行交换。

.. code-block:: java

	private Node getOverloadedPeer(Node node, int protocolID) {
		int linkableID = FastConfig.getLinkable(protocolID);
		Linkable linkable = (Linkable) node.getProtocol(linkableID);
		Node neighborNode = null;
		double maxdiff = 0.0;

		for (int i = 0; i < linkable.degree(); ++i) {
			Node peer = linkable.getNeighbor(i); 

			if (!peer.isUp()) // only if the neighbor is active
	            continue;

            AvgBalance n = (AvgBalance) 
			peer.getProtocol(protocolID); 

			if (n.quota == 0)
				continue; 

			if (value >= average && n.value >= average)
				continue;

			if (value <= average && n.value <= average)
				continue; 

			double d = Math.abs(value - n.value);

			if (d > maxdiff) {
				neighborNode = peer;
				maxdiff = d;
			}
		}

		return neighborNode;
	}

	private Node getUnderloadedPeer(Node node, int protocolID) {
		int linkableID = FastConfig.getLinkable(protocolID);
		Linkable linkable = (Linkable) node.getProtocol(linkableID);
		Node neighborNode = null;
		double maxdiff = 0.0;

		for (int i = 0; i < linkable.degree(); ++i) {
			Node peer = linkable.getNeighbor(i); 

			if (!peer.isUp()) // only if the neighbor is active
				continue;
			AvgBalance n = (AvgBalance) peer.getProtocol(protocolID);
	
			if (n.quota == 0)
				continue;
			if (value >= average && n.value >= average)
				continue;
			if (value <= average && n.value <= average)
				continue;

			double d = Math.abs(value - n.value);

			if (d < maxdiff) {
				neighborNode = peer;
				maxdiff = d;
			}
		}
		return neighborNode;
	}

查找最大和最小负载的邻居节点的代码是很相似的，在这里都展示是出于完整性的缘故。

**3.协议的评估**

负载均衡协议是为了减少负载的变化 ，而变化可以使用aggregation.AverageObserver或者loadbalance.LBObserver(它们是非常相似的)来进行分析，出于这个标准，两个协议几乎拥有相同的性能，并独立于最初使用的分布。然而，AVGBalance协议相对BasicBalance来说提升了整体的负载传输，AVGBalance传输了一个可证明是最小的负载。

我们可以实现一个control来观察正被传输的负载：

.. code-block:: java

	package example.loadbalance;

	import peersim.config.*;
	import peersim.core.*;
	import peersim.util.*;

	public class QuotaObserver implements Control {

		// The protocol to operate on.
		
		private static final String PAR_PROT = "protocol";

		// The name of this observer in the configuration file.
		
		private final String name;

		/** Protocol identifier*/
		private final int pid;

		// 构造方法
		public QuotaObserver(String name) {
			this.name = name;
			pid = Configuration.getPid(name + "." + PAR_PROT);
		}

		public boolean execute() {
			IncrementalStats stats = new IncrementalStats();
			for (int i = 0; i < Network.size(); i++) {
				BasicBalance protocol = (BasicBalance) Network.get(i).getProtocol(pid);
				stats.add(protocol.quota);
		}
		/* 打印统计量*/
		System.out.println(name + ": " + stats);
		return false;
		}	
	}

原理是很简单的，在每一个仿真周期中，它收集剩余的quota并在终端上打印统计数据，从这些统计数据和配额的初始值就可以计算出已经被传输的负载。

