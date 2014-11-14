---
layout: post
title: VB VBS VBA Excel Programming
---

> Difference on VB VBA and VBS. Some points on Excel programming.

## VB VBS VBA difference

你们怎么都在讲VB啊.其实VBA是VB的分支(应该可以这样比喻吧),vba是依附于其它应用程序的,因此,它也较vb提供了更多的对象.同时vba不需要vb环境.反之,如果vb要使用vba下的对象,却需要vba的环境(即必须安装应用程序).
vbs是解释型脚本语言,vb尽管也是解释型语言,但它仍是可以编绎的(即可以在没有vb的环境中运行).vba也不能编绎,这点与vbs一样.
vbs本身没有提供任何窗口型对象(vb和vba的那些控件,同样有窗口).vbs提供最简洁环境,与任何应用程序具有无关性.
熟悉了vbs,就熟悉了vba的语法,但vba的环境,远比vbs丰富得多.
个人意见,想到哪写到哪.欢迎指正.

## VBA Excel programming
直到90 年代早期,使应用程序自动化还是充满挑战性的领域.对每个需要自动化的应用程
序,人们不得不学习一种不同的**自动化语言**.例如:可以用EXCEL 的宏语言来使EXCEL 自动化,使
用WORDBASIC 使WORD 自动化,等等.微软决定让它开发出来的应用程序共享一种通用的自动化
语言--VisualBasicForApplication(VBA),可以认为VBA 是非常流行的应用程序开发
语言VASUALBASIC 的子集.实际上VBA 是"寄生于"VB 应用程序的版本.VBA 和VB 的区别包括如
下几个方面:

1. VB 是设计用于创建标准的应用程序,而VBA 是使已有的应用程序(EXCEL 等)自动化
2. VB 具有自己的开发环境,而VBA 必须寄生于已有的应用程序.
3. 要运行VB 开发的应用程序,用户不必安装VB,因为VB 开发出的应用程序是可执行文件
(*.EXE),而VBA 开发的程序必须依赖于它的"父"应用程序,例如EXCEL.

## 宏

指一系列EXCEL 能够执行的VBA语句。宏是由excel界面录制的（有后台代码）,实际还是保存成了vba代码，而vba是直接由代码编写的，宏的外观表现可由后台代码修改而变化。

宏和VBA是不一样的。宏只是VBA中最简单的应用。宏能够自动执行一系列操作，是因为其背后是一组VBA的程序代码，在录制宏的过程中，计算机把记录的操作都转换成了VBA代码。

Excel（包括其他Office 程序）允许用户录制一段宏，并将其记录为VBA 代码。对于开发者，使用这一功能，一方面可以节省时间，将录制的宏代码作为开发的基础，另一方面，对于不熟悉的操作，例如如何绘制图表，如何删除一行之类操作，可以录制一个宏并，通过查看其VBA 代码进行学习。录制的宏可以在VBA 集成开发环境（IDE）中修改编辑，可以为宏指定按钮、快捷键；而实际编写的代码也可以象宏一样在运行。

宏实际上就是一个简单的VBA的Sub过程，它保存在模块里，以Sub开头，以End Sub结尾，执行时就从第一句逐句执行，直到End Sub结束。

**录制宏的局限性**
 宏代码绝不等于VBA，它只是VBA里最简单的运用，尽管许多Excel过程都可以用录制宏来完成，但是通过宏代码还是无法完成许多的工作，如：
（1）不可以建立公式，函数；
（2）录制的宏没有判断或循环的功能；
（3）不能进行人机交互；人机交互能力差,即用户无法进行输入,计算机无法给出提示
（4）无法显示用户窗体；
（5）无法与其他软件或文件进行互动。

Refer to [什么是宏，什么是VBA，两者有区别吗？](http://t.excelhome.net/forum.php?mod=viewthread&tid=4172&highlight=&page=2)


## Excel Programming Basics

如何打开VBA IDE编辑器

1. 通过快捷键“ALT + F11”打开VB编辑器IDE
2. 也可以通过Developer Mode打开VB和Macro
3. 还可以在邮件点击work shett，选择View Code

VBA中给变量赋值使用set和不使用set的区别
'给普通变量赋值使用LET ，只是LET 可以省略。
'给对象变量赋值使用SET，SET 不能省略。例如ADODB.RecordSet等对象。

{% highlight vb.net linenos %}
Sub AA()
    Dim arr As String
    arr = "hello" '本句也可写成LET arr = "hello"
End Sub
Sub bb()
    Dim arr As String
    Set arr = "hello"  ' 这样写是错误的。
End Sub
{% endhighlight %}

VBA中变量用dim定义和不用dim定义而直接使用有何区别？
'DIM 语句 的作用似乎声明变量并分配存储空间。
'如果不指定数据类型或对象类型，也就是不用DIM定义，且在模块中没有 Deftype 语句，
'则该变量按缺省设置是 Variant 类型。