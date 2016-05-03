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
**_2-2 FAIR schedeler_**  
**_2-2-1 FAIR schedeler_**  
**_2-2-2 FAIR pool schedeler_**  


 
```sh
# Update package source
sudo apt-get update
```
