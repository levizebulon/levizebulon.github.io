---
title: autosub语音识别字幕生成工具
date: 2018-12-30 20:41:40
tags:
- 工具
---

### [autosub](https://github.com/agermanidis/autosub)

> Autosub是一种用于自动语音识别和字幕生成的实用程序。它将视频或音频文件作为输入，执行语音活动检测以查找语音区域，向Google Web Speech API发出并行请求以生成这些区域的转录，将它们翻译为不同的语言，最后保存结果字幕到磁盘。

### 安装

#### 首先安装FFmpeg
[download](https://www.ffmpeg.org/download.html)

#### pip 安装autosub
[autosub](https://pypi.org/project/autosub/)
~~~bash
pip install autosub
~~~

### 注意：该程序只能在python2.7环境下使用，python3使用会报错

可以在virtualenv环境下配置python2.7
参考[virtualenv](http://linuzb.cn/2019/01/04/virtulenv/)
~~~bash
mkdir autosub_env
cd autosub_env/
virtualenv venv --python=python2.7
source venv/bin/activate
pip install autosub
~~~

### 查看帮助
~~~
autosub -h
autosub --list-languages	# 所有可用的源/目标语言
~~~

### srt-translator 
> 字幕翻译软件

> Translating subtitles in format SubRip from one natural language to other. It is based on Google Translate without API and therefore without payment. Translator have automatic and manual spell checkers.

[下载](https://sourceforge.net/projects/srt-tran/)
