利用Jenkins+Gitlab搭建持续集成(CI)环境
========================================

:date: 2013-09-08 22:04
:tags: jenkins, gitlab, distcc, ci

这次实习的任务之一就是搭建一个持续集成(Continuous Integration)环境。

我们选择Jenkins作为持续集成工具，其优点是提供web GUI配置界面，方便配置，还可以安装很多第三方插件（plugin）进行定制与扩展，功能强大。

其次选择Gitlab作为git server。Gitlab的功能和Github差不多，但是是开源的，可以用来搭建私有git server，也提供非常强大的web GUI，比如开发者互相review源代码的时候就会很方便。

本文首先介绍整个系统的结构，然后再一一叙述各个组件的安装及使用方法。

**1.系统概览**

系统结构如下图所示:

.. figure:: ../statics/pics/ci_1.png
	:alt: figure_1

系统的工作流程大概分为以下几步:

1> 开发者将新版本push到git server (Gitlab)。

2> Gitlab随后触发jenkins master结点进行一次build。(通过web hook或者定时检测)

3> jenkins master结点将这个build任务分配给若干个注册的slave结点中的一个，这个slave结点根据一个事先设置好的脚本进行build。这个脚本可以做的事情很多，比如编译，测试，生成测试报告等等。这些原本需要手动完成的任务都可以交给jenkins来做。

4> 我们在build中要进行编译，这里使用了分布式编译器distcc来加快编译速度。

**notes**

jenkins的工作原理是先将源代码从gitlab中拷贝一份到本地，然后根据设置的脚本进行build。我们可以看出，整个系统的关键就是那个build脚本，用来告诉jenkins在一次集成中需要执行的任务。

**2.Jenkins的安装与配置**

**1> 安装Jenkins**

首先说如何安装
`jenkins <http://jenkins-ci.org/>`_
，一定要安装最新版本才不会出各种奇怪的问题，参考官网wiki,
`Installing Jenkins on Ubuntu <https://wiki.jenkins-ci.org/display/JENKINS/Installing+Jenkins+on+Ubuntu>`_
上的指示，

.. code-block:: console
	
	$ wget -q -O - http://pkg.jenkins-ci.org/debian/jenkins-ci.org.key | sudo apt-key add -
	$ sudo sh -c 'echo deb http://pkg.jenkins-ci.org/debian binary/ > /etc/apt/sources.list.d/jenkins.list'
	$ sudo apt-get update
	$ sudo apt-get install jenkins
	
即可以得到最新版的jenkins。

Jenkins安装完毕后，可以通过浏览器访问其Dashboard，例如192.168.16.183：8080，IP地址即为Jenkins所在机器的IP地址。

**2> Jenkins插件安装**

Jenkins其实没有什么需要特别配置的，由于这次任务中需要利用Jenkins与git，gitlab协作，所以需要安装一些插件。在主面板上点击Manage Jenkins -> Manage Plugins。

由于公司使用代理连接外网，首先需要为Jenkins插件安装配置proxy。点击Advanced标签即进入proxy设置页面。

Aailable标签下就是可以安装的插件。

要让Jenkins可以自动build git repo中的代码，需要安装GIT Client Plugin和GIT Plugin。

要想Jenkins可以收到Gitlab发来的hook从而自动build，需要安装 Gitlab Hook Plugin。

要让Jenkins可以在build完成之后根据TAP（test anything protocol）文件生成graph，需要安装 TAP Plugin。

**3> Jenkins Job**

我们尝试新建一个Jenkins Job，必须填的内容不是很多，而且每一项后面都有意一个问号，点击后会有关于这项的一些提示。

（1）首先是Project name；

（2）然后在Source Code Management 中选择Git，我们只需要提供Repository URL即可，这个URL可以是Jenkins机器上的一个本地repo，例如/home/woody/repo；也可以是一个远程机器上的repo，例如Gitlab上通过ssh连接的repo： git@192.168.16.194:user/test_gitlab_repo.git

注意，我们需要在Jenkins机器上设置Git username 和 email，不然后面会build不成功。

