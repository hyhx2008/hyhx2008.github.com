tesseract-ocr使用方法总结
==========================

:date: 2012-12-01 16:34
:tags: ocr Linux

实习的时候接触到了一个挺不错的OCR(光学字符识别)工具tesseract，这里总结一下linux下tesseract的简单用法。

tesseract-ocr是由google维护的一个开源ocr库，可以识别多种格式图片中的文字，可以直接作为一个工具使用，也可以利用tesseract提供的api在自己的程序中使用ocr功能。

`项目主页 <http://code.google.com/p/tesseract-ocr/>`_

**1.tesseract-ocr安装**

ubuntu和debian下tesseract的安装非常简单，但是需要安装的包比较多:

.. code-block:: console

	$sudo aptitude install tesseract-ocr tesseract-ocr-dev tesseract-ocr-eng

tesseract-ocr即为ocr库，tesseract-ocr-dev是使用api时需要用到的头文件。

tesseract-ocr-eng是语言包，这里安装的是英文，如果要识别其他语言，需要安装其他语言包。

（语言包安装的位置在/usr/share/tesseract-ocr/tessdata/，该位置将作为初始化api的一个参数）

为了识别tiff格式的图片，最好再安装以下两个库:

.. code-block:: console

	$sudo aptitude install leptonica libtiff

这样tesseract-ocr和一些相关的包就安装完成了。

**2.tesseract-ocr使用**

作为一个命令行程序，tesseract可以用以下命令识别一张图片中的文字:

.. code-block:: console

	tesseract photo.tif 1.txt

以上命令将从photo.tif中识别的文字存储在1.txt中。

**3.tesseract api 使用**

tesseract现在已经出到3.0.1版本了，网上的资料也大多基于3.0.1版本，比如
`Tesseract3.01在VS2008下面的使用 <http://www.cnblogs.com/zsb517/archive/2012/06/06/2537540.html>`_ 和
`OCR识别引擎tesseract使用方法 <http://blog.csdn.net/foxwit/article/details/6547465>`_ 。

不幸的是，Debian下安装的是2.0.4版本，api的命名与用法和新版本有所不同。在项目主页上我也没有找到api的使用手册之类的东西，只能自己琢磨。

我发现查看tesseract的source code是一个很好的学习api用法的途径，源码可以在下面的链接中下载:
`tesseract 2.0.4 Source <http://code.google.com/p/tesseract-ocr/downloads/detail?name=tesseract-2.04.tar.gz&can=2&q=>`_
，
`tesseract 3.0.1 Source <http://code.google.com/p/tesseract-ocr/downloads/detail?name=tesseract-3.01.tar.gz&can=2&q=>`_
。

下载并解压后在ccmain文件夹中的
**tesseractmain.cpp**
就是tesseract工具的主程序，是很好的api用法参考。api的声明在
**baseapi.h**
中(ccmain中或/usr/include下都可以找到)。

所有需要include的头文件都在 /usr/include/tesseract 和 /usr/include/lptonica 中，需要链接的库在/usr/lib中。

api的具体用法参见后面附的源码的注释。

编译真的是最痛苦的事情，不清楚需要链接哪些库，于是/usr/lib里用下面的命令找到相关的库，全都链接进来:

.. code-block:: console

	$ls | grep tesseract
	$ls | grep leptonica
	$ls | grep tiff

结果编译还是不通过，提示一大堆未定义的函数，发现都和png有关系，于是又用

.. code-block:: console

	$ls | grep png

找到一个库链接后才编译成功。我编译用的Makefile附在最后。

**4.api使用的例子**

新建一个ocr_test.cpp文件，将下面的代码粘贴进去即可。

.. code-block:: c++

	#include <mfcpch.h>
	#include <ctype.h>
	#include "applybox.h"
	#include "control.h"
	#include "tessvars.h"
	#include "tessedit.h"
	#include "baseapi.h"
	#include "pageres.h"
	#include "imgs.h"
	#include "varabled.h"
	#include "tprintf.h"
	#include "stderr.h"
	#include "notdll.h"
	#include "mainblk.h"
	#include "output.h"
	#include "globals.h"
	#include "blread.h"
	#include "tfacep.h"
	#include "callnet.h"
	

	int main(int argc,char **argv){

	    if(argc!=3)
		{
			printf("usage:%s <tiff file> <txt file>/n",argv[0]);
			return -1;
		}

		char *image_file=argv[1];	//程序的第一个参数为图片路径
		char *txt_file=argv[2];		//第二个参数为保存识别出的字符的文件路径

		STRING text_out;			//存储识别出的字符
																	       
		TessBaseAPI  api;
		IMAGE image;
		
		api.SimpleInit("/usr/share/tesseract-ocr/tessdata/", NULL,false); // 初始化函数，tesseract还提供其他的初始化函数 参考 baseapi.h
		
		if (image.read_header(argv[1]) < 0)
		{
			printf("read image header error!\n");
			exit(-1);
		}
		
		if (image.read(image.get_ysize()) < 0)
		{
			printf("read image error!\n");
			exit(-1);
		}                                     
		//读取图片并判断是否读取成功		

		int bytes_per_line = check_legal_image_size(image.get_xsize(),image.get_ysize(),image.get_bpp());
		
		char* text = api.TesseractRect(image.get_buffer(),image.get_bpp()/8,bytes_per_line,0,0,image.get_xsize(),image.get_ysize()); 
		//tesseract核心api，通过该函数识别出图片中的字符
		
		text_out += text;
		
		printf("output: %s\n");
		
		delete [] text;
		
		FILE* fout = fopen(txt_file, "w");
		
		fwrite(text_out.string(), 1, text_out.length(), fout);
		
		fclose(fout);
		
		return 0;
	}


**5.编译并运行**

新建Makefile如下所示。

.. code-block:: console

	LDFLAGS= -ltesseract_full -ltesseract_pageseg -ltesseract_training -ltesseract_textord -ltesseract_wordrec -ltesseract_classify -ltesseract_dict -ltesseract_ccstruct -ltesseract_cutil -ltesseract_viewer -ltesseract_ccutil -ltesseract_image -ltesseract_main -llept -ltiff -ltiffxx -lpthread -lpng -lpng12

	INCLUDES= -I/usr/include/tesseract/ -I/usr/include/leptonica/

	all:ocr

	ocr:
		g++ -g -o ocr ocr_test.cpp $(LDFLAGS) $(INCLUDES) 

	clean:
		rm ocr

用下面的命令运行样例程序:

.. code-block:: console

	$make
	$ocr photo.tif 1.txt

The End!!
