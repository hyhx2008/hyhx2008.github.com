PeerSim使用教程(4)-构建一个拓扑生成器
======================================

:date: 2013-04-04 21:45
:tags: PeerSim

本文描述了在PeerSim中如何构建一个新的拓扑生成器。

**1.什么是拓扑？为什么它很重要？**

在一个大型的动态P2P系统中，节点没有关于整个网络的信息，而所有的节点都可能拥有一些邻居节点，即节点能”感知”的peers，这种”感知”的关系就定义了一个覆盖网络，这是P2P系统中的一个基本概念。

很多P2P协议都需要在多个不同的网络拓扑上进行实验。PeerSim中的peersim.dynamic.Wire*类已经包含了很多拓扑结构，可以直接用来对linkable协议进行初始化，本教程将展示如何构建一个自定义的拓扑生成器。

**2.一个模拟Internet的简单模型**

下面我们将编写一个拓扑生成器来构建类似于Internet的树状拓扑，整个构建过程基于一个特定的，与位置相关的preferential attachment方法，编写规则很简单，并且会考虑几何和网络的限制以更好地模拟真实的网络。
Preferential attachment由参数a来调整，这个参数能扩大或减少几何位置所带来的影响。

这个规则的策略如下：给定一个单位正方形，将x0置于中心，即x0 = (0.5,0.5)，这个节点被称为root，令W()为与root相隔的跳数(hops)，对于i=1 … n-1，随机在单位正方形中选择一个xi，然后选择使下面的表达式值最小的节点xj来连接它：

.. figure:: ../statics/pics/peersim_tutorial_4_figure_1.png

在这里dist()是欧几里德距离而a (alpha)是权重参数，显然，

W(xi) = W(xj) + 1

通过这个方案我们得到了一个x0以为根的树。这个拓扑中每个节点(除了root外)的出度都为1,如果想更深入地理解这个模型，可以参考下面的文章：

* Heuristically Optimized Trade-offs: A New Paradigm for Power Laws in the Internet

* Degree distributions of the FKP network model

* On Power-Law Relationships of the Internet Topology

**3.需要编写的代码**

我们的目标是编写一个可以根据 a(alpha) 参数生成所需拓扑的PeerSim组件，并且能对生成的拓扑进行分析。
这个拓扑可以在仿真过程中逐步生成，也可以用一个步骤生成拓扑，在这里我们倾向后者。为了构建需要的拓扑结构，我们需要下面的组件(注意这只是其中一种方案)。

* 一个protocol 类，可以存储坐标，它不具备行为元素，只是一个普通的容器
* 一个initializer 类，可以为每个节点设置坐标值
* 一个control 类， 可在一个任意的linkable协议中根据坐标连接拓扑(在节点间添加link)
* 一个observer 类，将拓扑结构打印到一个文件中(例如用GnuPlot对图进行可视化)
* 一个observer 类，用来收集节点入度的分布的统计数据
* 一个observer 类，用来测试对随机节点失效的健壮性

在下节我们将看到，一些我们列出来的类是PeerSim中的基本组件，它们都实现了Linkable接口，Linkable以模块化的方式为用户提供了一个能处理任何拓扑结构的抽象。

**4.代码编写**

**a.Protocol类**

.. code-block:: java

	import peersim.core.Protocol;

	public class InetCoordinates implements Protocol {

		/** 2d coordinates components.*/
		private double x, y;

		public InetCoordinates(String prefix) {
			/* Un-initialized coordinates defaults to -1.*/
			x = y = -1;
		}

		public Object clone() {
			InetCoordinates inp = null;
			try {
				inp = (InetCoordinates) super.clone();
			} catch (CloneNotSupportedException e) {
			} // never happens

			return inp;
		}

		public double getX() {
			return x;
		}
	
		public void setX(double x) {
			this.x = x;
		}

		public double getY() {
			return y;
		}

		public void setY(double y) {
			this.y = y;
		}
	}

这个类只存储坐标，而links则会存储在其它任意实现了Linkable接口的协议中。

