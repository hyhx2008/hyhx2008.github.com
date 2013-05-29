Linux内核编译步骤
==================

:date: 2012-07-18 22:00
:tags: linux, kernel, compile

实习的时候需要用到3.0.35-rt56版本的linux内核，这里总结一下Debian下Linux内核编译的步骤, ubuntu下可能略有不同。

首先进入到内核源码的目录，然后按照以下步骤编译:

**1.配置内核**

.. code-block:: console

	$make menuconfig

这条命令可以在一个GUI下设置用户需要的内核参数，并生成编译需要的.config配置文件。

**2.编译内核**

.. code-block:: console

	$make -j 2

后面的-j参数指的是可以允许多少个module同时编译，-j 2 就是允许2个module同时编译，也可以省略-j参数。如果CPU是多核的话，可以加上该参数提高编译速度。

接下来就是漫长的等待。。。

**3.加入模块**

.. code-block:: console

	$sudo make modules_install
	$sudo make install

**4.更新引导文件**

.. code-block:: console

	$sudo update-initramfs -c -k "kernel version"

这里的kernel version可以在make menuconfig的时候自己设置名称，具体应该写什么需要在/boot文件夹下查看:

.. code-block:: console

	$ls -lF /boot

比如内核版本为2.6.36，则/boot下就会有一个vmlinuz-2.6.36文件，把vmlinuz-后面的字符放在kernel version的位置即可。

**5.更新grub**

.. code-block:: console

	$sudo update-grub

整个编译过程大概也得一两个小时左右吧，耐心等待。

The End！！

