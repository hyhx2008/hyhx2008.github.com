PeerSim使用教程(2)-解析Cycle-based模式仿真
===========================================

:date: 2013-03-21 20:10
:tags: PeerSim, Cycle-based

PeerSim提供的文档并不多，一共三篇，可以在其官网上下载，这里给一个链接:
`PeerSim: Documentation <http://peersim.sourceforge.net/#docs>`_
，是学习PeerSim必看资料。

英文有时不是很好理解，网上有个哥们将其翻译成中文了，不过他的网页最近好像失效了，这里和后面的教程基本上是整理了一下他的翻译，同时也感谢那位哥们的工作。

本文介绍了PeerSim的基本概念，并解析了两个示例以更清晰地说明PeeSim的仿真流程。

Peersim支持两种仿真模式，即Cycle-based模型和传统的Event-based模型，本文专注于前者的讲解。

Cycle-based模型是一个简化的模型，拥有更好的伸缩性及性能，在拥有4GB内存的情况下，Event-driven模式目前最多支持十万节点级别，而Cycle-based模式则支持千万个节点级别。 但是Cycle-based模型缺少对传输层的仿真和并行处理，节点之间是直接通信的，仿真核心以一定的顺序周期性地给以节点控制。在运行时，可以进行任意的操作，如调用其它对象的方法并执行一些计算。

Cycle-based模型损失了一些真实性，虽然一些简单的协议可以忽略这些差别，但是在选择使用这个模型时，需要注意这些区别。我们可以相对简单地将Cycle-based的仿真移植到Event-driven引擎上，但在本文中不讨论这个话题。

**1.PeerSim仿真的生命周期**

PeerSim鼓励基于接口的模块化编程，每一个组件都能被其它实现了相同接口的组件代替，一般的仿真过程如下：

	1> 选择网络大小(即节点数量)。

	2> 选择要实验的一个或多协议并进行初始化。

	3> 选择一个或多个Control对象来监视感兴趣的属性，并在仿真时修改一些参数(比如，网络大小，协议的内部状态，等等)。

	4> 根据配置文件，调用Simulator类运行仿真。

在仿真时创建的对象都是实现了一个或多个接口的类的实例，主要的接口如下表所示：

.. figure:: ../statics/pics/peersim_tutorial_2_table_1.png
	:align: center
	:alt: figure

Cycle-based仿真的生命周期是这样的：

	1> 读取配置文件（通过命令行参数传递进来），然后仿真器初始化网络中的节点和节点中的协议，每个节点都拥有相同的协议栈。节点和协议的实例是通过克隆来创建的，只有一个原型是通过构造方法创建，其它的节点和协议都是从这个原型中克隆而来。基于这个原因，协议类中clone方法的实现是很重要的。

	2> 初始化操作，设置每个协议的初始状态。初始化阶段是由一个或多个Control对象控制运行的，仅在实验开始时运行一次。在配置文件中，初始化的组件可以由init前缀识别，在下面讨论的initializer对象也是Controls，但为了标记其功能以区别于一般的Control对象，它被配置用来在初始阶段运行。

	3> 在初始化完成后，Cycle-based引擎在每一个周期中调用所有组件（Protocols和Controls）一次，直到完成了指定的周期数，或者某个组件决定终止仿真为止。在PeerSim中每一个对象（Controls和Protocols）都被赋以一个Scheduler对象，它定义了什么时候本组件将会被执行。在默认情况下，所有对象都会在每个周期中运行。但我们也可以配置一个Protocol或Control只在某些特定的周期中运行，也可以在每一个周期中指定组件的执行顺序。

下图展示了对controls和protocols的调度，其中C代表Control而P代表一个Protocol。图下方的数字代表PeerSim的周期，在最后一个周期后，可以运行一个Control来获取最后的快照(snapshot)。


下图展示了对controls和protocols的调度，其中C代表Control而P代表一个Protocol。图下方的数字代表PeerSim的周期，在最后一个周期后，可以运行一个Control来获取最后的快照(snapshot)。

.. figure:: ../statics/pics/peersim_tutorial_2_figure_1.png
	:alt: figure
在一个Control收集数据时，数据将会被格式化并发送到标准输出或重定向到一个文件以进行后续的处理。

**2.配置文件**

配置文件只是一个普通的ASCII文本，本质上就是一组由key-value对组成的java.util.Properties，以#开头的行代表注释。

若要在命令行模式下进行仿真，可以用下面的命令：

.. code-block:: console

	java -cp  peersim.Simulator config-edexample.txt

-cp后面需要注明classpath，例如：

