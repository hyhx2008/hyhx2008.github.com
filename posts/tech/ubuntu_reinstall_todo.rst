重新安装unbutu后的todo list
============================

:slug: ubuntu_reinstall_todo
:lang: zh
:date: 2012-11-11 21:44
:tags: ubuntu

经常把虚拟机里ubuntu搞坏，重装后总记不清还要做什么设置，安什么软件。

写在这里做个提醒吧，以后再慢慢添加。

1.安装aptitude

.. code-block:: console

    $sudo apt-get install aptitude

2.安装vim

.. code-block:: console

	$sudo aptitude install vim

并修改/usr/share/vim/vimrc 。

3.添加sudoers

.. code-block:: console
	
	$sudo visudo

加入 hy ALL = （ALL）ALL。

ubuntu下好像这样做后还是不能使用su命令切换到root，百度解决办法:

.. code-block:: console

	$sudo passwd

修改一下密码就好了。

4.安装openssh

为了能让secureCRT连上虚拟机里的ubuntu，需要安装openssh。

.. code-block:: console

	$sudo aptitude install openssh-server openssh-client

再用ifconfig查一下ip，填到secureCRT里就好了。

5.安装中文输入法

.. code-block:: console

	$sudo aptitude install ibus ibus-pinyin ibus-gtk ibus-qt4

暂时想到这么多。。。。



