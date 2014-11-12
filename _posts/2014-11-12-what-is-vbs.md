---
layout: post
title: What is VBS?
---

VBScript是Visual Basic Script的简称，即 Visual Basic 描述语言，有时也被缩写为VBS。VBScript是微软开发的一种脚本语言，可以看作是VB语言的简化版，与VBA的关系也非常密切。它具有原语言容易学习的特性。目前这种语言广泛应用于网页和ASP程序制作，同时还可以直接作为一个可执行程序。用于调试简单的VB语句非常方便。
由于VBScript可以通过Windows脚本宿主调用COM，因而可以使用Windows操作系统中可以被使用的程序库，比如它可以使用Microsoft Office的库，尤其是使用Microsoft Access和Microsoft SQL Server的程序库，当然它也可以使用其它程序和操作系统本身的库。

在实践中VBScript一般被用在以下*三个方面*： 

##Windows操作系统
  VBScript可以被用来自动地完成重复性的Windows操作系统任务。在Windows操作系统中，VBScript可以在Windows Script Host的范围内运行。Windows操作系统可以自动辨认和执行*.VBS和*.WSF两种文件格式，此外Internet Explorer可以执行HTA和CHM文件格式。VBS和WSF文件完全是文字式的，它们只能通过少数几种对话窗口与用户通讯。HTA和CHM文件使用HTML格式，它们的程序码可以象HTML一样被编辑和检查。在WSF、HTA和CHM文件中VBScript和JavaScript的程序码可以任意混合。HTA文件实际上是加有VBS、JavaScript成分的HTML文件。CHM文件是一种在线帮助，用户可以使用专门的编辑程序将HTML程序编辑为CHM。 

##网页浏览器（客户方的VBS） 
  网页中的VBS可以用来指挥客户方的网页浏览器（浏览器执行VBS程序）。VBS与JavaScript在这一方面是竞争者，它们可以用来实现动态HTML，甚至可以将整个程序结合到网页中来。至今为止VBS在客户方面未能占优势，因为它只获得因为它只获得Microsoft Internet Explorer的支持（Mozilla Suite可以通过装置一个外挂来支持VBS）。而JavaScript则受到所有网页浏览器的支持。在Internet Explorer中VBS和JavaScript使用同样的权限，它们只能有限地使用Windows操作系统中的对象。 

##网页服务器（服务器方面的VBS）
  在网页服务器方面VBS是微软的Active Server Pages的一部分，它与JavaServer Pages和PHP是竞争对手。在这里VBS的程序码直接嵌入到HTML页内，这样的网页以ASP结尾。网页服务器Internet信息服务执行ASP页内的程序部分并将其结果转化为HTML传递给网页浏览器供用户使用。这样服务器可以进行数据库闻讯并将其结果放到HTML网页中。 

##VBScript主要的优点

由于VBScript由操作系统，而不是由网页浏览器解释，它的文件比较小、易学。 

在所有2000 / 98SE以后的Windows版本都可直接使用。 

可以使用其它程序和可使用的物件（尤其Microsoft Office）。 

##缺点

现在VBS无法作为电子邮件的附件了。Microsoft Outlook拒绝接受VBS为附件，收信人无法直接使用VBS附件。

1. [PHP, Javascript and VBScript Language Summary](http://phplens.com/phpeverywhere/node/view/30)
2. [VBScript是什么？有什么优缺点？](http://www.cnblogs.com/BeyondTechnology/archive/2011/01/09/1931508.html)

