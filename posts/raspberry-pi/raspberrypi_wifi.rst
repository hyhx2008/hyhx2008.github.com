Raspberry Pi配置USB无线网卡连接wifi
====================================

:tags: raspberry-pi
:date: 2013-05-29 22:18:00

树莓派自身不带无线网卡，所以要想让树莓派连上无线网络，就需要插上一个USB无线网卡。

首先可以在树莓派的
`外设兼容列表 <http://elinux.org/RPi_VerifiedPeripherals#USB_Wi-Fi_Adapters>`_
中选择一款兼容的USB无线网卡，最好是那种即插即用的(out of the box)，这样就不需要安装驱动了。
在日本只能买到 Buffalo WLI-UC-GNM，本以为需要手动安装驱动，没想到直接被系统识别了。

我们可以使用命令

.. code-block:: console
	
	ifconfig -a

来查看接口信息。如果有wlan0这项，就说明新加的网卡已经被识别了。但是IP地址之类的信息还没有。

接下来我们需要配置接口，即编辑配置文件。

.. code-block:: console

	sudo nano /etc/network/interfaces
	
将interfaces中的配置改为:

.. code-block:: console

	auto lo
	iface lo inet loopback
	iface eth0 inet dhcp
	auto wlan0
	allow-hotplug wlan0
	iface wlan0 inet dhcp
        wpa-ssid “XX-Network”
        wpa-psk “XX1234567″
		
其中wpa-ssid后引号中的内容为你的无线网络的名称，wpa-psk后引号中的内容为密码。(这里无线路由器需选择wpa加密方式)

最后我们需要重新启动树莓派:

.. code-block:: console

	sudo shutdown “now” -r
	
重新启动后，以上的设置才会生效，这时再次查看接口信息时，发现wlan0已经成功获得了IP地址。

