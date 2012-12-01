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

未完待续。。。。
