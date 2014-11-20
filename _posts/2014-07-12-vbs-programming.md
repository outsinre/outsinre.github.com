---
layout: post
title: VBS Programing Tutorial
---

<div class="message">
一个很详尽的VBS介绍。
</div>

Vbs脚本编程简明教程之一—为什么要使用Vbs？

在Windows中，学习计算机操作也许很简单，但是很多计算机工作是重复性劳动，例如你每周也许需要对一些计算机文件进行复制、粘贴、改名、删除，也许你每天启动计算机第一件事情就是打开WORD，切换到你喜爱的输入法进行文本编辑，同时还要播放优美的音乐给工作创造一个舒心的环境，当然也有可能你经常需要对文本中的某些数据进行整理，把各式各样的数据按照某种规则排列起来……。这些事情重复、琐碎，使人容易疲劳。

第三方软件也许可以强化计算机的某些功能，但是解决这些重复劳动往往事倍功半，我也尝试过使用计算机语言编写程序来解决这些问题，但是随之而来的命令、语法、算法、系统框架和类库常常让我觉得这样是否有必要，难道就是因为猪毛比较难拔，所以我就要去学习机械，为自己设计一个拔猪毛机(?)吗？

Vbs是一种Windows脚本，它的全称是:Microsoft Visual Basic Script Editon.(微软公司可视化BASIC脚本版)，VBS是Visual Basic的的一个抽象子集，是系统内置的，用它编写的脚本代码不能编译成二进制文件，直接由Windows系统执行（实际是一个叫做宿主host的解释源代码并执行），高效、易学，但是大部分高级语言能干的事情，它基本上都具备，它可以使各种各样的任务自动化，可以使你从重复琐碎的工作中解脱出来，极大的提高工作效率。

我个人认为Vbs脚本其实就是一种计算机编程语言，但是由于缺少计算机程序设计语言中的部分要素，对于事件的描述能力较弱，所以称为脚本，它最方便的地方就是提供了对COM对象的简便支持。那么什么是COM对象呢？

我这样理解，COM对象就是一些具有特定函数功能项程序模块，他们一般以ocx或者dll作为扩展名，你只要找到包含有你需要的功能的模块文件，并在脚本中规范的引用，就可以实现特定的功能，也就是说Vbs脚本就是调用现成的“控件”作为对象，用对象的属性和方法实现目的，完全免去了编写代码、设计算法等等麻烦。说白了，我不是觉得拔猪毛麻烦么？我发觉xx机（比如真空离心器）有一个功能可以实现脱毛，ok，我把它拿来给猪脱毛。什么？大材小用？太浪费资源了？天哪，那是计算机芯片的事情，死道友不死贫道，反正我的事情是方便快速的解决了，这就行了。

最方便的是它甚至不需要专门的开发环境，在你的计算机中，只要有notepad，就可以编写Vbs脚本了，并且可以直接执行。

Vbs脚本编程简明教程之二—如何开始第一个Vbs脚本？

就像多数计算机教程一样，我们从“Hello World！”程序开始我们的练习。什么？不知道是什么意思？就是说大部分的计算机程序设计教程开篇入门都是编写一个小程序，执行这个程序的结果就是在计算机的屏幕上或者dos窗口中显示一行文字：Hello World！好了，我们开始吧。

打开你的“记事本”程序，在编辑窗口填写：

msgbox "Hello World!"

然后用鼠标单击“文件”菜单，单击“保存”，把“保存在”一栏设为桌面，在“文件名”一栏中填写kk.vbs，单击“保存”就可以了。然后最小化“记事本”窗口，在桌面上寻找你刚刚保存的kk.vbs，然后双击。看到弹出的对话框了没有，单击“确定”，对话框消失了。难看了点，不过确实是你编写的第一个脚本程序。

说明之一：上面的操作中，保存位置放在桌面，仅仅是为了执行方便，你保存到其他的地方完全没有问题，只要你知道你保存在什么地方就可以了，什么？是废话，自己保存的当然知道保存在那里了。不，自己保存的文件自己找不到的人我见的多了去了。文件名你可以随意填写，不一定非要写kk，只要符合Windows的文件命名规则就可以了，但是扩展名必须是vbs，什么?不知道什么是扩展名？就是文件名中“.”后的那部分，简单说，就是vbs脚本文件命名时必须是：xxx.vbs，其中xxx你随意。

说明之二：在记事本编辑窗口中写的这行是什么意思？

Msgbox是VBS内建的函数，每一个函数都可以完成一定的功能，你只需要按照语法要求，在函数的相应部分填写相应的内容就可以了，这部分内容我们称为参数，当然函数执行的结果我们称为返回值，一个函数可以有返回值也可以没有，可以有参数也可以没有。你不用了解函数是怎么运作的，只要了解这个函数能干什么就行了。

Msgbox语法：msgbox "对话框内容", , "对话框的标题"

你不妨用记事本打开刚才的文件在编辑窗口中输入：

msgbox "Hello World!" , , "系统提示"

执行一下，看看效果和位置。

说明之三：如果执行失败，看看你的标点符号，所有的标点符号必须是在英文状态下输入的。

当然，这个脚本实在是太简单了，甚至连最简单的交互都没有，所以你可以把脚本这样修改一下：

Dim name

name=Inputbox("请输入你的名字:","名称")

Msgbox name, , "您的名字是"

保存执行一下，看到弹出的对话框了么？填入你的名字，点确定，看到结果了吗？

说明之一：第一句是定义变量，dim是定义变量的语句

其格式为：dim 变量1,变量2……，

Vbs只有一种变量类型，所以不用声明变量类型。系统会自动分辨变量类型。

说明之二：inputbox是VBS内建的函数，可以接受输入的内容，其语法格式为：

Inputbox("对话框内容","对话框标题")

第二句的意思是接受用户的输入，并把输入结果传递给变量name。

好了，到此脚本基本的输入输出函数都有了，已经可以完成一些比较简单的功能了，你可以编写一个简单的脚本，然后拷贝的“程序”—>“启动”中，然后重新启动计算机看看结果。

Vbs脚本编程简明教程之三—Vbs的基本语法（牢记）

VBScript基础知识

一、变量

1、所有单引号后面的内容都被解释为注释。

2、在VBScript中，变量的命名规则遵循标准的命名规则，需要注意的是：在VBScript中对变量、方法、函数和对象的引用是不区分大小写的。在申明变量时，要显式地申明一个变量，需要使用关键字DIm来告诉VBScript你要创建一个变量，并将变量名称跟在其后。申明多个同类型变量，可以用逗号分隔。注意：VBScript中不允许在申明变量的时候同时给变量赋值。但是允许在一行代码内同时对两个变量进行赋值，中间用冒号分隔。

3、你可以使用OptionExplicit来告诉宿主变量必须先声明后使用。

4、VBScript在定义时只有一种变量类型，在实际使用中需要使用类型转换函数来将变量转换成相应的变量类型。

Cbool函数将变量转换成布尔值；

Cbyte函数将变量转换为0到255之间的整数。

Ccur函数、Cdbl函数和Csng函数将变量转换为浮点数值，前者只精确到小数点后四位，后两者要更加精确，数值的范围也要大的多。

Cdate函数将变量转换为日期值。

Cint函数和Clng函数将变量转换为整数，后者的范围比前者要大的多。

Cstr函数将变量转换为字符串。

二、数组

