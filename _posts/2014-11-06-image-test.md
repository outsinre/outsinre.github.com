---
layout: post
comments: true
title: Image and File
---
<div class="message">
注意这些方法细微区别。主要看是否background图片，是否relative/absolute定位，是否表格图片。
</div>

Add a file by markup.  
[A PDF](/assets/fx-3650P_3950P_EN.pdf)
<br />

Add a file by `href`. <br />
<a href="/assets/fx-3650P_3950P_EN.pdf" target="_blank">fx-3650P_3950P_EN</a>
<br />

Add an image by markup.
![Test]({{site.baseurl}}assets/plane.jpg)   

One way around this is to size images in relative units, rather than absolute pixel dimensions. The most common relative solution is to set the max-width of the image at 100%. Specifying only the `width` of images may cause a doubling or tripling of the cycles that many browsers must process to layout the new, resized page. While each of these cycles typically take less than a millisecond, they stack up, especially if there are multiple scalable elements on the page. Addressing `height` in the same declaration can reduce this issue.
<img src="{{site.baseurl}}assets/hknight.jpg" alt="Hong Kong Night" style="max-width:100%;height:auto;">
<br />

<img src="{{site.baseurl}}assets/plane.jpg" alt="Hong Kong Night" style="float:right; width:50%; margin-left:0.5rem; max-width:738px;">
A better, albeit more complex approach to fluid images is to measure the width of the image as a percentage of the overall width of the page. For example, let’s say you had an image that had a natural size of 500px × 300px in a 1200px wide document. Below 1200px, the document will be fluid. The calculation of how much width the image takes up as a percentage of the document is easy: (500 / 1200 ) × 100 = 41.66%.

We can plug this number in as the width for the image; typically, this would be done inline, as each image will often be a different dimension. For this method, put the **image before the text**.
<br />

Place text over image, this is combination of `background url` and `position`. For `background`, the `width` and `height` of the image should be included. The `border-radius` is set to `10px` which overwrites the default in `poole.css` file.
<div style="position:relative; background:url({{site.baseurl}}assets/hknight.jpg); width:738px; height:284px; border-radius:10px;">
  <div style="position:absolute; bottom:0.5em; left:0.5em; width:400px; font-weight:bold; color:#fff;">(some text ...some text ...some text ...)</div>
  <p style="position:absolute; top:1em; right:2em; width:120px; padding:4px; background-color:#fff; font-weight:bold; font-size:11px;">(some text ...some text ...some text ...)</p>
</div>

Place text over image, position not accurate.
<div style="background:url({{site.baseurl}}assets/hknight.jpg) no-repeat; width:738px; height:284px; text-align:center; border-radius:10px;">
  <span style="color:#fcc">some text ...</span>
</div>

Place text over image, by <span style="color:red; font-weight:bold">table</span>, position not accurate. No `border-radius` attribute here. So border angle is sharpe.
<TABLE BORDER="0" cellpadding="5" CELLSPACING="0">
<TR>
<TD WIDTH="738" HEIGHT="284" BACKGROUND="{{site.baseurl}}assets/hknight.jpg" VALIGN="bottom">
<FONT SIZE="+1" COLOR="yellow">some text ...</FONT></TD>
</TR>
</TABLE>

Place text over image. `border-radius` is acctually attribute of `img`. Attention: `border-radius`, `margin` `max-width` and `overflow` is already specified in `poole.css` and `lanyon.css` files. Here is just a complete example to show the method.
<div style="position:relative; overflow:hidden; border-radius:5px">
  <img src="{{site.baseurl}}assets/hknight.jpg" style="max-width:100%; height:auto; margin:0 0 0.5rem 0;">
  <p style="position:absolute; left:100px; top:55px; color:#fff; margin:0.5em;">(text accurately positioned)</p>
</div>

Place text over image with `CSS` file support with css support.
<link rel="stylesheet" href="{{ site.baseurl }}public/css/znhoo.css">
<div class="txtimg">
  <img src="{{site.baseurl}}assets/hknight.jpg">
  <div class="bl">(text to appear at the bottom left of the image)</div>
  <p class="tr"> (text to appear at the top right of the image)</p>
</div>

>Use the last two methods! They are very good and complete.

## Reference
[Main](http://demosthenes.info/blog/586/CSS-Fluid-Image-Techniques-for-Responsive-Site-Design)
[1](http://www.jianshu.com/p/05289a4bc8b2)
[2](http://isnowfy.github.io/about-simple-cn.html)
[3](https://docs.webplatform.org/wiki/css/properties/background-image)
[4](http://css-tricks.com/text-blocks-over-image/)
[5](http://www.htmldog.com/guides/css/intermediate/backgroundimages/)
[6](http://zhidao.baidu.com/question/296249405.html)
[7](http://bbs.csdn.net/topics/120076193)
[8](http://www.the-art-of-web.com/css/textoverimage/)
