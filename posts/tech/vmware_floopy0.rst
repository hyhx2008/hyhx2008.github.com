VMware虚拟机启动时总提示cann't connect to floopy0的问题
============================================================

:date: 2012-11-13 17:26
:tags: ubuntu, VMware

在VMware中装了ubuntu，启动时总提示不能连接到设备floopy0，关也关不掉，很讨厌。

百度了一下解决办法: 修改虚拟机的.vmx文件。

1.将floppy0.autodetect这项改为FALSE:

.. code-block:: console

	floppy0.autodetect = "FALSE"

2.添加一句:

.. code-block:: console

	floppy0.startConnected = "FALSE"

The End!!

