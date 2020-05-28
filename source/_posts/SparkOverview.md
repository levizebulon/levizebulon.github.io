---
title: SparkOverview
date: 2020-05-28 10:12:42
tags:
- software
- distribute system
---
# [Spark Overview](https://spark.apache.org/docs/latest/index.html#spark-overview)

Apache的Spark是一种快速和通用集群计算系统。它提供了在Java中，Scala中，Python的和R高层API，和优化引擎，支持一般的执行的图。它还支持一组丰富的更高级别的工具，包括SPARK SQL为SQL和结构化数据的处理，MLlib机器学习，GraphX用于图形处理和Spark Streaming。

# Security
Spark中的安全性在默认情况下是关闭的。这可能意味着您在默认情况下很容易受到攻击。请在下载并运行Spark之前查看Spark安全性。

# Downloading

# 运行示例和Shell

Spark附带了几个示例程序. Scala, Java, Python and R examples are in the examples`/src/main directory`. To run one of the Java or Scala sample programs, use `bin/run-example <class> [params]` in the top-level Spark directory. (在幕后，这将调用更通用的spark-submit脚本来启动应用程序). 例如,
<pre>
    <code>./bin/run-example SparkPi 10</code>
</pre>
您还可以通过修改过的Scala shell交互式地运行Spark。这是学习框架的好方法。
<pre><code>./bin/spark-shell --master local[2]
</code></pre>

`--master`选项指定一个分布式集群的master URL，或者`local`指定一个线程在local运行local URL，或者`loacl[N]`在本地运行N个线程。您应该首先使用`local`进行测试。要获得完整的选项列表，请使用`--help`选项运行Spark shell。

Spark提供了Python的API，使用`bin/pyspark`在Python解释器中交互式地运行Spark。
<pre><code>./bin/pyspark --master local[2]
</code></pre>
还用Python提供了示例应用程序。例如
<pre><code>./bin/spark-submit examples/src/main/python/pi.py 10
</code></pre>

<p>Spark also provides an experimental <a href="sparkr.html">R API</a> since 1.4 (only DataFrames APIs included).
To run Spark interactively in a R interpreter, use <code>bin/sparkR</code>:</p>

<pre><code>./bin/sparkR --master local[2]
</code></pre>

<p>Example applications are also provided in R. For example,</p>

<pre><code>./bin/spark-submit examples/src/main/r/dataframe.R
</code></pre>

