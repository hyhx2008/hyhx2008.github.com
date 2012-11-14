Festival tts 使用总结
=====================

:date: 2012-08-24 22:00
:tags: festival

`Festival-tts <http://www.cstr.ed.ac.uk/projects/festival/>`_
是一个免费的tts (text to speech) 工具，可以将字符串转为声音播放出来或者存为声音文件。

首先安装:

.. code-block:: console

	$sodu aptitude install festival
	$sudo aptitude install festival-dev

可以用下面的命令测试一下:

.. code-block:: console

	$echo hellow world | festival --tts

此时可以听到喇叭中念出hello world，如果没有声音可能是声卡驱动的问题。

如果想在自己的程序中使用festival tts的功能，则需要用到festival提供的API，可以参考festiva的文档:
`festival:API <http://www.cstr.ed.ac.uk/projects/festival/manual/festival_28.html#SEC126>`_
。

这里以C++的API为例，介绍两个常用的函数。

首先程序需要include一个头文件:

.. code-block:: c++

	#include <festival.h>

两个常用API为:

.. code-block:: c++

	festival_initialize(int load_init_files,int heap_size);

	festival_say_text(char* text);

从函数名就可以看出这两个API的作用，第一个为初始化函数，需要在使用其他API之前调用，第二个函数即将text转为声音播放出来。

下面给出一个样例程序，

.. code-block:: c++

	#include <festival/festival.h>
	
	int main(int argc, char **argv)	
	{

		//EST_Wave wave;
		int heap_size = 210000;  // default scheme heap size
		int load_init_files = 1; // we want the festival init files loaded

		festival_initialize(load_init_files,heap_size);

		// Say simple file   
		//festival_say_file("/etc/motd");

		//festival_eval_command("(voice_ked_diphone)");

		// Say some text;
 		festival_say_text("hello world");

		// Convert to a waveform    
		//festival_text_to_wave("hello world",wave);

		//wave.save("/tmp/wave.wav","riff");    
		// festival_say_file puts the system in async mode so we better
		// wait for the spooler to reach the last waveform before exiting    
		// This isn't necessary if only festival_say_text is being used (and your own wave playing stuff)

		//festival_wait_for_spooler();

		return 0;

	}

其他一些可用的API被我注释掉了，感兴趣的话可以尝试一下。

程序很简单，麻烦的是编译，需要include很多头文件并连接一大堆库，搞了半天才可以编译通过，下面的编译命令供参考:

.. code-block:: console

	g++ festival_src.cpp -o festival_app -lpthread -lFestival -I/usr/include/festival -I/usr/include/speech_tools -L/usr/lib/speech_tools/lib -lestools -lestbase -leststring

The End!!