clone方法必须新定义并且捕捉和压制所有的异常（它们永远不会被抛出），因为这里只有基本类型，不需要深拷贝操作。

坐标组件并不是public的，但可以通过getter/setter方法来存取 ，这是很重要的，因为我们可以使用peersim.vector包以一个弹性化的方式来初始化坐标值，但在本文中我们并没有使用这个包。

**b.初始化类**

.. code-block:: java

	package example.hot;
	import peersim.config.Configuration;
	import peersim.core.CommonState;
	import peersim.core.Control;
	import peersim.core.Network;
	import peersim.core.Node;
	public class InetInitializer implements Control {

		private static final String PAR_PROT = "protocol";

		/** Protocol identifier, obtained from config property*/
		private static int pid;

		public InetInitializer(String prefix) {
			pid = Configuration.getPid(prefix + "." + PAR_PROT);
		}

		/**
		* Initialize the node coordinates. The first node in the Network
		* is the root node by default and it is located in the middle
		* (the center of the square) of the surface area.*/
		public boolean execute() {
			// Set the root: the index 0 node by default.
			Node n = Network.get(0);
			InetCoordinates prot = (InetCoordinates) n.getProtocol(pid);
			prot.setX(0.5);
			prot.setY(0.5);
			// Set coordinates x,y
			for (int i = 1; i < Network.size(); i++) {
				n = Network.get(i);
				prot = (InetCoordinates) n.getProtocol(pid);
				prot.setX(CommonState.r.nextDouble());
				prot.setY(CommonState.r.nextDouble());
			}
			return false;
		}
	}

初始化类应当实现Control接口中唯一的execute方法，构造方法从配置文件中读取唯一的参数(protocol)，它声明了持有坐标的协议。

这个类是很简单的，它生成了一致的随机坐标(x和y),唯一的例外是root节点，默认情况下，它的下标是0，固定为(0.5,0.5)。

为了生成随机数，CommonState中的静态的数据域r必须总是使用，因为这样保证了实验的可重复性。

**c.Wiring类**

这个类继承了peersim.dynamic.WireGraph，它实现了Control接口并提供了处理拓扑的通用功能，同时也提供了一个图的接口。
wiring的逻辑应该放在由子类调用的wire方法中，并且在默认情况下将下标为0的节点视为root。

这个类需要从配置文件中读取 a(配置文件中的alpha)和坐标容器的protocol ID(配置文件中的coord_protocol)，这是由类的的构造方法来完成的，其它的参数，比如 protocol 是由父类继承的，它是一个实现了Linkable接口的协议。

.. code-block:: java

	package example.hot;

	import peersim.config.Configuration;
	import peersim.core.Linkable;
	import peersim.core.Network;
	import peersim.core.Node;
	import peersim.dynamics.WireGraph;
	import peersim.graph.Graph;

	public class WireInetTopology extends WireGraph {

		private static final String PAR_ALPHA = "alpha";
		private static final String PAR_COORDINATES_PROT = "coord_protocol";

		// A parameter that affects the distance importance.
		private final double alpha;

		// Coordinate protocol pid.
		private final int coordPid;

		public WireInetTopology(String prefix) {
			super(prefix);
			alpha = Configuration.getDouble(prefix + "." + PAR_ALPHA, 0.5);
			coordPid = Configuration.getPid(prefix + "." + PAR_COORDINATES_PROT);
		}

		/**
		* Performs the actual wiring.
		* @param g
		* a peersim.graph.Graph interface object to work on.*/																														
		public void wire(Graph g) {

			// Contains the distance in hops from the root node
			// for each node. 

			int[] hops = new int[Network.size()];

			// connect all the nodes other than roots
			for (int i = 1; i < Network.size(); ++i) {
				Node n = (Node) g.getNode(i);
				// Look for a suitable parent node between those
				// allready part of the overlay topology: alias
				// FIND THE MINIMUM!
				// Node candidate = null;
				int candidate_index = 0;
				double min = Double.POSITIVE_INFINITY;

				for (int j = 0; j < i; j++) {
					Node parent = (Node) g.getNode(j);
					double jHopDistance = hops[j];
					double value = jHopDistance +
						(alpha * distance(n, parent, coordPid));
					if (value < min) {
					// candidate = parent;
					// best parent node to connect to
						min = value;
						candidate_index = j;
					}
				}
				hops[i] = hops[candidate_index] + 1;
				g.setEdge(i, candidate_index);
			}
		}

		private static double distance(Node new_node,Node old_node, int coordPid) {
			double x1 = ((InetCoordinates) new_node.getProtocol(coordPid)).getX();
			double x2 = ((InetCoordinates) old_node.getProtocol(coordPid)).getX();
			double y1 = ((InetCoordinates) new_node.getProtocol(coordPid)).getY();
			double y2 = ((InetCoordinates) old_node.getProtocol(coordPid)).getY();

			if (x1 == -1 || x2 == -1 || y1 == -1 || y2 == -1)

			// NOTE: in release 1.0 the line above incorrectly

			// contains | -s instead of ||. Use latest CVS version,
			
			// or fix it by hand.
				throw new RuntimeException(
					"Found un-initialized coordinate. Use e.g.,\
					InetInitializer class in the config file.");

			return Math.sqrt((x1 - x2) * (x1 - x2)
						+ (y1 - y2) * (y1 - y2));
		}
	}

