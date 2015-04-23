Gentoo字体设置
=============

1. 字体准备
-----------

安装宋体(SimSun)和微软雅黑(Microsoft YaHei)作为主要中文字体。实测文泉驿微米黑和文泉驿正黑在低分辨率下渲染效果不好，建议不用，最好删除。

宋体和雅黑可以在微软系统里拷贝或者下载，具体为四个字体文件: `msyh.ttf, msyhbd.ttf, simsun.ttc, simsunb.ttf`. 然后将其放置在`~/.fonts/`目录下：
	
	$ cd ~/.fonts
	$ mkfontdir
	$ mkfontscale
	$ fc-cache -fv

英文请安装`croscorefonts`代替微软英文字体（兼容）。

	$ sudo emerge -av media-fonts/croscorefonts



2. 安装infinality
-----------------

给`media-libs/freetype`加上`infinality` USE, 并重新emerge：
	
	$ sudo emerge -av1 media-libs/freetype

这会自动依赖安装`fontconfig-infinality`和`eselect-infinality`. 如果没有请手动安装。


3. eselect freetype
-------------------

设置freetype效果：

	$ eselect lcdfilter set 14

此处我选择windows-7效果，实测选哪个没有影响，只要给freetype打上`infinality` USE就好。选windows-7就好。


4. eselect fontconfig
---------------------

特别注意下面我没有开启`52-infinality.conf`. 也就是说我没有启用`fontconfig-infinality`额外安装进来的任何fontconfig配置文件，而是采取了自己定制fontconfig配置文件的方法（见下条)。请严格照着这个列表相应选择开启和关闭。(P.S. Gentoo真好，否则你就一个个的`ln -s`去吧).

启用`50-user.conf`代表启用自己写的fontconfig配置。

注：此处你的文件没我的多不要紧，说明你安装的字体少一些。把有的条目设置关闭开启就好。

`
Available fontconfig .conf files (* is enabled):
  [1]   10-autohint.conf
  [2]   10-no-sub-pixel.conf
  [3]   10-scale-bitmap-fonts.conf *
  [4]   10-sub-pixel-bgr.conf
  [5]   10-sub-pixel-rgb.conf *
  [6]   10-sub-pixel-vbgr.conf
  [7]   10-sub-pixel-vrgb.conf
  [8]   10-unhinted.conf
  [9]   11-lcdfilter-default.conf
  [10]  11-lcdfilter-legacy.conf
  [11]  11-lcdfilter-light.conf
  [12]  20-unhint-small-dejavu-sans-mono.conf *
  [13]  20-unhint-small-dejavu-sans.conf *
  [14]  20-unhint-small-dejavu-serif.conf *
  [15]  20-unhint-small-vera.conf *
  [16]  25-unhint-nonlatin.conf
  [17]  30-metric-aliases.conf *
  [18]  30-urw-aliases.conf *
  [19]  31-cantarell.conf *
  [20]  40-nonlatin.conf
  [21]  45-latin.conf *
  [22]  49-sansserif.conf *
  [23]  50-user.conf *
  [24]  51-local.conf
  [25]  52-infinality.conf
  [26]  57-dejavu-sans-mono.conf *
  [27]  57-dejavu-sans.conf *
  [28]  57-dejavu-serif.conf *
  [29]  60-latin.conf *
  [30]  60-liberation.conf
  [31]  62-croscore-arimo.conf *
  [32]  62-croscore-cousine.conf *
  [33]  62-croscore-symbolneu.conf *
  [34]  62-croscore-tinos.conf *
  [35]  63-source-pro.conf
  [36]  65-fonts-persian.conf *
  [37]  65-khmer.conf
  [38]  65-nonlatin.conf
  [39]  69-unifont.conf *
  [40]  70-no-bitmaps.conf
  [41]  70-yes-bitmaps.conf
  [42]  80-delicious.conf *
  [43]  90-synthetic.conf *
  [44]  99pdftoopvp.conf
`

5. 用户fontconfig配置文件
-------------------------

以下内容请写于`~/.fonts.conf`或者`~/.config/fontconfig/fonts.conf`, 具体哪个请参考`/etc/fonts/conf.d/50-user.conf`. 前者较旧，后者在新版本fontconfig里才支持。


<?xml version='1.0'?>
<!DOCTYPE fontconfig SYSTEM 'fonts.dtd'>
<fontconfig>


	<!-- ******************************************************************  -->
	<!-- *************************** ALIASES ******************************  -->
	<!-- ******************************************************************  -->

	<!-- Default fonts - Linux Appearance -->
	<alias>
		<family>sans-serif</family>
		<prefer>
			<family>Arimo</family>
			<family>DejaVu Sans</family>
			<family>SimSun</family>
			<family>Microsoft YaHei</family>
		</prefer>
	</alias>
	<alias>
		<family>serif</family>
		<prefer>
			<family>Tinos</family>
			<family>DejaVu Serif</family>
			<family>SimSun</family>
		</prefer>
	</alias>
	<alias>
		<family>monospace</family>
		<prefer>
			<family>Cousine</family>
			<family>DejaVu Sans Mono</family>
		</prefer>
	</alias>

	<!-- ******************************************************************  -->
	<!-- ************************* RENDERING SETTINGS *********************  -->
	<!-- ******************************************************************  -->

	<!-- basic settings -->
	<match target="font">
		<edit name="rgba" mode="assign">
			<const>rgb</const>
		</edit>
		<edit name="autohint" mode="assign">
			<bool>false</bool>
		</edit>
		<edit name="hinting" mode="assign">
			<bool>true</bool>
		</edit>
		<edit name="antialias" mode="assign">
			<bool>true</bool>
		</edit>
		<edit name="hintstyle" mode="assign">
			<const>hintfull</const>
		</edit>
		<edit name="lcdfilter" mode="assign">
			<const>lcddefault</const>
		</edit>
	</match>


	<!-- Microsoft Yahei specific settings -->
	<match target="font">
		<test name="family">
			<string>Microsoft YaHei</string>
		</test>
		<edit name="rgba" mode="assign">
			<const>rgb</const>
		</edit>
		<edit name="autohint" mode="assign">
			<bool>false</bool>
		</edit>
		<edit name="hinting" mode="assign">
			<bool>true</bool>
		</edit>
		<edit name="antialias" mode="assign">
			<bool>true</bool>
		</edit>
		<edit name="hintstyle" mode="assign">
			<const>hintfull</const>
		</edit>
		<edit name="lcdfilter" mode="assign">
			<const>lcdlight</const>
		</edit>
	</match>

</fontconfig>


6. 一些注解
-----------

在`~/.fonts.conf`文件里，第一段对字体优先级做了设置。其中特别设置了SimSun为serif和sans-serif字体，并且比雅黑优先。如果你想让雅黑优先，就在sans-serif那里把SimSun一行删掉。

后边两段分别针对全部字体和微软雅黑微调做了设置。这里全局开启`hintfull`和`lcddefault`, 然后对微软雅黑单独设置了`hintfull`和`lcdlight`, 因为我实测这种设置雅黑最好看。

另，配置后需重启应用程序(如firefox)方可看到效果。
另另，不要同时使用第另一方字体微调软件(如gnome-tweak-tool）配置字体,以免覆盖。


7. Good luck!
-------------

-END-
