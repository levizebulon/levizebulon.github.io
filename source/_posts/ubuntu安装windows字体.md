---
title: ubuntu安装windows字体
date: 2019-04-21 23:16:24
tags:
- Others
---
将所有的 Windows 字体复制到 '''/usr/share/fonts''' 目录下并使用一下命令安装字体:
bash~~~
mkdir /usr/share/fonts/WindowsFonts
cp /Windowsdrive/Windows/Fonts/* /usr/share/fonts/WindowsFonts
chmod 755 /usr/share/fonts/WindowsFonts/*
~~~

最后，使用命令行重新生成 fontconfig 缓存：
bash~~~
fc-cache
~~~
