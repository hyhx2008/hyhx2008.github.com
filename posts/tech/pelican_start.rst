使用pelican生成github博客
============================

:date: 2012-11-11 22:20
:tags: GitHub, pelican

自从马星给我推荐在github上写博客到现在已经好久了，今天我终于算是学会了。
我看人家都是用jekyll生成静态页面的，果然和专业到技术宅差得太多，自己尝试折腾了一天也没弄出什么名堂。
马星觉得自己很菜，我居然比马星还菜。。。。。
在马星的诱惑下，用了这个不知到他从哪找来的工具，说实话倒是挺好用的。

这里记录一下生成博客的步骤（以下内容大多copy自
`马星的blog <http://x-wei.github.com/pelican_github_blog.html>`_
）。


**1.配置git**

注册github后首先要配置git。可以参考
`github:help:set up git <https://help.github.com/articles/set-up-git#platform-linux>`_
。

设置用户名和邮箱，邮箱为注册github时的邮箱地址。

.. code-block:: console

	$git config --global user.name "hy"

	$git config --global user.email "hyxxxxxx@163.com"

可以用下面的命令查看配置。

.. code-block:: console

	$git config -l

github上说可以用https方法提交工程，但是没有成功过，所以还是需要利用ssh。这里需要生成ssh keys，
具体步骤参考
`github:help:generating ssh keys <https://help.github.com/articles/generating-ssh-keys#platform-linux>`_
。

然后就可以进行git clone 和 git push等操作了。


**2.生成github page**

需要在github上新建一个仓库(repository)，这个仓库的名称必须为
**your_id.github.com**
。然后将一个index.html文件上传到master分支后，就可以访问域名your_id.github.com看到自己的主页了。

**3.安装和使用pelican**

pelican安装需要用到python-pip:

.. code-block:: console
	
	$sudo aptitude install python-pip

然后再用pip安装python:

.. code-block:: console

	$sudo pip install pelican

这样pelican就安装完成了。

pelican的使用也很简单, 需要在仓库根目录下新建一个配置文件settings.py, 内容大概如下所示:

.. code-block:: python

	# -*- coding: utf-8 -*-
	import sys

	TIMEZONE = 'Asia/Tokyo'

	DEFAULT_LANG = 'zhs'

	SITENAME = "H.y's Blog"
	AUTHOR = 'hyhx2008'

	DISQUS_SITENAME = 'hysblog'
	GITHUB_URL = 'https://github.com/hyhx2008'
	SITEURL = 'http://hyhx2008.github.com'
	GOOGLE_ANALYTICS = 'UA-36075477-1'
	#TAG_FEED_ATOM = 'feeds/%s.atom.xml'
	TAG_CLOUD_STEPS = 4
	FEED_RSS = 'feeds/all.rss.xml'
	#DEFAULT_ORPHANS=3
	DEFAULT_PAGINATION = 10
	DELETE_OUTPUT_DIRECTORY = True
	DEFAULT_CATEGORY ='tech'
	OUTPUT_PATH = '.'

	PATH = 'posts'

	THEME='./pelican-themes/bootstrap'


	LINKS = (('x-wei', 'http://x-wei.github.com'),
	         ('farseerfc', "http://farseerfc.github.com/"),
			          )

	SOCIAL = (
	          ('github', 'https://github.com/hyhx2008'),
			            )

各项的含义可以参见
`pelican:settings <https://pelican.readthedocs.org/en/2.8/settings.html>`_
。

settings.py中有一项PAHT=`posts`, 指的是放置reST格式文件的目录，所以新建一个posts文件夹，然后将博客用
`reST <http://docutils.sourceforge.net/rst.html>`_
格式写好之后放在posts文件夹下，即可用pelican生成静态页面了。在仓库根目录下用:

.. code-block:: console

	$pelican -s settings.py

然后就可以看到生成的index.html了。

pelican还可以使用现成的模版，主题可以在github上下载:

.. code-block:: console

	$git clone https://github.com/farseerfc/pelican-themes

settings.py中的THEME项用来指定要使用的主题模版。

如果和我一样觉得麻烦的话，可以在github上clone一个现成的博客修改学习，比如:

.. code-block:: console

	$git clone git@github.com:hyhx2008/hyhx2008.github.com.git

**4.将博客上传到github**

使用以下三条命令:

.. code-block:: console

	$git add .
	$git commit -a -m "commit message"
	$git push origin master

收到一封页面修改成功的邮件后，就可以到自己的主页 your_id.github.com 查看了。

**5.关于reST格式文件的编辑**

pelican支持markdown和reST两种格式，由于reST的语法高亮比较容易，马星推荐我使用这个格式。附上两个教程: 
`中文教程 <https://beinggeekbook.readthedocs.org/en/latest/rst.html>`_
, 
`官方英文教程 <http://docutils.sourceforge.net/rst.html>`_
。

在linux环境下可以使用具有实时预览功能的ReText编辑器编辑reST文件，但是后来发现vim中支持reST文件的语法高亮，写起来也挺方便的。

The End!!