**d.Observers**

前面提到的observer有些已经由PeerSim提供了相应的实现。例如，为了计算节点度的分布，用户可以使用peersim.reports.DegreeStats；
为了检验网络的健壮性，可以使用peersim.reports.RandRemoval：它打印生成的clusters的数目及大小，并作为随机删除的节点数量的函数。
然而，为了将拓扑转换为可绘图的形式，需要自行编写observer：
InetObserver实现了Control接口和对应的execute方法，我们继承了peersim.reports.GraphObserver，这个模板类能简化对图的观察。

构造方法根据配置文件进行初始化，其中参数protocol引用了Protocol ID，它拥有”who knows who”的关系（它必须是一个Linkable 协议），这是由超类继承而来。
其它的参数，coord_protocol和file_base，分别是坐标容器的协议名和将要使用的文件名前缀。
这样，最终由程序生成的文件名是：file_base + %08d + .dat，中间的8位数字是指周期数，
因为作为一个control对象，observer可以在每个周期中运行，在这种情况下每次应该生成一个不同的文件。

.. code-block:: java

	package example.hot;

	import java.io.FileOutputStream;
	import java.io.IOException;
	import java.io.PrintStream;
	import peersim.config.Configuration;
	import peersim.core.Node;
	import peersim.graph.Graph;
	import peersim.reports.GraphObserver;
	import peersim.util.FileNameGenerator;

	public class InetObserver extends GraphObserver {

		private static final String PAR_FILENAME_BASE = "file_base";
		private static final String PAR_COORDINATES_PROT
						= "coord_protocol";

		private final String graph_filename;
		private final FileNameGenerator fng;
		private final int coordPid;

		public InetObserver(String prefix) {
			super(prefix);
			coordPid = Configuration.getPid(prefix + "."
					+ PAR_COORDINATES_PROT);
			graph_filename = Configuration.getString(prefix + "."
					+ PAR_FILENAME_BASE, "graph_dump");
			fng = new FileNameGenerator(graph_filename, ".dat");
		}

		// Control interface method.
		public boolean execute() {
			try {
				updateGraph();
				System.out.print(name + ": ");
				// initialize output streams
				String fname = fng.nextCounterName();
				FileOutputStream fos = new FileOutputStream(fname);
				System.out.println("Writing to file " + fname);
				PrintStream pstr = new PrintStream(fos);
				// dump topology:
				graphToFile(g, pstr, coordPid);
				fos.close();
			} catch (IOException e) {
				throw new RuntimeException(e);
			}
			return false;
		}

		private static void graphToFile(Graph g, PrintStream ps,int coordPid) {
			for (int i = 1; i < g.size(); i++) {
				Node current = (Node) g.getNode(i);
				double x_to = ((InetCoordinates)
					current.getProtocol(coordPid)).getX();
				double y_to = ((InetCoordinates)
					current.getProtocol(coordPid)).getY();
	
				for (int index : g.getNeighbours(i)) {
					Node n = (Node) g.getNode(index);
					double x_from = ((InetCoordinates)
						n.getProtocol(coordPid)).getX();
					double y_from = ((InetCoordinates)
						n.getProtocol(coordPid)).getY();
					ps.println(x_from + " " + y_from);
					ps.println(x_to + " " + y_to);
					ps.println();
				}
			}
		}
	}