数组的定义与变量非常类似，只需要在变量后描述这个数组的个数和维数。需要注意的是：数组的下标总是从0开始，而以数组定义中数值减一结束。也就是说你以要定义一个有十个数据的数组，将这样书写代码：dImarray（9），同样，当你要访问第五个元素时，实际的代码是array(4)。当然，你可以通过不指定数组的个数和维数来申明动态数组。等到数组的个数和维数固定后，使用关键字redim来改变数组。注意，在改变数组的大小时，数组的数据会被破坏，使用关键字preserve来保护数据。例如：

RedIm空格preserve空格array括号个数逗号维数括号

三、操作符

在VBScript运算符中，加减乘除都是我们常用的符号，乘方使用的是 ^ ，取模使用的Mod。

在比较操作符中，等于、小于、大于、小于等于、大于等于都与我们常用的符号是一致的，而不等于是小于和大于连用。

逻辑运算符为：和操作—>AND     非操作—>NOT     或操作—>OR；

你可以使用操作符 + 和操作符 & 来连接字符串，一般使用&操作符；

另外还有一个比较特殊的操作符Is用来比较对象，例如按钮对象，如果对象是同一类型，结果就是真，如果对象不是同一类型，结果就是假。

四、条件语句主要有if……then语句和selectcase语句两种形式

在if……then语句中，其基本形式为：

If 条件 then

处理条件的语句；

……

Endif

基本形式只能对单个条件进行验证，如果有两个条件，则需要在基本形式中添加单行语句else，如果还有更多的条件需要验证，则需要添加语句

Elseif 条件 then

处理条件语句

在selectcase语句中，其基本形式为：

Select case 变量

Case 条件值

处理条件语句

并对上两句进行重复

最后一句应为

case else

处理语句

当然不要忘记将条件结束语句End select放在最后一行

注意：在执行字符串比较时，需要特别注意大小写，一般情况下，我们在比较前，使用lcase函数将字符串转换成小写，使用ucase函数将字符串转换成大写大写。

五、循环控制语句

循环控制语句有for……next循环、for……each循环、do……while循环、do……until循环、while循环五种形式。

在使用循环控制语句前，首先要对循环条件进行判断，如果循环次数是有固定次数的，那么使用For……next循环，其结构为：

For   计数器变量＝开始计数值 to 最后计数值

执行循环体

Next

如果是需要对数组或对象集合中的每一个元素进行判断，则需要使用for……each循环，其结构为：

For each 循环计数变量 in 要查看的对象或数组

执行处理语句

Next

注意：在上述两种循环中随时可以使用exit for来退出循环

如果你希望在条件满足时执行一段代码则使用do……while语句，结构为：

Do while 条件

执行循环体

Loop

如果你希望在条件不满足时执行代码，则使用do……until语句，结构为：

Do　until　条件

执行循环体

Loop

当然，在这两种循环语句中，你可以使用exit do来退出循环

最后一种循环语句是条件满足时一直执行循环，

While 条件

执行循环体

Wend

六、使用过程

常用的过程有两种，一种为函数，给调用者返回值，一种为子程序，无返回值，还有一种叫事件的特殊子程序，用的比较少。

函数的基本定义方法为：

Function 函数名称（参数列表）

函数代码

函数名称＝某值‘用来返回值

end function

子程序一些都类似，不过没有返回值

注意：尽管在定义子程序的时候，参数列表要加括号，但在调用子程序的时候，参数列表不加括号，括号只在函数中使用。另外，子程序不能在表达式中使用。

而函数只能出现在赋值语句的右边，或者表达式中，函数不能直接使用，如果必须直接使用函数，则必须使用call语句调用，并取消返回值。

Vbs脚本编程简明教程之四—如何利用Vbs运行外部程序？

Vbs只提供了编程的一个基本框架，用户可以使用Vbs来定义变量、过程和函数，vbs也提供了一些内部函数和对象，但是Vbs没有提供任何命令来访问Windows系统内部的部件，但是值得庆幸的是，Vbs虽然不能自己完成这些任务，但是它提供了一条极为方便、功能也相当强的命令——CreateObject，这条命令可以访问windows系统内安装的所有com对象，并且可以调用这些部件中存放的命令。

于是问题解决了，比如说，我手头有1000个小文本，我首先要对每一个文本的语法进行查错和修改，然后按照预先定义好的规则对这些文本进行排序，最后将这些文本合并成为一个文件。正常情况下，我们需要把打开第一个小文本，然后把它复制到WORD中，然后利用里面的除错功能进行除错和修改，然后再导入到EXCEL中进行排序，将这个过程重复1000遍，然后再将所有得到的文本复制到一个大文本中。实在是太枯燥、工作量太大了。有了Vbs和CreateObject，问题得到解决，我只需要找到相应的模块，调用相应的功能就可以了，作为脚本，把一个枯燥的过程重复1000次，本就是它的拿手好戏。

好了，我们走入正题，从最简单的——只启动一个程序开始。

WSH也就是用来解析Vbs的宿主，本身包含了几个个常用对象：

1、Scripting.FileSystemObject —> 提供一整套文件系统操作函数

2、Scripting.Dictionary —> 用来返回存放键值对的字典对象

3、Wscript.Shell —> 提供一套读取系统信息的函数，如读写注册表、查找指定文件的路径、读取DOS环境变量，读取链接中的设置

4、Wscript.NetWork —>

提供网络连接和远程打印机管理的函数。（其中，所有Scripting对象都存放在SCRRUN.DLL文件中，所有的Wscript对象都存放在WSHOM.ocx文件中。）

现在我们需要的是第三个对象，好了，让我们先连接一下对象看看，在记事本的编辑窗口中输入：

Set objShell = CreateObject(“Wscript.Shell”)

objShell.Run “notepad”

同样，保存执行。那么看到了一个什么样的结果呢？在桌面上又打开了一个记事本。

说明之一：Set是Vbs指令，凡是将一对象引用赋给变量，就需要使用set关键字。那么什么是对象引用呢？凡是字符串、数值、布尔值之外的变量都是对象引用。Objshell是变量名，可以随意修改。

说明之二：反是正确引用的对象，其本身内置有函数和变量，其引用方法为在变量后加“. ”，后紧跟其实现功能的函数就可以了。Objshell.run

的意思就是调用Wscript.shell中的运行外部程序的函数——run，notepad是记事本程序的文件名。当然你也可以改成“calc”，这是计算器的文件名，winword是word的文件名，等等吧，所有可执行文件的文件名都可以。但是需要注意的是，如果你要执行的可执行文件存放的地方不是程序安装的常用路径，一般情况下，需要提供合法的路径名，但是run在运行解析时，遇到空格会停止，解决的方法是使用双引号，例如：在我的机器上运行qq，代码为：

objshell.run """C:"Program Files"QQ2006"QQ.exe""" ‘注：三个引号

好，我们再进一步，启动两个程序会如何呢？

输入如下代码：

Set objShell = CreateObject(“Wscript.Shell”)

objShell.Run “notepad”

objShell.Run “calc”

执行会如何呢？两个程序基本上同时启动了。如果我们需要先启动notepad再启动calc将如何呢？很简单在需要顺序执行的代码后加 , , True参数就可以了。

好了输入代码：

Set objShell = CreateObject(“Wscript.Shell”)

objShell.Run “notepad” ,,true

objShell.Run “calc”

看看执行的结果怎么样吧！

总结：run函数有三个参数，第一个参数是你要执行的程序的路径，第二个程序是窗口的形式，0是在后台运行；1表示正常运行；2表示激活程序并且显示为最小化；3表示激活程序并且显示为最大化；一共有10个这样的参数我只列出了4个最常用的。

第三个参数是表示这个脚本是等待还是继续执行，如果设为了true,脚本就会等待调用的程序退出后再向后执行。