.. code-block:: console

	java -cp D:\library\peersim-1.0.5\jep-2.3.0.jar;D:\library\peersim-1.0.5\djep-1.0.0.jar;D:\library\peersim-1.0.5\peersim-1.0.5.jar;D:\library\peersim-1.0.5\peersim-doclet.jar peersim.Simulator D:\library\peersim-1.0.5\example\config-edexample.txt	

**2.1 配置文件example 1**

本例将创建一个由50000个节点组成的固定P2P随机拓扑，选定的协议是使用average函数的Aggregation协议，每个节点中用于求平均的值使用一个区间在(0,100)的线性分布来初始化，最后再定义一个Control监视平均值。

注：Aggregation是聚集的意思，这里是指对一个分布于网络中的数值集合运行一个特定的函数进行计算（如求平均数，最大值，最小值等等）。在Gossip-based Aggregation协议中，每个节点周期性地选择一个邻居节点进行通讯（基于覆盖网），并且在每次通讯时，基于前一个取得的近似值，相互更新它们下次计算的近似值。

example1就放在config-example1.txt中：

.. code-block:: c
	:linenos:

	# PEERSIM EXAMPLE 1

	random.seed 1234567890
	simulation.cycles 30

	control.shf Shuffle

	network.size 50000
 
	protocol.lnk IdleProtocol

	protocol.avg example.aggregation.AverageFunction
	protocol.avg.linkable lnk
  
	init.rnd WireKOut
	init.rnd.protocol lnk
	init.rnd.k 20

	init.peak example.aggregation.PeakDistributionInitializer
	init.peak.value 10000
	init.peak.protocol avg

	init.lin LinearDistribution
	init.lin.protocol avg
	init.lin.max 100
	init.lin.min 1

	# you can change this to select the peak initializer instead
	include.init rnd lin

	control.avgo example.aggregation.AverageObserver
	control.avgo.protocol avg

上面的配置中，一部份是全局属性，另一部分对应单个组件的实例。如simulation.cycles是全局属性，而protocol.lnk.xxx则定义了lnk协议的xxx参数。

第6行的control.shf Shuffle，Shuffle类是用来重新洗牌，在每次重新洗牌后，在一个Cycle-based类型的仿真周期中，节点迭代的次序将会变成随机的，这个类只对Cycle-based类型的仿真起作用。

每个组件都有一个名字，比如lnk。对于协议，这个名字将会被映射到一个在PeerSim引擎中称为protocol ID的数值型索引，虽然这个索引不出现在配置文件中，但在仿真时需要使用它来访问协议，这在后面将进一步解释。

一个组件，即protocol或control由下面的语法来声明：

**<protocol|init|control>.string_id [full_path_]classname**

注意到类的全路径是可选的，事实上PeerSim可以在类路径中搜索类名，只有在多个类拥有相同的名称时必须使用全路径。init前缀定义了一个Initializer对象，它实现了Control接口。

组件的参数（如果有的话）则以下面的语法定义：

**<protocol|init|control>.string_id.parameter_name**

例如，第10行定义了第一个协议，键部份包含了它的类型，而值则是组件的类名，由于IdleProtocol类在peersim包中，所以不必使用全路径。

可以为每一个组件声明参数，如第13行；而从第3行到第8行一些全局的仿真属性被引入，如仿真的总周期数和覆盖网的大小。Shuffle control对每一个周期中节点的访问顺序进行重新洗牌。

*从第10行到第13行，引入了两个协议:*

	1> IdleProtocol是存储邻居节点链路的一个静态容器，在进行静态拓扑建模的时候尤其有用，这个协议的唯一功能是作为其它协议的邻居信息的源，它没有实现CDProtocol接口但实现了Linkable接口，Linkable接口提供了到邻居节点的链路。

	2> AverageFunction是聚集协议的求平均数版本。它的参数（linkable）是很重要的，aggregation协议需要与邻居节点交流但是本身没有邻居节点列表。在模块化的方式中，它能应用于任何覆盖网络 ；定义覆盖网的协议栈应当在这里指定，参数linkable的值是实现了Linkable接口的协议的类名（在这里是IdleProtocol）。

从15行到26行用于初始化之前声明的所有组件。前面声明了3个初始化组件，但只有其中的2个被使用了(见29行)。第一个初始化器，peersim.init.WireKOut，进行的是对静态覆盖网的布线，特别的，节点以度数k随机地与其它节点相连接。

