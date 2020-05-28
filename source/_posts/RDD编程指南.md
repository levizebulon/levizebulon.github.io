---
title: RDD编程指南
date: 2020-05-28 11:15:51
tags:
- programming
- distribute system
---

# 概述
在较高的级别上，每个Spark应用程序由一个驱动程序组成，该驱动程序运行用户的主要功能，并在集群上执行各种并行操作。Spark提供的主要抽象是一个弹性分布式数据集(RDD)，它是一个跨集群节点分区的元素集合，可以并行操作。RDDs的创建是从Hadoop文件系统(或任何其他Hadoop支持的文件系统)中的一个文件开始的，或者是从驱动程序中已有的Scala集合开始的，然后对其进行转换。用户也可以要求Spark将RDD持久化到内存中。

Spark中的第二个抽象是可以在并行操作中使用的共享变量。默认情况下，当Spark作为不同节点上的一组任务并行运行一个函数时，它会将函数中使用的每个变量的副本发送给每个任务。有时候，一个变量需要在任务之间共享，或者在任务和驱动程序之间共享。Spark支持两种类型的共享变量:广播变量(broadcast variable)和累加器(accumulator)，广播变量可用于在所有节点的内存中缓存一个值，累加器是只添加到其中的变量，比如计数器和和。

本指南用Spark支持的每种语言展示了这些特性。It is easiest to follow along with if you launch Spark’s interactive shell – either `bin/spark-shell` for the Scala shell or `bin/pyspark` for the Python one.

# Linking with Spark

Python
Spark 2.4.5适用于python2.7 +或python3.4 +。它可以使用标准的CPython解释器，因此可以使用像NumPy这样的C库。它也适用于PyPy 2.3+。
可以使用运行时包含Spark的`bin/spark-submit`脚本运行Python中的Spark应用程序，也可以将其包含在setup.py中，如下所示：
<pre><code class="language-python" data-lang="python"><span></span>    <span class="n">install_requires</span><span class="o">=</span><span class="p">[</span>
        <span class="s1">'pyspark=={site.SPARK_VERSION}'</span>
    <span class="p">]</span></code></pre>
要在Python中运行Spark应用程序而无需pip安装PySpark，请使用位于Spark目录中的`bin/spark-submit`脚本。 该脚本将加载Spark的Java/Scala库，并允许您将应用程序提交到集群。 您还可以使用`bin/pyspark`启动交互式Python Shell。

