Ubuntu下Java的安装与配置
========================

:date: 2013-04-15 21:03:00
:tags: ubuntu, linux, java

第一次在linux环境下安装jdk，这里记录一下安装和配置的过程供以后参考。

**1.下载**

首先在Oracle官网上下载jdk，这里给一个链接：
`jdk7-downloads <http://www.oracle.com/technetwork/java/javase/downloads/jdk7-downloads-1880260.html>`_

选择合适的jdk版本下载，这里我下载的是jdk-7u17-linux-i566.tar.gz。

**2.安装**

在任意文件夹下解压这个压缩包，例如在/usr/java下：

.. code-block:: console

    sudo mkdir /usr/java
    sudo cp ~/Downloads/jdk-7u17-linux-i586.tar.gz /usr/java/
    cd /usr/java
    sudo tar -xvf jdk-7u17-linux-i585.tar.gz

这样jdk就安装在/usr/java/jdk1.7.0_17目录下。

**3.修改环境变量**

修改~/.bashrc文件：

.. code-block:: console

    vim ~/.bashrc

添加以下内容：

.. code-block:: console

    export JAVA_HOME=/usr/java/jdk_1.7.0_17
    export JRE_HOME=${JAVA_HOME}/jre
    export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
    export PATH=${JAVA_HOME}/bin:$PATH

**4修改系统默认的jdk**

.. code-block:: console

    $ sudo update-alternatives --install /usr/bin/java java /usr/java/jdk1.7.0_17/bin/java 300 
    $ sudo update-alternatives --install /usr/bin/javac javac /usr/java/jdk1.7.0_17/bin/javac 300 

    $ sudo update-alternatives --config java 
    $ sudo update-alternatives --config javac

**5.测试**

可以先在终端输入 java -version，若能看到java的版本信息，则说明修改成功。

也可也写一个简单的java程序测试一下，例如helloworld.java：

.. code-block:: java

    publci class helloworld
    {
        public static void main(String[] args)
        {
            System.out.println("Hello World!!");
        }
    }

然后编译并运行：

.. code-block:: console

    javac helloworld.java
    java helloworld

若能看到输出结果，证明java安装成功。


