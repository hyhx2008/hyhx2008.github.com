Ubuntu网络设置
=================

:date: 2013-04-15 21:35:00
:tags: ubuntu, linux, network

**1.配置ip**

ubuntu的网络配置信息放在 /etc/network/interfaces 中;

如果配置动态获取ip，则在上述文件中加入以下内容：

.. code-block:: console

    auto eth0
    iface eth0 inet dhcp

如果配置静态ip，则添加如下内容：

.. code-block:: console
    
    auto eth0 
    iface eth0 inet static
    address 192.168.1.155
    netmask 255.255.255.0
    gateway 192.168.1.1
                                 
要是配置生效，需要重启网卡：

.. code-block:: console

    ifconfig eth0 down
    ifconfig eth0 up

                                        
接着用ifconfig命令查看ip是否配置成功，若还有没有配置成功，则需重启下网络服务

.. code-block:: console

    /etc/init.d/networking restart
                                                                                        
**2.配置dns服务器**

有时候需要手动配置dns，则可以参考如下步骤：

ubuntu 的dns服务器信息，放在 /etc/resolv.conf中。

要添加dns服务器地址，如202.112.125.53，则在上述文件中加入

.. code-block:: console

    nameserver  202.112.125.53
