利用Jenkins+Gitlab搭建持续集成(CI)环境
========================================

:date: 2013-09-08 22:04
:tags: jenkins, gitlab, distcc, ci

这次实习的任务之一就是搭建一个持续集成(Continuous Integration)环境。

我们选择Jenkins作为持续集成工具，其优点是提供web GUI配置界面，方便配置，还可以安装很多第三方插件（plugin）进行定制与扩展，功能强大。

其次选择Gitlab作为git server。Gitlab的功能和Github差不多，但是是开源的，可以用来搭建私有git server。

本文首先介绍整个系统的结构，然后再一一叙述各个组建的安装及使用方法。

**1.系统概览**