（3）下面需要在Build中选择Add build step，这里以最熟悉的shell为例，选择Execute shell。Jenkins的工作原理很简单，就是从刚才的Repository URL里clone到当前的一个workspace中并切换到最新的branch然后执行Execute shell中的command，如果每条command都返回0则build成功，否则则算失败。 所以我们的shell command可以是简单的编译指令例如：

.. code-block:: console
	
	#!/bin/sh
	make
	
（这需要你的git repo中有一个makfile）

也可以执行repo中的另一个脚本，例如

.. code-block:: console

	#!/bin/sh
	perl test.pl
	
甚至可以什么都不做，或者是只输入一些字符。

（4）接下来我们需要设置一个触发选项，在Build Triggers中，Jenkins提供三个选择

Build after other projects are built    顾名思义

Build periodically    周期性地build

Poll SCM              周期性的检测SCM（如Git）中是否有新的提交，如果有则build

选择后面两项我们都需要在Schedule中设置周期，点击后面的问号可以查看的设置周期的语法。

到这一步结束后我们就可以Save配置了，可以在面板中点击Build Now看看是否可以正常工作。在下面的Build History中有该Job的所有build历史，点击任意一条进去后可以进行查看，特别说明可以在Console Output中看到执行这次build时控制台的输出信息。

（5）如果安装了TAP Plugin，我们还可以配置Job在build完成后根据TAP result生成一副Graph例如下图：

.. figure:: ../statics/pics/ci_2.png
	:alt: figure_2
	
这幅图显示了每次build中成功与失败次数的走势。

想要得到这张图的话，需要在Configure的Post-build Action中选择Add post-build action -> Publish TAP result。 然后在test result 中输入你的TAP result文件的路径，例如tap.tap指的就是当前文件下的tap.tap文件。也就是说在你的Execute Shell中，你需要在编译完成后根据结果自己生成一个TAP文件，TAP的语法可以参考Jenkins TAP Plugin的说明。

**4> 搭建Jenkins Master - Slaves 架构**

我们在配置Jenkins时，有一个# of executors 选项，这项代表了Jenkins可以同时处理的Job的数量。

Jenkins还支持将Job分给Slave处理的功能。这就需要我们为Jenkins配置一些Slaves。

在Dashboard中点击 Mamage Jenkins -> Manage Nodes -> New Node

然后输入Node name 并选择Dumb Slave 点击OK后进入配置页面。需要填的内容如下图所示：

.. figure:: ../statics/pics/ci_3.png
	:alt: figure_3
	
# of executors指的是该slave上允许同时处理的job数量； 其他选项也都顾名思义， 我们这里选择用SSH的方式进行连接，Host即为Slave的IP地址。

Slave上不需要安装Jenkins，只需要安装java环境和git即可：

.. code-block:: console

	$ sudo aptitude install default-jre
	$ sudo aptitude install git

还需要为slave的jenkins用户配置git user.name & user.email

这里需要说明一个地方，Jenkins在工作时是利用一个名为jenkins的用户登入机器的。

在master节点上，安装Jenkins时自动为系统添加了一个名为jenkins的用户，由于slave机器上不需要安装jenkins，所以我们需要在slave机器上手动添加一个名为jenkins的用户。而且Jenkins master只能通过不用输入密码的SSH方式连接slave，需要在master上用jenkins用户生成一对ssh密钥，并把公钥加入slave机器上jenkins用户的用户目录里.ssh/authorized_keys中。

注意，没有密码的用户在ubuntu下可能切换不成功，我们用下面的方法：

.. code-block:: console

	$ sudo su jenkins
	$ bash
	
**3.Gitlab的安装与配置**

Gitlab的功能和Github差不多，都是作为git server。

Gitlab是开源的，我们可以利用Gitlab搭建自己私有的git server。

我们可以在bitnami上下载
`Gitlab <http://bitnami.com/stack/gitlab>`_
。

公司有现成的Gitlab，所以我没有尝试Gitlab的安装，看上去成功安装并不容易。

