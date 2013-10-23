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

**notes:**jenkins的工作原理是先将源代码从gitlab中拷贝一份到本地，然后根据设置的脚本进行build。我们可以看出，整个系统的关键就是那个build脚本，用来告诉jenkins在一次集成中需要执行的任务。

**2.Jenkins的安装与配置**

**1> 安装Jenkins**

首先说如何安装
`jenjins <https://wiki.jenkins-ci.org/display/JENKINS/Home>`_
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


**3.Gitlab的安装与配置**

**4.distcc的安装与配置**

