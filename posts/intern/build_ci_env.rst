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

notes:jenkins的工作原理是先将源代码从gitlab中拷贝一份到本地，然后根据设置的脚本进行build。我们可以看出，整个系统的关键就是那个build脚本，用来告诉jenkins在一次集成中需要执行的任务。

**2.Jenkins的安装与配置**

**3.Gitlab的安装与配置**

**4.distcc的安装与配置**

