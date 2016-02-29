---
layout: post
title: Fcitx chinese-pyim 词库
---

＃ 概念

1. 拼音文件 —— Pinyin File is a text file with pinyin and one character per line, separated with space. One available file is in the source of fcitx, named *gbkpy.org*.
2. 词库文件 —— Phrase File is a text file with full pinyin separated with `'` and the corresponding phrase. The default phrase file can be found in the source of fcitx, namely *pyPhrase.org*.

    拼音文件和词库文件在转换词库时非常中要。
3. Fcitx 的安装不难。不过要用好的话，就应该设置好拼音文件文件和词库文件。

    拼音文件基本不变。目前比较好的词库是来自搜狗拼音输入法，可以在官网免费下载 *\*.scel* 结尾的词库文件
4. Fcitx 和 chinese-pyim 无法直接使用搜狗拼音输入法的 *\*.scel* 词库文件，需要经过工具转换为中间格式 *\*.org* 格式。经过排序，去重后，

    将 org 格式转换成 Fcitx 的二进制 *\*.mb* 格式；或者转换成 chinese-pyim 的普通 文本 *\*.pyim* 格式。org 格式既可以转成 *chinese-pyim* 的 pyim 格式，也可以转换成 Fcitx 的 MB 格式。所以，有必要维护一份共同的 org 中间格式词库文件。

    本文着重转换方法。
5. 在 Linux 上，转换之前，所有的文件必需是 UTF-8 编码。用 `enca file` 列出文件编码，用 `enca -x UTF-8 file` 把文件转换成 UTF-8 编码。

    如果编码非 UTF-8, 可能转换出错，即使转换成功，或者转换过程丢失很多词组。
