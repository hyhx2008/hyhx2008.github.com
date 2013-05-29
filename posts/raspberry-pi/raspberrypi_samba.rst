Raspberry Pi中samba服务器的配置
================================

:tags: raspberry-pi samba
:date: 2013-05-29 23:04:00

为了可以在笔记本上访问树莓派的文件系统，我们可以在树莓派上安装samba服务器。

因为Raspbian是从Debian改来的，所以参考Debian下samba服务器的配置。

**1.安装samba**

.. code-block:: console

	sudo apt-get install samba samba-common-bin

**2.配置samba**

首先需要为samba创建一个用户:

.. code-block:: console

	sudo touch /etc/samba/smbpasswd
	sudp smbpasswd -a pi
	
然后我们需要编辑samba的配置文件(这里用到了vim，没有安装的话也可以用别的编辑器)

.. code-block:: console

	sudo vim /etc/samba/smb.conf 
	
然后在配置文件最后加入以下内容:

.. code-block:: console

	[home]
		comment = laowang's data 
		path = /home 
		valid users = pi
		public = no 
		writable = yes 
		printable = no 
		create mask = 0777
		
中括号内的内容为这个共享的位置起一个名称；path指出了共享的目录。

**3.重启samba服务**

最后我们需要重启samba使设置生效。并且修改共享目录的权限，使samba客户端可读可写。

.. code-block:: console
	
	sudo /etc/init.d/samba restart 
	sudo chmod 775 /home
	
**4.通过客户端访问共享的文件**

树莓派联网后，我们就可以在同一个局域网下通过samba客户端访问树莓派共享的文件夹了。

windows下非常简单，只需要在计算机中添加一个网络位置即可。

我们也可以用手机或者平板电脑访问，android里好像有不错的软件可以作为samba客户端。ios下就只发现一个叫TIOD的软件，做得一般。

