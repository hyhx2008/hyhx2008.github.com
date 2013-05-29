Raspberry Pi系统安装及启动
============================

:tags: raspberry-pi, raspbian
:date: 2013-05-29 21:27:00

最近有些无聊，心血来潮买了一个Raspberry Pi(树莓派)。Raspberry Pi是一款基于Linux系统的个人电脑，配备一枚700MHz的ARM处理器，
512M内存，支持SD卡和Ethernet，拥有两个USB接口，以及 HDMI和RCA输出支持。

首先秀一张开箱图:

.. figure:: ../statics/pics/raspberrypi_start_1.png

**1.安装raspbian系统到SD卡**

树莓派用SD卡充当硬盘，为了速度，我专门买了一张16G class10的SD卡。

树莓派支持多个系统，官方建议的是Raspbian，这是一个从Debian改过来的系统。首先需要在官网下载系统镜像
`Dowload Raspbian <http://www.raspberrypi.org/downloads>`_
。

然后我们需要一个工具
`Win32 Disk Imager <http://sourceforge.net/projects/win32diskimager/>`_
将系统写入SD卡中。

下载并解压，然后运行win32diskimager。工具的界面很简单，我们只需要在"Image File"中选择系统镜像的位置(注意镜像的路径中不能有中文)，
在"Device"下选择SD卡的盘符，然后点击"write"按钮。

接下来就是等待至工具提示"Write Successful"后，即可将SD卡安全移除。

我们会发现SD卡的空间变成74MB了。。。百度了一下，高手的解释是windows下看不到linux的分区。

**2.初次启动Raspberry Pi**

将写入系统的SD卡插入树莓派，并连上键盘鼠标后，插上电源树莓派就会自动启动了。

由于是第一次启动，我们会看到一个raspi-config的配置工具。我们可以在这个工具中对系统进行一些配置。

.. figure:: ../statics/pics/raspberrypi_start_2.gif

如果以后需要更改某些配置，也可以在命令行中输入raspi-gonfig来运行这个工具。

这里不做详细展开，只是提一下哪些选项需要设置。

1> expand_rootfs 可以将文件系统扩展到整张SD卡中。

2> configure_keyboard 可以选择键盘布局。

3> change_pass 修改密码。

4> change_locale 选择字符编码方式。这里需要将默认的en_GB改为en_US.UTF-8。(用空格键选择和反选)

5> change_timezone 修改时区。

6> ssh 这里我启动了ssh server(enable ssh)，为了可以在笔记本上远程登录树莓派。

配置完成后选择Finish并按回车，就会进入登录界面，输入用户名(pi)和密码(默认是raspberry)即可进入系统的shell。

如果要进入图形界面，输入命令"startx"即可。

用树莓派可以做的事情有很多，这里给一个连接
`34 个使用 Raspberry Pi 的酷创意 <http://linuxtoy.org/archives/cool-ideas-for-raspberry-pi.html>`_
，以后有空了漫漫尝试。



