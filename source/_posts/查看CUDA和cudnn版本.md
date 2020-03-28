---
title: 查看CUDA和cudnn版本
date: 2019-01-08 14:01:17
tags:
---
#### 1. 查看CUDA版本
~~~bash
cat /usr/local/cuda/version.txt
~~~

#### 2. 查看cudnn版本
~~~bash
cat /usr/local/cuda/include/cudnn.h | grep CUDNN_MAJOR -A 2
~~~
根据输出判断：eg：
~~~bash
#define CUDNN_MAJOR      5
#define CUDNN_MINOR      1
#define CUDNN_PATCHLEVEL 10
--
#define CUDNN_VERSION    (CUDNN_MAJOR * 1000 + CUDNN_MINOR * 100 + CUDNN_PATCHLEVEL)

#include "driver_types.h"
~~~
CuDNN版本为 5.1.10
