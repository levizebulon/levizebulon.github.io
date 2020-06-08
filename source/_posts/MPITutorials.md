---
title: MPITutorials
date: 2020-05-26 17:42:38
tags:
- HPC
---
ref:[Message Passing Interface (MPI)](https://computing.llnl.gov/tutorials/mpi/)

## 摘要
消息传递接口标准（MPI）是基于MPI论坛共识的消息传递库标准，MPI论坛有40多个参与组织，其中包括供应商，研究人员，软件库开发人员和用户。消息传递接口的目标是为消息传递建立一种可移植，高效且灵活的标准，该标准将广泛用于编写消息传递程序。因此，MPI是第一个标准化的，独立于供应商的消息传递库。使用MPI开发消息传递软件的优势与可移植性，效率和灵活性的设计目标紧密匹配。 MPI不是IEEE或ISO标准，但实际上已成为在HPC平台上编写消息传递程序的“行业标准”。

本教程的目的是教那些不熟悉MPI的人如何根据MPI标准开发和运行并行程序。提出的主要主题集中于对新MPI程序员最有用的主题。本教程首先介绍MPI入门，背景和基本信息。接下来是对MPI例程的详细介绍，这些例程对新MPI程序员最有用，包括MPI环境管理，点对点通信和集体通信例程。提供了C和Fortran中的大量示例以及实验室练习。

教程材料还包括更高级的主题，例如“派生数据类型”，“组和Communicator管理例程”以及“虚拟拓扑”。但是，这些实际上并没有在讲座中介绍，而是为那些有兴趣的人提供“进一步的阅读”。

## 什么是MPI

### 一种接口规范

- M P I = Message Passing Interface
- MPI是针对消息传递库的开发人员和用户的规范。 就其本身而言，它不是一个库-而是有关该库应该是什么的规范。
- 简而言之，消息传递接口的目的是为编写消息传递程序提供广泛使用的标准。 该接口尝试是：
  - Practical 
  - Portable 
  - Efficient 
  - Flexible 
- MPI标准已进行了许多修订，最新版本为MPI-3.x。
- 已经为C和Fortran90语言绑定定义了接口规范：
- 实际的MPI库实现在支持的MPI标准的版本和功能方面有所不同。 开发人员/用户将需要意识到这一点。
  
### 编程模型
- MPI最初是为分布式内存体系结构设计的，该体系结构在当时（1980年代-1990年代初期）变得越来越流行。
- 随着架构趋势的变化，共享内存SMP通过网络进行组合，从而创建了混合分布式内存/共享内存系统。
- MPI实现者调整了其库，以无缝处理两种类型的基础内存体系结构。 他们还采用/开发了处理不同互连和协议的方式。
- 如今，MPI几乎可以在任何硬件平台上运行：
  - Distributed Memory 
  - Shared Memory 
  - Hybrid
- 但是，无论机器的基础物理体系结构如何，编程模型显然仍然是分布式内存模型。
- 所有并行性都是明确的：程序员负责正确识别并行性并使用MPI构造实现并行算法。

### 文档
- 有关MPI标准所有版本的文档，请访问：[http：//www.mpi-forum.org/docs/](http：//www.mpi-forum.org/docs/)。
  
##  LLNL MPI Implementations and Compilers

## 开始学习

### 一般的MPI编程结构
![prog_structure](MPITutorials/prog_structure.gif)

~~~bash
#include "mpi.h"
#include <stdio.h>
#include <stdlib.h>

int main (int argc, char *argv[])
{
int numtasks, rank, dest, source, rc, count, tag=1;
char inmsg, outmsg='x';
MPI_Status Stat;

MPI_Init(&argc,&argv);
MPI_Comm_size(MPI_COMM_WORLD, &numtasks);
MPI_Comm_rank(MPI_COMM_WORLD, &rank);

if (rank == 0) {
  dest = 1;
  source = 1;
  rc = MPI_Send(&outmsg, 1, MPI_CHAR, dest, tag, MPI_COMM_WORLD);
  rc = MPI_Recv(&inmsg, 1, MPI_CHAR, source, tag, MPI_COMM_WORLD, &Stat);
  }

else if (rank == 1) {
  dest = 0;
  source = 0;
  rc = MPI_Recv(&inmsg, 1, MPI_CHAR, source, tag, MPI_COMM_WORLD, &Stat);
  rc = MPI_Send(&outmsg, 1, MPI_CHAR, dest, tag, MPI_COMM_WORLD);
  }

MPI_Finalize();
}
~~~

### 头文件
- 所有进行MPI库调用的程序都必需。
<table border="1">
    <tr>
        <th>C include file </th>
        <th>Fortran include file</th>
    </tr>
    <tr>
        <td>#include "mpi.h" </td>
        <td>	include 'mpif.h' </td>
    </tr>
</table>

- MPI调用格式
<table>
    <tr>
        <th colspan=2>C Binding</th>
    </tr>
    <tr>
        <th>Format:</th>
        <td>rc = MPI_Xxxxx(parameter, ... )</td>
    </tr>
    <tr>
        <th>Example:</th>
        <td>rc = MPI_Bsend(&buf,count,type,dest,tag,comm) </td>
    <tr>
        <th>Error code:</th>
        <td>Returned as "rc". MPI_SUCCESS if successful </td>
    </tr>
</table>

### Communicators and Groups:
- MPI使用称为通信器和组的对象来定义哪些进程集合可以相互通信。
- 大多数MPI例程都要求您指定一个通讯器作为参数。
- 大多数MPI例程都要求您指定一个通讯器作为参数。
- 通讯器和组将在后面详细介绍。 现在，只要需要通信器，就只需使用MPI_COMM_WORLD-它是预定义的通信器，它包含所有MPI进程。
  ![comm_world](MPITutorials/comm_world.gif)

### Rank:

- 在通信器中，每个进程都有自己的唯一整数标识符，该标识符在进程初始化时由系统分配。 rank有时也称为“任务ID”。 rank是连续的，从零开始。
- 程序员用来指定消息的源和目标。 通常由应用程序有条件地用来控制程序的执行（如果rank = 0则执行此操作，如果rank = 1则执行此操作）。

### Error Handling: 
- 如上面“ MPI调用的格式”部分中所述，大多数MPI例程都包含返回/错误代码参数。
- 但是，根据MPI标准，如果发生错误，MPI调用的默认行为是中止。 这意味着您可能将无法捕获MPI_SUCCESS（零）以外的返回/错误代码。
- 该标准确实提供了覆盖此默认错误处理程序的方法。 此处提供有关如何执行此操作的讨论。 您也可以参考[http://www.mpi-forum.org/docs/](http://www.mpi-forum.org/docs/)上相关MPI标准文档的错误处理部分。
- 向用户显示的错误类型取决于实现。

## 环境管理程序
这套例程用于询问和设置MPI执行环境，并涵盖了多种目的，例如初始化和终止MPI环境，查询rank的identity，查询MPI库的版本等。大多数常用的例程如下所述。

### [MPI_Init](https://computing.llnl.gov/tutorials/mpi/man/MPI_Init.txt)

初始化MPI执行环境。 必须在每个MPI程序中调用此函数，必须在任何其他MPI函数之前调用此函数，并且在MPI程序中只能调用一次。 对于C程序，MPI_Init可以用于将命令行参数传递给所有进程，尽管这不是标准要求的，并且取决于实现。
~~~c
MPI_Init (&argc,&argv)
MPI_INIT (ierr) 
~~~

### [MPI_Comm_size](https://computing.llnl.gov/tutorials/mpi/man/MPI_Comm_size.txt)
返回指定通信器中MPI进程的总数，例如MPI_COMM_WORLD。 如果通信器是MPI_COMM_WORLD，则它表示应用程序可用的MPI任务数。

~~~c
MPI_Comm_size (comm,&size)
MPI_COMM_SIZE (comm,size,ierr) 
~~~

### [MPI_Comm_rank](https://computing.llnl.gov/tutorials/mpi/man/MPI_Comm_rank.txt)

返回指定通信器中调用MPI进程的rank。 最初，在通信器MPI_COMM_WORLD中，将为每个进程分配一个介于0和任务数-1之间的唯一整数rank。 该rank通常称为任务ID。 如果一个进程与其他通讯器相关联，那么在每个通讯器中也将具有唯一的rank。

~~~c
MPI_Comm_rank (comm,&rank)
MPI_COMM_RANK (comm,rank,ierr)
~~~

### [MPI_Abort](https://computing.llnl.gov/tutorials/mpi/man/MPI_Abort.txt)

终止与通信器关联的所有MPI进程。 在大多数MPI实现中，无论指定哪个通信器，它都会终止所有进程。

~~~c
MPI_Abort (comm,errorcode)
MPI_ABORT (comm,errorcode,ierr) 
~~~

### [MPI_Get_processor_name](https://computing.llnl.gov/tutorials/mpi/man/MPI_Get_processor_name.txt)

返回进程名称。 还返回名称的长度。 “名称”的缓冲区的大小必须至少为MPI_MAX_PROCESSOR_NAME个字符。 返回到“名称”中的是与实现相关的-可能与“主机名”或“主机” shell命令的输出不同。

~~~c
MPI_Get_processor_name (&name,&resultlength)
MPI_GET_PROCESSOR_NAME (name,resultlength,ierr)
~~~

### [MPI_Get_version](https://computing.llnl.gov/tutorials/mpi/man/MPI_Get_version.txt)

返回由库实现的MPI标准的版本和Subversion。

~~~c
MPI_Get_version (&version,&subversion)
MPI_GET_VERSION (version,subversion,ierr) 
~~~

### [MPI_Initialized](https://computing.llnl.gov/tutorials/mpi/man/MPI_Initialized.txt)

指示是否已调用MPI_Init-将标志返回为逻辑true（1）或false（0）。 MPI要求每个进程仅一次调用MPI_Init。 对于想要使用MPI并准备在必要时调用MPI_Init的模块，这可能会带来问题。 MPI_Initialized解决了此问题。

~~~c
MPI_Initialized (&flag)
MPI_INITIALIZED (flag,ierr) 
~~~

### [MPI_Wtime](https://computing.llnl.gov/tutorials/mpi/man/MPI_Wtime.txt)

返回调用处理器上经过的挂钟时间，以秒为单位（双精度）。

~~~c
MPI_Wtime ()
MPI_WTIME () 
~~~

### [MPI_Wtick](https://computing.llnl.gov/tutorials/mpi/man/MPI_Wtick.txt)

返回MPI_Wtime的分辨率（以秒为单位）（双精度）。
~~~c
MPI_Wtick ()
MPI_WTICK () 
~~~

### [MPI_Finalize](https://computing.llnl.gov/tutorials/mpi/man/MPI_Finalize.txt)
终止MPI执行环境。 此函数应该是每个MPI程序中最后一个调用的MPI例程-此后不得再调用其他MPI例程。

~~~c
MPI_Finalize ()
MPI_FINALIZE (ierr) 
~~~

## Examples: Environment Management Routines
~~~c
// required MPI include file  
#include "mpi.h"
#include <stdio.h>
int main(int argc, char *argv[]) {
   int  numtasks, rank, len, rc; 
   char hostname[MPI_MAX_PROCESSOR_NAME];

   // initialize MPI  
   MPI_Init(&argc,&argv);

   // get number of tasks 
   MPI_Comm_size(MPI_COMM_WORLD,&numtasks);

   // get my rank  
   MPI_Comm_rank(MPI_COMM_WORLD,&rank);

   // this one is obvious  
   MPI_Get_processor_name(hostname, &len);
   printf ("Number of tasks= %d My rank= %d Running on %s\n", numtasks,rank,hostname);


        // do some work with message passing 


   // done with MPI  
   MPI_Finalize();
}
~~~

## MPI Exercise 1

**[ GO TO THE EXERCISE HERE ](https://computing.llnl.gov/tutorials/mpi/exercise.html)**

## 点对点通讯程序

### 一般概念

####  First, a Simple Example: 

**以后再翻译**

#### 点对点操作的类型

- MPI点对点操作通常涉及两个(而且只有两个)不同MPI任务之间的消息传递。一个任务执行发送操作，另一个任务执行匹配的接收操作。
- 有不同类型的发送和接收例程用于不同的目的。例如
  - 同步发送
  - 阻塞发送 / 阻塞接收
  - 非阻塞发送和非阻塞接收
  - 缓冲发送
  - 联合发送/接收
  - “准备好”发送
- 任何类型的发送例程都可以与任何类型的接收例程配对。
- MPI还提供了几个与发送-接收操作相关的例程，比如那些用于等待消息到达或探测消息是否到达的例程。
  
#### Buffering:

- 在一个完美的世界中，每个发送操作都将与它匹配的接收操作完全同步。这种情况很少发生。无论如何，MPI实现必须能够在两个任务不同步时处理存储数据。
- 考虑以下两种情况
  - 发送操作在接收准备就绪前5秒发生，在等待接收时，消息在哪里？
  - 多个发送到达同一个接收任务，一次只能接收一个发送——正在“备份”的消息会发生什么
- MPI实现(不是MPI标准)决定在这些类型的情况下对数据进行什么处理。通常，保留一个系统缓冲区来保存传输中的数据。例如
  此处应该有图片
- 系统缓冲区空间为：
  - 对程序员来说是不透明的，完全由MPI库管理
  - 一种很容易耗尽的有限资源
  - 通常是神秘的，没有很好的记录
  - 能够存在于发送端、接收端或两者都存在
  - 可以提高程序性能，因为它允许发送-接收操作是异步的。
- 用户管理的地址空间(即你的程序变量)称为**应用程序缓冲区**。MPI还提供了一个用户管理的发送缓冲区。

#### 阻塞和非阻塞:

- 大多数MPI点对点例程可以在阻塞或非阻塞模式下使用。
- **阻塞：**
  - 阻塞发送例程只会在安全修改应用程序缓冲区(您的发送数据)以便重用后“返回”。安全意味着修改不会影响接收任务所需的数据。安全并不意味着实际收到了数据——它很可能位于系统缓冲区中。
  - 阻塞发送可以是同步的，这意味着与接收任务发生握手以确认安全发送。
  - 阻塞发送可以是异步的，如果一个系统缓冲区被用来保存数据，最终交付到接收。
  - 阻塞接收只在数据已经到达之后，并且应用程序准备使用的时候“返回”。
- **非阻塞：**
  - 非阻塞发送和接收例程的行为类似——它们几乎会立即返回。它们不等待任何通信事件完成，比如消息从用户内存复制到系统缓冲区空间，或者消息的实际到达。
  - 非阻塞操作只是“请求”MPI库在能够执行操作时执行该操作。用户无法预测何时会发生这种情况。
  - 除非您知道请求的非阻塞操作实际上是由库执行的，否则修改应用程序缓冲区(变量空间)是不安全的。有一些“等待”的例程用来做这个。
  - 非阻塞通信主要用于通信重叠计算和利用可能的性能增益。

<table cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top">
<th>Blocking Send</th>
<th>Non-blocking Send</th>
</tr><tr valign="top">
<td width="50%"><pre>myvar = 0;

for (i=1; i&lt;ntasks; i++) {
   task = i;
   <font color="#DF4442">MPI_Send (&amp;myvar ... ... task ...);
</font>   myvar = myvar + 2

   /* do some work */

   }

</pre></td><td width="50%"><pre>myvar = 0;

for (i=1; i&lt;ntasks; i++) {
   task = i;
   <font color="#DF4442">MPI_Isend (&amp;myvar ... ... task ...);</font>
   myvar = myvar + 2;

   /* do some work */

   <font color="#DF4442">MPI_Wait (...);</font>
   }

</pre></td>
</tr><tr valign="top">
<td align="center"><b>Safe.  Why?<b></b></b></td>
<td align="center"><b>Unsafe. Why?<b></b></b></td>
</tr></tbody></table>

#### 顺序和公平
- **顺序:**
  - MPI保证消息不会相互超越。
  - 如果发送方向同一目的地连续发送两条消息(消息1和消息2)，并且两者匹配相同的receive，则receive操作将在消息2之前接收消息1。
  - 如果接收者连续发送了两个Receive (Receive 1和Receive 2)，并且两者都在寻找相同的消息，则Receive 1将在Receive 2之前接收该消息。
  - 如果有多个线程参与通信操作，则顺序规则不适用。
- **公平：**
  - MPI不能保证公平-它取决于程序员来防止“饥饿操作”。
  - 示例:任务0向任务2发送一条消息。但是，task 1发送一个与task 2接收到的消息相匹配的竞争消息。只有一个发送将完成。

### MPI消息传递例程参数

MPI点对点通信例程通常具有采用以下格式之一的参数列表：

<table width="90%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr><td bgcolor="#FOF5FE"><b>Blocking sends
    </b></td><td><tt><b><nobr>
    MPI_Send(buffer,count,type,dest,tag,comm) 
</nobr></b></tt></td></tr><tr><td bgcolor="#FOF5FE"><b>Non-blocking sends
    </b></td><td><tt><b><nobr>
    MPI_Isend(buffer,count,type,dest,tag,comm,request) 
</nobr></b></tt></td></tr><tr><td bgcolor="#FOF5FE"><b>Blocking receive
    </b></td><td><tt><b><nobr>
    MPI_Recv(buffer,count,type,source,tag,comm,status) 
</nobr></b></tt></td></tr><tr><td bgcolor="#FOF5FE"><b>Non-blocking receive
    </b></td><td><tt><b><nobr>
    MPI_Irecv(buffer,count,type,source,tag,comm,request) </nobr></b></tt></td>
</tr></tbody></table>

### Buffer

引用要发送或接收的数据的程序（应用程序）地址空间。 在大多数情况下，这只是发送/接收的变量名。 对于C程序，此参数通过引用传递，并且通常必须在前面加上一个＆符：＆var1

### Data Count
指示要发送的特定类型的数据元素的数量。

### Data Type
出于可移植性的考虑，MPI预定义了其基本数据类型。 下表列出了标准要求的那些内容。

<table width="90%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr><th colspan="2">C Data Types</th>
    <th colspan="2">Fortran Data Types</th>
</tr><tr><td bgcolor="#FOF5FE"><tt><b>MPI_CHAR</b></tt></td>
    <td>char</td>
    <td bgcolor="#FOF5FE"><tt><b>MPI_CHARACTER</b></tt></td>
    <td>character(1)</td>
</tr><tr><td bgcolor="#FOF5FE"><tt><b>MPI_WCHAR</b></tt></td>
    <td>wchar_t - wide character</td>
    <td bgcolor="#FOF5FE">&nbsp;</td>
    <td>&nbsp;</td>
</tr><tr><td bgcolor="#FOF5FE"><tt><b>MPI_SHORT</b></tt></td>
    <td>signed short int</td>
    <td bgcolor="#FOF5FE">&nbsp;</td>
    <td>&nbsp;</td>
</tr><tr><td bgcolor="#FOF5FE"><tt><b>MPI_INT</b></tt></td>
    <td>signed int</td>
    <td bgcolor="#FOF5FE"><tt><b>MPI_INTEGER<br><font color="gray">MPI_INTEGER1
    <br>MPI_INTEGER2<br>MPI_INTEGER4</font></b></tt></td>
    <td>integer<br>integer*1<br>integer*2<br>integer*4</td>
</tr><tr><td bgcolor="#FOF5FE"><tt><b>MPI_LONG</b></tt></td>
    <td>signed long int</td>
    <td bgcolor="#FOF5FE">&nbsp;</td>
    <td>&nbsp;</td>
</tr><tr><td bgcolor="#FOF5FE"><tt><b>MPI_LONG_LONG_INT
    <br>MPI_LONG_LONG</b></tt></td>
    <td>signed long long int</td>
    <td bgcolor="#FOF5FE">&nbsp;</td>
    <td>&nbsp;</td>
</tr><tr><td bgcolor="#FOF5FE"><tt><b>MPI_SIGNED_CHAR</b></tt></td>
    <td>signed char</td>
    <td bgcolor="#FOF5FE">&nbsp;</td>
    <td>&nbsp;</td>
</tr><tr><td bgcolor="#FOF5FE"><tt><b>MPI_UNSIGNED_CHAR</b></tt></td>
    <td>unsigned char</td>
    <td bgcolor="#FOF5FE">&nbsp;</td>
    <td>&nbsp;</td>
</tr><tr><td bgcolor="#FOF5FE"><tt><b>MPI_UNSIGNED_SHORT</b></tt></td>
    <td>unsigned short int</td>
    <td bgcolor="#FOF5FE">&nbsp;</td>
    <td>&nbsp;</td>
</tr><tr><td bgcolor="#FOF5FE"><tt><b>MPI_UNSIGNED</b></tt></td>
    <td>unsigned int</td>
    <td bgcolor="#FOF5FE">&nbsp;</td>
    <td>&nbsp;</td>
</tr><tr><td bgcolor="#FOF5FE"><tt><b>MPI_UNSIGNED_LONG</b></tt></td>
    <td>unsigned long int</td>
    <td bgcolor="#FOF5FE">&nbsp;</td>
    <td>&nbsp;</td>
</tr><tr><td bgcolor="#FOF5FE"><tt><b>MPI_UNSIGNED_LONG_LONG</b></tt></td>
    <td>unsigned long long int</td>
    <td bgcolor="#FOF5FE">&nbsp;</td>
    <td>&nbsp;</td>
</tr><tr><td bgcolor="#FOF5FE"><tt><b>MPI_FLOAT</b></tt></td>
    <td>float</td>
    <td bgcolor="#FOF5FE"><tt><b>MPI_REAL<br><font color="gray">MPI_REAL2
    <br>MPI_REAL4<br>MPI_REAL8</font>
    </b></tt></td><td>real<br>real*2<br>real*4<br>real*8</td>
</tr><tr><td bgcolor="#FOF5FE"><tt><b>MPI_DOUBLE</b></tt></td>
    <td>double</td>
    <td bgcolor="#FOF5FE"><tt><b>MPI_DOUBLE_PRECISION</b></tt></td>
    <td>double precision</td>
</tr><tr><td bgcolor="#FOF5FE"><tt><b>MPI_LONG_DOUBLE</b></tt></td>
    <td>long double</td>
    <td bgcolor="#FOF5FE">&nbsp;</td>
    <td>&nbsp;</td>
</tr><tr><td bgcolor="#FOF5FE"><tt><b>MPI_C_COMPLEX<br>MPI_C_FLOAT_COMPLEX</b></tt></td>
    <td>float _Complex</td>
    <td bgcolor="#FOF5FE"><tt><b>MPI_COMPLEX</b></tt></td>
    <td>complex</td>
</tr><tr><td bgcolor="#FOF5FE"><tt><b>MPI_C_DOUBLE_COMPLEX</b></tt></td>
    <td>double _Complex</td>
    <td bgcolor="#FOF5FE"><tt><b><font color="gray">MPI_DOUBLE_COMPLEX</font></b></tt></td>
    <td>double complex</td>
</tr><tr><td bgcolor="#FOF5FE"><tt><b>MPI_C_LONG_DOUBLE_COMPLEX</b></tt></td>
    <td>long double _Complex</td>
    <td bgcolor="#FOF5FE">&nbsp;</td>
    <td>&nbsp;</td>
</tr><tr><td bgcolor="#FOF5FE"><tt><b>MPI_C_BOOL</b></tt></td>
    <td>_Bool</td>
    <td bgcolor="#FOF5FE"><tt><b>MPI_LOGICAL</b></tt></td>
    <td>logical</td>

</tr><tr><td bgcolor="#FOF5FE"><tt><b>MPI_INT8_T 
<br>MPI_INT16_T<br>MPI_INT32_T <br>MPI_INT64_T</b></tt></td>
    <td>int8_t<br>int16_t<br>int32_t <br>int64_t</td>
    <td bgcolor="#FOF5FE">&nbsp;</td>
    <td>&nbsp;</td>
</tr><tr><td bgcolor="#FOF5FE"><tt><b>MPI_UINT8_T 
<br>MPI_UINT16_T <br>MPI_UINT32_T <br>MPI_UINT64_T </b></tt></td>
    <td>uint8_t<br>uint16_t<br>uint32_t<br>uint64_t</td>
    <td bgcolor="#FOF5FE">&nbsp;</td>
    <td>&nbsp;</td>
</tr><tr><td bgcolor="#FOF5FE"><tt><b>MPI_BYTE</b></tt></td>
    <td>8 binary digits </td>
    <td bgcolor="#FOF5FE"><tt><b>MPI_BYTE</b></tt></td>        
    <td>8 binary digits </td>
</tr><tr><td bgcolor="#FOF5FE"><tt><b>MPI_PACKED</b></tt></td>
    <td>data packed or unpacked with MPI_Pack()/
        MPI_Unpack</td>
    <td bgcolor="#FOF5FE"><tt><b>MPI_PACKED</b></tt></td>
    <td>data packed or unpacked with MPI_Pack()/
        MPI_Unpack</td>
</tr></tbody></table>

**Notes:**
- 程序员还可以创建自己的数据类型（请参阅[派生数据类型](https://computing.llnl.gov/tutorials/mpi/#Derived_Data_Types)）。
- MPI_BYTE和MPI_PACKED与标准C或Fortran类型不对应。
  <li>Types shown in <font color="gray"><b>GRAY FONT</b></font> are recommended if
        possible.  
    </li>
- 一些实现可能包括其他基本数据类型（MPI_LOGICAL2，MPI_COMPLEX32等）。 检查MPI头文件。

### Destination
发送例程的参数，指示应在何处传递消息。 指定为接收进程的rank。

### Source 
接收例程的参数，指示消息的始发进程。 指定为发送进程的rank。 可以将其设置为通配符MPI_ANY_SOURCE，以接收来自任何任务的消息。

### Tag
程序员分配的用于标识消息的任意非负整数。 发送和接收操作应与消息标签匹配。 对于接收操作，通配符MPI_ANY_TAG可以用于接收任何消息，而不管其标签如何。 MPI标准保证可以将整数0-32767用作标记，但是大多数实现允许的范围远大于此范围。

### Communicator 
指示通信上下文或源或目标字段对其有效的进程集。 除非程序员明确创建新的通信器，否则通常使用预定义的通信器MPI_COMM_WORLD。

### Status 
对于接收操作，指示消息的来源和消息的标签。 在C语言中，此参数是指向预定义结构MPI_Status（例如stat.MPI_SOURCE stat.MPI_TAG）的指针。 在Fortran中，它是大小为MPI_STATUS_SIZE（例如stat（MPI_SOURCE）stat（MPI_TAG））的整数数组。 此外，可以通过MPI_Get_count例程从Status获得所接收的实际字节数。 如果稍后将查询消息的来源，标签或大小，则可以替换常量MPI_STATUS_IGNORE和MPI_STATUSES_IGNORE。

### Request 
由非阻塞发送和接收操作使用。 由于非阻塞操作可能会在获得请求的系统缓冲区空间之前返回，因此系统会发出唯一的“请求编号”。 程序员稍后（在WAIT类型的例程中）使用此系统分配的“句柄”来确定非阻塞操作的完成。 在C语言中，此参数是指向预定义结构MPI_Request的指针。 在Fortran中，它是整数。

## Blocking Message Passing Routines
下面介绍了更常用的MPI阻塞消息传递例程。

### [MPI_Send](https://computing.llnl.gov/tutorials/mpi/man/MPI_Send.txt)
基本阻塞发送操作。 例程仅在发送任务中的应用程序缓冲区可供重用之后才返回。 注意，该例程可以在不同的系统上以不同的方式实现。 MPI标准允许使用系统缓冲区，但不需要使用它。 一些实现可能实际上使用同步发送（在下面讨论）来实现基本的阻塞发送。
<p>
<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Send (&amp;buf,count,datatype,dest,tag,comm)  <br>
    MPI_SEND (buf,count,datatype,dest,tag,comm,ierr)
</b></tt></nobr></td></tr><tr></tr></tbody></table>
</p>

### [MPI_Recv](https://computing.llnl.gov/tutorials/mpi/man/MPI_Recv.txt)

接收消息并阻塞，直到接收任务中的应用程序缓冲区中有所需数据为止。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Recv (&amp;buf,count,datatype,source,tag,comm,&amp;status) <br> 
    MPI_RECV (buf,count,datatype,source,tag,comm,status,ierr) 
</b></tt></nobr></td></tr><tr></tr></tbody></table>

### [MPI_Ssend ](https://computing.llnl.gov/tutorials/mpi/man/MPI_Ssend.txt)

同步阻塞发送：发送消息并进行阻塞，直到发送任务中的应用程序缓冲区可供重新使用且目标进程已开始接收消息为止。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Ssend (&amp;buf,count,datatype,dest,tag,comm)  <br>
    MPI_SSEND (buf,count,datatype,dest,tag,comm,ierr)
</b></tt></nobr></td></tr><tr></tr></tbody></table>

### [MPI_Sendrecv](https://computing.llnl.gov/tutorials/mpi/man/MPI_Sendrecv.txt)

在阻塞之前发送一个消息并发布一个接收。然后将一直阻塞直到发送应用程序缓冲区可以自由重用和接收应用程序缓冲区包含接收到的消息。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Sendrecv (&amp;sendbuf,sendcount,sendtype,dest,sendtag,  <br>
        <font color="#FFFFFF">......</font> 
                 &amp;recvbuf,recvcount,recvtype,source,recvtag,   <br>
        <font color="#FFFFFF">......</font> 
                 comm,&amp;status)   <br>
    MPI_SENDRECV (sendbuf,sendcount,sendtype,dest,sendtag,  <br> 
        <font color="#FFFFFF">......</font> 
                 recvbuf,recvcount,recvtype,source,recvtag,  <br>
        <font color="#FFFFFF">......</font> 
                 comm,status,ierr)
    </b></tt></nobr><p>
</p></td></tr><tr></tr></tbody></table>

### [MPI_Wait](https://computing.llnl.gov/tutorials/mpi/man/MPI_Wait.txt)

### [MPI_Waitany](https://computing.llnl.gov/tutorials/mpi/man/MPI_Waitany.txt)

### [MPI_Waitall](https://computing.llnl.gov/tutorials/mpi/man/MPI_Waitall.txt)

### [MPI_Waitsome](https://computing.llnl.gov/tutorials/mpi/man/MPI_Waitsome.txt)

MPI_Wait会阻塞，直到指定的非阻塞发送或接收操作完成为止。 对于多个非阻塞操作，程序员可以指定任何，全部或部分完成。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Wait     (&amp;request,&amp;status)  <br>
    MPI_Waitany  (count,&amp;array_of_requests,&amp;index,&amp;status)  <br>
    MPI_Waitall  (count,&amp;array_of_requests,&amp;array_of_statuses)  <br>
    MPI_Waitsome (incount,&amp;array_of_requests,&amp;outcount,  <br>
        <font color="#FFFFFF">......</font> 
        &amp;array_of_offsets, &amp;array_of_statuses)  <br>
    MPI_WAIT     (request,status,ierr)  <br>
    MPI_WAITANY  (count,array_of_requests,index,status,ierr)  <br>
    MPI_WAITALL  (count,array_of_requests,array_of_statuses,  <br>
        <font color="#FFFFFF">......</font> 
                 ierr)  <br>
    MPI_WAITSOME (incount,array_of_requests,outcount,  <br>
        <font color="#FFFFFF">......</font> 
                 array_of_offsets, array_of_statuses,ierr)
</b></tt></nobr></td></tr><tr></tr></tbody></table>

### [MPI_Probe](https://computing.llnl.gov/tutorials/mpi/man/MPI_Probe.txt)

对消息执行阻止测试。 “通配符” MPI_ANY_SOURCE和MPI_ANY_TAG可用于测试来自任何来源或带有任何标签的消息。 对于C例程，实际的源和标签将在状态结构中作为status.MPI_SOURCE和status.MPI_TAG返回。 对于Fortran例程，它们将以整数数组status（MPI_SOURCE）和status（MPI_TAG）返回。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Probe (source,tag,comm,&amp;status)  <br>
    MPI_PROBE (source,tag,comm,status,ierr)
</b></tt></nobr></td></tr><tr></tr></tbody></table>

### [MPI_Get_count ](https://computing.llnl.gov/tutorials/mpi/man/MPI_Get_count.txt)

返回接收到的数据类型的元素的来源，标签和数量。 可以与阻塞和非阻塞接收操作一起使用。 对于C例程，实际的源和标签将在状态结构中作为status.MPI_SOURCE和status.MPI_TAG返回。 对于Fortran例程，它们将以整数数组status（MPI_SOURCE）和status（MPI_TAG）返回。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Get_count (&amp;status,datatype,&amp;count)  <br>
    MPI_GET_COUNT (status,datatype,count,ierr)
</b></tt></nobr></td></tr><tr></tr></tbody></table>

## Examples: Blocking Message Passing Routines

<ul>
<p>
Task 0 pings task 1 and awaits return ping
</p><p>

<table width="90%" cellspacing="0" cellpadding="15" border="1"><tbody><tr><td> <!---outer table--->
<table width="100%" cellspacing="0" cellpadding="0" border="0">
<tbody><tr>
<td colspan="2" width="30" bgcolor="FOF5FE" align="center"></td>
<td bgcolor="FOF5FE"><b><br>&nbsp;&nbsp;&nbsp;&nbsp;
C Language - Blocking Message Passing Example
</b><p></p></td>

</tr><tr valign="top"><td colspan="3"><p></p></td>

</tr><tr valign="top">
<td width="30"><pre><font color="#AAAAAA"> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35</font></pre></td>

<td width="1" bgcolor="#7099cc"></td>

<td><pre>   #include <font color="#df4442">"mpi.h"</font>
   #include &lt;stdio.h&gt;

   main(int argc, char *argv[])  {
   int numtasks, rank, dest, source, rc, count, tag=1;  
   char inmsg, outmsg='x';
   <font color="#DF4442">MPI_Status Stat</font>;   <font color="#AAAAAA">// required variable for receive routines</font>

   <font color="#DF4442">MPI_Init</font>(&amp;argc,&amp;argv);
   <font color="#DF4442">MPI_Comm_size</font>(MPI_COMM_WORLD, &amp;numtasks);
   <font color="#DF4442">MPI_Comm_rank</font>(MPI_COMM_WORLD, &amp;rank);

   <font color="#AAAAAA">// task 0 sends to task 1 and waits to receive a return message</font>
   if (rank == 0) {
     dest = 1;
     source = 1;
     <font color="#DF4442">MPI_Send</font>(&amp;outmsg, 1, MPI_CHAR, dest, tag, MPI_COMM_WORLD);
     <font color="#DF4442">MPI_Recv</font>(&amp;inmsg, 1, MPI_CHAR, source, tag, MPI_COMM_WORLD, &amp;Stat);
     } 

   <font color="#AAAAAA">// task 1 waits for task 0 message then returns a message</font>
   else if (rank == 1) {
     dest = 0;
     source = 0;
     <font color="#DF4442">MPI_Recv</font>(&amp;inmsg, 1, MPI_CHAR, source, tag, MPI_COMM_WORLD, &amp;Stat);
     <font color="#DF4442">MPI_Send</font>(&amp;outmsg, 1, MPI_CHAR, dest, tag, MPI_COMM_WORLD);
     }

   <font color="#AAAAAA">// query recieve Stat variable and print message details</font>
   <font color="#DF4442">MPI_Get_count</font>(&amp;Stat, MPI_CHAR, &amp;count);
   printf("Task %d: Received %d char(s) from task %d with tag %d \n",
          rank, count, Stat.MPI_SOURCE, Stat.MPI_TAG);

   <font color="#DF4442">MPI_Finalize</font>();
   }

</pre></td>
</tr></tbody></table>
</td></tr></tbody></table>  <!---outer table--->

<br><br><br>

<table width="90%" cellspacing="0" cellpadding="15" border="1"><tbody><tr><td> <!---outer table--->
<table width="100%" cellspacing="0" cellpadding="0" border="0">
<tbody><tr>
<td colspan="2" width="30" bgcolor="FOF5FE" align="center"><img src="../images/page01.gif"></td>
<td bgcolor="FOF5FE"><b><br>&nbsp;&nbsp;&nbsp;&nbsp;
Fortran - Blocking Message Passing Example
</b><p></p></td>

</tr><tr valign="top"><td colspan="3"><p></p></td>

</tr><tr valign="top">
<td width="30"><pre><font color="#AAAAAA"> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>36</font></pre></td>

<td width="1" bgcolor="#7099cc"></td>

<td><pre>   program ping
   include <font color="#DF4442">'mpif.h'</font>

   integer numtasks, rank, dest, source, count, tag, ierr
   integer <font color="#DF4442">stat(MPI_STATUS_SIZE)</font>   <font color="#AAAAAA">! required variable for receive routines</font>
   character inmsg, outmsg
   outmsg = 'x'
   tag = 1

   call <font color="#DF4442">MPI_INIT</font>(ierr)
   call <font color="#DF4442">MPI_COMM_RANK</font>(MPI_COMM_WORLD, rank, ierr)
   call <font color="#DF4442">MPI_COMM_SIZE</font>(MPI_COMM_WORLD, numtasks, ierr)

   <font color="#AAAAAA">! task 0 sends to task 1 and waits to receive a return message</font>
   if (rank .eq. 0) then
      dest = 1
      source = 1
      call <font color="#DF4442">MPI_SEND</font>(outmsg, 1, MPI_CHARACTER, dest, tag, MPI_COMM_WORLD, ierr)
      call <font color="#DF4442">MPI_RECV</font>(inmsg, 1, MPI_CHARACTER, source, tag, MPI_COMM_WORLD, stat, ierr)

   <font color="#AAAAAA">! task 1 waits for task 0 message then returns a message</font>
   else if (rank .eq. 1) then
      dest = 0
      source = 0
      call <font color="#DF4442">MPI_RECV</font>(inmsg, 1, MPI_CHARACTER, source, tag, MPI_COMM_WORLD, stat, err)
      call <font color="#DF4442">MPI_SEND</font>(outmsg, 1, MPI_CHARACTER, dest, tag, MPI_COMM_WORLD, err)
   endif

   <font color="#AAAAAA">! query recieve Stat variable and print message details</font>
   call <font color="#DF4442">MPI_GET_COUNT</font>(stat, MPI_CHARACTER, count, ierr)
   print *, 'Task ',rank,': Received', count, 'char(s) from task', &amp;
            stat(MPI_SOURCE), 'with tag',stat(MPI_TAG)

   call <font color="#DF4442">MPI_FINALIZE</font>(ierr)

   end

</pre></td>
</tr></tbody></table>
</td></tr></tbody></table>  <!---outer table--->
</p></ul>

## 非阻塞消息传递例程

下面描述了更常用的MPI非阻塞消息传递例程。

### [MPI_Isend](https://computing.llnl.gov/tutorials/mpi/man/MPI_Isend.txt)

标识内存中用作发送缓冲区的区域。程序立即继续进行，而不等待从应用程序缓冲区复制消息。返回一个通信请求句柄来处理挂起的消息状态。直到随后调用MPI_Wait或MPI_Test表明非阻塞发送已经完成,程序才可以修改应用程序缓冲区。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Isend (&amp;buf,count,datatype,dest,tag,comm,&amp;request) <br>
    MPI_ISEND (buf,count,datatype,dest,tag,comm,request,ierr)
</b></tt></nobr></td></tr><tr></tr></tbody></table>

### [MPI_Irecv](https://computing.llnl.gov/tutorials/mpi/man/MPI_Irecv.txt)

标识内存中用作接收缓冲区的区域。程序将立即继续，而无需实际等待消息被接收并复制到应用程序缓冲区中。返回一个通信请求句柄来处理挂起的消息状态。程序必须使用调用MPI_Wait或MPI_Test来确定非阻塞接收操作何时完成，请求的消息在应用程序缓冲区中可用。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Irecv (&amp;buf,count,datatype,source,tag,comm,&amp;request) <br>
    MPI_IRECV (buf,count,datatype,source,tag,comm,request,ierr)
</b></tt></nobr></td></tr><tr></tr></tbody></table>

### [MPI_Issend](https://computing.llnl.gov/tutorials/mpi/man/MPI_Issend.txt)

非阻塞同步发送。与MPI_Isend()类似，除了MPI_Wait()或MPI_Test()表示目标进程何时收到消息。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Issend (&amp;buf,count,datatype,dest,tag,comm,&amp;request) <br>
    MPI_ISSEND (buf,count,datatype,dest,tag,comm,request,ierr)
</b></tt></nobr></td></tr><tr></tr></tbody></table>

### [MPI_Test](https://computing.llnl.gov/tutorials/mpi/man/MPI_Test.txt)

### [MPI_Testany](https://computing.llnl.gov/tutorials/mpi/man/MPI_Testany.txt)

### [MPI_Testall](https://computing.llnl.gov/tutorials/mpi/man/MPI_Testall.txt)

### [MPI_Testsome](https://computing.llnl.gov/tutorials/mpi/man/MPI_Testsome.txt)

MPI测试检查指定的非阻塞发送或接收操作的状态。如果操作已经完成，“flag”参数将返回逻辑true(1)，如果没有，则返回逻辑false(0)。对于多个非阻塞操作，程序员可以指定任意、全部或部分补全。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Test     (&amp;request,&amp;flag,&amp;status) <br>
    MPI_Testany  (count,&amp;array_of_requests,&amp;index,&amp;flag,&amp;status)<br>
    MPI_Testall  (count,&amp;array_of_requests,&amp;flag,&amp;array_of_statuses)<br>
    MPI_Testsome (incount,&amp;array_of_requests,&amp;outcount,<br>
        <font color="#FFFFFF">......</font> 
                 &amp;array_of_offsets, &amp;array_of_statuses)<br>
    MPI_TEST     (request,flag,status,ierr)<br>
    MPI_TESTANY  (count,array_of_requests,index,flag,status,ierr)<br>
    MPI_TESTALL  (count,array_of_requests,flag,array_of_statuses,ierr)<br>
    MPI_TESTSOME (incount,array_of_requests,outcount,<br>
        <font color="#FFFFFF">......</font> 
                 array_of_offsets, array_of_statuses,ierr)<br>
</b></tt></nobr></td></tr><tr></tr></tbody></table>

### [MPI_Iprobe](https://computing.llnl.gov/tutorials/mpi/man/MPI_Iprobe.txt)

对消息执行非阻塞测试。“通配符”MPI_ANY_SOURCE和MPI_ANY_TAG可以用于测试来自任何源或带有任何标记的消息。如果消息已经到达，那么整数“flag”参数将返回逻辑true(1)，如果没有到达，则返回逻辑false(0)。对于C例程，实际的源和标记将在状态结构中作为status返回MPI_SOURCE和status.MPI_TAG。对于Fortran例程，它们将以整数数组状态(MPI源)和状态(MPI标记)的形式返回。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Iprobe (source,tag,comm,&amp;flag,&amp;status)<br>
    MPI_IPROBE (source,tag,comm,flag,status,ierr)
</b></tt></nobr></td></tr><tr></tr></tbody></table>

### 示例:非阻塞消息传递例程

环拓扑中的最近邻交换

<table width="90%" cellspacing="0" cellpadding="15" border="1"><tbody><tr><td> <!---outer table--->
<table width="100%" cellspacing="0" cellpadding="0" border="0">
<tbody><tr>
<td colspan="2" width="30" bgcolor="FOF5FE" align="center"><img src="../images/page01.gif"></td>
<td bgcolor="FOF5FE"><b><br>&nbsp;&nbsp;&nbsp;&nbsp;
C Language - Non-blocking Message Passing Example</b><p></p></td>

</tr><tr valign="top"><td colspan="3"><p></p></td>

</tr><tr valign="top">
<td width="30"><pre><font color="#AAAAAA"> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34</font></pre></td>

<td width="1" bgcolor="#7099cc"></td>

<td><pre>   #include <font color="#DF4442">"mpi.h"</font>
   #include &lt;stdio.h&gt;

   main(int argc, char *argv[])  {
   int numtasks, rank, next, prev, buf[2], tag1=1, tag2=2;
   <font color="#DF4442">MPI_Request reqs[4]</font>;   <font color="#AAAAAA">// required variable for non-blocking calls</font>
   <font color="#DF4442">MPI_Status stats[4]</font>;   <font color="#AAAAAA">// required variable for Waitall routine</font>

   <font color="#DF4442">MPI_Init</font>(&amp;argc,&amp;argv);
   <font color="#DF4442">MPI_Comm_size</font>(MPI_COMM_WORLD, &amp;numtasks);
   <font color="#DF4442">MPI_Comm_rank</font>(MPI_COMM_WORLD, &amp;rank);
   
   <font color="#AAAAAA">// determine left and right neighbors</font>
   prev = rank-1;
   next = rank+1;
   if (rank == 0)  prev = numtasks - 1;
   if (rank == (numtasks - 1))  next = 0;

   <font color="#AAAAAA">// post non-blocking receives and sends for neighbors</font>
   <font color="#DF4442">MPI_Irecv</font>(&amp;buf[0], 1, MPI_INT, prev, tag1, MPI_COMM_WORLD, &amp;reqs[0]);
   <font color="#DF4442">MPI_Irecv</font>(&amp;buf[1], 1, MPI_INT, next, tag2, MPI_COMM_WORLD, &amp;reqs[1]);

   <font color="#DF4442">MPI_Isend</font>(&amp;rank, 1, MPI_INT, prev, tag2, MPI_COMM_WORLD, &amp;reqs[2]);
   <font color="#DF4442">MPI_Isend</font>(&amp;rank, 1, MPI_INT, next, tag1, MPI_COMM_WORLD, &amp;reqs[3]);
  
   <font color="#AAAAAA">   // do some work while sends/receives progress in background</font>

   <font color="#AAAAAA">// wait for all non-blocking operations to complete</font>
   <font color="#DF4442">MPI_Waitall</font>(4, reqs, stats);
  
   <font color="#AAAAAA">   // continue - do more work</font>

   <font color="#DF4442">MPI_Finalize</font>();
   }

</pre></td>
</tr></tbody></table>
</td></tr></tbody></table>

## 集体通信例程

#### 集体作业的类型
- **同步**——进程等待，直到组中的所有成员都达到同步点。
- **数据传送**——广播(broadcast)，散布(scatter)和收集(gather),all to all. 
- **集合计算(归约)**——组中的一个成员从其他成员那里收集数据，并对该数据执行操作(最小、最大、加、乘等)。

#### Scope:
- 集合通信例行程序必须包括通信者范围内的所有进程。
  - 默认情况下，所有进程都是通信器MPI_COMM_WORLD中的成员。
  - 程序员可以定义其他通信器。有关详细信息，请参阅[ Group and Communicator Management Routines](https://computing.llnl.gov/tutorials/mpi/#Group_Management_Routines)部分。
- 如果通信器中甚至有一个任务不参与，就可能发生意外的行为，包括程序失败。
- 确保通信器中的所有进程都参与到任何集体操作中是程序员的责任。

#### 编程注意事项和限制

- 集体通信例程不接受消息标签参数。
- 进程子集内的集体操作是通过首先将子集划分为新组，然后将新组附加到新通信器来完成的((discussed in the [Group and Communicator Management Routines](https://computing.llnl.gov/tutorials/mpi/#Group_Management_Routines) section). )
- 只能与MPI预定义的数据类型一起使用——而不能与MPI[派生的数据类型](https://computing.llnl.gov/tutorials/mpi/#Derived_Data_Types)一起使用。
- MPI-2扩展了大多数集合操作，允许在通信器之间移动数据(这里没有介绍)。
- 对于MPI-3，集合操作可以是阻塞的，也可以是非阻塞的。本教程只介绍阻塞操作。

### 集合通信例程

#### [MPI_Barrier](https://computing.llnl.gov/tutorials/mpi/man/MPI_Barrier.txt)

同步操作。在组中创建barrier同步。每个任务在到达MPI_Barrier调用时都会阻塞，直到组中的所有任务都到达相同的MPI_Barrier调用为止。然后所有的任务都可以继续进行。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Barrier (comm)<br>
    MPI_BARRIER (comm,ierr)
</b></tt></nobr></td></tr><tr></tr></tbody></table>

#### [MPI_Bcast](https://computing.llnl.gov/tutorials/mpi/man/MPI_Bcast.txt)

数据移动操作。从rank为“root”的进程向组中的所有其他进程广播(发送)消息。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Bcast (&amp;buffer,count,datatype,root,comm)   <br>
    MPI_BCAST (buffer,count,datatype,root,comm,ierr)
</b></tt></nobr></td></tr><tr></tr></tbody></table>

#### [MPI_Scatter](https://computing.llnl.gov/tutorials/mpi/man/MPI_Scatter.txt)

数据移动操作。将来自单个源任务的不同消息分发到组中的每个任务。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Scatter (&amp;sendbuf,sendcnt,sendtype,&amp;recvbuf,   <br>
        <font color="#FFFFFF">......</font> 
                recvcnt,recvtype,root,comm)  <br>
    MPI_SCATTER (sendbuf,sendcnt,sendtype,recvbuf,  <br> 
        <font color="#FFFFFF">......</font> 
                recvcnt,recvtype,root,comm,ierr)
</b></tt></nobr></td></tr><tr></tr></tbody></table>

#### [MPI_Gather](https://computing.llnl.gov/tutorials/mpi/man/MPI_Gather.txt)

数据移动操作。将组中每个任务的不同消息收集到单个目标任务。这个例程是MPI_Scatter的反向操作。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Gather (&amp;sendbuf,sendcnt,sendtype,&amp;recvbuf,  <br>
        <font color="#FFFFFF">......</font> 
               recvcount,recvtype,root,comm)  <br>
    MPI_GATHER (sendbuf,sendcnt,sendtype,recvbuf,  <br>
        <font color="#FFFFFF">......</font> 
               recvcount,recvtype,root,comm,ierr)  
</b></tt></nobr></td></tr><tr></tr></tbody></table>

#### [MPI_Allgather](https://computing.llnl.gov/tutorials/mpi/man/MPI_Allgather.txt)

数据移动操作。数据与组中所有任务的连接。组中的每个任务实际上在组内执行一对所有的广播操作。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Allgather (&amp;sendbuf,sendcount,sendtype,&amp;recvbuf,  <br>
        <font color="#FFFFFF">......</font> 
                  recvcount,recvtype,comm) <br>
    MPI_ALLGATHER (sendbuf,sendcount,sendtype,recvbuf, <br> 
        <font color="#FFFFFF">......</font> 
                  recvcount,recvtype,comm,info)
</b></tt></nobr></td></tr><tr></tr></tbody></table>

#### [ MPI_Reduce ](https://computing.llnl.gov/tutorials/mpi/man/MPI_Reduce.txt)

集体计算操作。对组中的所有任务应用reduce操作，并将结果放在一个任务中。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Reduce (&amp;sendbuf,&amp;recvbuf,count,datatype,op,root,comm) <br>
    MPI_REDUCE (sendbuf,recvbuf,count,datatype,op,root,comm,ierr)
</b></tt></nobr></td></tr><tr></tr></tbody></table>

下面显示了预定义的MPI精简操作。用户还可以使用[MPI_Op_create](https://computing.llnl.gov/tutorials/mpi/man/MPI_Op_create.txt)例程定义自己的reduce函数。

<table width="90%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="TOP">
<th colspan="2">MPI Reduction Operation</th> 
<th>C Data Types</th>
<th>Fortran Data Type</th>
</tr><tr valign="TOP">
<td bgcolor="#FOF5FE"><tt><b> MPI_MAX    
</b></tt></td><td>maximum       
</td><td>integer, float      
</td><td>integer, real, complex  
</td></tr><tr valign="TOP">
<td bgcolor="#FOF5FE"><tt><b>MPI_MIN    
</b></tt></td><td>minimum       
</td><td>integer, float      
</td><td>integer, real, complex  
</td></tr><tr valign="TOP">
<td bgcolor="#FOF5FE"><tt><b>MPI_SUM    
</b></tt></td><td>sum          
</td><td>integer, float      
</td><td>integer, real, complex  
</td></tr><tr valign="TOP">
<td bgcolor="#FOF5FE"><tt><b>MPI_PROD    
</b></tt></td><td>product      
</td><td>integer, float      
</td><td>integer, real, complex  
</td></tr><tr valign="TOP">
<td bgcolor="#FOF5FE"><tt><b>MPI_LAND    
</b></tt></td><td>logical AND   
</td><td>integer           
</td><td>logical                 
</td></tr><tr valign="TOP">
<td bgcolor="#FOF5FE"><tt><b>MPI_BAND    
</b></tt></td><td>bit-wise AND  
</td><td>integer, MPI_BYTE   
</td><td>integer, MPI_BYTE      
</td></tr><tr valign="TOP">
<td bgcolor="#FOF5FE"><tt><b>MPI_LOR    
</b></tt></td><td>logical OR    
</td><td>integer            
</td><td>logical                 
</td></tr><tr valign="TOP">
<td bgcolor="#FOF5FE"><tt><b>MPI_BOR    
</b></tt></td><td>bit-wise OR   
</td><td>integer, MPI_BYTE   
</td><td>integer, MPI_BYTE       
</td></tr><tr valign="TOP">
<td bgcolor="#FOF5FE"><tt><b>MPI_LXOR    
</b></tt></td><td>logical XOR   
</td><td>integer           
</td><td>logical                 
</td></tr><tr valign="TOP">
<td bgcolor="#FOF5FE"><tt><b>MPI_BXOR    
</b></tt></td><td>bit-wise XOR  
</td><td>integer, MPI_BYTE   
</td><td>integer, MPI_BYTE      
</td></tr><tr valign="TOP">
<td bgcolor="#FOF5FE"><tt><b>MPI_MAXLOC  
</b></tt></td><td>max value and location
</td><td>float, double and long double         
</td><td>real, complex,double precision        
</td></tr><tr valign="TOP">
<td bgcolor="#FOF5FE"><tt><b>MPI_MINLOC  
</b></tt></td><td>min value and location 
</td><td>float, double and long double         
</td><td>real, complex, double precision        
</td></tr></tbody></table>

- MPI_Reduce手册页中的说明:操作总是被假定为关联的。所有预定义的操作都假定是可交换的。用户可以定义被假定为关联而非交换的操作。归约的“规范”评估顺序由组中进程的rank决定。但是，实现可以利用结合性，或者结合性和交换性来改变计算顺序。这可能会改变非严格结合性和可交换性操作(如浮点加法)的归约结果。强烈建议实现MPI_REDUCE，以便当函数被应用到相同的参数上时，以相同的顺序出现时，可以获得相同的结果。注意，这可能会阻止优化利用处理器的物理位置。

#### [ MPI_Allreduce ](https://computing.llnl.gov/tutorials/mpi/man/MPI_Allreduce.txt)

集体计算操作+数据移动。应用归约操作并将结果放置到组中的所有任务中。这相当于MPI_Reduce后再进行MPI_Bcast。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Allreduce (&amp;sendbuf,&amp;recvbuf,count,datatype,op,comm) <br>
    MPI_ALLREDUCE (sendbuf,recvbuf,count,datatype,op,comm,ierr)
</b></tt></nobr></td></tr><tr></tr></tbody></table>

#### [ MPI_Reduce_scatter ](https://computing.llnl.gov/tutorials/mpi/man/MPI_Reduce_scatter.txt)

集体计算操作+数据移动。首先对组中所有任务的向量进行element-wise的归约。接下来，结果向量被分割成不相交的片段并分布在各个任务中。这相当于在MPI_Reduce之后执行MPI_Scatter操作。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Reduce_scatter (&amp;sendbuf,&amp;recvbuf,recvcount,datatype, <br>
        <font color="#FFFFFF">......</font>
         op,comm) <br>
    MPI_REDUCE_SCATTER (sendbuf,recvbuf,recvcount,datatype, <br>
        <font color="#FFFFFF">......</font>
         op,comm,ierr)
</b></tt></nobr></td></tr><tr></tr></tbody></table>

#### [ MPI_Alltoall ](https://computing.llnl.gov/tutorials/mpi/man/MPI_Alltoall.txt)

数据移动操作。组中的每个任务执行scatter操作，按照索引的顺序向组中的所有任务发送不同的消息。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Alltoall (&amp;sendbuf,sendcount,sendtype,&amp;recvbuf, <br>
        <font color="#FFFFFF">......</font>
                 recvcnt,recvtype,comm) <br>
    MPI_ALLTOALL (sendbuf,sendcount,sendtype,recvbuf, <br>
        <font color="#FFFFFF">......</font>
                 recvcnt,recvtype,comm,ierr)
</b></tt></nobr></td></tr><tr></tr></tbody></table>

#### [ MPI_Scan ](https://computing.llnl.gov/tutorials/mpi/man/MPI_Scan.txt)

对整个任务组的归约操作执行扫描操作。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Scan (&amp;sendbuf,&amp;recvbuf,count,datatype,op,comm) <br>
    MPI_SCAN (sendbuf,recvbuf,count,datatype,op,comm,ierr)
</b></tt></nobr></td></tr><tr></tr></tbody></table>

### 例子:集体通信

对数组的行执行分散操作

<table width="90%" cellspacing="0" cellpadding="15" border="1"><tbody><tr><td> <!---outer table--->
<table width="100%" cellspacing="0" cellpadding="0" border="0">
<tbody><tr>
<td colspan="2" width="30" bgcolor="FOF5FE" align="center"><img src="../images/page01.gif"></td>
<td bgcolor="FOF5FE"><b><br>&nbsp;&nbsp;&nbsp;&nbsp;
C Language - Collective Communications Example
</b><p></p></td>

</tr><tr valign="top"><td colspan="3"><p></p></td>

</tr><tr valign="top">
<td width="30"><pre><font color="#AAAAAA"> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33</font></pre></td>

<td width="1" bgcolor="#7099cc"></td>

<td><pre>   #include <font color="#DF4442">"mpi.h"</font>
   #include &lt;stdio.h&gt;
   #define SIZE 4

   main(int argc, char *argv[])  {
   int numtasks, rank, sendcount, recvcount, source;
   float sendbuf[SIZE][SIZE] = {
     {1.0, 2.0, 3.0, 4.0},
     {5.0, 6.0, 7.0, 8.0},
     {9.0, 10.0, 11.0, 12.0},
     {13.0, 14.0, 15.0, 16.0}  };
   float recvbuf[SIZE];

   <font color="#DF4442">MPI_Init</font>(&amp;argc,&amp;argv);
   <font color="#DF4442">MPI_Comm_rank</font>(MPI_COMM_WORLD, &amp;rank);
   <font color="#DF4442">MPI_Comm_size</font>(MPI_COMM_WORLD, &amp;numtasks);

   if (numtasks == SIZE) {
     <font color="#AAAAAA">// define source task and elements to send/receive, then perform collective scatter</font>
     source = 1;
     sendcount = SIZE;
     recvcount = SIZE;
     <font color="#DF4442">MPI_Scatter</font>(sendbuf,sendcount,MPI_FLOAT,recvbuf,recvcount,
                 MPI_FLOAT,source,MPI_COMM_WORLD);

     printf("rank= %d  Results: %f %f %f %f\n",rank,recvbuf[0],
            recvbuf[1],recvbuf[2],recvbuf[3]);
     }
   else
     printf("Must specify %d processors. Terminating.\n",SIZE);

   <font color="#DF4442">MPI_Finalize</font>();
   }

</pre></td>
</tr></tbody></table>
</td></tr></tbody></table>

<table width="90%" cellspacing="0" cellpadding="15" border="1"><tbody><tr><td> <!---outer table--->
<table width="100%" cellspacing="0" cellpadding="0" border="0">
<tbody><tr>
<td colspan="2" width="30" bgcolor="FOF5FE" align="center"><img src="../images/page01.gif"></td>
<td bgcolor="FOF5FE"><b><br>&nbsp;&nbsp;&nbsp;&nbsp;
Fortran - Collective Communications Example
</b><p></p></td>

</tr><tr valign="top"><td colspan="3"><p></p></td>

</tr><tr valign="top">
<td width="30"><pre><font color="#AAAAAA"> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>36</font></pre></td>

<td width="1" bgcolor="#7099cc"></td>

<td><pre>   program scatter
   include <font color="#DF4442">'mpif.h'</font>

   integer SIZE
   parameter(SIZE=4)
   integer numtasks, rank, sendcount, recvcount, source, ierr
   real*4 sendbuf(SIZE,SIZE), recvbuf(SIZE)

   <font color="#AAAAAA">! Fortran stores this array in column major order, so the 
   ! scatter will actually scatter columns, not rows.</font>
   data sendbuf /1.0, 2.0, 3.0, 4.0, &amp;
                 5.0, 6.0, 7.0, 8.0, &amp;
                 9.0, 10.0, 11.0, 12.0, &amp;
                 13.0, 14.0, 15.0, 16.0 /

   call <font color="#DF4442">MPI_INIT</font>(ierr)
   call <font color="#DF4442">MPI_COMM_RANK</font>(MPI_COMM_WORLD, rank, ierr)
   call <font color="#DF4442">MPI_COMM_SIZE</font>(MPI_COMM_WORLD, numtasks, ierr)

   if (numtasks .eq. SIZE) then
      <font color="#AAAAAA">! define source task and elements to send/receive, then perform collective scatter</font>
      source = 1
      sendcount = SIZE
      recvcount = SIZE
      call <font color="#DF4442">MPI_SCATTER</font>(sendbuf, sendcount, MPI_REAL, recvbuf, recvcount, MPI_REAL, &amp;
                       source, MPI_COMM_WORLD, ierr)

      print *, 'rank= ',rank,' Results: ',recvbuf 

   else
      print *, 'Must specify',SIZE,' processors.  Terminating.' 
   endif

   call <font color="#DF4442">MPI_FINALIZE</font>(ierr)

   end

</pre></td>
</tr></tbody></table>
</td></tr></tbody></table>

## 派生数据类型

- 如前所述，MPI预定义了它的基本数据类型

<table width="90%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr><th colspan="2">C Data Types</th>
<th>Fortran Data Types</th>
</tr><tr valign="top">
<td width="33%"><pre>MPI_CHAR
MPI_WCHAR
MPI_SHORT
MPI_INT
MPI_LONG
MPI_LONG_LONG_INT 
MPI_LONG_LONG	 	 
MPI_SIGNED_CHAR
MPI_UNSIGNED_CHAR
MPI_UNSIGNED_SHORT
MPI_UNSIGNED_LONG
MPI_UNSIGNED
MPI_FLOAT
MPI_DOUBLE
MPI_LONG_DOUBLE
</pre></td>
<td width="33%"><pre>MPI_C_COMPLEX
MPI_C_FLOAT_COMPLEX
MPI_C_DOUBLE_COMPLEX
MPI_C_LONG_DOUBLE_COMPLEX	 	 
MPI_C_BOOL
MPI_LOGICAL
MPI_C_LONG_DOUBLE_COMPLEX 	 
MPI_INT8_T 
MPI_INT16_T
MPI_INT32_T 
MPI_INT64_T	 	 
MPI_UINT8_T 
MPI_UINT16_T 
MPI_UINT32_T 
MPI_UINT64_T
MPI_BYTE
MPI_PACKED
</pre></td>
<td width="33%"><pre>MPI_CHARACTER
MPI_INTEGER
MPI_INTEGER1 
MPI_INTEGER2
MPI_INTEGER4
MPI_REAL
MPI_REAL2 
MPI_REAL4
MPI_REAL8
MPI_DOUBLE_PRECISION
MPI_COMPLEX
MPI_DOUBLE_COMPLEX
MPI_LOGICAL
MPI_BYTE
MPI_PACKED
</pre></td>
</tr></tbody></table>

- MPI还提供了根据MPI基本数据类型序列定义自己的数据结构的工具。这种用户定义的结构称为派生数据类型。
- 基本数据类型是连续的。派生数据类型允许您以方便的方式指定非连续数据，并将其视为连续数据。
- MPI提供了几种构造派生数据类型的方法
  - 连续的（Contiguous）
  - 向量（Vector）
  - Indexed
  - Struct

### 派生数据类型例程

#### [ MPI_Type_contiguous](https://computing.llnl.gov/tutorials/mpi/man/MPI_Type_contiguous.txt)

最简单的构造函数。通过对现有数据类型进行计数复制，生成新的数据类型。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Type_contiguous (count,oldtype,&amp;newtype) <br>
    MPI_TYPE_CONTIGUOUS (count,oldtype,newtype,ierr)
</b></tt></nobr></td></tr><tr></tr></tbody></table>

#### [MPI_Type_vector](https://computing.llnl.gov/tutorials/mpi/man/MPI_Type_vector.txt)

#### [MPI_Type_hvector](https://computing.llnl.gov/tutorials/mpi/man/MPI_Type_hvector.txt)

类似于连续，但允许在位移中有规则的间隙(stride)。MPI_Type_hvector与MPI_Type_vector相同，不同的是跨步是以字节为单位指定的。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Type_vector (count,blocklength,stride,oldtype,&amp;newtype)<br> 
    MPI_TYPE_VECTOR (count,blocklength,stride,oldtype,newtype,ierr)
</b></tt></nobr></td></tr><tr></tr></tbody></table>

#### [MPI_Type_indexed](https://computing.llnl.gov/tutorials/mpi/man/MPI_Type_indexed.txt)

#### [ MPI_Type_hindexed ](https://computing.llnl.gov/tutorials/mpi/man/MPI_Type_hindexed.txt)

提供输入数据类型的位移数组作为新数据类型的映射。MPI_Type_hindexed与MPI_Type_indexed是相同的，除了偏移量是用字节指定的。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Type_indexed (count,blocklens[],offsets[],old_type,&amp;newtype)<br>
    MPI_TYPE_INDEXED (count,blocklens(),offsets(),old_type,newtype,ierr)
</b></tt></nobr></td></tr><tr></tr></tbody></table>

#### [MPI_Type_struct](https://computing.llnl.gov/tutorials/mpi/man/MPI_Type_struct.txt)

新的数据类型是根据组件数据类型的完全定义映射形成的。

**NOTE:** 该函数在MPI-2.0中不提倡使用，在MPI-3.0中被MPI_Type_creat_struct替代

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Type_struct (count,blocklens[],offsets[],old_types,&amp;newtype)<br>
    MPI_TYPE_STRUCT (count,blocklens(),offsets(),old_types,newtype,ierr)
</b></tt></nobr></td></tr><tr></tr></tbody></table>

#### [MPI_Type_extent](https://computing.llnl.gov/tutorials/mpi/man/MPI_Type_extent.txt)

返回指定数据类型的大小(以字节为单位)。对于需要指定字节偏移量的MPI子例程很有用。

**NOTE:** 该函数在MPI-2.0中不赞成使用，在MPI-3.0中被MPI_Type_get_extent替换

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Type_extent (datatype,&amp;extent)<br>
    MPI_TYPE_EXTENT (datatype,extent,ierr)
</b></tt></nobr></td></tr><tr></tr></tbody></table>

#### [MPI_Type_commit](https://computing.llnl.gov/tutorials/mpi/man/MPI_Type_commit.txt)

向系统提交新的数据类型。所有用户构造(派生)数据类型都需要。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Type_commit (&amp;datatype)<br>
    MPI_TYPE_COMMIT (datatype,ierr)
</b></tt></nobr></td></tr><tr></tr></tbody></table>

#### [MPI_Type_free](https://computing.llnl.gov/tutorials/mpi/man/MPI_Type_free.txt)

释放指定的数据类型对象。如果在循环中创建了许多数据类型对象，那么使用这个例程对于防止内存耗尽尤其重要。

<table width="75%" cellspacing="0" cellpadding="5" border="1">
<tbody><tr valign="top"><td><nobr><tt><b> 
    MPI_Type_free (&amp;datatype)<br>
    MPI_TYPE_FREE (datatype,ierr)
</b></tt></nobr></td></tr><tr></tr></tbody></table>

### 示例:连续派生数据类型

创建表示数组中的一行的数据类型，并将另一行分发给所有进程。
<table width="90%" cellspacing="0" cellpadding="15" border="1"><tbody><tr><td> <!---outer table--->
<table width="100%" cellspacing="0" cellpadding="0" border="0">
<tbody><tr>
<td colspan="2" width="30" bgcolor="FOF5FE" align="center"><img src="../images/page01.gif"></td>
<td bgcolor="FOF5FE"><b><br>&nbsp;&nbsp;&nbsp;&nbsp;
C Language - Contiguous Derived Data Type Example
</b><p></p></td>

</tr><tr valign="top"><td colspan="3"><p></p></td>

</tr><tr valign="top">
<td width="30"><pre><font color="#AAAAAA"> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>36<br>37<br>38<br>39<br>40<br>41<br>42<br>43</font></pre></td>

<td width="1" bgcolor="#7099cc"></td>

<td><pre>   #include <font color="#DF4442">"mpi.h"</font>
   #include &lt;stdio.h&gt;
   #define SIZE 4

   main(int argc, char *argv[])  {
   int numtasks, rank, source=0, dest, tag=1, i;
   float a[SIZE][SIZE] =
     {1.0, 2.0, 3.0, 4.0,
      5.0, 6.0, 7.0, 8.0,
      9.0, 10.0, 11.0, 12.0,
      13.0, 14.0, 15.0, 16.0};
   float b[SIZE];

   <font color="#DF4442">MPI_Status stat</font>;
   <font color="#DF4442">MPI_Datatype rowtype</font>;   <font color="#AAAAAA">// required variable</font>

   <font color="#DF4442">MPI_Init</font>(&amp;argc,&amp;argv);
   <font color="#DF4442">MPI_Comm_rank</font>(MPI_COMM_WORLD, &amp;rank);
   <font color="#DF4442">MPI_Comm_size</font>(MPI_COMM_WORLD, &amp;numtasks);

   <font color="#AAAAAA">// create contiguous derived data type</font>
   <font color="#DF4442">MPI_Type_contiguous</font>(SIZE, MPI_FLOAT, &amp;rowtype);
   <font color="#DF4442">MPI_Type_commit</font>(&amp;rowtype);

   if (numtasks == SIZE) {
      <font color="#AAAAAA">// task 0 sends one element of rowtype to all tasks</font>
      if (rank == 0) {
         for (i=0; i&lt;numtasks; i++)
           <font color="#DF4442">MPI_Send</font>(&amp;a[i][0], 1, rowtype, i, tag, MPI_COMM_WORLD);
         }

      <font color="#AAAAAA">// all tasks receive rowtype data from task 0</font>
      <font color="#DF4442">MPI_Recv</font>(b, SIZE, MPI_FLOAT, source, tag, MPI_COMM_WORLD, &amp;stat);
      printf("rank= %d  b= %3.1f %3.1f %3.1f %3.1f\n",
             rank,b[0],b[1],b[2],b[3]);
      }
   else
      printf("Must specify %d processors. Terminating.\n",SIZE);

   <font color="#AAAAAA">// free datatype when done using it</font>
   <font color="#DF4442">MPI_Type_free</font>(&amp;rowtype);
   <font color="#DF4442">MPI_Finalize</font>();
   }

</pre></td>
</tr></tbody></table>
</td></tr></tbody></table>

<table width="90%" cellspacing="0" cellpadding="15" border="1"><tbody><tr><td> <!---outer table--->
<table width="100%" cellspacing="0" cellpadding="0" border="0">
<tbody><tr>
<td colspan="2" width="30" bgcolor="FOF5FE" align="center"><img src="../images/page01.gif"></td>
<td bgcolor="FOF5FE"><b><br>&nbsp;&nbsp;&nbsp;&nbsp;
Fortran - Contiguous Derived Data Type Example
</b><p></p></td>

</tr><tr valign="top"><td colspan="3"><p></p></td>

</tr><tr valign="top">
<td width="30"><pre><font color="#AAAAAA"> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>36<br>37<br>38<br>39<br>40<br>41<br>42<br>43<br>44<br>45<br>46</font></pre></td>

<td width="1" bgcolor="#7099cc"></td>

<td><pre>   program contiguous
   include <font color="#DF4442">'mpif.h'</font>

   integer SIZE
   parameter(SIZE=4)
   integer numtasks, rank, source, dest, tag, i,  ierr
   real*4 a(0:SIZE-1,0:SIZE-1), b(0:SIZE-1)
   integer <font color="#DF4442">stat(MPI_STATUS_SIZE)</font>
   integer <font color="#DF4442">columntype</font>   <font color="#AAAAAA">! required variable</font>
   tag = 1

   <font color="#AAAAAA">! Fortran stores this array in column major order</font>
   data a  /1.0, 2.0, 3.0, 4.0, &amp;
            5.0, 6.0, 7.0, 8.0, &amp;
            9.0, 10.0, 11.0, 12.0, &amp; 
            13.0, 14.0, 15.0, 16.0 /

   call <font color="#DF4442">MPI_INIT</font>(ierr)
   call <font color="#DF4442">MPI_COMM_RANK</font>(MPI_COMM_WORLD, rank, ierr)
   call <font color="#DF4442">MPI_COMM_SIZE</font>(MPI_COMM_WORLD, numtasks, ierr)

   <font color="#AAAAAA">! create contiguous derived data type</font>
   call <font color="#DF4442">MPI_TYPE_CONTIGUOUS</font>(SIZE, MPI_REAL, columntype, ierr)
   call <font color="#DF4442">MPI_TYPE_COMMIT</font>(columntype, ierr)
  
   if (numtasks .eq. SIZE) then
      <font color="#AAAAAA">! task 0 sends one element of columntype to all tasks</font>
      if (rank .eq. 0) then
         do i=0, numtasks-1
         call <font color="#DF4442">MPI_SEND</font>(a(0,i), 1, columntype, i, tag, MPI_COMM_WORLD,ierr)
         end do
      endif

      <font color="#AAAAAA">! all tasks receive columntype data from task 0</font>
      source = 0
      call <font color="#DF4442">MPI_RECV</font>(b, SIZE, MPI_REAL, source, tag, MPI_COMM_WORLD, stat, ierr)
      print *, 'rank= ',rank,' b= ',b
   else
      print *, 'Must specify',SIZE,' processors.  Terminating.' 
   endif

   <font color="#AAAAAA">! free datatype when done using it</font>
   call <font color="#DF4442">MPI_TYPE_FREE</font>(columntype, ierr)
   call <font color="#DF4442">MPI_FINALIZE</font>(ierr)

   end

</pre></td>
</tr></tbody></table>
</td></tr></tbody></table>

 Sample program output: 

 <pre>rank= 0  b= 1.0 2.0 3.0 4.0
rank= 1  b= 5.0 6.0 7.0 8.0
rank= 2  b= 9.0 10.0 11.0 12.0
rank= 3  b= 13.0 14.0 15.0 16.0
</pre>

### 示例:向量派生的数据类型

创建表示数组中的列的数据类型，并将不同的列分发给所有进程。

<table width="90%" cellspacing="0" cellpadding="15" border="1"><tbody><tr><td> <!---outer table--->
<table width="100%" cellspacing="0" cellpadding="0" border="0">
<tbody><tr>
<td colspan="2" width="30" bgcolor="FOF5FE" align="center"><img src="../images/page01.gif"></td>
<td bgcolor="FOF5FE"><b><br>&nbsp;&nbsp;&nbsp;&nbsp;
C Language - Vector Derived Data Type Example
</b><p></p></td>

</tr><tr valign="top"><td colspan="3"><p></p></td>

</tr><tr valign="top">
<td width="30"><pre><font color="#AAAAAA"> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>36<br>37<br>38<br>39<br>40<br>41<br>42<br>43<br>44</font></pre></td>

<td width="1" bgcolor="#7099cc"></td>

<td><pre>   #include <font color="#DF4442">"mpi.h"</font>
   #include &lt;stdio.h&gt;
   #define SIZE 4

   main(int argc, char *argv[])  {
   int numtasks, rank, source=0, dest, tag=1, i;
   float a[SIZE][SIZE] = 
     {1.0, 2.0, 3.0, 4.0,  
      5.0, 6.0, 7.0, 8.0, 
      9.0, 10.0, 11.0, 12.0,
     13.0, 14.0, 15.0, 16.0};
   float b[SIZE]; 

   <font color="#DF4442">MPI_Status stat</font>;
   <font color="#DF4442">MPI_Datatype columntype</font>;   <font color="#AAAAAA">// required variable</font>


   <font color="#DF4442">MPI_Init</font>(&amp;argc,&amp;argv);
   <font color="#DF4442">MPI_Comm_rank</font>(MPI_COMM_WORLD, &amp;rank);
   <font color="#DF4442">MPI_Comm_size</font>(MPI_COMM_WORLD, &amp;numtasks);
   
   <font color="#AAAAAA">// create vector derived data type</font>
   <font color="#DF4442">MPI_Type_vector</font>(SIZE, 1, SIZE, MPI_FLOAT, &amp;columntype);
   <font color="#DF4442">MPI_Type_commit</font>(&amp;columntype);

   if (numtasks == SIZE) {
      <font color="#AAAAAA">// task 0 sends one element of columntype to all tasks</font>
      if (rank == 0) {
         for (i=0; i&lt;numtasks; i++) 
            <font color="#DF4442">MPI_Send</font>(&amp;a[0][i], 1, columntype, i, tag, MPI_COMM_WORLD);
         }
 
      <font color="#AAAAAA">// all tasks receive columntype data from task 0</font>
      <font color="#DF4442">MPI_Recv</font>(b, SIZE, MPI_FLOAT, source, tag, MPI_COMM_WORLD, &amp;stat);
      printf("rank= %d  b= %3.1f %3.1f %3.1f %3.1f\n",
             rank,b[0],b[1],b[2],b[3]);
      }
   else
      printf("Must specify %d processors. Terminating.\n",SIZE);

   <font color="#AAAAAA">// free datatype when done using it</font>
   <font color="#DF4442">MPI_Type_free</font>(&amp;columntype);
   <font color="#DF4442">MPI_Finalize</font>();
   }

</pre></td>
</tr></tbody></table>
</td></tr></tbody></table>

<table width="90%" cellspacing="0" cellpadding="15" border="1"><tbody><tr><td> <!---outer table--->
<table width="100%" cellspacing="0" cellpadding="0" border="0">
<tbody><tr>
<td colspan="2" width="30" bgcolor="FOF5FE" align="center"><img src="../images/page01.gif"></td>
<td bgcolor="FOF5FE"><b><br>&nbsp;&nbsp;&nbsp;&nbsp;
Fortran - Vector Derived Data Type Example
</b><p></p></td>

</tr><tr valign="top"><td colspan="3"><p></p></td>

</tr><tr valign="top">
<td width="30"><pre><font color="#AAAAAA"> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>36<br>37<br>38<br>39<br>40<br>41<br>42<br>43<br>44<br>45<br>46</font></pre></td>

<td width="1" bgcolor="#7099cc"></td>

<td><pre>   program vector
   include <font color="#DF4442">'mpif.h'</font>

   integer SIZE
   parameter(SIZE=4)
   integer numtasks, rank, source, dest, tag, i,  ierr
   real*4 a(0:SIZE-1,0:SIZE-1), b(0:SIZE-1)
   integer <font color="#DF4442">stat(MPI_STATUS_SIZE)</font>
   integer <font color="#DF4442">rowtype</font>   <font color="#AAAAAA">! required variable</font>
   tag = 1

   <font color="#AAAAAA">! Fortran stores this array in column major order</font>
   data a  /1.0, 2.0, 3.0, 4.0, &amp;
            5.0, 6.0, 7.0, 8.0,  &amp;
            9.0, 10.0, 11.0, 12.0, &amp;
            13.0, 14.0, 15.0, 16.0 /

   call <font color="#DF4442">MPI_INIT</font>(ierr)
   call <font color="#DF4442">MPI_COMM_RANK</font>(MPI_COMM_WORLD, rank, ierr)
   call <font color="#DF4442">MPI_COMM_SIZE</font>(MPI_COMM_WORLD, numtasks, ierr)

   <font color="#AAAAAA">! create vector derived data type</font>
   call <font color="#DF4442">MPI_TYPE_VECTOR</font>(SIZE, 1, SIZE, MPI_REAL, rowtype, ierr)
   call <font color="#DF4442">MPI_TYPE_COMMIT</font>(rowtype, ierr)
  
   if (numtasks .eq. SIZE) then
      <font color="#AAAAAA">! task 0 sends one element of rowtype to all tasks</font>
      if (rank .eq. 0) then
         do i=0, numtasks-1
         call <font color="#DF4442">MPI_SEND</font>(a(i,0), 1, rowtype, i, tag, MPI_COMM_WORLD, ierr)
         end do
      endif

      <font color="#AAAAAA">! all tasks receive rowtype data from task 0</font>
      source = 0
      call <font color="#DF4442">MPI_RECV</font>(b, SIZE, MPI_REAL, source, tag, MPI_COMM_WORLD, stat, ierr)
      print *, 'rank= ',rank,' b= ',b
   else
      print *, 'Must specify',SIZE,' processors.  Terminating.' 
   endif

   <font color="#AAAAAA">! free datatype when done using it</font>
   call <font color="#DF4442">MPI_TYPE_FREE</font>(rowtype, ierr)
   call <font color="#DF4442">MPI_FINALIZE</font>(ierr)

   end

</pre></td>
</tr></tbody></table>
</td></tr></tbody></table>

 Sample program output: 

 <pre>rank= 0  b= 1.0 5.0 9.0 13.0
rank= 1  b= 2.0 6.0 10.0 14.0
rank= 2  b= 3.0 7.0 11.0 15.0
rank= 3  b= 4.0 8.0 12.0 16.0
</pre>

### 示例:索引派生数据类型

通过提取数组的可变部分来创建数据类型并分发给所有任务。

<table width="90%" cellspacing="0" cellpadding="15" border="1"><tbody><tr><td> <!---outer table--->
<table width="100%" cellspacing="0" cellpadding="0" border="0">
<tbody><tr>
<td colspan="2" width="30" bgcolor="FOF5FE" align="center"><img src="../images/page01.gif"></td>
<td bgcolor="FOF5FE"><b><br>&nbsp;&nbsp;&nbsp;&nbsp;
C Language - Indexed Derived Data Type Example
</b><p></p></td>

</tr><tr valign="top"><td colspan="3"><p></p></td>

</tr><tr valign="top">
<td width="30"><pre><font color="#AAAAAA"> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>36<br>37<br>38<br>39<br>40<br>41<br>42<br>43</font></pre></td>

<td width="1" bgcolor="#7099cc"></td>

<td><pre>   #include <font color="#DF4442">"mpi.h"</font>
   #include &lt;stdio.h&gt;
   #define NELEMENTS 6

   main(int argc, char *argv[])  {
   int numtasks, rank, source=0, dest, tag=1, i;
   int blocklengths[2], displacements[2];
   float a[16] = 
     {1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, 
      9.0, 10.0, 11.0, 12.0, 13.0, 14.0, 15.0, 16.0};
   float b[NELEMENTS]; 

   <font color="#DF4442">MPI_Status stat</font>;
   <font color="#DF4442">MPI_Datatype indextype</font>;   <font color="#AAAAAA">// required variable</font>

   <font color="#DF4442">MPI_Init</font>(&amp;argc,&amp;argv);
   <font color="#DF4442">MPI_Comm_rank</font>(MPI_COMM_WORLD, &amp;rank);
   <font color="#DF4442">MPI_Comm_size</font>(MPI_COMM_WORLD, &amp;numtasks);

   blocklengths[0] = 4;
   blocklengths[1] = 2;
   displacements[0] = 5;
   displacements[1] = 12;
   
   <font color="#AAAAAA">// create indexed derived data type</font>
   <font color="#DF4442">MPI_Type_indexed</font>(2, blocklengths, displacements, MPI_FLOAT, &amp;indextype);
   <font color="#DF4442">MPI_Type_commit</font>(&amp;indextype);

   if (rank == 0) {
     for (i=0; i&lt;numtasks; i++) 
      <font color="#AAAAAA">// task 0 sends one element of indextype to all tasks</font>
        <font color="#DF4442">MPI_Send</font>(a, 1, indextype, i, tag, MPI_COMM_WORLD);
     }

   <font color="#AAAAAA">// all tasks receive indextype data from task 0</font>
   <font color="#DF4442">MPI_Recv</font>(b, NELEMENTS, MPI_FLOAT, source, tag, MPI_COMM_WORLD, &amp;stat);
   printf("rank= %d  b= %3.1f %3.1f %3.1f %3.1f %3.1f %3.1f\n",
          rank,b[0],b[1],b[2],b[3],b[4],b[5]);
   
   <font color="#AAAAAA">// free datatype when done using it</font>
   <font color="#DF4442">MPI_Type_free</font>(&amp;indextype);
   <font color="#DF4442">MPI_Finalize</font>();
   }

</pre></td>
</tr></tbody></table>
</td></tr></tbody></table>

<table width="90%" cellspacing="0" cellpadding="15" border="1"><tbody><tr><td> <!---outer table--->
<table width="100%" cellspacing="0" cellpadding="0" border="0">
<tbody><tr>
<td colspan="2" width="30" bgcolor="FOF5FE" align="center"><img src="../images/page01.gif"></td>
<td bgcolor="FOF5FE"><b><br>&nbsp;&nbsp;&nbsp;&nbsp;
Fortran - Indexed Derived Data Type Example
</b><p></p></td>

</tr><tr valign="top"><td colspan="3"><p></p></td>

</tr><tr valign="top">
<td width="30"><pre><font color="#AAAAAA"> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>36<br>37<br>38<br>39<br>40<br>41<br>42<br>43<br>44<br>45<br>46<br>47</font></pre></td>

<td width="1" bgcolor="#7099cc"></td>

<td><pre>   program indexed
   include <font color="#DF4442">'mpif.h'</font>

   integer NELEMENTS
   parameter(NELEMENTS=6)
   integer numtasks, rank, source, dest, tag, i,  ierr
   integer blocklengths(0:1), displacements(0:1)
   real*4 a(0:15), b(0:NELEMENTS-1)
   integer <font color="#DF4442">stat(MPI_STATUS_SIZE)</font>
   integer <font color="#DF4442">indextype</font>   <font color="#AAAAAA">! required variable</font>
   tag = 1

   data a  /1.0, 2.0, 3.0, 4.0, 5.0, 6.0, 7.0, 8.0, &amp;
            9.0, 10.0, 11.0, 12.0, 13.0, 14.0, 15.0, 16.0 /

   call <font color="#DF4442">MPI_INIT</font>(ierr)
   call <font color="#DF4442">MPI_COMM_RANK</font>(MPI_COMM_WORLD, rank, ierr)
   call <font color="#DF4442">MPI_COMM_SIZE</font>(MPI_COMM_WORLD, numtasks, ierr)

   blocklengths(0) = 4
   blocklengths(1) = 2
   displacements(0) = 5
   displacements(1) = 12

   <font color="#AAAAAA">! create indexed derived data type</font>
   call <font color="#DF4442">MPI_TYPE_INDEXED</font>(2, blocklengths, displacements, MPI_REAL, &amp;
                         indextype, ierr)
   call <font color="#DF4442">MPI_TYPE_COMMIT</font>(indextype, ierr)
  
   if (rank .eq. 0) then
      <font color="#AAAAAA">! task 0 sends one element of indextype to all tasks</font>
      do i=0, numtasks-1
      call <font color="#DF4442">MPI_SEND</font>(a, 1, indextype, i, tag, MPI_COMM_WORLD, ierr)
      end do
   endif

   <font color="#AAAAAA">! all tasks receive indextype data from task 0</font>
   source = 0
   call <font color="#DF4442">MPI_RECV</font>(b, NELEMENTS, MPI_REAL, source, tag, MPI_COMM_WORLD, &amp;
                 stat, ierr)
   print *, 'rank= ',rank,' b= ',b

   <font color="#AAAAAA">! free datatype when done using it</font>
   call <font color="#DF4442">MPI_TYPE_FREE</font>(indextype, ierr)
   call <font color="#DF4442">MPI_FINALIZE</font>(ierr)

   end

</pre></td>
</tr></tbody></table>
</td></tr></tbody></table>

 Sample program output: 

 <pre>rank= 0  b= 6.0 7.0 8.0 9.0 13.0 14.0
rank= 1  b= 6.0 7.0 8.0 9.0 13.0 14.0
rank= 2  b= 6.0 7.0 8.0 9.0 13.0 14.0
rank= 3  b= 6.0 7.0 8.0 9.0 13.0 14.0
</pre>

### 示例:结构派生的数据类型

创建表示粒子的数据类型，并将这些粒子的数组分发给所有进程。

<table width="90%" cellspacing="0" cellpadding="15" border="1"><tbody><tr><td> <!---outer table--->
<table width="100%" cellspacing="0" cellpadding="0" border="0">
<tbody><tr>
<td colspan="2" width="30" bgcolor="FOF5FE" align="center"><img src="../images/page01.gif"></td>
<td bgcolor="FOF5FE"><b><br>&nbsp;&nbsp;&nbsp;&nbsp;
C Language - Struct Derived Data Type Example
</b><p></p></td>

</tr><tr valign="top"><td colspan="3"><p></p></td>

</tr><tr valign="top">
<td width="30"><pre><font color="#AAAAAA"> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>36<br>37<br>38<br>39<br>40<br>41<br>42<br>43<br>44<br>45<br>46<br>47<br>48<br>49<br>50<br>51<br>52<br>53<br>54<br>55<br>56<br>57<br>58<br>59<br>60<br>61<br>62<br>63<br>64<br>65<br>66</font></pre></td>

<td width="1" bgcolor="#7099cc"></td>

<td><pre>   #include <font color="#DF4442">"mpi.h"</font>
   #include &lt;stdio.h&gt;
   #define NELEM 25

   main(int argc, char *argv[])  {
   int numtasks, rank, source=0, dest, tag=1, i;

   typedef struct {
     float x, y, z;
     float velocity;
     int  n, type;
     }          Particle;
   Particle     p[NELEM], particles[NELEM];
   <font color="#DF4442">MPI_Datatype particletype, oldtypes[2]</font>;   <font color="#AAAAAA">// required variables</font>
   int          blockcounts[2];

   <font color="#AAAAAA">// MPI_Aint type used to be consistent with syntax of</font>
   <font color="#AAAAAA">// MPI_Type_extent routine</font>
   <font color="#DF4442">MPI_Aint    offsets[2], extent</font>;

   <font color="#DF4442">MPI_Status stat</font>;

   <font color="#DF4442">MPI_Init</font>(&amp;argc,&amp;argv);
   <font color="#DF4442">MPI_Comm_rank</font>(MPI_COMM_WORLD, &amp;rank);
   <font color="#DF4442">MPI_Comm_size</font>(MPI_COMM_WORLD, &amp;numtasks);
 
   <font color="#AAAAAA">// setup description of the 4 MPI_FLOAT fields x, y, z, velocity</font>
   offsets[0] = 0;
   oldtypes[0] = MPI_FLOAT;
   blockcounts[0] = 4;

   <font color="#AAAAAA">// setup description of the 2 MPI_INT fields n, type</font>
   <font color="#AAAAAA">// need to first figure offset by getting size of MPI_FLOAT</font>
   <font color="#DF4442">MPI_Type_extent</font>(MPI_FLOAT, &amp;extent);
   offsets[1] = 4 * extent;
   oldtypes[1] = MPI_INT;
   blockcounts[1] = 2;

   <font color="#AAAAAA">// define structured type and commit it</font>
   <font color="#DF4442">MPI_Type_struct</font>(2, blockcounts, offsets, oldtypes, &amp;particletype);
   <font color="#DF4442">MPI_Type_commit</font>(&amp;particletype);

   <font color="#AAAAAA">// task 0 initializes the particle array and then sends it to each task</font>
   if (rank == 0) {
     for (i=0; i&lt;NELEM; i++) {
        particles[i].x = i * 1.0;
        particles[i].y = i * -1.0;
        particles[i].z = i * 1.0; 
        particles[i].velocity = 0.25;
        particles[i].n = i;
        particles[i].type = i % 2; 
        }
     for (i=0; i&lt;numtasks; i++) 
        <font color="#DF4442">MPI_Send</font>(particles, NELEM, particletype, i, tag, MPI_COMM_WORLD);
     }
 
   <font color="#AAAAAA">// all tasks receive particletype data</font>
   <font color="#DF4442">MPI_Recv</font>(p, NELEM, particletype, source, tag, MPI_COMM_WORLD, &amp;stat);

   printf("rank= %d   %3.2f %3.2f %3.2f %3.2f %d %d\n", rank,p[3].x,
        p[3].y,p[3].z,p[3].velocity,p[3].n,p[3].type);

   <font color="#AAAAAA">// free datatype when done using it</font>
   <font color="#DF4442">MPI_Type_free</font>(&amp;particletype);
   <font color="#DF4442">MPI_Finalize</font>();
   }

</pre></td>
</tr></tbody></table>
</td></tr></tbody></table>

<table width="90%" cellspacing="0" cellpadding="15" border="1"><tbody><tr><td> <!---outer table--->
<table width="100%" cellspacing="0" cellpadding="0" border="0">
<tbody><tr>
<td colspan="2" width="30" bgcolor="FOF5FE" align="center"><img src="../images/page01.gif"></td>
<td bgcolor="FOF5FE"><b><br>&nbsp;&nbsp;&nbsp;&nbsp;
Fortan - Struct Derived Data Type Example
</b><p></p></td>

</tr><tr valign="top"><td colspan="3"><p></p></td>

</tr><tr valign="top">
<td width="30"><pre><font color="#AAAAAA"> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>36<br>37<br>38<br>39<br>40<br>41<br>42<br>43<br>44<br>45<br>46<br>47<br>48<br>49<br>50<br>51<br>52<br>53<br>54<br>55<br>56<br>57<br>58<br>59<br>60<br>61<br>62<br>63</font></pre></td>

<td width="1" bgcolor="#7099cc"></td>

<td><pre>   program struct
   include <font color="#DF4442">'mpif.h'</font>

   integer NELEM
   parameter(NELEM=25)
   integer numtasks, rank, source, dest, tag, i,  ierr
   integer <font color="#DF4442">stat(MPI_STATUS_SIZE)</font>

   type Particle
   sequence
   real*4 x, y, z, velocity
   integer n, type
   end type Particle

   type (Particle) p(NELEM), particles(NELEM)
   integer <font color="#DF4442">particletype, oldtypes(0:1)</font>   <font color="#AAAAAA">! required variables</font>
   integer blockcounts(0:1), offsets(0:1), extent
   tag = 1

   call <font color="#DF4442">MPI_INIT</font>(ierr)
   call <font color="#DF4442">MPI_COMM_RANK</font>(MPI_COMM_WORLD, rank, ierr)
   call <font color="#DF4442">MPI_COMM_SIZE</font>(MPI_COMM_WORLD, numtasks, ierr)

   <font color="#AAAAAA">! setup description of the 4 MPI_REAL fields x, y, z, velocity</font>
   offsets(0) = 0
   oldtypes(0) = MPI_REAL
   blockcounts(0) = 4

   <font color="#AAAAAA">! setup description of the 2 MPI_INTEGER fields n, type</font> 
   <font color="#AAAAAA">! need to first figure offset by getting size of MPI_REAL</font>
   call <font color="#DF4442">MPI_TYPE_EXTENT</font>(MPI_REAL, extent, ierr)
   offsets(1) = 4 * extent
   oldtypes(1) = MPI_INTEGER
   blockcounts(1) = 2

   <font color="#AAAAAA">! define structured type and commit it</font> 
   call <font color="#DF4442">MPI_TYPE_STRUCT</font>(2, blockcounts, offsets, oldtypes, &amp;
                        particletype, ierr)
   call <font color="#DF4442">MPI_TYPE_COMMIT</font>(particletype, ierr)
  
   <font color="#AAAAAA">! task 0 initializes the particle array and then sends it to each task</font>
   if (rank .eq. 0) then
      do i=0, NELEM-1
      particles(i) = Particle ( 1.0*i, -1.0*i, 1.0*i, 0.25, i, mod(i,2) )
      end do

      do i=0, numtasks-1
      call <font color="#DF4442">MPI_SEND</font>(particles, NELEM, particletype, i, tag, &amp;
                    MPI_COMM_WORLD, ierr)
      end do
   endif

   <font color="#AAAAAA">! all tasks receive particletype data</font>
   source = 0
   call <font color="#DF4442">MPI_RECV</font>(p, NELEM, particletype, source, tag, &amp;
                 MPI_COMM_WORLD, stat, ierr)

   print *, 'rank= ',rank,' p(3)= ',p(3)

   <font color="#AAAAAA">! free datatype when done using it</font>
   call <font color="#DF4442">MPI_TYPE_FREE</font>(particletype, ierr)
   call <font color="#DF4442">MPI_FINALIZE</font>(ierr)
   end

</pre></td>
</tr></tbody></table>
</td></tr></tbody></table>

 Sample program output: 

 <pre>rank= 0   3.00 -3.00 3.00 0.25 3 1
rank= 2   3.00 -3.00 3.00 0.25 3 1
rank= 1   3.00 -3.00 3.00 0.25 3 1
rank= 3   3.00 -3.00 3.00 0.25 3 1
</pre>

## 组和通信子管理例程

###  Groups vs. Communicators: 

- 组是一组有序的进程集合。组中的每个进程都与一个惟一的整数rank相关联。rank值从0到N-1，其中N是组中的进程数。在MPI中，组在系统内存中表示为一个对象。程序员只能通过一个“句柄”来访问它。组总是与通信器对象关联。
- 通信器包含一组可以相互通信的进程。所有MPI消息必须指定一个通信器。从最简单的意义上说，通信器是一个额外的“标记”，必须包含在MPI调用中。像组一样，通信器在系统内存中表示为对象，程序员只能通过“句柄”访问它。例如，包含所有任务的通信器的句柄是MPI_COMM_WORLD。
- 从程序员的角度来看，一个组和一个通信者是一体的。组例程主要用于指定应该使用哪些进程来构造通信器。

### 组和通信器对象的主要用途
1. 允许您根据功能将任务组织为任务组。
2. 启用跨相关任务子集的集体通信操作。
3. 为实现用户定义的虚拟拓扑提供基础
4. 提供安全通讯

### 编程注意事项和限制

- 组/通信子是动态的——它可以在程序执行期间创建和销毁
- 过程可以是在一个以上的组/通信子。他们在各组/通信子的rank是唯一的。
- MPI提供了40多个与组、通信器和虚拟拓扑相关的例程。
- Typical usage: 
  1. 使用MPI_COMM_group从MPI_COMM_WORLD中提取全局group的句柄
  2. 使用MPI_group_incl形成新组作为全局组的子集
  3. 使用MPI_Comm_Create为新组创建新的通信器
  4. 使用MPI_Comm_rank确定新通信器中的新rank
  5. 使用任何MPI消息传递例程进行通信
  6. 完成后，使用MPI_Comm_free和MPI_group_free释放新的通信和组(可选)

### Group and Communicator Management Routines

为单独的集体通信交换创建两个不同的进程组。还需要创建新的通信器。

<table width="90%" cellspacing="0" cellpadding="15" border="1"><tbody><tr><td> <!---outer table--->
<table width="100%" cellspacing="0" cellpadding="0" border="0">
<tbody><tr>
<td colspan="2" width="30" bgcolor="FOF5FE" align="center"><img src="../images/page01.gif"></td>
<td bgcolor="FOF5FE"><b><br>&nbsp;&nbsp;&nbsp;&nbsp;
C Language - Group and Communicator Example
</b><p></p></td>

</tr><tr valign="top"><td colspan="3"><p></p></td>

</tr><tr valign="top">
<td width="30"><pre><font color="#AAAAAA"> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>36<br>37<br>38<br>39<br>40<br>41<br>42<br>43</font></pre></td>

<td width="1" bgcolor="#7099cc"></td>

<td><pre>   #include <font color="DF4442">"mpi.h"</font>
   #include &lt;stdio.h&gt;
   #define NPROCS 8

   main(int argc, char *argv[])  {
   int        rank, new_rank, sendbuf, recvbuf, numtasks,
              ranks1[4]={0,1,2,3}, ranks2[4]={4,5,6,7};
   <font color="DF4442">MPI_Group  orig_group, new_group</font>;   <font color="#AAAAAA">// required variables</font>
   <font color="DF4442">MPI_Comm   new_comm</font>;   <font color="#AAAAAA">// required variable</font>

   <font color="DF4442">MPI_Init</font>(&amp;argc,&amp;argv);
   <font color="DF4442">MPI_Comm_rank</font>(MPI_COMM_WORLD, &amp;rank);
   <font color="DF4442">MPI_Comm_size</font>(MPI_COMM_WORLD, &amp;numtasks);

   if (numtasks != NPROCS) {
     printf("Must specify MP_PROCS= %d. Terminating.\n",NPROCS);
     <font color="DF4442">MPI_Finalize</font>();
     exit(0);
     }

   sendbuf = rank;

   <font color="#AAAAAA">// extract the original group handle</font>
   <font color="DF4442">MPI_Comm_group</font>(MPI_COMM_WORLD, &amp;orig_group);

   <font color="#AAAAAA">//  divide tasks into two distinct groups based upon rank</font>
   if (rank &lt; NPROCS/2) {
     <font color="DF4442">MPI_Group_incl</font>(orig_group, NPROCS/2, ranks1, &amp;new_group);
     }
   else {
     <font color="DF4442">MPI_Group_incl</font>(orig_group, NPROCS/2, ranks2, &amp;new_group);
     }

   <font color="#AAAAAA">// create new new communicator and then perform collective communications</font>
   <font color="DF4442">MPI_Comm_create</font>(MPI_COMM_WORLD, new_group, &amp;new_comm);
   <font color="DF4442">MPI_Allreduce</font>(&amp;sendbuf, &amp;recvbuf, 1, MPI_INT, MPI_SUM, new_comm);

   <font color="#AAAAAA">// get rank in new group</font>
   <font color="DF4442">MPI_Group_rank</font> (new_group, &amp;new_rank);
   printf("rank= %d newrank= %d recvbuf= %d\n",rank,new_rank,recvbuf);

   <font color="DF4442">MPI_Finalize</font>();
   }

</pre></td>
</tr></tbody></table>
</td></tr></tbody></table>

<table width="90%" cellspacing="0" cellpadding="15" border="1"><tbody><tr><td> <!---outer table--->
<table width="100%" cellspacing="0" cellpadding="0" border="0">
<tbody><tr>
<td colspan="2" width="30" bgcolor="FOF5FE" align="center"><img src="../images/page01.gif"></td>
<td bgcolor="FOF5FE"><b><br>&nbsp;&nbsp;&nbsp;&nbsp;
Fortran - Group and Communicator Example
</b><p></p></td>

</tr><tr valign="top"><td colspan="3"><p></p></td>

</tr><tr valign="top">
<td width="30"><pre><font color="#AAAAAA"> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>36<br>37<br>38<br>39<br>40<br>41<br>42</font></pre></td>

<td width="1" bgcolor="#7099cc"></td>

<td><pre>   program group
   include <font color="DF4442">'mpif.h'</font>

   integer NPROCS
   parameter(NPROCS=8)
   integer rank, new_rank, sendbuf, recvbuf, numtasks
   integer ranks1(4), ranks2(4), ierr
   integer <font color="DF4442">orig_group, new_group, new_comm</font>   <font color="#AAAAAA">! required variables</font>
   data ranks1 /0, 1, 2, 3/, ranks2 /4, 5, 6, 7/

   call <font color="DF4442">MPI_INIT</font>(ierr)
   call <font color="DF4442">MPI_COMM_RANK</font>(MPI_COMM_WORLD, rank, ierr)
   call <font color="DF4442">MPI_COMM_SIZE</font>(MPI_COMM_WORLD, numtasks, ierr)

   if (numtasks .ne. NPROCS) then
     print *, 'Must specify NPROCS= ',NPROCS,' Terminating.'
     call <font color="DF4442">MPI_FINALIZE</font>(ierr)
     stop
   endif

   sendbuf = rank

   <font color="#AAAAAA">! extract the original group handle</font>
   call <font color="DF4442">MPI_COMM_GROUP</font>(MPI_COMM_WORLD, orig_group, ierr)

   <font color="#AAAAAA">! divide tasks into two distinct groups based upon rank</font>
   if (rank .lt. NPROCS/2) then
      call <font color="DF4442">MPI_GROUP_INCL</font>(orig_group, NPROCS/2, ranks1, new_group, ierr)
   else 
      call <font color="DF4442">MPI_GROUP_INCL</font>(orig_group, NPROCS/2, ranks2, new_group, ierr)
   endif

   <font color="#AAAAAA">! create new new communicator and then perform collective communications</font>
   call <font color="DF4442">MPI_COMM_CREATE</font>(MPI_COMM_WORLD, new_group, new_comm, ierr)
   call <font color="DF4442">MPI_ALLREDUCE</font>(sendbuf, recvbuf, 1, MPI_INTEGER, MPI_SUM, new_comm, ierr)

   <font color="#AAAAAA">! get rank in new group</font>
   call <font color="DF4442">MPI_GROUP_RANK</font>(new_group, new_rank, ierr)
   print *, 'rank= ',rank,' newrank= ',new_rank,' recvbuf= ', recvbuf

   call <font color="DF4442">MPI_FINALIZE</font>(ierr)
   end

</pre></td>
</tr></tbody></table>
</td></tr></tbody></table>

 Sample program output: 

 <pre>rank= 7 newrank= 3 recvbuf= 22
rank= 0 newrank= 0 recvbuf= 6
rank= 1 newrank= 1 recvbuf= 6
rank= 2 newrank= 2 recvbuf= 6
rank= 6 newrank= 2 recvbuf= 22
rank= 3 newrank= 3 recvbuf= 6
rank= 4 newrank= 0 recvbuf= 22
rank= 5 newrank= 1 recvbuf= 22
</pre>

##  虚拟拓扑(Virtual Topologies)

###  What Are They? 

- In terms of MPI, a virtual topology describes a mapping/ordering of MPI processes into a geometric "shape". 
- MPI支持的两种主要拓扑类型是笛卡尔(网格)和图。
- MPI拓扑是虚拟的——并行机的物理结构和进程拓扑之间可能没有关系。
- 虚拟拓扑构建在MPI通信器和组之上。
- 必须由应用程序开发人员“编程”。

###  Why Use Them? 

- Convenience 
  - 虚拟拓扑可能对具有特定通信模式(与MPI拓扑结构匹配的模式)的应用程序有用。
  - 例如，对于需要对基于网格的数据进行4路最近邻通信的应用程序，笛卡尔拓扑可能很方便。
- Communication Efficiency 
  - 一些硬件架构可能会对相继相隔很远的“节点”之间的通信进行惩罚。
  - 特定的实现可以根据给定并行机的物理特性来优化进程映射。
  - 进程到MPI虚拟拓扑的映射依赖于MPI实现，可能完全被忽略。

### 虚拟拓扑的例程

 Create a 4 x 4 Cartesian topology from 16 processors and have each process exchange its rank with four neighbors. 

 <table width="90%" cellspacing="0" cellpadding="15" border="1"><tbody><tr><td> <!---outer table--->
<table width="100%" cellspacing="0" cellpadding="0" border="0">
<tbody><tr>
<td colspan="2" width="30" bgcolor="FOF5FE" align="center"><img src="../images/page01.gif"></td>
<td bgcolor="FOF5FE"><b><br>&nbsp;&nbsp;&nbsp;&nbsp;
C Language - Cartesian Virtual Topology Example
</b><p></p></td>

</tr><tr valign="top"><td colspan="3"><p></p></td>

</tr><tr valign="top">
<td width="30"><pre><font color="#AAAAAA"> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>36<br>37<br>38<br>39<br>40<br>41<br>42<br>43<br>44<br>45<br>46<br>47<br>48<br>49<br>50<br>51<br>52<br>53<br>54</font></pre></td>

<td width="1" bgcolor="#7099cc"></td>

<td><pre>   #include <font color="#DF4442">"mpi.h"</font>
   #include &lt;stdio.h&gt;
   #define SIZE 16
   #define UP    0
   #define DOWN  1
   #define LEFT  2
   #define RIGHT 3

   main(int argc, char *argv[])  {
   int numtasks, rank, source, dest, outbuf, i, tag=1, 
      inbuf[4]={MPI_PROC_NULL,MPI_PROC_NULL,MPI_PROC_NULL,MPI_PROC_NULL,}, 
      nbrs[4], dims[2]={4,4}, 
      periods[2]={0,0}, reorder=0, coords[2];

   <font color="#DF4442">MPI_Request reqs[8]</font>;
   <font color="#DF4442">MPI_Status stats[8]</font>;
   <font color="#DF4442">MPI_Comm cartcomm</font>;   <font color="#AAAAAA">// required variable</font>

   <font color="#DF4442">MPI_Init</font>(&amp;argc,&amp;argv);
   <font color="#DF4442">MPI_Comm_size</font>(MPI_COMM_WORLD, &amp;numtasks);

   if (numtasks == SIZE) {
      <font color="#AAAAAA">// create cartesian virtual topology, get rank, coordinates, neighbor ranks</font>
      <font color="#DF4442">MPI_Cart_create</font>(MPI_COMM_WORLD, 2, dims, periods, reorder, &amp;cartcomm);
      <font color="#DF4442">MPI_Comm_rank</font>(cartcomm, &amp;rank);
      <font color="#DF4442">MPI_Cart_coords</font>(cartcomm, rank, 2, coords);
      <font color="#DF4442">MPI_Cart_shift</font>(cartcomm, 0, 1, &amp;nbrs[UP], &amp;nbrs[DOWN]);
      <font color="#DF4442">MPI_Cart_shift</font>(cartcomm, 1, 1, &amp;nbrs[LEFT], &amp;nbrs[RIGHT]);

      printf("rank= %d coords= %d %d  neighbors(u,d,l,r)= %d %d %d %d\n",
             rank,coords[0],coords[1],nbrs[UP],nbrs[DOWN],nbrs[LEFT],
             nbrs[RIGHT]);

      outbuf = rank;

      <font color="#AAAAAA">// exchange data (rank) with 4 neighbors</font>
      for (i=0; i&lt;4; i++) {
         dest = nbrs[i];
         source = nbrs[i];
         <font color="#DF4442">MPI_Isend</font>(&amp;outbuf, 1, MPI_INT, dest, tag, 
                   MPI_COMM_WORLD, &amp;reqs[i]);
         <font color="#DF4442">MPI_Irecv</font>(&amp;inbuf[i], 1, MPI_INT, source, tag, 
                   MPI_COMM_WORLD, &amp;reqs[i+4]);
         }

      <font color="#DF4442">MPI_Waitall</font>(8, reqs, stats);
   
      printf("rank= %d                  inbuf(u,d,l,r)= %d %d %d %d\n",
             rank,inbuf[UP],inbuf[DOWN],inbuf[LEFT],inbuf[RIGHT]);  }
   else
      printf("Must specify %d processors. Terminating.\n",SIZE);
   
   <font color="#DF4442">MPI_Finalize</font>();
   }

</pre></td>
</tr></tbody></table>
</td></tr></tbody></table>

<table width="90%" cellspacing="0" cellpadding="15" border="1"><tbody><tr><td> <!---outer table--->
<table width="100%" cellspacing="0" cellpadding="0" border="0">
<tbody><tr>
<td colspan="2" width="30" bgcolor="FOF5FE" align="center"><img src="../images/page01.gif"></td>
<td bgcolor="FOF5FE"><b><br>&nbsp;&nbsp;&nbsp;&nbsp;
Fortran - Cartesian Virtual Topology Example
</b><p></p></td>

</tr><tr valign="top"><td colspan="3"><p></p></td>

</tr><tr valign="top">
<td width="30"><pre><font color="#AAAAAA"> 1<br> 2<br> 3<br> 4<br> 5<br> 6<br> 7<br> 8<br> 9<br>10<br>11<br>12<br>13<br>14<br>15<br>16<br>17<br>18<br>19<br>20<br>21<br>22<br>23<br>24<br>25<br>26<br>27<br>28<br>29<br>30<br>31<br>32<br>33<br>34<br>35<br>36<br>37<br>38<br>39<br>40<br>41<br>42<br>43<br>44<br>45<br>46<br>47<br>48<br>49<br>50<br>51<br>52<br>53<br>54<br>55<br>56<br>57<br>58</font></pre></td>

<td width="1" bgcolor="#7099cc"></td>

<td><pre>   program cartesian
   include <font color="#DF4442">'mpif.h'</font>

   integer SIZE, UP, DOWN, LEFT, RIGHT
   parameter(SIZE=16)
   parameter(UP=1)
   parameter(DOWN=2)
   parameter(LEFT=3)
   parameter(RIGHT=4)
   integer numtasks, rank, source, dest, outbuf, i, tag, ierr, &amp;
           inbuf(4), nbrs(4), dims(2), coords(2), periods(2), reorder
   integer <font color="#DF4442">stats(MPI_STATUS_SIZE, 8), reqs(8)</font>
   integer <font color="#DF4442">cartcomm</font>   <font color="#AAAAAA">! required variable</font>
   data inbuf /MPI_PROC_NULL,MPI_PROC_NULL,MPI_PROC_NULL,MPI_PROC_NULL/, &amp;
        dims /4,4/, tag /1/, periods /0,0/, reorder /0/ 

   call <font color="#DF4442">MPI_INIT</font>(ierr)
   call <font color="#DF4442">MPI_COMM_SIZE</font>(MPI_COMM_WORLD, numtasks, ierr)
  
   if (numtasks .eq. SIZE) then
      <font color="#AAAAAA">! create cartesian virtual topology, get rank, coordinates, neighbor ranks</font>
      call <font color="#DF4442">MPI_CART_CREATE</font>(MPI_COMM_WORLD, 2, dims, periods, reorder, &amp;
                           cartcomm, ierr)
      call <font color="#DF4442">MPI_COMM_RANK</font>(cartcomm, rank, ierr)
      call <font color="#DF4442">MPI_CART_COORDS</font>(cartcomm, rank, 2, coords, ierr)
      call <font color="#DF4442">MPI_CART_SHIFT</font>(cartcomm, 0, 1, nbrs(UP), nbrs(DOWN), ierr)
      call <font color="#DF4442">MPI_CART_SHIFT</font>(cartcomm, 1, 1, nbrs(LEFT), nbrs(RIGHT), ierr)

      write(*,20) rank,coords(1),coords(2),nbrs(UP),nbrs(DOWN), &amp;
                  nbrs(LEFT),nbrs(RIGHT)

      <font color="#AAAAAA">! exchange data (rank) with 4 neighbors</font>
      outbuf = rank
      do i=1,4
         dest = nbrs(i)
         source = nbrs(i)
         call <font color="#DF4442">MPI_ISEND</font>(outbuf, 1, MPI_INTEGER, dest, tag, &amp;
                       MPI_COMM_WORLD, reqs(i), ierr)
         call <font color="#DF4442">MPI_IRECV</font>(inbuf(i), 1, MPI_INTEGER, source, tag, &amp;
                       MPI_COMM_WORLD, reqs(i+4), ierr)
      enddo

      call <font color="#DF4442">MPI_WAITALL</font>(8, reqs, stats, ierr)

      write(*,30) rank,inbuf

   else
     print *, 'Must specify',SIZE,' processors.  Terminating.' 
   endif

   call <font color="#DF4442">MPI_FINALIZE</font>(ierr)

   20 format('rank= ',I3,' coords= ',I2,I2, &amp;
             ' neighbors(u,d,l,r)= ',I3,I3,I3,I3 )
   30 format('rank= ',I3,'                 ', &amp;
             ' inbuf(u,d,l,r)= ',I3,I3,I3,I3 )

   end

</pre></td>
</tr></tbody></table>
</td></tr></tbody></table>

 Sample program output: (partial) 

 <pre>rank=   0 coords=  0 0 neighbors(u,d,l,r)=  -1  4 -1  1
rank=   0                  inbuf(u,d,l,r)=  -1  4 -1  1
rank=   8 coords=  2 0 neighbors(u,d,l,r)=   4 12 -1  9
rank=   8                  inbuf(u,d,l,r)=   4 12 -1  9
rank=   1 coords=  0 1 neighbors(u,d,l,r)=  -1  5  0  2
rank=   1                  inbuf(u,d,l,r)=  -1  5  0  2
rank=  13 coords=  3 1 neighbors(u,d,l,r)=   9 -1 12 14
rank=  13                  inbuf(u,d,l,r)=   9 -1 12 14
...
...
rank=   3 coords=  0 3 neighbors(u,d,l,r)=  -1  7  2 -1
rank=   3                  inbuf(u,d,l,r)=  -1  7  2 -1
rank=  11 coords=  2 3 neighbors(u,d,l,r)=   7 15 10 -1
rank=  11                  inbuf(u,d,l,r)=   7 15 10 -1
rank=  10 coords=  2 2 neighbors(u,d,l,r)=   6 14  9 11
rank=  10                  inbuf(u,d,l,r)=   6 14  9 11
rank=   9 coords=  2 1 neighbors(u,d,l,r)=   5 13  8 10
rank=   9                  inbuf(u,d,l,r)=   5 13  8 10
</pre>

## 简要介绍一下MPI-2和MPI-3

###  MPI-2: 

- 有意地，MPI-1规范没有解决几个“困难的”问题。出于权宜之计，这些问题在1998年被推迟到第二个规范，称为MPI-2。
- MPI-2是对MPI-1的重大修订，增加了新的功能和修正。
- MPI-2新功能的关键领域
  - **动态进程**——删除MPI静态进程模型的扩展。提供在作业启动后创建新进程的例程。
  - **One-Sided Communications** - provides routines for one directional communications. Include shared memory operations (put/get) and remote accumulate operations. 
  - **扩展的集合操作**-允许应用集合操作的内部通信
  - **外部接口**——定义允许开发人员在MPI之上构建的例程，比如调试器和分析器。
  - **其他语言绑定**——描述c++绑定并讨论Fortran-90问题。
  - **Parallel I/O** - describes MPI support for parallel I/O. 

###  MPI-3: 

- MPI-3标准于2012年采用，包含了对MPI-1和MPI-2功能的重要扩展，包括
  - **非阻塞的集合操作**——允许集合中的任务在没有阻塞的情况下执行操作，可能提供性能改进。
  - **新的单边通信操作**-更好地处理不同的内存模型。
  - **邻域集合**——扩展了分布式图和笛卡尔进程拓扑，增加了通信能力。
  - **Fortran** 2008 Bindings - expanded from Fortran90 bindings 
  - **MPIT Tool Interface** - allows the MPI implementation to expose certain internal variables, counters, and other states to the user (most likely performance tools). 
  - **Matched Probe** - fixes an old bug in MPI-2 where one could not probe for messages in a multi-threaded environment. 

###  More Information on MPI-2 and MPI-3: 

- 
    MPI Standard documents: [http://www.mpi-forum.org/docs/](http://www.mpi-forum.org/docs/) 