为了测试，还是在虚拟机环境下配置了Gitlab，直接下载了Virtual Machine镜像，由于镜像是Vmware的，还需要将其转为Virtual Box镜像，方法参见 http://wiki.bitnami.com/Virtual_Appliances_Quick_Start_Guide。

在虚拟机上跑起来后，发现是一个没有GUI的ubuntu。。。系统用户名和密码都是bitnami。

我们还可以通过访问其IP地址登入Gitlab，初始用户名为user，密码为bitnami，看上去和github很象。

****

由于在这次任务中Jenkins需要从Gitlab中获取文件并build，所以我们需要在Gitlab的工程中设置一些东西。

首先是settings -> Deploy Keys, 这里需要加入Jenkins所在机器jenkins用户的公钥。

如果Jenkins中使用slave，还需要将slave的公钥加入deloy keys中。 因为slave的工作原理是收到master的指示直接clone repository。

然后是一个叫Web Hooks的东东。还记得我们在Jenkins中安装过Gitlab Hook Plugin么，如果设置了Web Hooks，Gitlab就会在每次push上来后发送一条消息到指定的地址（即hook的地址），Jenkins Gitlab Hook Plugin收到消息后立即build。

不过我按照Gitlab Hook Plugin的说明设置里hooks后并没有什么作用，可能是那个插件的bug吧。。

后来大哥告诉我一个巧妙的办法，就是将Jenkins Job里的build now连接作为hook地址，例如http://192.168.16.183:8080/job/test_gitlab/build?delay=0sec

这样每次Gitlab收到push后就会促使Jenkins立即build。

**4.distcc的安装与配置**

distcc是一个分布式的编译器，他可以将编译任务分配给多个其他机器上的destccd-daemon，从而加速编译过程。

1> 安装

ubuntu下distcc很容易安装

.. code-block:: console

	$ sudo aptitude install distcc

2> 配置

distcc分为前端和守护进程，前端的用法和gcc差不多，用来编译源代码文件。

守护进程即distcc-daemon，需要一些配置。

守护进程的配置文件在 /etc/default/distcc 中， 

.. code-block:: console

	$ sudo vim /etc/default/distcc

	STARTDISTCC="true"      	   //这项允许distccd启动

	ALLOWEDNETS=“192.168.16.0/24”  //这项指出里允许那些IP的distcc连接上来， /24指的是子网掩码前24位为1，后面为0，即代表192.168.16.0 ～ 192.168.16.255
	LISTENER=“192.168.16.183”      //这项应该填本机的IP地址，即需要监听的IP地址

	ZEROCONF = “false”             //这项指出不开启zeroconf，我们先讲不开启zeroconf，后面再讨论使用它的情况

这样一个distccd就配置完成了。 

通过以下命令可以开启和停止distccd

.. code-block:: console

	$ sudo service distcc start
	$ sudo service distcc stop

在进行编译前，因为distccd都没有开启zeroconf，所以distcc无法知道有哪些host可供使用，所以这里需要在环境变量中加入，例如

.. code-block:: console

	export DISTCC_HOSTS="192.168.16.183 192.168.16.198"

使用下面的命令可以查看hosts是否配置成功。

.. code-block:: console

	$ distcc --show-hosts

然后就可以使用distcc进行分布式编译，例如编译linux kernel，

.. code-block:: console

	$ make CC=distcc

*************

下面讲当我们为distccd开启ZEROCONF时即配置

.. code-block:: console

	ZEROCONF=“true”

这就说明distcc可以不需要手动配置hosts地址即可以发现可用的hosts，具体原理不是很清楚，反正用命令

.. code-block:: console

	$ distcc --show-hosts 

可以看到distccd的地址。

但是这里貌似对IPv6的解析上有一个bug，百度里一下说需要关闭avahi的IPv6

编辑 /etc/avahi/avahi-daemon.conf

修改以下内容

.. code-block:: console

	use-ipv6=no

即可。

然后还需要在调用distcc的用户的环境变量中加入

.. code-block:: console
	
	export DISTCC_HOSTS="+zeroconf"

*************

我们还可以在调用distcc的用户中用以下命令查看编译的进度

.. code-block:: console

	$ distccmom-text 5
