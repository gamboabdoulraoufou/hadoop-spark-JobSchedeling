**_hadoop-spark-JobSchedeling_**

When running on a cluster, each Spark application (instance of SparkContext) gets an independent set of executor JVMs (with CPU and RAM) that only run tasks and store data for that application. Spark provides several facilities for scheduling resources between computations:
 - multiple applications / users can use your cluster simultaniously  
 - within each Spark application, multiple “jobs” (Spark actions) may be running concurrently  

**_1- Schedeling accros application_**  
There are also different options to manage allocation accros application, depending on the cluster manager (YARN, Standalone and Mesos):
- static partitioning of resources
- dynamic ressources allocation

In this post, I will show the different way to allocate ressources on Standalone mode.

**_1-1 static partitioning of resources_**
**_1-1-1 Interactive mode_**
```sh
# go to spark home
cd spark-install
./bin/pyspark --num-executors 1 --executor-memory 2G --total-executor-cores 2
```

**_1-1-2 Application mode_**
Add these line on top of your .py file

```python
# -*- coding: utf-8 -*-
import csv

from pyspark import SparkConf, SparkContext


appName = 'Spark demo App'
master = 'spark://spark-cluster-m:7077'
conf = SparkConf().setAppName(appName).setMaster(master)
sc = SparkContext(conf=conf)

```

**_1-2 Schedeling within an application_**

**_2- Schedeling within application_**
**_2-1 FIFO schedeler_**
**_2-2 FAIR schedeler_**
**_2-2-1 FAIR schedeler_**
**_2-2-2 FAIR pool schedeler_**


 
```sh
# Update package source
sudo apt-get update
```
