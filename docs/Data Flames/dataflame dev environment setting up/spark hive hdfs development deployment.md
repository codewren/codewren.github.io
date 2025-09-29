# Spark + Hive + HDFS Development Deployment

## Spark + Hive + HDFS Development Deployment

### Summary of Service <a href="#summary-of-service" id="summary-of-service"></a>

#### Spark <a href="#spark" id="spark"></a>

Spark 3.3.1

ref [https://spark.apache.org/docs/3.3.1/](https://spark.apache.org/docs/3.3.1/)

> Quote:
>
> Spark runs on Java 8/11/17, Scala 2.12/2.13, Python 3.7+ and R 3.5+. Java 8 prior to version 8u201 support is deprecated as of Spark 3.2.0. When using the Scala API, it is necessary for applications to use the same version of Scala that Spark was compiled for. For example, when using Scala 2.13, use Spark compiled for 2.13, and compile code/applications for Scala 2.13 as well.

#### Hadoop: HDFS, Yarn <a href="#hadoop-hdfs-yarn" id="hadoop-hdfs-yarn"></a>

version hadoop 3.2.4

ref [https://hadoop.apache.org/release/3.2.4.html](https://hadoop.apache.org/release/3.2.4.html)

#### Hive <a href="#hive" id="hive"></a>

hive 3.1.3.

recommendation: java 1.8

ref [https://medium.com/@marshallgo/how-hive-3-works-with-hadoop-3-d25c84bde37b](https://medium.com/@marshallgo/how-hive-3-works-with-hadoop-3-d25c84bde37b)

### JDK 1.8 <a href="#jdk-1.8" id="jdk-1.8"></a>

Binary release: [https://wiki.openjdk.org/display/jdk8u](https://wiki.openjdk.org/display/jdk8u)

> Quote:
>
> actually download website: github: adoptium/temurin8-binaries, eclipse
>
> [https://github.com/adoptium/temurin8-binaries/releases/tag/jdk8u392-b08](https://github.com/adoptium/temurin8-binaries/releases/tag/jdk8u392-b08)

### Installation of Service <a href="#installation-of-service" id="installation-of-service"></a>

#### Hadoop 3.2.4 Installation <a href="#hadoop-3.2.4-installation" id="hadoop-3.2.4-installation"></a>

single host, pseudo-distribution mode,

ref \[1] [https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html#Pseudo-Distributed\_Operation](https://hadoop.apache.org/docs/stable/hadoop-project-dist/hadoop-common/SingleCluster.html#Pseudo-Distributed_Operation)

ref \[2] [https://www.cloudduggu.com/hadoop/installation-single-node/](https://www.cloudduggu.com/hadoop/installation-single-node/)
