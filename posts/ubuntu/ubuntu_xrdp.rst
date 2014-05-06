Ubuntu下配置远程桌面xrdp
========================

:date: 2014-04-15 21:03:00
:tags: ubuntu, linux, remote desktop, xrdp

想通过windows的远程桌面访问ubunt的桌面，尝试VNC不成功，于是百度出利用xrdp也可以实现。

**1.安装**

.. code-block:: console

    sudo apt-get install xrdp

**2.配置**

配置文件在/etc/xrdp/xrdp.ini中，我没有进行特别的修改。

**3.安装gnome桌面环境**

只安装xrdp后就进行连接（端口号3389）时，只会显示一个桌面背景。。。能做的操作就是新建文件夹。。。

百度之后说是因为ubuntu13.04的桌面环境不支持，所以需要换成gnome。

.. code-block:: console

    sudo apt-get install gnome-session-fallback
	echo "gnome-session --session=gnome-fallback" > ~/.xsession
	
**4.重启xrdp**

.. code-block:: console

    sudo service xrdp restart
	
The end！！
	