在execute方法中我们必须调用 updateGraph方法(a GraphObserver protected method)以检验实际的图中是否发生了变化，
这是为了在很多observer运行于同一个图中的时候节省构造图的时间。如果许多observers观察同一个图的无向版本，那节省时间的时间是很显著的。
注意在execute方法中使用的IO库函数可能抛出一些异常，这里任意的异常都被捕获并重新作为运行时异常抛出，它们会导致仿真的终止。

静态的功能方法graphToFile将实际的拓扑结构写到磁盘中，对于每个节点n，收集其x和y坐标，而对于节点n的每个邻居节点i，其坐标将会是下面的格式：

1> n.neighbor(i).x n.neighbor(i).y \newline

2> n.x n.y \newline

3> \newline}

这种格式很适合于用GnuPlot来绘图，请注意循环是从下标1而不是0开始的，这是因为节点0是root,它没有向外的连接。

**5.运行实验**

下面是本实验所对应的配置文件：

.. code-block:: c

	# Complex Network file:
	# random.seed 1234567890
	simulation.cycles 1
	network.size 10000
	protocol.link IdleProtocol
	protocol.coord example.hot.InetCoordinates
	init.0 example.hot.InetInitializer
	init.0.protocol coord
	init.1 example.hot.WireInetTopology
	init.1.protocol link #the linkable to be wired
	init.1.coord_protocol coord
	init.1.alpha 4
	control.io example.hot.InetObserver
	control.io.protocol link
	control.io.coord_protocol coord
	control.io.file_base graph
	control.degree DegreeStats
	control.degree.protocol link
	control.degree.undir
	control.degree.method freq
	include.control io degree

.. figure:: ../statics/pics/peersim_tutorial_4_figure_2.png

Figure 1: Topology and in-degree distribution with a 4

.. figure:: ../statics/pics/peersim_tutorial_4_figure_3.png

Figure 2: Topology and in-degree distribution with a 20

.. figure:: ../statics/pics/peersim_tutorial_4_figure_4.png

Figure 3: Topology and in-degree distribution with a 100

它根据init.0部分的参数生成了具有10000个节点的覆盖网络。下面的图展示了在a不同的情况下生成的拓扑。
事实上，它影响了系统的聚类行为并且它与网络的大小紧密相关:
如果，拓扑将变得越来越聚集，在a很小时，则拓扑会变成星形结构。 如果，拓扑将趋向于随机分布而不是聚集在一起。

DegreeStats可以用来收集节点度的统计信息，然而，应当慎重地使用它。
因为在PeerSim的默认情况下，“度”是指“出度”，然而我们感兴趣的是“入度”，那怎么样才能观察入度呢？
首先我们将图视为无向图（通过undir参数）,然后我们进行频率统计（freq参数）来绘图，observer会输出类似于下面的数据：

.. code-block:: console

	1 9838
	2 38
	3 19
	4 14
	5 7
	6 7
	7 7
	8 4
	9 3
	10 3
	11 1
	12 5
	...
	...
	543 1
	566 1
	620 1
	653 1
	2153 1

第一列对应于度数，而第二列是指拥有相应度数的节点数量，我们可以肯定除了root以外，其它每个节点都只有一个out-link，同时所有的link都是严格单向的，因而为了取得入度我们只需要从第一列简单地减去1即可。