# 在集群上启动
[Spark集群模式](https://spark.apache.org/docs/latest/cluster-overview.html)概述解释了在集群上运行的关键概念。Spark既可以自己运行，也可以在多个现有集群管理器上运行。它目前提供了几个部署选项
<ul>
  <li><a href="spark-standalone.html">Standalone Deploy Mode</a>: simplest way to deploy Spark on a private cluster</li>
  <li><a href="running-on-mesos.html">Apache Mesos</a></li>
  <li><a href="running-on-yarn.html">Hadoop YARN</a></li>
  <li><a href="running-on-kubernetes.html">Kubernetes</a></li>
</ul>

# Where to Go from Here

<p><strong>Programming Guides:</strong></p>
<ul>
  <li><a href="quick-start.html">Quick Start</a>: a quick introduction to the Spark API; start here!</li>
  <li><a href="rdd-programming-guide.html">RDD Programming Guide</a>: overview of Spark basics - RDDs (core but old API), accumulators, and broadcast variables</li>
  <li><a href="sql-programming-guide.html">Spark SQL, Datasets, and DataFrames</a>: processing structured data with relational queries (newer API than RDDs)</li>
  <li><a href="structured-streaming-programming-guide.html">Structured Streaming</a>: processing structured data streams with relation queries (using Datasets and DataFrames, newer API than DStreams)</li>
  <li><a href="streaming-programming-guide.html">Spark Streaming</a>: processing data streams using DStreams (old API)</li>
  <li><a href="ml-guide.html">MLlib</a>: applying machine learning algorithms</li>
  <li><a href="graphx-programming-guide.html">GraphX</a>: processing graphs</li>
</ul>

<p><strong>API Docs:</strong></p>

<ul>
  <li><a href="api/scala/index.html#org.apache.spark.package">Spark Scala API (Scaladoc)</a></li>
  <li><a href="api/java/index.html">Spark Java API (Javadoc)</a></li>
  <li><a href="api/python/index.html">Spark Python API (Sphinx)</a></li>
  <li><a href="api/R/index.html">Spark R API (Roxygen2)</a></li>
  <li><a href="api/sql/index.html">Spark SQL, Built-in Functions (MkDocs)</a></li>
</ul>

<p><strong>Deployment Guides:</strong></p>

<ul>
  <li><a href="cluster-overview.html">Cluster Overview</a>: overview of concepts and components when running on a cluster</li>
  <li><a href="submitting-applications.html">Submitting Applications</a>: packaging and deploying applications</li>
  <li>Deployment modes:
    <ul>
      <li><a href="https://github.com/amplab/spark-ec2">Amazon EC2</a>: scripts that let you launch a cluster on EC2 in about 5 minutes</li>
      <li><a href="spark-standalone.html">Standalone Deploy Mode</a>: launch a standalone cluster quickly without a third-party cluster manager</li>
      <li><a href="running-on-mesos.html">Mesos</a>: deploy a private cluster using
  <a href="https://mesos.apache.org">Apache Mesos</a></li>
      <li><a href="running-on-yarn.html">YARN</a>: deploy Spark on top of Hadoop NextGen (YARN)</li>
      <li><a href="running-on-kubernetes.html">Kubernetes</a>: deploy Spark on top of Kubernetes</li>
    </ul>
  </li>
</ul>

<p><strong>Other Documents:</strong></p>

<ul>
  <li><a href="configuration.html">Configuration</a>: customize Spark via its configuration system</li>
  <li><a href="monitoring.html">Monitoring</a>: track the behavior of your applications</li>
  <li><a href="tuning.html">Tuning Guide</a>: best practices to optimize performance and memory use</li>
  <li><a href="job-scheduling.html">Job Scheduling</a>: scheduling resources across and within Spark applications</li>
  <li><a href="security.html">Security</a>: Spark security support</li>
  <li><a href="hardware-provisioning.html">Hardware Provisioning</a>: recommendations for cluster hardware</li>
  <li>Integration with other storage systems:
    <ul>
      <li><a href="cloud-integration.html">Cloud Infrastructures</a></li>
      <li><a href="storage-openstack-swift.html">OpenStack Swift</a></li>
    </ul>
  </li>
  <li><a href="building-spark.html">Building Spark</a>: build Spark using the Maven system</li>
  <li><a href="https://spark.apache.org/contributing.html">Contributing to Spark</a></li>
  <li><a href="https://spark.apache.org/third-party-projects.html">Third Party Projects</a>: related third party Spark projects</li>
</ul>

<p><strong>External Resources:</strong></p>

<ul>
  <li><a href="https://spark.apache.org">Spark Homepage</a></li>
  <li><a href="https://spark.apache.org/community.html">Spark Community</a> resources, including local meetups</li>
  <li><a href="http://stackoverflow.com/questions/tagged/apache-spark">StackOverflow tag <code>apache-spark</code></a></li>
  <li><a href="https://spark.apache.org/mailing-lists.html">Mailing Lists</a>: ask questions about Spark here</li>
  <li><a href="http://ampcamp.berkeley.edu/">AMP Camps</a>: a series of training camps at UC Berkeley that featured talks and
exercises about Spark, Spark Streaming, Mesos, and more. <a href="http://ampcamp.berkeley.edu/6/">Videos</a>,
<a href="http://ampcamp.berkeley.edu/6/">slides</a> and <a href="http://ampcamp.berkeley.edu/6/exercises/">exercises</a> are
available online for free.</li>
  <li><a href="https://spark.apache.org/examples.html">Code Examples</a>: more are also available in the <code>examples</code> subfolder of Spark (<a href="https://github.com/apache/spark/tree/master/examples/src/main/scala/org/apache/spark/examples">Scala</a>,
 <a href="https://github.com/apache/spark/tree/master/examples/src/main/java/org/apache/spark/examples">Java</a>,
 <a href="https://github.com/apache/spark/tree/master/examples/src/main/python">Python</a>,
 <a href="https://github.com/apache/spark/tree/master/examples/src/main/r">R</a>)</li>
</ul>