第2个和第3个初始化器是初始化aggregation协议的可选方案，在这里是指需要求平均的初值。初始化器设置初始值遵循peak分布或线性分布。Peak的意思是只有一个节点拥有与0不同的值。而线性则代表节点被拥有一个线性增加的值。两个初始化都需要一个指定了协议来进行初始化（协议参数）的协议名。额外的参数是PeakDistributionInitializer的range(max,min参数)。

使用peak还是linear分布是由include.init属性来决定的(29行)，它指定了选择哪个初始化器。这个属性也定义了组件运行的顺序，注意到默认的顺序(即如果没有include属性)，是根据字母排序的，对于protocol和control的include属性也是如此。

最后，31行和32行声明了最后一个组件：aggregation.AverageObserver。它使用的唯一参数是protocol，它引用了aggregation.AverageFunction协议类型，所以这个参数的值是avg。

注释掉第3行的seed，运行这个仿真，得到的结果将是：

.. code-block:: console

	control.avgo: 0 1.0 100.0 50000 50.49999999999998 816.7990066335468 1 1
	control.avgo: 1 1.2970059401188023 99.38519770395408 50000 50.50000000000005 249.40673287686545 1 1
	control.avgo: 2 9.573571471429428 84.38874902498048 50000 50.500000000000085 77.89385877895182 1 1
	control.avgo: 3 23.860361582231647 71.93627224106982 50000 50.49999999999967 24.131366707228402 1 1
	control.avgo: 4 34.920915967147465 68.92828482118958 50000 50.49999999999994 7.702082905414273 1 1
	control.avgo: 5 42.37228198409946 59.94511004870823 50000 50.49999999999987 2.431356211088775 1 1
	control.avgo: 6 45.19621912151794 54.855516163070746 50000 50.499999999999844 0.7741451706754877 1 1
	control.avgo: 7 47.68716274528092 53.11433934745646 50000 50.49999999999949 0.24515365729069857 1 1
	control.avgo: 8 48.97706271318158 52.38916238021276 50000 50.50000000000026 0.07746523384731269 1 1
	control.avgo: 9 49.59674440194668 51.46963472637451 50000 50.49999999999937 0.024689348817011823 1 1
	control.avgo: 10 49.946490417215266 51.13343750384934 50000 50.50000000000048 0.007807022577928414 2 1
	control.avgo: 11 50.18143472395333 50.858337267869565 50000 50.49999999999982 0.002493501256296898 2 1
	control.avgo: 12 50.30454978101492 50.67203454827276 50000 50.500000000000206 7.90551008686205E-4 1 1
	control.avgo: 13 50.3981394834783 50.60093898689035 50000 50.49999999999967 2.518940347803474E-4 1 1
	control.avgo: 14 50.449347314832124 50.54962989951735 50000 50.5000000000003 8.071623184942779E-5 1 1
	control.avgo: 15 50.47368195506415 50.52608817343459 50000 50.49999999999999 2.566284350168338E-5 1 1
	control.avgo: 16 50.48510475374435 50.518871021756894 50000 50.50000000000012 8.191527862075119E-6 1 1
	control.avgo: 17 50.49082426764112 50.51000681641142 50000 50.49999999999945 2.570199757692886E-6 1 1
	control.avgo: 18 50.494810505765045 50.50556221303088 50000 50.5000000000003 8.197012224814065E-7 1 1
	control.avgo: 19 50.496876367842034 50.50296444951085 50000 50.499999999999524 2.640584231868471E-7 1 1
	control.avgo: 20 50.498457906558905 50.50182062146254 50000 50.500000000000334 8.565428611988968E-8 1 1
	control.avgo: 21 50.49905541635283 50.50096466374638 50000 50.49999999999974 2.721171621666857E-8 1 1
	control.avgo: 22 50.49946061473347 50.500553628252945 50000 50.49999999999975 8.590349265230611E-9 1 1
	control.avgo: 23 50.49972602272376 50.500315571370415 50000 50.5000000000004 2.6248542064007986E-9 2 1
	control.avgo: 24 50.4998450606816 50.50018053311878 50000 50.50000000000005 8.845012874999227E-10 1 1
	control.avgo: 25 50.499894793874255 50.500096923965216 50000 50.50000000000079 1.864501428663076E-10 1 2
	control.avgo: 26 50.4999267984512 50.500056126785694 50000 50.5000000000003 8.594896829690765E-11 1 1
	control.avgo: 27 50.49996613170552 50.50003198608762 50000 50.50000000000017 1.9554527178661528E-11 1 1
	control.avgo: 28 50.49997903068333 50.500019172164286 50000 50.499999999999766 3.274246411310768E-11 1 1
	control.avgo: 29 50.49998958653935 50.5000099409645 50000 50.50000000000045 0.0 1 1