其实，run做为函数，前面还有一个接受返回值的变量，一般来说如果返回为0，表示成功执行，如果不为0，则这个返回值就是错误代码，可以通过这个代码找出相应的错误。

Vbs脚本编程简明教程之五—错误处理

引发错误的原因有很多，例如用户输入了错误类型的值，或者脚本找不到必需的文件、目录或者驱动器，我们可以使用循环技术来处理错误，但是VBS本身也提供了一些基本技术来进行错误的检测和处理。

1、最常见的错误是运行时错误，也就是说错误在脚本正在运行的时候发生，是脚本试图进行非法操作的结果。例如零被作为除数。在vbs中，任何运行时错误都是致命的，此时，脚本将停止运行，并在屏幕上显示一个错误消息。你可以在脚本的开头添加

On Error Resume Next

这行语句可以告诉vbs在运行时跳过发生错误的语句，紧接着执行跟在它后面的语句。

发生错误时，该语句将会把相关的错误号、错误描述和相关源代码压入错误堆栈。

2、虽然On Error Resume

Next语句可以防止vbs脚本在发生错误时停止运行，但是它并不能真正处理错误，要处理错误，你需要在脚本中增加一些语句，用来检查错误条件并在错误发生时处理它。

vbscript提供了一个对象err对象，他有两个方法clear，raise，5个属性：description，helpcontext，helpfile，number，source

err对象不用引用实例，可以直接使用，例如：

on error resume next

a=11

b=0

c=a/b

if err.number<>0 then

wscript.echo err.number & err.description & err.source

end if

Vbs脚本编程简明教程之六—修改注册表

Vbs中修改注册表的语句主要有：

1、读注册表的关键词和值：

可以通过把关键词的完整路径传递给wshshell对象的regread方法。例如：

set ws=wscript.createobject("wscript.shell")

v=ws.regread("HKEY_LOCAL_MACHINE"SOFTWARE"Microsoft"Windows"CurrentVersion"Run"nwiz")

wscript.echo v

2、写注册表

使用wshshell对象的regwrite方法。例子：

path="HKEY_LOCAL_MACHINE"SOFTWARE"Microsoft"Windows"CurrentVersion"Run""

set ws=wscript.createobject("wscript.shell")

t=ws.regwrite(path & "jj","hello")

这样就把

HKEY_LOCAL_MACHINE"SOFTWARE"Microsoft"Windows"CurrentVersion"Run"jj这个键值改成了hello.不过要注意：这个键值一定要预先存在。

如果要创建一个新的关键词，同样也是用这个方法。

path="HKEY_LOCAL_MACHINE"SOFTWARE"Microsoft"Windows"CurrentVersion"run"sssa2000"love""

set ws=wscript.createobject("wscript.shell")

val=ws.regwrite(path,"nenboy")

val=ws.regread(path)

wscript.echo val

 

删除关键字和值

使用regdelete方法，把完整的路径传递给regdelete就可以了

例如

val=ws.regdel(path)

注意，如果要删除关键词的值的话一定要在路径最后加上“"”，如果不加斜线，就会删除整个关键词。

当然，从现在的角度看，还是使用WMI的注册表处理功能也许更好些。

Vbs脚本编程简明教程之七—FSO的常见对象和方法

文件系统是所有操作系统最重要的部分之一，脚本经常会需要对文件及文件夹进行访问和管理，在Vbs中对桌面和文件系统进行访问的顶级对象是FileSystemObject(FSO)，这个对象特别复杂，是vbs进行文件操作的核心。此节内容应了如指掌。

FSO包含的常见对象有：

Drive对象：包含储存设备的信息，包括硬盘、光驱、ram盘、网络驱动器

Drives集合：提供一个物理和逻辑驱动器的列表

File 对象：检查和处理文件

Files 集合：提供一个文件夹中的文件列表

Folder对象：检查和处理文件夹

Folders集合：提供文件夹中子文件夹的列表

Textstream对象：读写文本文件

FSO的常见方法有：

BulidPath：把文件路径信息添加到现有的文件路径上

CopyFile：复制文件

CopyFolder：复制文件夹

CreateFolder：创建文件夹

CreateTextFile：创建文本并返回一个TextStream对象

DeleteFile：删除文件

DeleteFolder：删除文件夹及其中所有内容

DriveExits：确定驱动器是否存在

FileExits：确定一个文件是否存在

FolderExists：确定某文件夹是否存在

GetAbsolutePathName：返回一个文件夹或文件的绝对路径

GetBaseName：返回一个文件或文件夹的基本路径

GetDrive：返回一个dreve对象

GetDriveName：返回一个驱动器的名字

GetExtensionName：返回扩展名

GetFile：返回一个file对象

GetFileName：返回文件夹中文件名称

GetFolder：返回一个文件夹对象

GetParentFolderName：返回一个文件夹的父文件夹

GetSpecialFolder:返回指向一个特殊文件夹的对象指针

GetTempName:返回一个可以被createtextfile使用的随机产生的文件或文件夹的名称

MoveFile：移动文件

MoveFolder：移动文件夹

OpenTextFile：打开一个存在的文件并返回一个TextStream对象

Vbs脚本编程简明教程之八—FSO中文件夹的基本操作

1、使用fso

由于fso不是wsh的一部分，所以我们需要建立他的模型

例如

set fs=wscript.createobject(“scripting.filesystemobject”)

样就建立了fso的模型。如果要释放的话也很简单，

set fs=nothing

2、使用文件夹

在创建前，我们一般需要检查该文件夹是否存在例如：

dim fs,s //定义fs、s两个变量

set fs=wscript.createobject(“scripting.filesystemobject”) //fs为FSO实例

