---
layout: post
comments: true
title: Image and file
---

<!--add an image by img tag, img tag is self closing-->
<img src="{{site.baseurl}}assets/hknight.jpg" alt="">  

<!--add an image by markup -->
![Test]({{site.baseurl}}assets/plane.jpg)   

<!-- add a file by markup -->
[A PDF](/assets/fx-3650P_3950P_EN.pdf)  

<!-- add a file by link -->
<a href="/assets/fx-3650P_3950P_EN.pdf" target="_blank">fx-3650P_3950P_EN</a>  

<!--method 1, place text over image this is the best way. this is combination of method 2 and method 3 -->
<div style="position: relative; background: url({{site.baseurl}}assets/hknight.jpg); width: 738px; height: 284px;">
	<div style="position: absolute; bottom: 0; left: 0.5em; width: 400px; font-weight: bold; color: #fff;">
		<p>(text to appear at the bottom left of the image)</p>
	</div>

	<p style="position: absolute; top: 1em; right: 2em; width: 120px; padding: 4px; background-color: #fff; font-weight: bold; font-size: 11px;"> (text to appear at the top right of the image) </p>
</div>

<br />

<!--method 2, place text over image, position not accurate -->
<div style="background:url({{site.baseurl}}assets/hknight.jpg) no-repeat;width:738px;height:284px;text-align:center">
	<span style="color:#fcc">添加文字...添加文字...添加文字...</span>
</div>

<br />

<!--method 3, place text over image, position not accurate -->
<div style="position:relative">
	<div style="position:absolute; left:50; top:0; color:#fff; font-weight:bold">图片文字</div>
	<img src="{{site.baseurl}}assets/hknight.jpg">
</div>
<br />

<!--method 4, place text over image, by table, position not accurate -->
<TABLE BORDER="0" cellpadding="5" CELLSPACING="0">
<TR>
<TD WIDTH="738" HEIGHT="284" BACKGROUND="{{site.baseurl}}assets/hknight.jpg" VALIGN="bottom">
<FONT SIZE="+1" COLOR="yellow">Joe Burns at Work</FONT></TD>
</TR>
</TABLE>

## Reference
[1](http://www.jianshu.com/p/05289a4bc8b2)
[2](http://isnowfy.github.io/about-simple-cn.html)
[3](https://docs.webplatform.org/wiki/css/properties/background-image)
[4](http://css-tricks.com/text-blocks-over-image/)
[5](http://www.htmldog.com/guides/css/intermediate/backgroundimages/)
[6](http://zhidao.baidu.com/question/296249405.html)
[7](http://bbs.csdn.net/topics/120076193)
[Main](http://www.the-art-of-web.com/css/textoverimage/)