Observer组件产生了很多数字，从第3列和第4列的数据（网络中的最大值和最小值），可以很容易地看到方差衰减得非常快，从第12个周期开始，几乎所有的节点都近似于真实的平均值50。可以用不同的数字或改变初始的分布（例如，使用aggregation.PeakDistributionInitializer）。同时，也可以替换覆盖网，比如你可以用Newscast来代替IdleProtocol。

**2.2 配置文件example 2**

第二个例子是前面例子的改进版本。现在Aggregation协议将运行于Newscast拓扑上并添加了一些扩展，例如，有一个Control对象用来改变网络的大小：在第5个周期至第10个周期间，每次调用时删除500个节点。

example2就放在config-example2.txt中：

.. code-block:: c
    :linenos:

	# PEERSIM EXAMPLE 2

	random.seed 1234567890

	simulation.cycles 30

	control.shf Shuffle

	network.size 50000

	protocol.lnk example.newscast.SimpleNewscast
	protocol.lnk.cache 20

	protocol.avg example.aggregation.AverageFunction
	protocol.avg.linkable lnk

	init.rnd WireKOut
	init.rnd.protocol lnk
	init.rnd.k 20

	init.pk example.aggregation.PeakDistributionInitializer
	init.pk.value 10000
	init.pk.protocol avg

	init.ld LinearDistribution
	init.ld.protocol 1
	init.ld.max 100
	init.ld.min 1

	# you can change this to include the linear initializer instead
	include.init rnd pk

	control.ao example.aggregation.AverageObserver
	control.ao.protocol avg

	control.dnet DynamicNetwork
	control.dnet.add -500
	#control.dnet.minsize 4000
	control.dnet.from 5
	control.dnet.until 10

在这里，全局参数与前面的例子相同，现在只讨论添加的扩展。

11行到12行选择了Newscast协议，它唯一的参数是缓存的大小。Newscast是一个流行性的内容分布和拓扑管理协议，系统中的每个peer都有一个部份的节点信息(事实上是一个固定大小的的节点描述符(node descriptor)的集合)，每个描述符是由peer地址和一个创建描述符的时间戳组成的元组。
每个节点通过选择一个随机的邻居并交换信息来更新自身的状态，在交换信息时，两个peer归并信息并且只留下最新的项。在这种方式中，陈旧的信息（描述符）从系统中删除。这个过程允许协议修复覆盖网拓扑，用最小的代价删除死链，这种特性在一个节点频繁加入退出的动态系统中是很有用的。

17到28行是初始化部分，与前面的例子相同，然而这里选择使用peak分布。为了将其转换为线性分布，在31行改变include init的属性。peak分布将用0初始化所有节点的值，除了取得value参数的那个节点除外。

在36到40行，DynamicNetwork是定义的最后一个组件，如前所述，一个Control对象可以用来修改仿真中的一些参数，这种改变可以在每个仿真周期中进行（默认的行为），或者使用一种更好的途径。示例中选择的对象每次在control执行时删除500个节点。

参数add指定了要添加的节点的数量，它可以是负值。而参数size则为网络大小设定了一个下限值，如果达到了下限，那不会再删除节点；参数from和until是一个可以为每个组件指定的一般化参数，它们指定了组件所要执行的周期，还有一个未使用的参数是step，如果是2，则表示每两个周期才执行一次。

**2.3 高级配置特性**

高级配置特性由Java Expression Parser提供，用于处理一些表达式。例如：

.. code-block:: c
	:linenos

	MAG 2
	SIZE 2^MAG

	A B+C
	B D+E
	C E+F
	D 1
	E F
	F 2

	# 等价于 A=7, B=3, C=4, D=1, E=2 and F=2

但是注意不允许递归定义。

对于组件的集合，可以指定执行的顺序，默认是根据组件名的字母顺序来排序的，但也可以显式地覆写为：

.. code-block:: c
	:linenos

	control.conn ConnectivityObserver
	control.myClass Class1
	control.1 Class2
	order.observer myClass conn 1

如果不是所有的名字都出现在这个列表中，那些缺失的对象会按字母顺序执行，例如：

.. code-block:: c
    
	order.observer myClass

会导致下面的运行顺序：observer.myClass, observer.1, observer.conn。

另外一个特性是告知仿真器哪些项是允许执行的：

.. code-block:: c

	include.control conn myClass

这样可以让control.conn和control.myClass以这种顺序执行，但如果这个列表为空，则什么都不会执行。