if (fs.folderexists(“c:"temp”)) then //判断c:"temp文件夹是否存在

s=”is available”

else

s=”not exist”

set foldr=fs.createfolder(“c:"temp”) //不存在则建立

end if

 

拷贝：

注意：如果c:"data 和d:"data都存在，脚本会出错，复制也就会停止，如果要强制覆盖，使用fs.copyfolder “c:"data”

“d:"data”，true

 

移动：

set fs=wscript.createobject(“scripting.filesystemobject”)

fs.movefolder “c:"data” “d:"data”

 

我们可以使用统配符，来方便操作：

例如， fs.movefolder :c:"data"te*” , “d:"working”

注意：在目的路径最后没有使用“"”也就是说我没有这样写：

fs.movefolder c:"data"te*” , “d:"working"”

这样写的话，如果d:"working 目录不存在，windows就不会为我们自动创建这个目录。

 

注意：上面我们所举的例子都是在利用fso提供的方法，如果使用folder对象也完全是可以的：

set fs= wscript.createobject(“scripting.filesystemobject”)

set f=fs.getfolder(“c:"data”)

f.delete //删除文件夹c:"data。如果有子目录，也会被删除

f.copy “d:"working”,true    //拷贝到d:"working

f.move “d:"temp”    //移动到d:"temp

一般指的就是系统文件夹："windows"system32，

临时文件夹，windows文件夹，在前几篇的时候，我们提过一下：例如

set fs=wscript.createobject(“scripting.filesystemobject”)

set wshshell=wscript.createobject(“wscript.shell”)

osdir=wshshell.expandenvironmentstrings(“%systemroot%”)

set f =fs.getfolder(osdir)

wscript.echo f

这个方法使用3种值：

0 表示windows文件夹，相关常量是windowsfolder

1 系统文件夹，相关常量是systemfolder

2 临时目录，相关常量temporaryfolder

例如：

set fs=wscript.createobject(“scripting.filesystemobject”)

set wfolder=fs.getspecialfolder(0) ‘返回windows目录

set wfolder=fs.getspecialfolder(1) ‘返回system32"

set wfolder=fs.getspecialfolder(2)'返回临时目录

当然，还有简单的方法那就是使用getspecialfolder() 3、特殊文件夹

set fs=wscript.createobject(“scripting.filesystemobject”)

fs.copyfolder “c:"data” “d:"data”

set fs=wscript.createobject(“scripting.filesystemobject”)

fs.deletefolder(“c:"windows”)

删除：

Vbs脚本编程简明教程之九—妙用SendKeys简化重复操作

每次开机的时候，你想自动登陆你的QQ或者博客吗？巧妙使用VBS中的SendKeys命令（这个命令的作用就是模拟键盘操作，将一个或多个按键指令发送到指定Windows窗口来控制应用程序运行），可以极大的方便我们的常用操作。其使用格式为：

Object.SendKeys string其中：

Object：为WshShell对象，即脚本的第一行为：

Set WshShell=WScript.CreateObject("WScript.Shell")

将Object替换为WshShell

“string”：表示要发送的按键指令字符串，需要放在英文双引号中。它包含如下内容：

1．基本键：一般来说，要发送的按键指令都可以直接用该按键字符本身来表示，例如要发送字母“x”，使用“WshShell.SendKeys

"x"”即可。当然，也可直接发送多个按键指令，只需要将按键字符按顺序排列在一起即可，例如，要发送按键“cfan”，可以使用

“WshShell.SendKeys "cfan"”。

2．特殊功能键：对于需要与Shift、Ctrl、Alt三个控制键组合的按键，SendKeys使用特殊字符来表示：Shift — +；Ctrl   —  ^；Alt — %

如要发送的组合按键是同时按下Ctrl＋E，需要用WshShell.SendKeys

"^e"”表示，如果要发送的组合按键是按住Ctrl键的同时按下E与C两个键，这时应使用小括号把字母键括起来，书写格式为“WshShell.SendKeys

"^(ec)"”，这里要注意它与“WshShell.SendKeys

"^ec"”的区别，后者表示组合按键是同时按住Ctrl和E键，然后松开Ctrl键，单独按下“C”字母键。

由于“+”、“^”这些字符用来表示特殊的控制按键了，如何表示这些按键呢？只要用大括号括住这些字符即可。例如，要发送加号“+”，可使用“WshShell.SendKeys

"{+}"”。另外对于一些不会生成字符的控制功能按键，也同样需要使用大括号括起来按键的名称，例如要发送回车键，需要用“WshShell.SendKeys

"{ENTER}"”表示，发送向下的方向键用

“WshShell.SendKeys "{DOWN}"”表示。

如果需要发送多个重复的单字母按键，不必重复输入该字母，SendKeys允许使用简化格式进行描述，使用格式为“{按键

数字}”。例如要发送10个字母“x”，则输入“WshShell.SendKeys "{x 10}"”即可。

例一：WshShell.SendKeys "^{ESC}u"

代码的含义为：按下Ctrl＋Esc组合键（相当于按Win键）打开“开始”菜单，接着按U键打开“关机”菜单。

例二：让VBS脚本自动在记事本中输入一行文字“hello, welcome to cfan”。

Dim WshShell

Set WshShell=WScript.CreateObject("WScript.Shell")

WshShell.Run "notepad"

WScript.Sleep 2000  

//本行的含义为是脚本暂停2秒，给notepad一个打开的时间，有时时间太短可能导致后面的字符无法进入编辑区

WshShell.AppActivate "无标题 - 记事本

"//AppActivate为寻找可执行程序的标题框，”无标题－记事本”内容你的自己打开看一下

WshShell.SendKeys "hello, welcome to cfan"

作业1:让脚本自动输入下面两段小短句

This is the most wonderful day of my life

because I'm here with you now

作业2：让脚本在输入短句后自动关闭记事本，并保存文件名为“test”，注意关闭记事本可以直接使用组合按键Alt＋F4来实现。

例三：制作能自动定时存盘的记事本

我们最常用的记事本没有Word、WPS那样的自动定时存盘功能，其实利用VBS脚本再加上SendKeys命令，就能弥补这个遗憾。打开记事本，输入以下内容（为容易描述和分析，把代码分为四个部分）：

'第一部分：定义变量和对象

Dim WshShell, AutoSaveTime, TXTFileName

AutoSaveTime=300000

Set WshShell=WScript.CreateObject("WScript.Shell")

TXTFileName=InputBox("请输入你要创建的文件名(不能用中文和纯数字)：")

'第二部分：打开并激活记事本

WshShell.Run "notepad"

WScript.Sleep 200

WshShell.AppActivate "无标题 - 记事本"

'第三部分：用输入的文件名存盘

WshShell.SendKeys "^s"

WScript.Sleep 300

WshShell.SendKeys TXTFileName

WScript.Sleep 300

WshShell.SendKeys "%s"

WScript.Sleep AutoSaveTime

'第四部分：自动定时存盘

While WshShell.AppActivate (TXTFileName)=True

WshShell.SendKeys "^s"

WScript.Sleep AutoSaveTime

Wend

WScript.Quit

将其保存为记事本.vbs，以后要使用记事本时，都通过双击这个脚本文件来打开。

程序说明：这个脚本的基本思路是定时向记事本发送Ctrl＋S这个存盘组合键。

第一部分：定义了脚本中需要用到的变量和对象。“AutoSaveTime”变量用来设置自动存盘间隔，单位为毫秒，这里设置为5分钟。“TXTFileName”变量通过输入框取得你要创建的文本文件名称。

第二部分：运行记事本，对于Windows本身提供的程序，比如计算器等，可直接在“WshShell.Run”后输入程序名称，如"calc"，对于非系统程序，则可输入完全路径，但要注意使用8.3格式输入，比如“"D:"Progra~1"Tencent"QQ.exe"”。

第三部分：这里用SendKeys命令执行了这样的操作流程（请注意每个操作之间延时命令的使用）：在记事本中按Ctrl＋S组合键→弹出保存文件的窗口→输入文件名→按Alt＋S组合键进行保存（默认保存在“我的文档”目录）。

第四部分：定时存盘的关键，通过“While……Wend”这个当条件为“真”时循环命令，实现自动存盘代码“WshShell.SendKeys

"^s"”和定时代码“WScript.Sleep

AutoSaveTime”的重复执行。因为不能让这个定时存盘循环一直执行，退出记事本后，必须自动退出脚本并结束循环，所以设计了一个循环判断条件“WshShell.AppActivate

TXTFileName=True”，当记事本运行中时，可以激活记事本窗口，这个条件运行结果为“True”，定时存盘循环一直执行，退出记事本后，脚本无法激活记事本窗口，就会跳出循环，执行“Wend”后面的“WScript.Quit”退出脚本。

例四：关机菜单立刻显身

打开记事本，输入以下命令，并将其保存为1.vbs：

set WshShell = CreateObject("WScript.Shell")

WshShell.SendKeys "^{ESC}u"

双击运行它，你会发现关机菜单立刻出现了。

将“WshShell.SendKeys "^{ESC}u"”改为“WshShell.SendKeys "^+{ESC}"”，运行一下看看是否打开了任务管理器。

让我们举个例子利用SendKeys自动上网并登陆博客

将下面的脚本复制到一个文本文件中，并将其文件名命名为：自动登陆.vbs，然后将拨号软件及本脚本一起复制到程序——启动项中，就可以实现自动拨号上网，并登陆到博客上。

代码如下：

Set wshshell=CreateObject("wscript.shell")

wshshell.AppActivate "连接 MAE-301U 拨号连接"

wscript.Sleep 20000

wshshell.SendKeys "{enter}"

wshshell.Run "iexplore"

WScript.Sleep 2000

wshshell.AppActivate "hao123网址之家---实用网址,搜索大全,尽在www.hao123.com - Microsoft

Internet Explorer" '引号中的内容修改为你的浏览器打开后标题栏中的内容

wshshell.SendKeys "%d"

wshshell.SendKeys "http://passport.baidu.com/?login"

wshshell.SendKeys "{enter}"

WScript.Sleep 2000

wshshell.SendKeys "此处修改为博客帐号"

wshshell.SendKeys "{tab}"

wshshell.SendKeys "此处修改为博客密码"

wshshell.SendKeys "{enter}"

Vbs脚本编程简明教程之十 —— Vbs脚本编程常用的编辑器

Vbs脚本常用的编辑器当然是notapad，不过这个编辑器的功能当然实在是太弱了一点，其实有很多的专用的脚本编辑器可以大大方便vbs脚本的编写。我常用的有两种：

1、VBSEDit汉化版

2、primalscript汉化版，可以对30多种脚本进行编辑

Vbs脚本编程简明教程之十一 ——FSO中文件的基本操作

一、文件属性：

在windows中，文件的属性一般用数字来表示：

0代表normal，即普通文件未设置任何属性。   1代表只读文件。

2代表隐藏文件。   4代表系统文件。   16代表文件夹或目录。

32代表存档文件。 1024代表链接或快捷方式。例如：

set fs=wscript.createobject(“scripting.filesystemobject”)

set f=fs.getfile(“d:"index.txt”)

msgbox f.Attributes ‘attributes函数的作用是显示文件属性

需要说明的是：msgbox显示的结果往往不是上面说明的数字，而是有关属性代表数字的和。

二、创建文件：object.createtextfile方法，注意创建前一般需要检查文件是否存在。

例如：set fso=wscript.createobject(“scripting.filesystemobject”)

if fso.fileexists(“c:"kk.txt”) then

msgbox “文件已存在”

else

set f=fso.createtextfile(“c:"kk.txt”)

end if

如需要强制覆盖已存在的文件，则在文件名后加true参数。

三、复制、移动、删除文件：使用copyfile方法、movefile方法、deletefile方法。例如：

set fso=wscript.createobject(“scripting.filesystemobject”)

fso.copyfile “c:"kk.txt”,”d:"1"kk.txt”,true   //如上文说述，true代表强制覆盖

fso.movefile “c:"kk.txt”, “d:"” //移动文件

fso.deletefile “c:"kk.txt” //删除文件

四、文件的读写：

1、打开文件：使用opentextfile方法

如：set ts=fso.opentextfile(“c:"kk.txt”,1,true)

说明：第二个参数为访问模式1为只读、2写入、8为追加

第三个参数指定如文件不存在则创建。

2、读取文件：read(x)读x个字符；readline读一行；readall全部读取

如：set ffile=fso.opentextfile(“c:"kk.txt”,1,true)

value=ffile.read(20)

line=ffile.readline

contents=ffile.readall

3、常见的指针变量：

atendofstream属性：当处于文件结尾的时候这个属性返回true。一般用循环检测是否到达文件末尾。例如：

do while ffile.atendofstream<>true

ffile.read(10)

loop

atendofline属性：如果已经到了行末尾，这个属性返回true。

Column属性(当前字符位置的列号)和line属性(文件当前行号)：在打开一个文件后，行和列指针都被设置为1。

4、在文件中跳行：skip(x) 跳过x个字符；skipline 跳过一行

5、在文件中写入字符：可以用2－写入和8－追加的方式来写入

其方法有：write(x)写入x字符串；writeline(x)写入x代表的一行

writeblanklines(n) 写入n个空行

注意：最后一定要使用close方法关闭文件。读文件后一定要关闭，才能以写的方式打开。

Vbs脚本编程简明教程之十二—使用系统对话框

在VBS脚本设计中，如果能使用windows提供的系统对话框，可以简化脚本的使用难度，使脚本人性化许多，很少有人使用，但VBS并非不能实现这样的功能，方法当然还是利用COM对象。

1、SAFRCFileDlg.FileSave对象：属性有：FileName —指定默认文件名；FileType —

指定文件扩展名；OpenFileSaveDlg —显示文件保存框体方法。

2、SAFRCFileDlg.FileOpen 对象：FileName —默认文件名属性；OpenFileOpenDlg —显示打开文件框体方法。

3、UserAccounts.CommonDialog对象：Filter —扩展名属性（"vbs File|*.vbs|All Files|*.*"）；

FilterIndex —指定

InitialDir —指定默认的文件夹

FileName —指定的文件名

Flags —对话框的类型

Showopen方法：

很简单，ok，让我们来举两个简单的例子：

例一：保存文件

Set objDialog = CreateObject("SAFRCFileDlg.FileSave")

Set objFSO = CreateObject("Scripting.FileSystemObject")

objDialog.FileName = "test"

objDialog.FileType = ".txt"

intReturn = objDialog.OpenFileSaveDlg

If intReturn Then

objFSO.CreateTextFile(objDialog.FileName & objdialog.filetype)

Else

Wscript.Quit

End If

注意：1、SAFRCFileDlg.FileSave对象仅仅是提供了一个方便用户选择的界面，本身并没有保存文件的功能，保存文件还需要使用FSO对象来完成。2、用FileType属性来指定默认的文件类型。3、在调用OpenFileSaveDlg方法时，最好把返回值保存到一变量中，用它可以判断用户按下的是确定还是取消。

例二：.打开文件

set objFile = CreateObject("SAFRCFileDlg.FileOpen")

intRet = objFile.OpenFileOpenDlg

if intret then

msgbox “文件打开成功！文件名为：” & objFile.filename

else

wscript.quit

end if

例三：比较复杂的打开文件对话框

Set objDialog = CreateObject("UserAccounts.CommonDialog")

objDialog.Filter = "vbs File|*.vbs"

objDialog.InitialDir = "c:""

tfile=objDialog.ShowOpen

if tfile then

strLoadFile = objDialog.FileName

msgbox strLoadFile

else

wscript.quit

end if

说明：在脚本中加入 objDialog.Flags = &H020 看看会出现什么结果。

Vbs脚本编程简明教程之十三 —使用dictionary对象

VBS中存在一个特殊的对象－dictionnary，是一个集合对象。一般情况霞，我把这个特殊的集合想象为数组，可以使用其中内建的函数完成存储和操纵数据等基本任务，无须担心数据是在哪些行列，而是使用唯一的键进行访问或者是一个只能运行在内存中的数据库，并只有两个字段分别是：key和item，在使用中，字段key是索引字段。

set sdict=CreateObject("Scripting.Dictionary")

sdict.add "a","apple"

sdict.add "b","banana"

sdict.add "c","copy"

for each key in sdict.keys

msgbox     "键名" &   key     & "是" & " = " & sdict (key)

next

sdict.removeall

这个脚本很简单，就是定义了一个 dictionary 对象的实例sdict，并加入了三条数据，然后对每一条数据进行了枚举，最后，将对象的实例清空。

Dictionary 对象的成员概要

属性和说明

CompareMode    设定或返回键的字符串比较模式

Count     只读。返回 Dictionary 里的键/条目对的数量

Item(key) 设定或返回指定的键的条目值

Key(key) 设定键值

方法和说明

Add(key,item) 增加键/条目对到 Dictionary

Exists(key) 如果指定的键存在，返回 True，否则返回 False

Items() 返回一个包含 Dictionary 对象中所有条目的数组

Keys() 返回一个包含 Dictionary 对象中所有键的数组

Remove(key) 删除一个指定的键/条目对

       RemoveAll()   删除全部键/条目对

Vbs脚本编程简明教程之十四—VBS内置函数

Abs 函数：返回数的绝对值。

Array 函数：返回含有数组的变体。

Asc 函数：返回字符串首字母的 ANSI 字符码。

Atn 函数：返回数值的反正切。

CBool 函数：返回已被转换为 Boolean 子类型的变体的表达式。

CByte 函数：返回已被转换为字节子类型的变体的表达式。

CCur 函数：返回已被转换为货币子类型的变体的表达式。

CDate 函数：返回已被转换为日期子类型的变体的表达式。

CDbl 函数：返回已被转换为双精度子类型的变体的表达式。

Chr 函数：返回与指定的 ANSI 字符码相关的字符。

CInt 函数：返回已被转换为整形子类型的变体的表达式。

CLng 函数；返回已被转换为Long子类型的变体的表达式。

Cos 函数：返回角度的余弦。

CreateObject 函数：创建并返回对“自动”对象的引用。

CSng 函数：返回已被转换为单精度子类型的变体的表达式。

CStr 函数：返回已被转换为字符串子类型的变体的表达式。

Date 函数：返回当前系统日期。

DateAdd 函数：返回的日期已经加上了指定的时间间隔。

DateDiff 函数：返回两个日期之间的间隔。

DatePart 函数：返回给定日期的指定部分。

DateSerial 函数：返回指定年月日的日期子类型的变体。

DateValue 函数：返回日期子类型的变体。

Day 函数：返回日期，取值范围为 1 至 31。

Eval 函数：计算表达式并返回结果。

Exp 函数：返回 e （自然对数的底）的多少次方。

Filter 函数：根据指定的筛选条件,返回含有字符串数组子集的、下限为 0 的数组。

Fix 函数：返回数的整数部分。

FormatCurrency 函数：返回的表达式为货币值格式，其货币符号采用系统控制面板中定义的。

FormatDateTime 函数：返回的表达式为日期和时间格式。

FormatNumber 函数：返回的表达式为数字格式。

FormatPercent 函数：返回的表达式为百分数（乘以 100）格式，后面有 % 符号。

GetObject 函数：返回从文件对“自动”对象的引用。

GetRef 函数：返回对能够绑定到一事件的过程的引用。

Hex 函数：返回一字符串，代表一个数的十六进制值。

Hour 函数：返回表示钟点的数字，取值范围为 0 至 23。

InputBox 函数：在对话框中显式一提示，等待用户输入文本或单击按钮，并返回文本框的内容。

InStr 函数：返回一个字符串在另一个字符串中首次出现的位置。

InStrRev 函数；返回一个字符串在另一个字符串中出现的位置，但是从字符串的尾部算起。

Int 函数：返回数的整数部分。

IsArray 函数：返回 Boolean 值，反映变量是否为数组。

IsDate 函数：返回 Boolean 值，反映表达式能否转换为日期。

IsEmpty 函数：返回 Boolean 值，反映变量是否已被初始化。

IsNull 函数：返回 Boolean 值，反映表达式是否含有无效数据(Null)。

IsNumeric 函数：返回 Boolean 值，反映表达式能否转换为数字。

IsObject 函数：返回 Boolean 值，反映表达式是否引用了有效的“自动”对象。

Join 函数：返回通过连接许多含有数组的子串而创建的字符串。

LBound 函数；返回指定维数数组的最小有效下标。

LCase 函数：返回的字符串已被转换为小写字母。

Left 函数：返回字符串最左边的指定数量的字符。

Len 函数：返回字符串中的字符数或存储变量所需的字节数。

LoadPicture 函数：返回图片对象。只用于 32 位平台。

Log 函数：返回数的自然对数。

LTrim 函数；返回去掉前导空格的字符串。

Mid 函数：从字符串中返回指定数量的字符。

Minute 函数：返回分钟数，取值范围为 0 至 59。

Month 函数：返回表示月份的数，取值范围为 1 至 12。

MonthName 函数：返回表示月份的字符串。

MsgBox 函数：在对话框中显示消息，等待用户单击按钮，并返回表示用户所击按钮的数值。

Now 函数：返回计算机的当前系统日期和时间。

Oct 函数：返回表示该数八进制数值的字符串。

Replace 函数：返回一字符串，其中指定的子串已被另一个子串替换了规定的次数。

RGB 函数：返回代表 RGB 颜色值的数字。

Right 函数：返回字符串最右边的指定数量的字符。

Rnd 函数：返回随机数。

Round 函数：返回指定位数、四舍五入的数。

RTrim 函数：返回去掉尾部空格的字符串副本。

ScriptEngine 函数：返回反映使用中的脚本语言的字符串。

ScriptEngineBuildVersion 函数：返回使用中的脚本引擎的编译版本号。

ScriptEngineMajorVersion 函数：返回使用中的脚本引擎的主版本号。

ScriptEngineMinorVersion 函数：返回使用中的脚本引擎的次版本号。

Second 函数：返回秒数，取值范围为 0 至 59。

Sgn 函数：返回反映数的符号的整数。

Sin 函数：返回角度的正弦值。

Space 函数：返回由指定数量的空格组成的字符串。

Split 函数：返回下限为 0 的、由指定数量的子串组成的一维数组。

Sqr 函数：返回数的平方根。

StrComp 函数：返回反映字符串比较结果的数值。

String 函数：返回指定长度的重复字符串。

StrReverse 函数：返回一字符串，其中字符的顺序与指定的字符串中的顺序相反。

Tan 函数：返回角度的正切值。

Time 函数：返回表示当前系统时间的“日期”子类型的“变体”。

Timer 函数：返回时经子夜 12：00 AM 后的秒数。

TimeSerial 函数：返回含有指定时分秒时间的日期子类型的变体。

TimeValue 函数：返回含有时间的日期子类型的变体。

Trim 函数：返回去掉前导空格或尾部空格的字符串副本。

TypeName 函数：返回一字符串，它提供了关于变量的变体子类型信息。

UBound 函数：返回指定维数数组的最大有效下标。

UCase 函数：返回的字符串已经被转换为大写字母。

VarType 函数：返回标识变体子类型的数值。

Weekday 函数：返回表示星期几的数值。

WeekdayName 函数：返回表示星期几的字符串。

Year 函数：返回表示年份的数值。

Vbs脚本编程简明教程之十五——响应事件

什么是事件？在我看来，事件就象我们手机上的闹钟，闹钟一响，我们就要去做某些特定的事情。或者这样说，事件就像警钟，当程序运行时，有特殊的事情发生，就会激发事件，事件本身就是一条消息，如果你编写的脚本要对事件进行处理，就需要一个特殊的过程或者函数来接受和处理事件。那么这个特殊的过程或者函数在程序运行时，就不断的监听，看系统是否传来了相应的事件，一旦接受到事件，脚本对此作出反应。

那么事件是从那里来的呢？是否需要我们在脚本中对事件进行编写呢？一般情况下，事件是某个程序在运行中的特殊状态发出的，我们不需要对事件进行编写，只需要编写处理事件的函数。比如说我们用vbs建立了ie的一个实例，那么当ie的窗口被关闭的时候，就会激发出一个叫做OnQuit的事件。

是不是脚本自然而然就能接受事件并进行处理呢？我们说不是的，在创建对象的时候，我们将使用WSH的createobject命令，例如：

Set objie=Wscript.createobject(“internetexplorer.application”,”event_”)

注意到了吗？多了一个参数，这个参数的作用是什么呢？它叫做事件接收端，当脚本连接的对象包含事件时，如果对象调用的事件是OnBegin，那么WSH将会在脚本中调用一个event_OnBegin的事件处理程序。当然事件接受端并不是固定的，如果对象将其定义为MyObj_的话，那么事件处理程序将是：MyObj_OnBegin。

是否很熟悉？在打造个性化QQ一讲中，曾经出现过Window_OnSize(cx,cy)函数，它其实就是一个事件处理程序。

让我们来举个实际的例子完整的看看事件的处理过程：

Set objie=WScript.CreateObject("InternetExplorer.Application","event_")

objie.Visible=True

MsgBox "请关闭浏览器窗口看看效果！",vbSystemModal

Wscript.sleep 6000

MsgBox "现在已经正常关闭了"

Sub event_onquit()

MsgBox "您确定要关闭浏览器吗？",vbSystemModal

End Sub

这段脚本打开了一个IE窗口，然后要求你关闭IE窗口，当你关闭窗口的时候，自动调用事件响应程序。

Vbs脚本编程简明教程之十六——访问ADO数据库

ADO是Microsoft提供和建议使用的新型数据访问接口，它是建立OLEDB之上的一个抽象层。微软公司在操作系统中默认提供了 Access 的 ODBC 驱动程序以及 JET 引擎，

一、对ADO对象的主要操作，一般包括6个方面：

1.连接到数据源。通常使用ADO的Connection对象。一般使用相应的属性打开到数据源的连接，设置游标的位置，设置默认的当前数据库，设置将使用的OLEDBProvider，直接提交SQL脚本等。

2.向数据源提交命令。通常涉及ADO的Command对象。可查询数据库并返回结果在Recordset对象中。

3.执行SELECT查询命令。在提交SQL脚本的任务时，不用创建一个Command对象，就可完成查询。

4.可以通过ADO的Recordset对象对结果进行操作。

5.更新数据到物理存储。

6.提供错误检测。通常涉及ADO的Error对象。

二、ADO中主要对象的功能

Recordset对象，用来封装查询的结果。

Field对象，用来表达一行结果中各子段的类型和值。

Error对象，用来检测和判断在数据库操作中出现的错误，比如连接失败。

在ADO中，许多对象名后多了一个"s"，比如Error->Errors，Field->Fields等等。添加"s"意味着是相应对象的Collection（集合）对象，比如Errors是Error对象的Collection对象。Collection有点像数组（Array），但不同的是，Collection可以以不同类型的数据或对象作为自己的元素，而数组中的各元素通常都是相同类型的。所以，在看到一个对象名最后是"s"，通常表明这是一个Collection对象，比如Errors中的各元素是由Error对象的实例组成的。

三、具体应用

1、创建mdb数据库

ADOX 是ADO 对象的扩展库。它可用于创建、修改和删除模式对象，如数据库和表格等。

其常用的对象有：Catalog—>创建数据库。Column—>表示表、索引或关键字的列。

Key—>表示数据库表中的关键字。

常用的方法有： Append 将对象添加到其集合。Delete 删除集合中的对象。

set cat= createobject("ADOX.Catalog")

cat.Create "Provider=Microsoft.Jet.OLEDB.4.0;Data Source=d:"shujuku.mdb"

Set tbl=createobject("ADOX.Table")     

tbl.Name ="MyTable"  

tbl.Columns.Append "姓名", 202 'adInteger  

tbl.Columns.Append "性别", 3 'adInteger  

tbl.Columns.Append "工作单位", 202 ,50 'adVarWChar  

cat.Tables.Append tbl    

不过你要操纵数据库就连一个数据库也不建，未免懒惰了点，用代码虽然可以完成，但是我觉得对数据约束完成的比较困难，本代码也就是示范个例子，并不推荐使用此类方法。

2、打开数据库

Provider=″Provider=Microsoft.Jet.OLEDB.4.0 ; Data Source="

Set Objconn = createobject("ADODB.Connection")

Objconn.Open Provider & "数据库名称"

3、创建记录集

Set Objrs = CreateObject("ADODB.Recordset")

4、执行SQL查询语句

Sql="SQL查询语句" '例如：Select count(*) from table1

Set objrs = objconn.execute(sql)

一般情况下，我们将绝大多数的操作转化为SQL语句完成。

常用的SQL语句

在学习SQL语句之前，让我们先来对数据库做一个基本的了解。一个数据库中可能包含了很多个基本单位叫做表。表格被分为“行”和“列”。每一行代表表的一个单独组成部分，每一列代表相同性质的一组数据。举例来说，如果我们有一个记载顾客资料的表格，行包括姓、名、地址、城市、国家、生日等。而一列则代表了所有的地址或者国家等。

一、建立数据表，我们前边说过利用ADOX.Catalog建立数据库和数据表的方法，但是用的似乎不是很多，一般情况下，如果我们需要在数据库中动态建立一个表，我们将工作交给SQL语句来做，其基本语法是：

CREATE TABLE [表格名]([列名1] 数据类型 , [列名2] 数据类型,... )

例如我们要建立一个基本顾客表：

Create table [顾客表]([姓名] text（8）, [性别] text（2），[住址] text(30))

二、插入数据项

insert into [数据表名称] (数据项1,数据项2,...) values (值1,值2,...)

insert into语句用来添加新的数据到数据库中的指定表。通过(数据项1,数据项2,...) values (值1,值2,...)来为新添加的数据赋初值。

三、删除数据项

delete from [数据表名称] where [数据项1] like [值1] and/or [数据项2] like [值2]

...

四、更新数据项

update [数据表名称] set 数据项1=值1,数据项2=值2,... where [数据项1] like [值1] and/or [数据项2] like

[值2] ...

该语句可以修改数据库中指定数据表内的指定数据，如果不是用where限定条件就表示修改该表内所有的数据条目。

五、查询数据项

select [数据内容] from [数据表名称] where [数据项1] like [值1] and/or [数据项2] like [值2] ...

order by [数据项] asc/desc

[数据内容]部分表示所要选取的表格中的数据项，使用*表示选取全部。[数据表名称]表示要从哪一个表格中选取，如果你没有接触过数据库可能很难了解什么是数据表格，没关系，我将在后面用到它的时候再说明。where表示选取的条件，使用like表示相等，也支持>=这样的判断符号，同时使用多个条件进行选取时中间要使用and进行连接。order

by决定数据的排列顺序，asc表示按照[数据项]中的数据顺序排列，desc表示倒序，默认情况为顺序。select语句中除select和from之外其它均为可选项，如果都不填写表示选取该数据表中的全部数据。例如：下面的语句查询某数据库中表名称为：testtable中姓名为“张三”的nickname字段和email字段。

SELECT nickname,email FROM testtable WHERE name='张三'

(一) 选择列表

选择列表(select_list)指出所查询列，它可以是一组列名列表、星号、表达式、变量(包括局部变量和全局变量)等构成。

1、选择所有列

例如，下面语句显示testtable表中所有列的数据：

SELECT * FROM testtable

2、选择部分列并指定它们的显示次序查询结果集合中数据的排列顺序与选择列表中所指定的列名排列顺序相同。

例如：SELECT nickname,email FROM testtable

3、更改列标题

在选择列表中，可重新指定列标题。定义格式为：列标题=列名

列名列标题如果指定的列标题不是标准的标识符格式时，应使用引号定界符，例如，下列语句使用汉字显示列标题：

SELECT 昵称=nickname,电子邮件=email FROM testtable

(二) FROM子句指定SELECT语句查询的表。

最多可指定256个表，它们之间用逗号分隔。如果选择列表中存在同名列，这时应使用对象名限定这些列所属的表或视图。例如在usertable和citytable表中同时存在cityid列，在查询两个表中的cityid时应加以限定。

(三) WHERE子句设置查询条件

WHERE子句设置查询条件，过滤掉不需要的数据行。例如下面语句查询年龄大于20的数据：

SELECT * FROM usertable WHERE age>20

WHERE子句可包括各种条件运算符：

比较运算符(大小比较)：>、>=、=、<、<=、<>、!>、!<

范围运算符(表达式值是否在指定的范围)：BETWEEN…AND…

NOT BETWEEN…AND…

列表运算符(判断表达式是否为列表中的指定项)：IN (项1,项2……)

NOT IN (项1,项2……)

模式匹配符(判断值是否与指定的字符通配格式相符):LIKE、NOT LIKE

空值判断符(判断表达式是否为空)：IS NULL、NOT IS NULL

逻辑运算符(用于多条件的逻辑连接)：NOT、AND、OR

1、范围运算符例：age BETWEEN 10 AND 30相当于age>=10 AND age<=30

2、列表运算符例：country IN ('Germany','China')

3、模式匹配符例：常用于模糊查找，它判断列值是否与指定的字符串格式相匹配。可用于char、

varchar、text、ntext、datetime和smalldatetime等类型查询。

可使用以下通配字符：

百分号%：可匹配任意类型和长度的字符，如果是中文，请使用两个百分号即%%。

下划线_：匹配单个任意字符，它常用来限制表达式的字符长度。

方括号[]：指定一个字符、字符串或范围，要求所匹配对象为它们中的任一个。

[^]：其取值也[] 相同，但它要求所匹配对象为指定字符以外的任一个字符。

例如：

限制以Publishing结尾，使用LIKE '%Publishing'

限制以A开头：LIKE '[A]%'

限制以A开头外：LIKE '[^A]%'

4、空值判断符例WHERE age IS NULL

5、逻辑运算符：优先级为NOT、AND、OR

最后，让我们用一个简单的例子结束这篇教程：

Objku = InputBox("请输入单位数据库的路径","默认位置","d:"jbqk.mdb")

Set Objconn = createobject("adodb.connection")

Objconn.open ="provider=Microsoft.Jet.OLEDB.4.0;Data Source=" & Objku

sql = "CREATE TABLE [单位资料](ID Autoincrement PRIMARY KEY,[姓名] text(8),[性别]

text(2),[科室] text(6),[住址] text(30))"

Objconn.execute(sql)

sql = "INSERT INTO [单位资料]([姓名],[性别],[科室],[住址]) VALUES('张三','男','行管科','解放路12号')"

Objconn.execute(sql)

sql = "INSERT INTO [单位资料]([姓名],[性别],[科室],[住址]) VALUES('李斯','女','市场科','五一路12号')"

Objconn.execute(sql)

sql = "DELETE FROM [单位资料] WHERE [姓名] = '张三' "

Objconn.execute(sql)

sql = "UPDATE [单位资料]"

sql = "SELECT COUNT(ID) FROM [单位资料]"

Vbs调用MsAgent组件，很有趣

Microsoft Agent是微软公司发布的一项代理软件开发技术，我们知道，在Office帮助系统中有一种叫作Office助手的代理软件，但其只允许Office各个组件调用，Agent动画人物可由任何Windows程序调用；

Agent支持文字气球和输入提示条，在输出语音的同时把文字输出至一个卡通式文字气球中。如果电脑系统中安装有Agent语音识别引擎，当用户可以通过声卡、麦克风与用户交谈。下午无事，就尝试着写了一段简单的代码调用MsAgent：

GenieID = "Genie"

GenieACS = "genie.acs"

ScriptComplete=0

Set AgentControl = WScript.CreateObject("Agent.Control.2","agent_")

AgentControl.Connected = True'连接控件

AgentControl.Characters.Load GenieID,GenieACS

Set Genie = AgentControl.Characters(GenieID)

Genie.LanguageID = &H409

Genie.MoveTo 900, 600

Genie.Show

Genie.MoveTo 900, 0

timespeak="good " & GetTimeOfDay()

Genie.Speak(timespeak)

Genie.Play "Acknowledge"     '承认

Genie.Speak("眨眼")

Genie.Play "Blink"     '眨眼

Genie.Speak("i love you")

Genie.Speak("回复动作")

genie.Play("RestPose")     '回復动作

Genie.Speak("向上")

genie.Play("GestureUp")     '向上

Genie.Speak("向下")

genie.Play("GestureDown")     '向下

Genie.Speak("伸出左手")

genie.Play("GestureLeft")     ' 伸出左手

Genie.Speak("伸出右手")

genie.Play("GestureRight")     ' 伸出右手

Genie.Speak("叹气")

genie.Play("Sad")     '嘆气

Genie.Speak("惊奇")

genie.Play("Surprised")     '惊奇

Genie.Speak("握掌")

genie.Play("Pleased")     '握掌

Genie.Speak("喇叭")

genie.Play("Announce")     '喇叭

Genie.Speak("眯眼")

genie.Play("Blink")     '瞇眼

Genie.Speak("无奈")

genie.Play("Decline")     '无奈

Genie.Speak("抓头")

genie.Play("Confused")     '抓头

Genie.Speak("鼓掌")

genie.Play("Congratulate")     '奖盃

Genie.Speak("回手")

genie.Play("Wave")     '挥手

Genie.Speak("惊讶")

genie.Play("Alert")     '惊讶

Genie.Speak("魔术棒1")

genie.Play("DoMagic1")     '魔术棒-1

Genie.Speak("魔术棒2")

genie.Play("DoMagic2")     '魔术棒-2

Genie.Speak("摊手")

genie.Play("Explain")     '摊手

Genie.Speak("敲门")

genie.Play("GetAttention")     '敲门

genie.Play("GetAttentionContinued")     '敲门-敲

genie.Play("GetAttentionReturn")     '敲门-放下

genie.Play("Greet")     '弯腰

genie.Play("Idle2_1")     '观察魔术棒

genie.Play("Idle2_2")     '两手在腹前交叉

genie.Play("Idle3_1")     '打呵欠

Genie.Speak("向上看")

genie.Play("LookUp")     '上看

genie.Play("LookDown")     '下看

genie.Play("LookLeft")     '左看

genie.Play("LookRight")     '右看

genie.Play("MoveUp")     '上移

genie.Play("MoveDown")     '下移

genie.Play("MoveLeft")     '左移

genie.Play("MoveRight")     '右移

genie.Play("Process")     '魔法调配

genie.Play("Read")     '阅读

Do     '此处存疑，高手请看最后

WScript.Sleep 1000

Loop Until ScriptComplete

Function GetTimeOfDay()

       Dim TimeOfDay

       Dim h

       h = Hour(Now())

       If h < 12 Then

           TimeOfDay = "Morning"

       ElseIf h < 17 Then

           TimeOfDay = "Afternoon"

       Else

           TimeOfDay = "Evening"

       End If

       GetTimeOfDay = TimeOfDay

End Function

Sub agent_dblclick(ByVal CharacterID, ByVal Button, ByVal Shift, ByVal X, ByVal

Y)

Genie.StopAll

MsgBox "白白，再见了！"

WScript.Quit

End Sub

本来代码写的就没有什么难度，可是写完之后，每次可爱的小人总是一闪而过，屏幕上什么也看不见，在网络上查找也找不出原因，我实验了好多次，终于发觉了加红的那段代码必不可少，程序异步执行，没有最后的代码，程序没有执行完就退出了


Reference:[Vbs脚本编程简明教程](http://blog.csdn.net/yjbnew/article/details/6732039)
