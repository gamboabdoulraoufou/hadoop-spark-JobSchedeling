### hadoop-spark-JobSchedeling  

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
./bin/pyspark --driver-cores 1 --driver-memory 2G --executor-cores 1 --executor-memory 2G

```

**_1-1-2 Application mode_**

```python
# -*- coding: utf-8 -*-
import time

from pyspark import SparkConf, SparkContext

appName = 'Wait 120 seconds'
Master = 'spark://spark-cluster-m:7077'
conf = SparkConf().setAppName(appName).setMaster(Master).set("spark.cores.max", "1")
conf.set("spark.driver.cores", "1").set("spark.driver.memory", "2g").set("spark.executor-cores", "1").set("spark.executor.memory", "2g")
sc = SparkContext(conf=conf)

# Run spark application during 60 seconds
time.sleep(120)

sc.stop()

```

**_1-1-3 Scheduling for both interactive and application mode_**  
Change spark configuration file `spark-defaults.conf`

```sh
cd /home/hadoop/spark-install/conf
sudo nano spark-defaults.conf
```

Change properties here
```sh
spark.driver.cores 1
spark.driver.memory 2g
spark.executor.cores 1
spark.executor.memory 2g
spark.cores.max 1
```


**_1-2 Dynamic Resource Allocation_**
application may give resources back to the cluster if they are no longer used and request them again later when there is demand. This feature is particularly useful if multiple applications share resources in your Spark cluster. If a subset of the resources allocated to an application becomes idle, it can be returned to the cluster’s pool of resources and acquired by other applications

```python
# -*- coding: utf-8 -*-
import time

from pyspark import SparkConf, SparkContext

appName = 'Dynamic allocation'
Master = 'spark://spark-cluster-m:7077'
conf = SparkConf().setAppName(appName).setMaster(Master)
conf.set("spark.shuffle.service.enabled", "true")
conf.set("spark.dynamicAllocation.enabled", "true")
conf.set("spark.dynamicAllocation.executorIdleTimeout", "60s")
conf.set("spark.dynamicAllocation.cachedExecutorIdleTimeout", "60s")
conf.set("spark.dynamicAllocation.initialExecutors", "2")
conf.set("spark.dynamicAllocation.maxExecutors", "2")
conf.set("spark.dynamicAllocation.minExecutors", "0")
conf.set("spark.dynamicAllocation.schedulerBacklogTimeout", "30s")

sc = SparkContext(conf=conf)

# Run spark application during 60 seconds
time.sleep(120)

sc.stop()

```

**_1-2-2 Scheduling for both interactive and application mode_**  
Change spark configuration file `spark-defaults.conf`

```sh
cd /home/hadoop/spark-install/conf
sudo nano spark-defaults.conf
```

Change properties here
spark.shuffle.service.enabled true 
spark.dynamicAllocation.enabled true 
spark.dynamicAllocation.executorIdleTimeout 30s
spark.dynamicAllocation.cachedExecutorIdleTimeout 30s 
spark.dynamicAllocation.initialExecutors 2 
spark.dynamicAllocation.maxExecutors 2 
spark.dynamicAllocation.minExecutors 0 
spark.dynamicAllocation.schedulerBacklogTimeout 30s

**_2- Schedeling within application_**  
Inside a given Spark application (SparkContext instance), multiple parallel jobs can run simultaneously if they were submitted from separate threads. By “job”, in this section, we mean a Spark action (e.g. save, collect) and any tasks that need to run to evaluate that action. Spark’s scheduler is fully thread-safe and supports this use case to enable applications that serve multiple requests (e.g. queries for multiple users).

By default, Spark’s scheduler runs jobs in FIFO fashion. Each job is divided into “stages” (e.g. map and reduce phases), and the first job gets priority on all available resources while its stages have tasks to launch, then the second job gets priority, etc. If the jobs at the head of the queue don’t need to use the whole cluster, later jobs can start to run right away, but if the jobs at the head of the queue are large, then later jobs may be delayed significantly.

**_2-1 FIFO schedeler_**  
conf.set("spark.scheduler.mode", "FIFO")

**_2-2 FAIR schedeler_**  
conf.set("spark.scheduler.mode", "FAIR")

**_2-2-1 FAIR schedeler_**  
**_2-2-2 FAIR pool schedeler_**  
sc.setLocalProperty("spark.scheduler.pool", "pool1")

Default Behavior of Pools
By default, each pool gets an equal share of the cluster (also equal in share to each job in the default pool), but inside each pool, jobs run in FIFO order. For example, if you create one pool per user, this means that each user will get an equal share of the cluster, and that each user’s queries will run in order instead of later queries taking resources from that user’s earlier ones.

Configuring Pool Properties
Specific pools’ properties can also be modified through a configuration file. Each pool supports three properties:

schedulingMode: This can be FIFO or FAIR, to control whether jobs within the pool queue up behind each other (the default) or share the pool’s resources fairly.
weight: This controls the pool’s share of the cluster relative to other pools. By default, all pools have a weight of 1. If you give a specific pool a weight of 2, for example, it will get 2x more resources as other active pools. Setting a high weight such as 1000 also makes it possible to implement priority between pools—in essence, the weight-1000 pool will always get to launch tasks first whenever it has jobs active.
minShare: Apart from an overall weight, each pool can be given a minimum shares (as a number of CPU cores) that the administrator would like it to have. The fair scheduler always attempts to meet all active pools’ minimum shares before redistributing extra resources according to the weights. The minShare property can therefore be another way to ensure that a pool can always get up to a certain number of resources (e.g. 10 cores) quickly without giving it a high priority for the rest of the cluster. By default, each pool’s minShare is 0.
The pool properties can be set by creating an XML file, similar to conf/fairscheduler.xml.template, and setting a spark.scheduler.allocation.file property in your SparkConf.

conf.set("spark.scheduler.allocation.file", "/path/to/file")

<?xml version="1.0"?>
<allocations>
  <pool name="production">
    <schedulingMode>FAIR</schedulingMode>
    <weight>1</weight>
    <minShare>2</minShare>
  </pool>
  <pool name="test">
    <schedulingMode>FIFO</schedulingMode>
    <weight>2</weight>
    <minShare>3</minShare>
  </pool>
</allocations>

A full example is also available in conf/fairscheduler.xml.template. Note that any pools not configured in the XML file will simply get default values for all settings (scheduling mode FIFO, weight 1, and minShare 0).


 
```sh
# Update package source
sudo apt-get update
```