6. 虽然 chinese-pyim (参考 emacs configuration 文） 和 Fcitx 都可以用 org 词库来转换，但有所不同：
    1. chinese-pyim 要求 org 格式的拼音分隔符是 `-`，而不是 Fcitx org 的 `'`。
    2. chinese-pyim 需要合并相同拼音前缀的行。
    3. 还需要去除同一行相同的候选词。

＃ 安装 Fcitx

Details refer to post *Gentoo Installation*.

# Fcitx 词库设置

Fcitx 自带的拼音文件 *gbkpy.org* 可以直接使用，但自带的词库文件 *pyPhrase.org* 很差劲，我们需要手动生成更好的。

1. 参考 [合并词库及方法步骤（2012-06-17 更新）](http://forum.ubuntu.org.cn/viewtopic.php?t=364764) 和 [120余万的搜狗细胞词库](http://forum.ubuntu.org.cn/viewtopic.php?t=252407).
2. 去 [搜狗词库官网](http://pinyin.sogou.com/dict/)下载各种词库文件。譬如放到 *~/Downloads/scel/\*.scel*.
3. 把 *\*.scel* 文件 转换成 *\*.org* 文件。

    ```bash
    ~ $ cd Downloads
    ~ $ mkdir org
    ~ $ cd scel
    ~ $ find -L . -type f -iname '*.scel' -exec scel2org -o ../org/{}.org {} \;
    or
    ~ $ for i in *.scel; do scel2org $i -o ../org/$i.org; done
    ~ $ cd ..
    ~ $ ls org
    ```
    这个命来还会打印出所有被转换的词库名称。*~/Downloads/org/* 下存放着转换后的 *\*.scel.org* 文件，此文件是普通文本文件，可以用 *less* 明令查看。
4. 合并所有的 *\*.scel.org* 文件为一个单一的 *org* 文件。

    ```bash
    ~ $ mkdir dict
    ~ $ cd dict
    ~ $ cat ../org/*.scel.org > 1.org
    ```
    得到 *1.org* 文件，合并词库文件，有利於提升输入法的性能。很明显从单个文件提词比从多个文件提词快。

    到目前威治我们得了一个 *org* 格式的大词库文件。
5. 从 Fcitx 源码包解压出 *~/Downloads/dict/pyPhrase.org* —— Fcitx 默认自带的词库文件，很糟糕，我们把它合并到刚刚生成的 *1.org* 中。

    ```bash
    ~ $ tar xf fcitx-4.2.4.1_dict.tar.xz
    ~ $ tar xf fcitx-4.2.4.1/data/pinyin.tar.gz # 解压出 pyPhrase.org
    ~ $ cat pyPhrase.org >> 1.org # 合并
    ~ $ sort 1.org > 2.org        # 排序
    ~ $ uniq 2.org > 3.org        # 去掉重复
    ```
    得到 Fcitx 的 *org* 格式词库文件。

    如果找不到 *pinyin.tar.gz* 文件，可以：

    ```bash
    ~ $ find -L fcitx-4.2.4.1 -type f -iname '*pinyin.tar*'
    ```
    下同。
6. 从 Fcitx 源码包解压出 *~/Downloads/dict/gbkpy.org* —— Fcitx 默认自带的拼音文件.

    开时转换 Fcitx 使用的 MB 格式。

    ```bash
    ~ $ cp fcitx-4.2.4.1/data/gbkpy.org .
    ~ $ man createPYMB
    ~ $ createPYMB gbkpy.org 3.org
    ```
    如果 *3.org* 文件非常大的话，可能需要10分钟左右，耐心等待。最后生成 4 个新文件:

    ```
    dict/
    ├── 3.org
    ├── gbkpy.org
    ├── pyERROR     词库中重复或有其它问题条目，有兴趣可参考，没事直接忽略
    ├── pyPhrase.ok 可以顺利转换称 Fcitx MB格式的 org 词库；和 pyERROR 一起就是 3.org
    ├── pyphrase.mb 最终 Fcitx 使用的词库文件，必须，用于覆盖 Fcitx 原 *pyphrase.mb* 文件。
    └── pybase.mb   MB 词库文件的配套字码库，必须，用于覆盖原 *pybase.mb* 文件。
    ```
    转换完毕。只需要用新转换得到的 *pybase.mb* 和 *pyphrase.mb* 文件复盖原文件即可。
7. 复盖 Fcitx 原文件。

    原文件一般位於系统路经 */usr/share/fcitx{/data,/pinyin}* 和用户经 *~/.config/fcitx{/data,/pinyin}*。

    如果所在目录有原文件，就覆盖，没有的话就复制过去。

    在我的 Gentoo 下，位於 *{/usr/share,~/.config}/fcitx/pinyin/* 目录下。最后选择放到用户目录下。
8. 上面描述里从 scel 到 org 再到 mb 格式的转换过程。

    如果能从网上下载到比较好的 org 或 mb 会更省时。注意：网上下载的词库在使用之前必需转换成 UTF-8 编码。

    第四个参考链接里的 *sougou-phrases-full.7z* 或 *fcitx-sougou-phrase-full.7z* 不错。

# chinese-pyim 词库设置

1. 上面得到的 3.org 经过转换也可以给 chinese-pyim 使用。
2. 用 sed 命令把 `'` 替换成 `-`

    ```
    sed -i "s/'/-/g" 3.org
    ```
3. 用 awk 合并相同拼音前缀的行，也就是汉字的同音字/词。譬如 *gai-shan 改善* 和 *gai-shan 改删* 两行就需要合并成 *gai-shan 改善 改删*，这是 chinese-pyim 的要求，Fcitx 并没有此要求。

    ```
    awk '{key=$1; $1=""; a[key]=a[key] $0}; END{ for (key in a) { print key a[key];}}' 3.org > 4.org
    ```
4. 重新排序。经过 awk 合并后，顺序打乱了：

    ```
    sort 4.org > 5.pyim
    ```
5. 在同一行去重复。譬如 *a-ba 阿爸 阿坝 阿爸* 就需要去除多余的 *阿爸*，这是由于第三步合并时，可能有两行里包含相同的词。

    ```
    awk '{printf("%s",$1); for (i=2;i<=NF;++i) if (!a[$1" "$i]++) printf(" %s", $i); printf("\n");}' 5.pyim > pyim-dict.pyim
    ```

# Ref

1. [合并词库及方法步骤（2012-06-17 更新）](http://forum.ubuntu.org.cn/viewtopic.php?t=364764)
2. [120余万的搜狗细胞词库](http://forum.ubuntu.org.cn/viewtopic.php?t=252407)
3. [fcitx archlinux](https://wiki.archlinux.org/index.php/Fcitx_%28简体中文%29)
4. [词库下载](https://code.google.com/p/hslinuxextra/downloads/list)
