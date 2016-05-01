# hadoop-spark-JobSchedeling

When running on a cluster, each Spark application (instance of SparkContext) gets an independent set of executor JVMs (with CPU and RAM) that only run tasks and store data for that application. Spark provides several facilities for scheduling resources between computations:
- multiple applications / users can use your cluster simultaniously  
- within each Spark application, multiple “jobs” (Spark actions) may be running concurrently  

### 1- Schedeling accros application
There are also different options to manage allocation accros application, depending on the cluster manager (YARN, Standalone and Mesos):
- static partitioning of resources
- dynamic ressources allocation

In this post, I will show the different way to allocate ressources on Standalone mode.

### 1-1 static partitioning of resources
** Interactive mode**
```sh
# go to spark home
cd spark-install
./bin/pyspark --num-executors 1 --executor-memory 2G --total-executor-cores 2
```

** Application mode **
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

### 1-2 Schedeling within an application

### 2- Schedeling within application
### 2-1 FIFO schedeler
### 2-2 FAIR schedeler
### 2-2-1 FAIR schedeler
### 2-2-2 FAIR pool schedeler


 
```sh
# Update package source
sudo apt-get update
```