如果您希望访问HDFS数据，则需要使用PySpark构建来链接到您的HDFS版本。在Spark主页上还可以找到用于普通HDFS版本的[预构建包](https://spark.apache.org/downloads.html)。

最后，您需要将一些Spark类导入程序。添加以下行
<pre><code class="language-python" data-lang="python"><span></span><span class="kn">from</span> <span class="nn">pyspark</span> <span class="kn">import</span> <span class="n">SparkContext</span><span class="p">,</span> <span class="n">SparkConf</span></code></pre>

PySpark在驱动程序和工作程序中都需要相同的Python minor版本。它在PATH中使用默认的python版本，例如，您可以指定`PYSPARK_PYTHON`要使用的python版本

<pre><code class="language-bash" data-lang="bash"><span></span>$ <span class="nv">PYSPARK_PYTHON</span><span class="o">=</span>python3.4 bin/pyspark
$ <span class="nv">PYSPARK_PYTHON</span><span class="o">=</span>/opt/pypy-2.5/bin/pypy bin/spark-submit examples/src/main/python/pi.py</code></pre>

# Initializing Spark

## Python

Spark程序必须做的第一件事是创建一个[SparkContext](https://spark.apache.org/docs/latest/api/python/pyspark.html#pyspark.SparkContext)对象，它告诉Spark如何访问集群。要创建SparkContext，首先需要构建一个[SparkConf](https://spark.apache.org/docs/latest/api/python/pyspark.html#pyspark.SparkContext)对象，该对象包含关于应用程序的信息。

<pre><code class="language-python" data-lang="python"><span></span><span class="n">conf</span> <span class="o">=</span> <span class="n">SparkConf</span><span class="p">()</span><span class="o">.</span><span class="n">setAppName</span><span class="p">(</span><span class="n">appName</span><span class="p">)</span><span class="o">.</span><span class="n">setMaster</span><span class="p">(</span><span class="n">master</span><span class="p">)</span>
<span class="n">sc</span> <span class="o">=</span> <span class="n">SparkContext</span><span class="p">(</span><span class="n">conf</span><span class="o">=</span><span class="n">conf</span><span class="p">)</span></code></pre>

`appName`参数是应用程序在集群UI上显示的名称。`master`是一个[Spark、Mesos或YARN集群的URL](https://spark.apache.org/docs/latest/submitting-applications.html#master-urls)，或者在本地模式下运行的特殊本地字符串。实际上，在集群上运行时，您不希望在程序中硬编码master，而是[使用spark-submit启动应用程序](https://spark.apache.org/docs/latest/submitting-applications.html)并在那里接收它。但是，对于本地测试和单元测试，可以通过`local`来运行Spark in-process。

### Using the Shell

在PySpark shell中，已经在名为sc的变量中为您创建了一个特殊的可识别解释程序的SparkContext。 制作自己的SparkContext将不起作用。 您可以使用`--master`参数设置上下文连接的主机，也可以通过将逗号分隔的列表传递给`--py-files`，将Python .zip，.egg或.py文件添加到运行时路径。 您还可以通过在`--packages`参数中提供逗号分隔的Maven坐标列表，从而将依赖项（例如Spark Packages）添加到Shell会话中。 可以存在依赖项的任何其他存储库（例如Sonatype）都可以传递给`--repositories`参数。 必要时，必须使用pip手动安装Spark软件包具有的所有Python依赖项（在该软件包的requirements.txt中列出）。 例如，要在正好四个内核上运行`bin/pyspark`，请使用：
<pre><code class="language-bash" data-lang="bash"><span></span>$ ./bin/pyspark --master local<span class="o">[</span><span class="m">4</span><span class="o">]</span></code></pre>

或者，也可以将code.py添加到搜索路径(以便以后能够导入代码)，使用

<code class="language-bash" data-lang="bash"><span></span>$ ./bin/pyspark --master local<span class="o">[</span><span class="m">4</span><span class="o">]</span> --py-files code.py</code>

要获得完整的选项列表，请运行`pyspark --help`。在幕后，pyspark调用更通用的[spark-submit脚本](https://spark.apache.org/docs/latest/submitting-applications.html)。

也可以在增强的Python解释器IPython中启动PySpark Shell。 PySpark可与IPython 1.0.0及更高版本一起使用。 要使用IPython，请在运行`bin / pyspark`时将`PYSPARK_DRIVER_PYTHON`变量设置为ipython：

<pre><code class="language-bash" data-lang="bash"><span></span>$ <span class="nv">PYSPARK_DRIVER_PYTHON</span><span class="o">=</span>ipython ./bin/pyspark</code></pre>

使用jupyter notebook(以前称为IPython notebook)

<code class="language-bash" data-lang="bash"><span></span>$ <span class="nv">PYSPARK_DRIVER_PYTHON</span><span class="o">=</span>jupyter <span class="nv">PYSPARK_DRIVER_PYTHON_OPTS</span><span class="o">=</span>notebook ./bin/pyspark</code>

您可以通过设置`PYSPARK_DRIVER_PYTHON_OPTS`来定制ipython或jupyter命令。

启动Jupyter Notebook服务器后，您可以从“文件”选项卡创建一个新的“ Python 2”笔记本。 在笔记本内部，您可以在笔记本中开始内联输入命令`％pylab inline`作为笔记本的一部分，然后再从Jupyter笔记本开始尝试Spark。

# 弹性分布式数据集Resilient Distributed Datasets (RDDs)

Spark围绕弹性分布式数据集(RDD)的概念，它是一组可以并行操作的元素的容错集合。创建RDDs有两种方法:并行化驱动程序中的现有集合，或者引用外部存储系统中的数据集，例如共享文件系统、HDFS、HBase或任何提供Hadoop InputFormat的数据源。

# 并行集合Parallelized Collections

## Python
通过在驱动程序中现有的iterable或集合上调用`SparkContext's parallelize`方法来创建并行集合。将集合的元素复制以形成可并行操作的分布式数据集。例如，下面是如何创建一个包含数字1到5的并行集合

<pre><code class="language-python" data-lang="python"><span></span><span class="n">data</span> <span class="o">=</span> <span class="p">[</span><span class="mi">1</span><span class="p">,</span> <span class="mi">2</span><span class="p">,</span> <span class="mi">3</span><span class="p">,</span> <span class="mi">4</span><span class="p">,</span> <span class="mi">5</span><span class="p">]</span>
<span class="n">distData</span> <span class="o">=</span> <span class="n">sc</span><span class="o">.</span><span class="n">parallelize</span><span class="p">(</span><span class="n">data</span><span class="p">)</span></code></pre>

一旦创建，分布式数据集(distData)就可以并行操作。例如，我们可以调用distData。减少(lambda a, b: a + b)将列表中的元素相加。稍后我们将描述对分布式数据集的操作。

并行集合的一个重要参数是要将数据集分割成多少个分区。Spark将为集群的每个分区运行一个任务。通常，集群中的每个CPU需要2-4个分区。通常，Spark尝试根据您的集群自动设置分区的数量。但是，您也可以通过将其作为第二个参数传递来手动设置它`parallelize`(例如，`sc.parallelize(data, 10))`。注意:代码中的某些地方使用术语片(分区的同义词)来保持向后兼容性。

# 外部数据集External Datasets

PySpark可以从Hadoop支持的任何存储源创建分布式数据集，包括本地文件系统、HDFS、Cassandra、HBase、Amazon S3等。Spark支持文本文件、[SequenceFiles](http://hadoop.apache.org/common/docs/current/api/org/apache/hadoop/mapred/SequenceFileInputFormat.html)和任何其他[Hadoop InputFormat](http://hadoop.apache.org/docs/stable/api/org/apache/hadoop/mapred/InputFormat.html)。

可以使用`SparkContext's textFile`方法创建文本文件RDD。 此方法获取文件的URI（计算机上的本地路径，或`hdfs：//，s3a：//`等URI），并将其读取为行的集合。 这是一个示例调用：

<pre><code class="language-python" data-lang="python"><span></span><span class="o">&gt;&gt;&gt;</span> <span class="n">distFile</span> <span class="o">=</span> <span class="n">sc</span><span class="o">.</span><span class="n">textFile</span><span class="p">(</span><span class="s2">"data.txt"</span><span class="p">)</span></code></pre>

创建distFile之后，就可以通过数据集操作对其进行操作。例如，我们可以使用映射将所有行的大小相加并按如下方式归约操作:`distFile.map(lambda s: len(s)).reduce(lambda a, b: a + b)`。

一些关于使用Spark读取文件的注意事项

- 如果使用本地文件系统上的路径，则该文件也必须在工作节点上的相同路径上可访问。将文件复制到所有worker，或者使用挂载在网络上的共享文件系统。
- 所有基于Spark文件的输入方法，包括`textFile`，都支持在目录、压缩文件和通配符上运行。例如，你可以使用`textFile("/my/directory"), textFile("/my/directory/*.txt"), and textFile("/my/directory/*.gz")`。
- `textFile`方法还接受一个可选的第二个参数，用于控制文件的分区数量。默认情况下，Spark为文件的每个块创建一个分区(在HDFS中，块的默认大小为128MB)，但是您也可以通过传递更大的值来请求更多的分区。注意，分区不能少于块。

除了文本文件之外，Spark's Python API还支持其他几种数据格式

- `SparkContext.wholeTextFiles`允许您读取包含多个小文本文件的目录，并以(文件名、内容)对的形式返回每个文件。这与textFile相反，textFile在每个文件中每行返回一条记录。
- `RDD.saveAsPickleFile` 和`SparkContext.pickleFile`支持以包含pickle的Python对象的简单格式保存RDD。批处理用于pickle序列化，默认批处理大小为10。
- SequenceFile and Hadoop Input/Output Formats

<p><strong>Note</strong> this feature is currently marked <code>Experimental</code> and is intended for advanced users. It may be replaced in future with read/write support based on Spark SQL, in which case Spark SQL is the preferred approach.</p>

**Writable Support**

PySpark SequenceFile支持在Java中加载一个键-值对的RDD，将可写对象转换为基本Java类型，并使用[Pyrolite](https://github.com/irmen/Pyrolite/) pickle生成的Java对象。当将键值对的RDD保存到SequenceFile时，PySpark执行相反的操作。它将Python对象解pickle为Java对象，然后将它们转换为可写对象。下面的可写内容会自动转换

<table class="table">
<tbody><tr><th>Writable Type</th><th>Python Type</th></tr>
<tr><td>Text</td><td>unicode str</td></tr>
<tr><td>IntWritable</td><td>int</td></tr>
<tr><td>FloatWritable</td><td>float</td></tr>
<tr><td>DoubleWritable</td><td>float</td></tr>
<tr><td>BooleanWritable</td><td>bool</td></tr>
<tr><td>BytesWritable</td><td>bytearray</td></tr>
<tr><td>NullWritable</td><td>None</td></tr>
<tr><td>MapWritable</td><td>dict</td></tr>
</tbody></table>

数组不是开箱即用的。在读取或写入时，用户需要指定自定义ArrayWritable子类型。在编写时，用户还需要指定将数组转换为自定义ArrayWritable子类型的自定义转换器。读取时，默认转换器将自定义ArrayWritable子类型转换为Java Object[]，然后将其pickle为Python元组。获取Python数组。对于基元类型的数组，用户需要指定自定义转换器。


