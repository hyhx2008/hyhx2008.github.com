Ubuntu网络设置
=================

:date: 2013-04-15 21:35:00
:tags: ubuntu, linux

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

**3.设置代理**

公司里链接外网需要设置http代理， ubuntu下可以用系统的GUI设置代理，但是这样终端中依旧无法连接外网，例如apt-get命令都不能使用。 百度了一下有以下设置方法供参考：

1） 如果仅仅是想配置apt-get命令的代理，可以在/etc/apt/apt.conf.d文件夹中修改（没有则新建）80proxy配置文件，在文件中加入以下内容。

.. code-block:: console

	Acquire::http::Proxy "http://username:passwd@133.144.14.240:8888/";

这里的proxy IP和端口号仅为示例，需按自身情况修改。注意不要忘记行末的分号。

2） 如果是想要终端暂时联网，只需要在终端中键入以下命令。

.. code-block:: console

	export http_proxy="http://username:passwd@133.144.14.240:8888/"
	export https_proxy="http://username:passwd@133.144.14.240:8888/"

如果有需要也可以加入其他协议的代理，如ftp。

3） 上面第二种方法是临时改变环境变量，将终端关闭或者新开一个终端后则不起作用，需要重新设置。一劳永逸的方法是直接在.bashrc文件中修改环境变量。

.. code-block:: console

	$ cd
	$ vim .bashrc

在文件末尾加入以下内容

.. code-block:: console

	http_proxy=http://username:passwd@133.144.14.240:8888/
	export http_proxy

	https_proxy=http://username:passwd@133.144.14.240:8888/
	export https_proxy

