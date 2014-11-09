---
layout: post
comments: true
title: Image and file
---
<div class="message">
注意这写图片的细微区别。主要看三点：是否background图片，是否relative/absolute定位，是否表格图片。如果是通过background图片加上去的，发现边角是方形的，而且不能自动适应手机。如果是通过img或markup添加的就会自动适应屏幕（譬如手机）大小，边角是圆弧的。
</div>
<!--add an image by img tag, img tag is self closing-->
<img src="{{site.baseurl}}assets/hknight.jpg" alt="Hong Kong Night">  

<!--add an image by markup -->
![Test]({{site.baseurl}}assets/plane.jpg)   

<!-- add a file by markup -->
[A PDF](/assets/fx-3650P_3950P_EN.pdf)  

<!-- add a file by link -->
<a href="/assets/fx-3650P_3950P_EN.pdf" target="_blank">fx-3650P_3950P_EN</a>  

<!--method 1, place text over image this is the best way. this is combination of method 2 and method 3 -->
<div style="position: relative; background: url({{site.baseurl}}assets/hknight.jpg); width: 738px; height: 284px; max-width: 100%;border-radius:10px;">
	<div style="position: absolute; bottom: 0; left: 0.5em; width: 400px; font-weight: bold; color: #fff;">
		<p>(some text ...some text ...some text ...)</p>
	</div>

	<p style="position: absolute; top: 1em; right: 2em; width: 120px; padding: 4px; background-color: #fff; font-weight: bold; font-size: 11px;"> (some text ...some text ...some text ...) </p>
</div>

<br />

<!--method 2, place text over image, position not accurate -->
<div style="background:url({{site.baseurl}}assets/hknight.jpg) no-repeat;width:738px;height:284px;text-align:center">
	<span style="color:#fcc">some text ...</span>
</div>

<br />

<!--method 3, place text over image, position not accurate -->
<div style="position:relative; width: 738px; height: 284px">
	<div style="position:absolute; left:50px; top:50px; color:#fff; font-weight:bold">some text ...</div>
	<img src="{{site.baseurl}}assets/hknight.jpg">
</div>
<br />

<!--method 4, place text over image, by table, position not accurate -->
<TABLE BORDER="0" cellpadding="5" CELLSPACING="0">
<TR>
<TD WIDTH="738" HEIGHT="284" BACKGROUND="{{site.baseurl}}assets/hknight.jpg" VALIGN="bottom">
<FONT SIZE="+1" COLOR="yellow">some text ...</FONT></TD>
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
