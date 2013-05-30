将Raspberry Pi作为BT下载机
============================

:date: 2013-05-30 21:54:00
:tags: raspberry-pi, transmission

今天尝试了一个有用的功能，就是将树莓派作为bt下载机。

由于树莓派耗电低，没噪音，可以让它一直开着下载bt资源。

一个名为
` Transmission <http://www.transmissionbt.com/>`_
的软件为我们提供了这个功能。它不但能作为树莓派上的bt客户端，还可以通过web图形界面进行控制，非常方便。

**1.安装Transmission**

.. code-block:: console
	
	sudo apt-get install transmission-daemon
	
**2.配置Transmission**

Transmission的配置是通过编辑其配置文件实现的，

.. code-block:: console 

	sudo vim /etc/transmission-daemon/settings.json
	
可配置的选项有很多，这里挑几个常用的讲一下。

1> "download-dir" : "/home/pi/bt"  设置下载后的资源保存的路径。

2> "rpc-authentication-required" : true  远程控制时需要验证

3> "rpc-enabled" : true  开启远程控制

4> "rpc-password" & "rpc-username" 这两项设置验证时的用户名和密码

5> "rpc-whitelist-enabled": false 关闭白名单功能(即允许从任何ip地址远程控制)

更详细的配置参数解释可以参考百度文库中的一篇文章:
`树莓派bt下载机 <http://wenku.baidu.com/view/075739d949649b6648d747d9.html>`_

**3.加载配置并重启服务**

.. code-block:: console

	sudo service transmission-daemon reload
	sudo service transmission-daemon restart
	
至此bt下载机就配置好了，现在我们可以在局域网中任何一台设备的浏览器中访问树莓派的IP地址加9091端口进入Transmission的web界面，比如192.168.1.115:9091
用户名和密码即为配置文件中所设置的(如果没有改动，默认都为transmission)。

Transmission的web界面非常简单，我们只需要提供.torrent种子文件或者是提供种子文件的链接地址，即可让树莓派执行下载任务。

The End！