---
title: "Set up an Apache Spark cluster and integrate with Jupyter Notebook"
date: 2019-07-01T12:00:00.000Z
lastmod: 2020-03-10T11:59:02.000Z
draft: false
description: "Step-by-step tutorial to set up an Apache Spark cluster and integrate it with Jupyter Notebook."
tags:
  - "Tutorials"
aliases:
  - /how-to-spark-cluster-jupyter/
featureimage: "feature.jpg"
---

## What is Spark?

Spark is a framework to make computations with large amounts of data. Why do you need something like Spark? Think for example about a small dataset that fit easily into memory, let’s say some Gb maximum. You will probably load the entire  dataframe using Pandas, R or your tool of choice and after some quick cleaning and visualization you will be almost done  with no major hassles related with computing performance if you are using a proper computer (or cloud infrastructure).

Now think that you have to process a 1Tb (or bigger) dataset and train a ML algorithm on it. Even with a powerful computer  it is crazy. Spark gives you two features you need to handle these data monsters:

-   **Parallel computing**: you use not one but many computers to speed your calculations.
-   **Fault tolerance**: you must be able to recover if one of your computers hangs in the middle of the process.

[How Spark works internally](https://jaceklaskowski.gitbooks.io/mastering-apache-spark/spark-overview.html) is out  of the scope of this tutorial and I will assume you are already familiar with that. Anyway you will need little  knowledge about Spark’s internals to set up and run you own cluster at home.

## What is a Spark cluster and what does ‘standalone’ mean?

### Spark clusters

A Spark cluster is just some computers running Spark and working together. A cluster consist on:

-   **Master**: is one of the computers that orchestrate how everything works. It distributes the work and take care of everything.
-   **Slaves**: these are the computers that get the job done. They process chunks of your massive datasets following the Map  Reduce paradigm. A computer can be master and slave at the same time.

### Standalone

It just mean that Spark is installed in every computer involved in the cluster. The cluster manager in use is provided  by Spark. There are other cluster managers like Apache Mesos and Hadoop YARN.

## Requirements

To follow this tutorial you need:

-   A couple of computers (minimum): this is a cluster.
-   Linux: it should also work for OSX, you have to be able to run shell scripts. I have not seen Spark running on native  windows so far.

For this tutorial I have used a MacBook Air with Ubuntu 17.04 and my desktop system with Windows 10 running  [Linux Subsystem for Windows](https://msdn.microsoft.com/es-es/commandline/wsl/install_guide) (yeah!) with Ubuntu 16.04 LTS.

If you don’t meet these simple requirements, please don’t panic, follow this steps and you are done:

-   Download [Oracle Virtualbox](https://www.virtualbox.org/).
-   Download [Linux](https://www.ubuntu.com/download/desktop).
-   [Create a Virtual Machine in Virtualbox and install Linux on it](http://www.psychocats.net/ubuntu/virtualbox).
-   [Clone that VM](https://protechgurus.com/clone-virtual-machine-virtualbox/) after following the installation tutorial steps.

And that’s all, you have 2 Linux machines to run your cluster.

## Tutorial

### Step 1: Install Java

**Note:** you will have to perform this step for all machines involved.

Spark needs Java to run. My recommendation is going with Open JDK8. Go to your Terminal and write the following commands:

```bash
$ sudo apt-get update
$ sudo apt-get upgrade
$ sudo apt-get install openjdk-8-jdk
```

Test your Java installation typing:

```bash
$ java -version
```

You should see the following output:

```text
openjdk version "1.8.0_131" OpenJDK Runtime Environment (build 1.8.0_131-8u131-b11-2ubuntu1.17.04.3-b11) OpenJDK 64-Bit Server VM (build 25.131-b11, mixed mode)
```

### Step 2: Install Spark

[Download](https://d3kbcqa49mib13.cloudfront.net/spark-2.2.0-bin-hadoop2.7.tgz) Spark. I have used 2.2.0 pre-built for  this tutorial. You can go to [Spark download page](https://spark.apache.org/downloads.html) and download it from there  or in case you don’t have access to a graphical desktop and have to use command line just run:

```bash
$ curl -O https://d3kbcqa49mib13.cloudfront.net/spark-2.2.0-bin-hadoop2.7.tgz
```

After that you will have the compressed source of Spark in your home directory. Extract the code and move the resulting folder to your home (recommended but not necessary) in case you downloaded it in Downloads folder.

Extract:

```bash
$ tar zxvf spark-2.2.0-bin-hadoop2.7.tgz
```

And move (optional):

```bash
$ mv spark-2.2.0-bin-hadoop2.7 ~
```

Finally you will set the needed environment variable `SPARK_HOME`.

Go to your home directory and open with a text editor the `.bashrc` file:

```bash
$ cd ~
$ sudo nano .bashrc
```

Add the following lines at the end of the file. It will set up the variable properly when your turn on your computer or start a session. Take into account that folder name will change in the future and will not match the one written here,  but the procedure is similar.

```bash
# Spark
export SPARK_HOME="/home/<your_username>/spark-2.2.0-bin-hadoop2.7/"
```

Save the file. If you are using nano just do `ctrl+x`, write `y` and press `return` to get  it done.

Now you can check your Spark installation. Go to Spark folder and execute pyspark:

```bash
$ cd spark-2.2.0-bin-hadoop2.7
$ bin/pyspark
```

If everything is properly installed you should see an output similar to this:

```text
Python 3.6.1 |Anaconda custom (64-bit)| (default, May 11 2017, 13:09:58) 
[GCC 4.4.7 20120313 (Red Hat 4.4.7-1)] on linux
Type "help", "copyright", "credits" or "license" for more information.
Using Spark's default log4j profile: org/apache/spark/log4j-defaults.properties
Setting default log level to "WARN".
To adjust logging level use sc.setLogLevel(newLevel). For SparkR, use setLogLevel(newLevel).
17/08/17 20:39:03 WARN Utils: Your hostname, david-MacBookAir resolves to a loopback address: 127.0.1.1; using 192.168.1.143 instead (on interface wlp2s0)
17/08/17 20:39:03 WARN Utils: Set SPARK_LOCAL_IP if you need to bind to another address
17/08/17 20:39:13 WARN ObjectStore: Version information not found in metastore. hive.metastore.schema.verification is not enabled so recording the schema version 1.2.0
17/08/17 20:39:13 WARN ObjectStore: Failed to get database default, returning NoSuchObjectException
17/08/17 20:39:14 WARN ObjectStore: Failed to get database global_temp, returning NoSuchObjectException
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   version 2.2.0
      /_/

Using Python version 3.6.1 (default, May 11 2017 13:09:58)
SparkSession available as 'spark'.
```

Up to this point, you may see this warning at Spark initialization:

```text
WARN NativeCodeLoader: Unable to load native-hadoop library for your platform... using builtin-java classes where applicable".
```

To solve this problem, we will have to install Hadoop. This is optional because Spark is going to run anyway, but I guess  there must be some performance improvements of using native Hadoop over some kind of adapters.

### Step 3 (Optional): Installing Hadoop

The process of installing Hadoop is pretty much the same as Spark, and I will go over it quickly. This is an optional step  and is not strictly necessary to run your cluster. You can also decide to do it later. The steps for installing Hadoop are:

-   Download Hadoop source [here](http://apache.rediris.es/hadoop/common/hadoop-2.8.0/hadoop-2.8.0.tar.gz). You are free  to install whatever version you like, but make sure you choose a version > 2.7 because it’s a Spark’s requirement. This   is pretty much the same as we did before with Spark. Feel free to use equivalent command line expressions.
-   Extract in your home directory.
-   Add the following lines at the end of your `.bashrc` file:

```bash
# Hadoop
export HADOOP_HOME="/home/<your_username>/hadoop-2.8.0"
export LD_LIBRARY_PATH=$HADOOP_HOME/lib/native:$LD_LIBRARY_PATH
```

-   Test your Hadoop installation by starting pyspark and make sure you don’t see the warning.

### Step 4: Launch the master server

Previous to launch the master server, you must check your ip to pass it as the host parameter. Type the following command:

```bash
$ ifconfig
```

You should see an output similar to this:

```text
lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        inet6 ::1  prefixlen 128  scopeid 0x10<host>
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 3735  bytes 7174785 (7.1 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 3735  bytes 7174785 (7.1 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

wlp2s0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.1.143  netmask xxx.xxx.xxx.x  broadcast xxx.xxx.x.xxx
        inet6 xxxx::xxxx:xxxx:xxxx:xxxx  prefixlen 64  scopeid 0x20<link>
        ether 14:10:9f:f3:a0:5e  txqueuelen 1000  (Ethernet)
        RX packets 67543  bytes 60399180 (60.3 MB)
        RX errors 0  dropped 0  overruns 0  frame 119733
        TX packets 52684  bytes 11712399 (11.7 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
        device interrupt 17  
```

Copy the `inet` value in the second text block. This is going to be `<your_master_ip>`. In this example you should copy `192.168.1.143`.

Now, go to your Spark installation directory and type:

```bash
$ ./sbin/start-master.sh -h <your_master_ip_>
```

You should see an output similar to:

```text
starting org.apache.spark.deploy.master.Master, logging to /home/<your_username>/spark-2.2.0-bin-hadoop2.7//logs/spark-<your_username>-org.apache.spark.deploy.master.Master-1-<your_username>-MacBookAir.out
```

Now you can go to `localhost:8080` in the same computer where you started the master server, or to `<your_master_ip>:8080` in  case you don’t have access to a web browser but have other devices connected to the same private network (e.g. a phone). In that URL you have access to the master server web user interface.

![Spark master server web UI](spark-master-web-ui.png)

In this web you can see just behind Spark logo an URL parameter similar to `spark://<your_master_ip>:7077`. This URL is  very important because is the one you are going to need when connecting slaves to your cluster and I will name it `<your_master_url>`.

### Step 5: Connect your slaves

Now you have a master server running, is time to start a couple of slave servers to get the job done. To start a slave  server you have to type the following command running from your Spark installation folder, using the URL you copied from  master server web interface:

```bash
$ ./sbin/start-slave.sh <your_master_url>
```

You should see an output very similar to the master’s one.

Perform this operation in every computer you want to connect to the cluster. In this tutorial I am going to run 2 slaves.

After that, if you go to the master server web interface again you should see several rows in the ‘Workers’ section where,  one per slave server you started. There is also useful info like worker’s memory, status, cores used and ip.

![Spark workers connected to the cluster](spark-workers-list.png)

You can also click the links in the Spark web app and go to worker’s page and tasks page. As you don’t have any running  app connected to the cluster, you can’t access running and completed tasks page.

So far you have a fully working Spark cluster running.

### Step 6: Integration with Jupyter

To get the most out of Spark is a good idea integrating with some interactive tool like Jupyter. If you already have  Jupyter installed and running, skip the following lines where I explain how to set up a local Jupyter server.

#### Installing Jupyter

The easiest way to install Jupyter is probably using [conda](https://conda.io/docs/intro.html)  (package, dependency and environment management). If you have [Anaconda Python distribution](https://www.continuum.io/downloads),  conda is already installed in your computer. If not, I strongly recommend you giving Anaconda a try, it really worth it.  In case you don’t want to install the full Anaconda Python (it includes a lot of libraries and needs about 350 Mb of disk)  you can opt for Miniconda, a lighter version which only includes Python and conda.

If you are using command line, just download the installation file (shell script) using curl and execute it with `'./'`

Once you have conda installed in your computer, let’s create a conda virtual environment called `jupyter` to avoid  messing the root one:

```bash
$ conda create -n jupyter
```

Now have a clean virtual environment to install Jupyter on it. To activate this environment type:

```bash
$ source activate jupyter
```

With the environment activated you should see `(jupyter)` at the beginning of your command prompt. Something like this:

```bash
(jupyter) david@david-MacBookAir:~$
```

It means that your environment is activated and all changes you do hereinafter (installing libraries, etc.) will be  applied to this conda virtual environment.

To install Jupyter, type the following command:

```bash
(jupyter) $ conda install notebook
```

This command will install Jupyter notebook and all its dependencies (IPython, Jupyter, etc.) so you don’t have to worry  about setting all these things up (thanks to conda package manager!).

Now it’s time to launch a Jupyter notebook and test your installation. Type:

```bash
(jupyter) $ jupyter notebook
```

If Jupyter is properly installed you should be able to go `localhost:8888/tree` URL in a web browser and see Jupyter folder tree.

![Jupyter Notebook folder tree](jupyter-folder-tree.png)

#### Installing findspark

findspark is a Python library that automatically allow you to import and use PySpark as any other Python library. There  are other options to make the integration (create a jupyter profile for Spark) but up to date findspark is imho the  faster and simpler one.

To install findspark run the following command:

```bash
(jupyter) $ pip install findspark
```

Now you have findpark installed in your jupyter virtual environment. Let’s create your first application.

#### Create your first Spark application

To create your first Spark app and start to make cool things with data, run the following script in a Jupyter cell:

```python
import findspark 
findspark.init() 

import pyspark 
sc = pyspark.SparkContext(master='<your_master_url>',
                          appName='<your_app_name>')
```

This script just initialize `findspark` so you can import `PySpark` as a library, and create an instance of Spark Context with  your master URL and app name (up to you) as parameters.

To test everything works well, you can display sc in your Jupyter notebook and should see an output like this:

![Spark Context displayed in Jupyter](spark-context-jupyter.png)

You can click the link and go to your app web ui, which is very interesting while you a running long tasks.

You should also connect to your master server web interface (`localhost:8080` or `<your_master_ip>:8080`) and see a new row  in Running Applications section, like this:

<!-- Image previously hosted at thegurus.tech — no longer available -->

To turn off your master and slaves servers, use the following commands from your Spark installation directory:

```bash
$ ./sbin/stop-master.sh
$ ./sbin/stop-slave.sh
```

To deactivate your jupyter conda virtual environment, just run the following command:

```bash
(jupyter) $ source deactivate jupyter
```

And that’s all!

There are many other things you can tweak and configure about Spark! If people is interested, I will create more content  about this amazing technology in the future as I am still learning new concepts everyday.

Hope you found this tutorial useful. If you see any error, bug, errand or whatever, just tell me or leave a comment and I  will try to fix it asap.