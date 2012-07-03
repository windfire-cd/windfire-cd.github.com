---
layout: post
title: "Rest vs Markdown"
date: 2012-06-29 14:20
comments: true
categories: 
---

Table of Contents

-   [概述](#id1)
-   [Markdown 安装以及工具链](#markdown)
-   [Markdown 语法](#id2)
-   [RestructuredText安装以及工具链](#restructuredtext)
    -   [依赖](#id4)
    -   [RestructuredText 安装](#id5)

-   [RestructuredText 语法](#id6)
-   [Markdown，RestructuredText技巧和应用](#markdown-restructuredtext)
    -   [RestructuredText](#id7)
    -   [Markdown](#id8)

-   [Markdown, RestructuredText的中文相关问题](#id9)
-   [Markdown, RestructuredText对比](#id10)
-   [参考](#id12)

[概述](#id13)
=============

介绍配置使用Markdown和RestructuredText以及相关对比。
以及这两天配置和使用的部分经验，所有资料来自于网络，最开始想要使用这两种工具的起因是ppt对于代码高亮支持不够理想，
然后发现若干网络上面的代码高亮很不错

[Markdown 安装以及工具链](#id14)
================================

需要安装 [Pandoc](http://johnmacfarlane.net/pandoc/index.html)
,这个比较好使，幻灯，代码高亮，pdf以及docx生成都可以使用这个工具也可以转换restructuredtext格式，但是似乎对于Markdown格式更加亲和。ubuntu的软件管理中存在该工具，如果需要更新到最新版本的进行如下操作：

{%  codeblock lang:bash %}
sudo apt-get install haskell-platform
sudo cabal update
sudo cabal install pandoc
{%  endcodeblock %}

在该网站上有若干 [Example](http://johnmacfarlane.net/pandoc/demos.html)
可以供参考

[Markdown 语法](#id15)
======================

[Markdown简明教程](http://wowubuntu.com/markdown/)

[RestructuredText安装以及工具链](#id16)
=======================================

[依赖](#id17)
-------------

**texlive**

在ubuntu上面有texlive
2009，如果希望安装更新的版本的话需要按照Texlive官方安装文档手动安装，没有特殊的apt-get可用，需要下载
[install-tl-unx.tar.gz](http://mirror.ctan.org/systems/texlive/tlnet/install-tl-unx.tar.gz)

在ubuntu 12.04上运行如下命令

{%  codeblock lang:bash %}
1 $sudo tar -xvf install-tl-unx.tar.gz && cd install-tl-unx && sudo ./install-tl
{%  endcodeblock %}


[RestructuredText 安装](#id18)
------------------------------

在官网
[Document](http://docutils.sourceforge.net/rst.html#user-documentation)
下载对应的\`Python\`源码文件，然后运行

{%  codeblock lang:python  %}
1 $sudo python setup.py install
{% endcodeblock %}

[RestructuredText 语法](#id19)
==============================

主要语法参见
[Document](http://docutils.sourceforge.net/rst.html#user-documentation),
[Primer](http://docutils.sourceforge.net/docs/user/rst/quickstart.html),
[QuickRef](http://docutils.sourceforge.net/docs/user/rst/quickref.html),
[Directives](http://docutils.sourceforge.net/docs/ref/rst/directives.html)

**目录级别**

一般按照:


> = - ` : ' " ~ ^ _ * + # < >


的顺序进行目录配置

**链接**

-   当前链接:

{% codeblock lang:rst %}
 `test <http://google.com.hk>`__
{% endcodeblock %}

-   匿名链接:

{% codeblock lang:rst %}
test anynomous link `link`__
__ http://google.com.hk
{% endcodeblock %}

-   外部链接:

{% codeblock lang:rst %}
test global link example_link_
.. _example_link: http://google.com.hk
{% endcodeblock %}

**图片**

-   当前图片:

{% codeblock lang:rst %}
.. image:: img/test.png
    :align: center
    :scale: 150%
    :alt: 额是
    :target: http://baidu.com
{% endcodeblock %}

-   全局图片:

{% codeblock lang:rst %}
.. |TestImg| image:: img/test.png
    :align: top
    :scale: 105%
    :alt: ceshi
    :target: http://baidu.com
{% endcodeblock %}

 文字中使用图片 |TestImg|

[Markdown，RestructuredText技巧和应用](#id20)
============================================

这里记录一些Markdown和RestructuredText的一些技巧和应用

[RestructuredText](#id21)
-------------------------

**生成html**

可以用pandoc, sphinx以及rst2html生成html。这三个工具特点如下：

> 1.  *pandoc* : Haskell语言库，转化格式较多，但是需要Haskell
>
> 2.  *sphinx* : Python官方文档库，比较强大，对中文支持好，
>       ~ 但是比较复杂转化pdf借助于texlive，texlive-2011对中文支持更好
>
> 3.  *rst2html* : 官方的转化程序，使用简单，对中文支持好，我们一直用它！
>
**语法高亮**

可以利用如下代码进行语法高亮

{% codeblock lang:rst %}
.. code:: lang-name
    :number-lines:

    source
{% endcodeblock %}

在新的Docutils中可以支持 *code*
指令，但是需要Pygments进行解析和高亮官方安装利用下面的命令

{%  codeblock lang:bash %}
1 $sudo apt-get install python-pygments
{% endcodeblock %}

但是官方比较老，支持代码高亮的只在Docutils
0.9版本上，所以最好源码安装根据官方安装文档
[Pygments-installation](http://pygments.org/docs/installation/) 进行安装

首先生成对应的sheet

{%  codeblock lang:bash %}
1 $pygmentize -S default -f html -a .highlight > style.css
{% endcodeblock %}

然后利用生成的css进行渲染

{%  codeblock lang:bash %}
1 $rst2html --stylesheet=style.css test.rst test.html
{% endcodeblock %}

**Slides**

rst2slides, landslide,
rst2s5等均支持ReST转化为幻灯片，但是从格式支持上来说，rst2s5更为合理

[Markdown](#id22)
-----------------

**语法高亮**

输入格式为

{%  codeblock %}
 ~~~~~~~~~~{.lang-name, .numberLines}
 source
 ~~~~~~~~~~
{% endcodeblock %}

其中 \~ 需要上下一致，lang-name 代表语言名称，.numberLines
代表是否标识代码行数

然后利用pandoc指定选项可以生成不同风格的代码高亮，例子如下

{%  codeblock lang:bash %}
pandoc code.txt -s --highlight-style pygments[tango|kate|..] -o example.html
{% endcodeblock %}

**在线博客**

利用octopress和github可以编写markdown格式的在线博客，具体步骤见 [Octopress](http://octopress.org/)

值得注意的是其语法高亮模式有多种，只有 [Backtick Code Blocks](http://octopress.org/docs/blogging/code/) 和*pandoc*
兼容具体格式如下

{% codeblock %}
>``` [language] [title] [url] [link text]
>code snippet
>```
>其中`只能是3个

{% endcodeblock %}

[Markdown, RestructuredText的中文相关问题](#id23)
=================================================

**Markdown**

利用 pandoc -s *source* -o *target*
可以处理大部分中文问题，注意需要指定-s, -o选项，否则会出现乱码

**RestructuredText**

利用 *rst2html* , *rst2tex* 等工具可以直接转化，不需要特殊注意中文问题

**中文pdf**

无论Markdown或者RestructuredText默认使用pdflatex处理pdf文档，所以都无法完美解决中文问题，这里我们需要使用xelatex解决中文字体问题，在texlive安装目录对应版本下的texmf-dist/tex/xelatex/目录下，新建zhfontcfg目录，在该目录中创建zhfontcfg.sty宏包内容如下

{% codeblock lang:tex %}
\ProvidesPackage{zhfontcfg}
\usepackage{fontspec,xunicode}
\defaultfontfeatures{Mapping=tex-text} %如果没有它，会有一些 tex 特殊字符无法正常使用，比如连字符。

% 中文断行
\XeTeXlinebreaklocale "zh"
\XeTeXlinebreakskip = 0pt plus 1pt minus 0.1pt

%将系统字体名映射为逻辑字体名称，主要是为了维护的方便
\newcommand\fontnamehei{WenQuanYi Micro Hei}
%\newcommand\fontnamesong{SimSun}
%\newcommand\fontnamekai{KaiTi_GB2312}
\newcommand\fontnamemono{WenQuanYi Micro Hei Mono}
%\newcommand\fontnameroman{Bitstream Vera Serif}

%设置文档正文字体为宋体
\setmainfont[BoldFont=\fontnamehei]{\fontnamehei}
\setsansfont[BoldFont=\fontnamehei]{\fontnamehei}
\setmonofont{\fontnamemono}

%楷体
%\newfontinstance\KAI{\fontnamekai}
%\newcommand{\kai}[1]{{\KAI #1}}

%黑体
\newfontinstance\HEI{\fontnamehei}
\newcommand{\hei}[1]{{\HEI #1}}

%英文
%\newfontinstance\ENF{\fontnameroman}
%\newcommand{\en}[1]{\,{\ENF #1}\,}
%\newcommand{\EN}{\,\ENF\,}S5_

{% endcodeblock %}

然后运行

{%  codeblock lang:bash %}
sudo mktexlsr
{% endcodeblock %}

其中设置正文字体通过

{%  codeblock lang:bash %}
fc-list :lang=zh-cn
{% endcodeblock %}

获取

利用Markdown和RestructuredText生成.tex文件，然后在文件前面添加\`usepackage{zhfontcfg}\`

最后用

{%  codeblock lang:bash %}
xelatex file.tex
{% endcodeblock %}

生成相应的pdf文件

**中文换行**

如果在编辑器中直接打一个回车进行换行，那么在生成的文件中会回车处出现一个空格这使W3C的一个标准，但是可以进行屏蔽

在Markdown中在末尾处直接打'\\n'然后回车可以屏蔽该现象

在RestructuredText中末尾处直接打个'\\'然后回车可以屏蔽该现象

[Markdown, RestructuredText对比](#id24)
=======================================

Markdown是github主推的格式，比较简洁，就是一个text to
html的格式其思想主要来源于email。ReST是Docutils的标记语法，Docutils是Python世界的文档工具集，Sphinx可以生成多种格式，有默认的生成工程过程，可以用\`Makefile\`进行管理
[[m\_or\_r]](#m-or-r)

但是在Stackflow上还有人说因为在mac上有实时的预览工具而选择Markdown

[参考](#id25)
=============

[[m\_or\_r]](#id11)

[http://ieqi.net/2012/04/13/markdown-%E8%BF%98%E6%98%AF-restructuredtext/](http://ieqi.net/2012/04/13/markdown-%E8%BF%98%E6%98%AF-restructuredtext/)
