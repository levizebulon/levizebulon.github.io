---
title: 解决ubuntu1604下wps不能使用搜狗输入法问题
date: 2019-01-01 16:42:28
tags:
- 问题
---
## 有时候会遇到Ubuntu下WPS无法使用搜狗输入法的问题，经过在网上搜索尝试，发现是环境变量没有正确配置(虽然之前一直正常使用),不过亲测以后再遇到这种问题按照如下方法修改即可

## 步骤

### Word部分
~~~bash
sudo vim /usr/bin/wps
~~~
在第一行 #!/bin/bash 下添加:
~~~bash
#wps word environment
export XMODIFIERS="@im=fcitx"
export QT_IM_MODULE="fcitx"
~~~

### ppt部分
~~~bash
sudo vim /usr/bin/wpp
~~~
在第一行 #!/bin/bash 下添加:
~~~bash
#wps ppt environment
export XMODIFIERS="@im=fcitx"
export QT_IM_MODULE="fcitx"
~~~

### Excel部分
~~~bash
sudo vim /usr/bin/et
~~~
在第一行 #!/bin/bash 下添加:
~~~bash
#excel word environment
export XMODIFIERS="@im=fcitx"
export QT_IM_MODULE="fcitx"
~~~
