windows下安装pelican
=====================

:date: 2013-05-29 20:23:00
:tags: pelican, github

用虚拟机在ubuntu下写了这么久的博客，实在是受不了电脑内存不够，一开虚拟机就开始卡的情况。

原来一直懒得搞，因为在linux下安装程序很方便。今天尝试了一下在windows下安装pelican，其实也不是很烦。

**1.安装python**

pelican是用python开发的，所以首先要在windows下安装python。

在python官网上下载python安装包:
`Download Python <http://www.python.org/download/>`_
。我是win8-64bit，所以选择Python3.3.2Windows X86-64 MSI Installer。安装的过程是全自动的，只需要选择路径和点下一步即可。

安装的过程中会询问是否将python添加到环境变量中，为了可以直接在cmd(命令提示符)中使用python命令，建议选是。如果不小心选错了，也可以手动
将python添加到环境变量中去。在path中加入

.. code-block:: console

	C:\Python33\Scripts;

即可(此处路径为示例，有可能不一样)。

可以在cmd中输入python查看是否安装成功。(ctrl+c退出)

**2.安装pip**

pip是一个帮助我们安装和管理python包的工具，我们通过pip来安装pelican。

首先在
`Download pip <https://pypi.python.org/pypi/pip>`_
下载pip。

然后解压下载的压缩包，例如pip-1.3.1.tar.gz。运行cmd并进入解压目录，输入

.. code-block:: console

	python setup.py install
	
这样pip就安装完成了。

**3.安装pelican**

有了pip再安装pelican就很容易了，只需要一条命令

.. code-block:: console
	
	pip install pelican
	
这样我们就在windows上安装了pelican。以后写博客就不再需要linux环境了。

如果不是很熟悉windows的cmd命令，这里推荐使用
`cygwin <http://www.cygwin.com/>`_
。它可以在windows上模拟unix环境，并且可以安装大部分常用的
工具及软件，比如git。安装完成后添加至环境变量，就可以在windows的cmd中使用linux的命令了。