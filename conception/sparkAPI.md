# Overview

At a high level, every Spark application consists of a *driver program* that runs the user’s `main` function and executes various *parallel operations* on a cluster. The main abstraction Spark provides is a *resilient distributed dataset* (RDD), which is a collection of elements partitioned across the nodes of the cluster that can be operated on in parallel. RDDs are created by starting with a file in the Hadoop file system (or any other Hadoop-supported file system), or an existing Scala collection in the driver program, and transforming it. Users may also ask Spark to *persist* an RDD in memory, allowing it to be reused efficiently across parallel operations. Finally, RDDs automatically recover from node failures.

A second abstraction in Spark is *shared variables* that can be used in parallel operations. By default, when Spark runs a function in parallel as a set of tasks on different nodes, it ships a copy of each variable used in the function to each task. Sometimes, a variable needs to be shared across tasks, or between tasks and the driver program. Spark supports two types of shared variables: *broadcast variables*, which can be used to cache a value in memory on all nodes, and *accumulators*, which are variables that are only “added” to, such as counters and sums.

This guide shows each of these features in each of Spark’s supported languages. It is easiest to follow along with if you launch Spark’s interactive shell – either `bin/spark-shell` for the Scala shell or `bin/pyspark` for the Python one.

# Linking with Spark

- [**Scala**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_scala_0)
- [**Java**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_java_0)
- [**Python**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_python_0)





```

```

``

```

```



```

```

``

# Initializing Spark

- [**Scala**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_scala_1)
- [**Java**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_java_1)
- [**Python**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_python_1)

``

``

```

```

The `appName` parameter is a name for your application to show on the cluster UI. `master` is a [Spark, Mesos or YARN cluster URL](https://spark.apache.org/docs/latest/submitting-applications.html#master-urls), or a special “local” string to run in local mode. In practice, when running on a cluster, you will not want to hardcode `master` in the program, but rather [launch the application with `spark-submit`](https://spark.apache.org/docs/latest/submitting-applications.html) and receive it there. However, for local testing and unit tests, you can pass “local” to run Spark in-process.

## Using the Shell

- [**Scala**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_scala_2)
- [**Python**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_python_2)

\````````````

```

```

``

```

```



```

```

\````[``](https://spark.apache.org/docs/latest/submitting-applications.html)

# Resilient Distributed Datasets (RDDs)

Spark revolves around the concept of a *resilient distributed dataset* (RDD), which is a fault-tolerant collection of elements that can be operated on in parallel. There are two ways to create RDDs: *parallelizing* an existing collection in your driver program, or referencing a dataset in an external storage system, such as a shared filesystem, HDFS, HBase, or any data source offering a Hadoop InputFormat.

## Parallelized Collections

- [**Scala**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_scala_3)
- [**Java**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_java_3)
- [**Python**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_python_3)

\``````

```

```

\````

One important parameter for parallel collections is the number of *partitions* to cut the dataset into. Spark will run one task for each partition of the cluster. Typically you want 2-4 partitions for each CPU in your cluster. Normally, Spark tries to set the number of partitions automatically based on your cluster. However, you can also set it manually by passing it as a second parameter to `parallelize` (e.g. `sc.parallelize(data, 10)`). Note: some places in the code use the term slices (a synonym for partitions) to maintain backward compatibility.

## External Datasets

- [**Scala**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_scala_4)
- [**Java**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_java_4)
- [**Python**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_python_4)



\````````

```

```

\````````



- 
- \````````
- ``



- \``````
- \````````
- \````````
- \````

## RDD Operations

RDDs support two types of operations: *transformations*, which create a new dataset from an existing one, and *actions*, which return a value to the driver program after running a computation on the dataset. For example, `map` is a transformation that passes each dataset element through a function and returns a new RDD representing the results. On the other hand, `reduce` is an action that aggregates all the elements of the RDD using some function and returns the final result to the driver program (although there is also a parallel `reduceByKey` that returns a distributed dataset).

All transformations in Spark are *lazy*, in that they do not compute their results right away. Instead, they just remember the transformations applied to some base dataset (e.g. a file). The transformations are only computed when an action requires a result to be returned to the driver program. This design enables Spark to run more efficiently. For example, we can realize that a dataset created through `map` will be used in a `reduce` and return only the result of the `reduce` to the driver, rather than the larger mapped dataset.

By default, each transformed RDD may be recomputed each time you run an action on it. However, you may also *persist* an RDD in memory using the `persist` (or `cache`) method, in which case Spark will keep the elements around on the cluster for much faster access the next time you query it. There is also support for persisting RDDs on disk, or replicated across multiple nodes.

### Basics

- [**Scala**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_scala_5)
- [**Java**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_java_5)
- [**Python**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_python_5)



```

```

\``````````

``

```

```

\````

### Passing Functions to Spark

- [**Scala**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_scala_6)
- [**Java**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_java_6)
- [**Python**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_python_6)



- 
- \````

```

```



```

```

\``````````



```

```

\``````

```

```

### Understanding closures 

One of the harder things about Spark is understanding the scope and life cycle of variables and methods when executing code across a cluster. RDD operations that modify variables outside of their scope can be a frequent source of confusion. In the example below we’ll look at code that uses `foreach()` to increment a counter, but similar issues can occur for other operations as well.

#### Example

Consider the naive RDD element sum below, which may behave differently depending on whether execution is happening within the same JVM. A common example of this is when running Spark in `local` mode (`--master = local[n]`) versus deploying a Spark application to a cluster (e.g. via spark-submit to YARN):

- [**Scala**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_scala_7)
- [**Java**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_java_7)
- [**Python**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_python_7)

```

```

#### Local vs. cluster modes

The behavior of the above code is undefined, and may not work as intended. To execute jobs, Spark breaks up the processing of RDD operations into tasks, each of which is executed by an executor. Prior to execution, Spark computes the task’s **closure**. The closure is those variables and methods which must be visible for the executor to perform its computations on the RDD (in this case `foreach()`). This closure is serialized and sent to each executor.

The variables within the closure sent to each executor are now copies and thus, when **counter** is referenced within the `foreach` function, it’s no longer the **counter** on the driver node. There is still a **counter** in the memory of the driver node but this is no longer visible to the executors! The executors only see the copy from the serialized closure. Thus, the final value of **counter** will still be zero since all operations on **counter** were referencing the value within the serialized closure.

In local mode, in some circumstances, the `foreach` function will actually execute within the same JVM as the driver and will reference the same original **counter**, and may actually update it.

To ensure well-defined behavior in these sorts of scenarios one should use an [`Accumulator`](https://spark.apache.org/docs/latest/rdd-programming-guide.html#accumulators). Accumulators in Spark are used specifically to provide a mechanism for safely updating a variable when execution is split up across worker nodes in a cluster. The Accumulators section of this guide discusses these in more detail.

In general, closures - constructs like loops or locally defined methods, should not be used to mutate some global state. Spark does not define or guarantee the behavior of mutations to objects referenced from outside of closures. Some code that does this may work in local mode, but that’s just by accident and such code will not behave as expected in distributed mode. Use an Accumulator instead if some global aggregation is needed.

#### Printing elements of an RDD

#### 打印RDD的元素

Another common idiom is attempting to print out the elements of an RDD using `rdd.foreach(println)` or `rdd.map(println)`. On a single machine, this will generate the expected output and print all the RDD’s elements. However, in `cluster` mode, the output to `stdout` being called by the executors is now writing to the executor’s `stdout` instead, not the one on the driver, so `stdout` on the driver won’t show these! To print all elements on the driver, one can use the `collect()` method to first bring the RDD to the driver node thus: `rdd.collect().foreach(println)`. This can cause the driver to run out of memory, though, because `collect()` fetches the entire RDD to a single machine; if you only need to print a few elements of the RDD, a safer approach is to use the `take()`: `rdd.take(100).foreach(println)`.

另一个常见的习惯用法是尝试使用rdd.foreach（println）或rdd.map（println）打印出RDD的元素。 在单台机器上，这将生成预期的输出并打印所有RDD的元素。 但是，在集群模式下，执行者正在调用“ stdout”的输出现在正在写入执行者的“ stdout”，而不是driver上的那个，因此driver上的“ stdout”不会显示这些信息！ 要在驱动程序上打印所有元素，可以使用`collect（）`方法首先将RDD带到driver节点：`rdd.collect（）`。然后`foreach（println）`。 但是，这可能导致driver用尽内存，因为`collect（）`将整个RDD提取到一台机器上。 如果只需要打印RDD的几个元素，一种更安全的方法是使用`take（）`：`rdd.take（100）.foreach（println）`。

### Working with Key-Value Pairs

- [**Scala**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_scala_8)
- [**Java**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_java_8)
- [**Python**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_python_8)



``

``

```

```

\````

\````

### Transformations

The following table lists some of the common transformations supported by Spark. Refer to the RDD API doc ([Scala](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.rdd.RDD), [Java](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/api/java/JavaRDD.html), [Python](https://spark.apache.org/docs/latest/api/python/pyspark.html#pyspark.RDD), [R](https://spark.apache.org/docs/latest/api/R/index.html)) and pair RDD functions doc ([Scala](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.rdd.PairRDDFunctions), [Java](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/api/java/JavaPairRDD.html)) for details.

| Transformation                                               | Meaning                                                      |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| **map**(*func*)                                              | Return a new distributed dataset formed by passing each element of the source through a function *func*. |
| **filter**(*func*)                                           | Return a new dataset formed by selecting those elements of the source on which *func* returns true. |
| **flatMap**(*func*)                                          | Similar to map, but each input item can be mapped to 0 or more output items (so *func* should return a Seq rather than a single item). |
| **mapPartitions**(*func*)                                    | Similar to map, but runs separately on each partition (block) of the RDD, so *func* must be of type Iterator<T> => Iterator<U> when running on an RDD of type T. |
| **mapPartitionsWithIndex**(*func*)                           | Similar to mapPartitions, but also provides *func* with an integer value representing the index of the partition, so *func* must be of type (Int, Iterator<T>) => Iterator<U> when running on an RDD of type T. |
| **sample**(*withReplacement*, *fraction*, *seed*)            | Sample a fraction *fraction* of the data, with or without replacement, using a given random number generator seed. |
| **union**(*otherDataset*)                                    | Return a new dataset that contains the union of the elements in the source dataset and the argument. |
| **intersection**(*otherDataset*)                             | Return a new RDD that contains the intersection of elements in the source dataset and the argument. |
| **distinct**([*numPartitions*]))                             | Return a new dataset that contains the distinct elements of the source dataset. |
| **groupByKey**([*numPartitions*])                            | When called on a dataset of (K, V) pairs, returns a dataset of (K, Iterable<V>) pairs. **Note:** If you are grouping in order to perform an aggregation (such as a sum or average) over each key, using `reduceByKey` or `aggregateByKey` will yield much better performance. **Note:** By default, the level of parallelism in the output depends on the number of partitions of the parent RDD. You can pass an optional `numPartitions` argument to set a different number of tasks. |
| **reduceByKey**(*func*, [*numPartitions*])                   | When called on a dataset of (K, V) pairs, returns a dataset of (K, V) pairs where the values for each key are aggregated using the given reduce function *func*, which must be of type (V,V) => V. Like in `groupByKey`, the number of reduce tasks is configurable through an optional second argument. |
| **aggregateByKey**(*zeroValue*)(*seqOp*, *combOp*, [*numPartitions*]) | When called on a dataset of (K, V) pairs, returns a dataset of (K, U) pairs where the values for each key are aggregated using the given combine functions and a neutral "zero" value. Allows an aggregated value type that is different than the input value type, while avoiding unnecessary allocations. Like in `groupByKey`, the number of reduce tasks is configurable through an optional second argument. |
| **sortByKey**([*ascending*], [*numPartitions*])              | When called on a dataset of (K, V) pairs where K implements Ordered, returns a dataset of (K, V) pairs sorted by keys in ascending or descending order, as specified in the boolean `ascending` argument. |
| **join**(*otherDataset*, [*numPartitions*])                  | When called on datasets of type (K, V) and (K, W), returns a dataset of (K, (V, W)) pairs with all pairs of elements for each key. Outer joins are supported through `leftOuterJoin`, `rightOuterJoin`, and `fullOuterJoin`. |
| **cogroup**(*otherDataset*, [*numPartitions*])               | When called on datasets of type (K, V) and (K, W), returns a dataset of (K, (Iterable<V>, Iterable<W>)) tuples. This operation is also called `groupWith`. |
| **cartesian**(*otherDataset*)                                | When called on datasets of types T and U, returns a dataset of (T, U) pairs (all pairs of elements). |
| **pipe**(*command*, *[envVars]*)                             | Pipe each partition of the RDD through a shell command, e.g. a Perl or bash script. RDD elements are written to the process's stdin and lines output to its stdout are returned as an RDD of strings. |
| **coalesce**(*numPartitions*)                                | Decrease the number of partitions in the RDD to numPartitions. Useful for running operations more efficiently after filtering down a large dataset. |
| **repartition**(*numPartitions*)                             | Reshuffle the data in the RDD randomly to create either more or fewer partitions and balance it across them. This always shuffles all data over the network. |
| **repartitionAndSortWithinPartitions**(*partitioner*)        | Repartition the RDD according to the given partitioner and, within each resulting partition, sort records by their keys. This is more efficient than calling `repartition` and then sorting within each partition because it can push the sorting down into the shuffle machinery. |

### Actions

The following table lists some of the common actions supported by Spark. Refer to the RDD API doc ([Scala](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.rdd.RDD), [Java](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/api/java/JavaRDD.html), [Python](https://spark.apache.org/docs/latest/api/python/pyspark.html#pyspark.RDD), [R](https://spark.apache.org/docs/latest/api/R/index.html))

and pair RDD functions doc ([Scala](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.rdd.PairRDDFunctions), [Java](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/api/java/JavaPairRDD.html)) for details.

| Action                                             | Meaning                                                      |
| :------------------------------------------------- | :----------------------------------------------------------- |
| **reduce**(*func*)                                 | Aggregate the elements of the dataset using a function *func* (which takes two arguments and returns one). The function should be commutative and associative so that it can be computed correctly in parallel. |
| **collect**()                                      | Return all the elements of the dataset as an array at the driver program. This is usually useful after a filter or other operation that returns a sufficiently small subset of the data. |
| **count**()                                        | Return the number of elements in the dataset.                |
| **first**()                                        | Return the first element of the dataset (similar to take(1)). |
| **take**(*n*)                                      | Return an array with the first *n* elements of the dataset.  |
| **takeSample**(*withReplacement*, *num*, [*seed*]) | Return an array with a random sample of *num* elements of the dataset, with or without replacement, optionally pre-specifying a random number generator seed. |
| **takeOrdered**(*n*, *[ordering]*)                 | Return the first *n* elements of the RDD using either their natural order or a custom comparator. |
| **saveAsTextFile**(*path*)                         | Write the elements of the dataset as a text file (or set of text files) in a given directory in the local filesystem, HDFS or any other Hadoop-supported file system. Spark will call toString on each element to convert it to a line of text in the file. |
| **saveAsSequenceFile**(*path*) (Java and Scala)    | Write the elements of the dataset as a Hadoop SequenceFile in a given path in the local filesystem, HDFS or any other Hadoop-supported file system. This is available on RDDs of key-value pairs that implement Hadoop's Writable interface. In Scala, it is also available on types that are implicitly convertible to Writable (Spark includes conversions for basic types like Int, Double, String, etc). |
| **saveAsObjectFile**(*path*) (Java and Scala)      | Write the elements of the dataset in a simple format using Java serialization, which can then be loaded using `SparkContext.objectFile()`. |
| **countByKey**()                                   | Only available on RDDs of type (K, V). Returns a hashmap of (K, Int) pairs with the count of each key. |
| **foreach**(*func*)                                | Run a function *func* on each element of the dataset. This is usually done for side effects such as updating an [Accumulator](https://spark.apache.org/docs/latest/rdd-programming-guide.html#accumulators) or interacting with external storage systems. **Note**: modifying variables other than Accumulators outside of the `foreach()` may result in undefined behavior. See [Understanding closures ](https://spark.apache.org/docs/latest/rdd-programming-guide.html#understanding-closures-a-nameclosureslinka)for more details. |

The Spark RDD API also exposes asynchronous versions of some actions, like `foreachAsync` for `foreach`, which immediately return a `FutureAction` to the caller instead of blocking on completion of the action. This can be used to manage or wait for the asynchronous execution of the action.

### Shuffle operations

Certain operations within Spark trigger an event known as the shuffle. The shuffle is Spark’s mechanism for re-distributing data so that it’s grouped differently across partitions. This typically involves copying data across executors and machines, making the shuffle a complex and costly operation.

#### Background

To understand what happens during the shuffle we can consider the example of the [`reduceByKey`](https://spark.apache.org/docs/latest/rdd-programming-guide.html#ReduceByLink) operation. The `reduceByKey` operation generates a new RDD where all values for a single key are combined into a tuple - the key and the result of executing a reduce function against all values associated with that key. The challenge is that not all values for a single key necessarily reside on the same partition, or even the same machine, but they must be co-located to compute the result.

In Spark, data is generally not distributed across partitions to be in the necessary place for a specific operation. During computations, a single task will operate on a single partition - thus, to organize all the data for a single `reduceByKey` reduce task to execute, Spark needs to perform an all-to-all operation. It must read from all partitions to find all the values for all keys, and then bring together values across partitions to compute the final result for each key - this is called the **shuffle**.

Although the set of elements in each partition of newly shuffled data will be deterministic, and so is the ordering of partitions themselves, the ordering of these elements is not. If one desires predictably ordered data following shuffle then it’s possible to use:

- `mapPartitions` to sort each partition using, for example, `.sorted`
- `repartitionAndSortWithinPartitions` to efficiently sort partitions while simultaneously repartitioning
- `sortBy` to make a globally ordered RDD

Operations which can cause a shuffle include **repartition** operations like [`repartition`](https://spark.apache.org/docs/latest/rdd-programming-guide.html#RepartitionLink) and [`coalesce`](https://spark.apache.org/docs/latest/rdd-programming-guide.html#CoalesceLink), **‘ByKey** operations (except for counting) like [`groupByKey`](https://spark.apache.org/docs/latest/rdd-programming-guide.html#GroupByLink) and [`reduceByKey`](https://spark.apache.org/docs/latest/rdd-programming-guide.html#ReduceByLink), and **join** operations like [`cogroup`](https://spark.apache.org/docs/latest/rdd-programming-guide.html#CogroupLink) and [`join`](https://spark.apache.org/docs/latest/rdd-programming-guide.html#JoinLink).

#### Performance Impact

The **Shuffle** is an expensive operation since it involves disk I/O, data serialization, and network I/O. To organize data for the shuffle, Spark generates sets of tasks - *map* tasks to organize the data, and a set of *reduce* tasks to aggregate it. This nomenclature comes from MapReduce and does not directly relate to Spark’s `map` and `reduce` operations.

Internally, results from individual map tasks are kept in memory until they can’t fit. Then, these are sorted based on the target partition and written to a single file. On the reduce side, tasks read the relevant sorted blocks.

Certain shuffle operations can consume significant amounts of heap memory since they employ in-memory data structures to organize records before or after transferring them. Specifically, `reduceByKey` and `aggregateByKey` create these structures on the map side, and `'ByKey` operations generate these on the reduce side. When data does not fit in memory Spark will spill these tables to disk, incurring the additional overhead of disk I/O and increased garbage collection.

Shuffle also generates a large number of intermediate files on disk. As of Spark 1.3, these files are preserved until the corresponding RDDs are no longer used and are garbage collected. This is done so the shuffle files don’t need to be re-created if the lineage is re-computed. Garbage collection may happen only after a long period of time, if the application retains references to these RDDs or if GC does not kick in frequently. This means that long-running Spark jobs may consume a large amount of disk space. The temporary storage directory is specified by the `spark.local.dir` configuration parameter when configuring the Spark context.

Shuffle behavior can be tuned by adjusting a variety of configuration parameters. See the ‘Shuffle Behavior’ section within the [Spark Configuration Guide](https://spark.apache.org/docs/latest/configuration.html).

## RDD Persistence

One of the most important capabilities in Spark is *persisting* (or *caching*) a dataset in memory across operations. When you persist an RDD, each node stores any partitions of it that it computes in memory and reuses them in other actions on that dataset (or datasets derived from it). This allows future actions to be much faster (often by more than 10x). Caching is a key tool for iterative algorithms and fast interactive use.

You can mark an RDD to be persisted using the `persist()` or `cache()` methods on it. The first time it is computed in an action, it will be kept in memory on the nodes. Spark’s cache is fault-tolerant – if any partition of an RDD is lost, it will automatically be recomputed using the transformations that originally created it.

In addition, each persisted RDD can be stored using a different *storage level*, allowing you, for example, to persist the dataset on disk, persist it in memory but as serialized Java objects (to save space), replicate it across nodes. These levels are set by passing a `StorageLevel` object ([Scala](https://spark.apache.org/docs/latest/api/scala/index.html#org.apache.spark.storage.StorageLevel), [Java](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/storage/StorageLevel.html), [Python](https://spark.apache.org/docs/latest/api/python/pyspark.html#pyspark.StorageLevel)) to `persist()`. The `cache()` method is a shorthand for using the default storage level, which is `StorageLevel.MEMORY_ONLY` (store deserialized objects in memory). The full set of storage levels is:

| Storage Level                          | Meaning                                                      |
| :------------------------------------- | :----------------------------------------------------------- |
| MEMORY_ONLY                            | Store RDD as deserialized Java objects in the JVM. If the RDD does not fit in memory, some partitions will not be cached and will be recomputed on the fly each time they're needed. This is the default level. |
| MEMORY_AND_DISK                        | Store RDD as deserialized Java objects in the JVM. If the RDD does not fit in memory, store the partitions that don't fit on disk, and read them from there when they're needed. |
| MEMORY_ONLY_SER (Java and Scala)       | Store RDD as *serialized* Java objects (one byte array per partition). This is generally more space-efficient than deserialized objects, especially when using a [fast serializer](https://spark.apache.org/docs/latest/tuning.html), but more CPU-intensive to read. |
| MEMORY_AND_DISK_SER (Java and Scala)   | Similar to MEMORY_ONLY_SER, but spill partitions that don't fit in memory to disk instead of recomputing them on the fly each time they're needed. |
| DISK_ONLY                              | Store the RDD partitions only on disk.                       |
| MEMORY_ONLY_2, MEMORY_AND_DISK_2, etc. | Same as the levels above, but replicate each partition on two cluster nodes. |
| OFF_HEAP (experimental)                | Similar to MEMORY_ONLY_SER, but store the data in [off-heap memory](https://spark.apache.org/docs/latest/configuration.html#memory-management). This requires off-heap memory to be enabled. |

**Note:** *In Python, stored objects will always be serialized with the Pickle library, so it does not matter whether you choose a serialized level. The available storage levels in Python include MEMORY_ONLY, MEMORY_ONLY_2, MEMORY_AND_DISK, MEMORY_AND_DISK_2, DISK_ONLY, and DISK_ONLY_2.*

Spark also automatically persists some intermediate data in shuffle operations (e.g. `reduceByKey`), even without users calling `persist`. This is done to avoid recomputing the entire input if a node fails during the shuffle. We still recommend users call `persist` on the resulting RDD if they plan to reuse it.

### Which Storage Level to Choose?

Spark’s storage levels are meant to provide different trade-offs between memory usage and CPU efficiency. We recommend going through the following process to select one:

- If your RDDs fit comfortably with the default storage level (`MEMORY_ONLY`), leave them that way. This is the most CPU-efficient option, allowing operations on the RDDs to run as fast as possible.
- If not, try using `MEMORY_ONLY_SER` and [selecting a fast serialization library](https://spark.apache.org/docs/latest/tuning.html) to make the objects much more space-efficient, but still reasonably fast to access. (Java and Scala)
- Don’t spill to disk unless the functions that computed your datasets are expensive, or they filter a large amount of the data. Otherwise, recomputing a partition may be as fast as reading it from disk.
- Use the replicated storage levels if you want fast fault recovery (e.g. if using Spark to serve requests from a web application). *All* the storage levels provide full fault tolerance by recomputing lost data, but the replicated ones let you continue running tasks on the RDD without waiting to recompute a lost partition.

### Removing Data

Spark automatically monitors cache usage on each node and drops out old data partitions in a least-recently-used (LRU) fashion. If you would like to manually remove an RDD instead of waiting for it to fall out of the cache, use the `RDD.unpersist()` method.

# Shared Variables

Normally, when a function passed to a Spark operation (such as `map` or `reduce`) is executed on a remote cluster node, it works on separate copies of all the variables used in the function. These variables are copied to each machine, and no updates to the variables on the remote machine are propagated back to the driver program. Supporting general, read-write shared variables across tasks would be inefficient. However, Spark does provide two limited types of *shared variables* for two common usage patterns: broadcast variables and accumulators.

## Broadcast Variables

Broadcast variables allow the programmer to keep a read-only variable cached on each machine rather than shipping a copy of it with tasks. They can be used, for example, to give every node a copy of a large input dataset in an efficient manner. Spark also attempts to distribute broadcast variables using efficient broadcast algorithms to reduce communication cost.

Spark actions are executed through a set of stages, separated by distributed “shuffle” operations. Spark automatically broadcasts the common data needed by tasks within each stage. The data broadcasted this way is cached in serialized form and deserialized before running each task. This means that explicitly creating broadcast variables is only useful when tasks across multiple stages need the same data or when caching the data in deserialized form is important.

Broadcast variables are created from a variable `v` by calling `SparkContext.broadcast(v)`. The broadcast variable is a wrapper around `v`, and its value can be accessed by calling the `value` method. The code below shows this:

- [**Scala**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_scala_9)
- [**Java**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_java_9)
- [**Python**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_python_9)

```

```

After the broadcast variable is created, it should be used instead of the value `v` in any functions run on the cluster so that `v` is not shipped to the nodes more than once. In addition, the object `v` should not be modified after it is broadcast in order to ensure that all nodes get the same value of the broadcast variable (e.g. if the variable is shipped to a new node later).

## Accumulators

Accumulators are variables that are only “added” to through an associative and commutative operation and can therefore be efficiently supported in parallel. They can be used to implement counters (as in MapReduce) or sums. Spark natively supports accumulators of numeric types, and programmers can add support for new types.

As a user, you can create named or unnamed accumulators. As seen in the image below, a named accumulator (in this instance `counter`) will display in the web UI for the stage that modifies that accumulator. Spark displays the value for each accumulator modified by a task in the “Tasks” table.

![Accumulators in the Spark UI](https://spark.apache.org/docs/latest/img/spark-webui-accumulators.png)

Tracking accumulators in the UI can be useful for understanding the progress of running stages (NOTE: this is not yet supported in Python).

- [**Scala**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_scala_10)
- [**Java**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_java_10)
- [**Python**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_python_10)

\````````



```

```

\````````

```

```



For accumulator updates performed inside **actions only**, Spark guarantees that each task’s update to the accumulator will only be applied once, i.e. restarted tasks will not update the value. In transformations, users should be aware of that each task’s update may be applied more than once if tasks or job stages are re-executed.

Accumulators do not change the lazy evaluation model of Spark. If they are being updated within an operation on an RDD, their value is only updated once that RDD is computed as part of an action. Consequently, accumulator updates are not guaranteed to be executed when made within a lazy transformation like `map()`. The below code fragment demonstrates this property:

- [**Scala**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_scala_11)
- [**Java**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_java_11)
- [**Python**](https://spark.apache.org/docs/latest/rdd-programming-guide.html#tab_python_11)

```

```

# Deploying to a Cluster

The [application submission guide](https://spark.apache.org/docs/latest/submitting-applications.html) describes how to submit applications to a cluster. In short, once you package your application into a JAR (for Java/Scala) or a set of `.py` or `.zip` files (for Python), the `bin/spark-submit` script lets you submit it to any supported cluster manager.

# Launching Spark jobs from Java / Scala

The [org.apache.spark.launcher](https://spark.apache.org/docs/latest/api/java/index.html?org/apache/spark/launcher/package-summary.html) package provides classes for launching Spark jobs as child processes using a simple Java API.

# Unit Testing

Spark is friendly to unit testing with any popular unit test framework. Simply create a `SparkContext` in your test with the master URL set to `local`, run your operations, and then call `SparkContext.stop()` to tear it down. Make sure you stop the context within a `finally` block or the test framework’s `tearDown` method, as Spark does not support two contexts running concurrently in the same program.

# Where to Go from Here

You can see some [example Spark programs](https://spark.apache.org/examples.html) on the Spark website. In addition, Spark includes several samples in the `examples` directory ([Scala](https://github.com/apache/spark/tree/master/examples/src/main/scala/org/apache/spark/examples), [Java](https://github.com/apache/spark/tree/master/examples/src/main/java/org/apache/spark/examples), [Python](https://github.com/apache/spark/tree/master/examples/src/main/python), [R](https://github.com/apache/spark/tree/master/examples/src/main/r)). You can run Java and Scala examples by passing the class name to Spark’s `bin/run-example` script; for instance:

```
./bin/run-example SparkPi
```

For Python examples, use `spark-submit` instead:

```
./bin/spark-submit examples/src/main/python/pi.py
```

For R examples, use `spark-submit` instead:

```
./bin/spark-submit examples/src/main/r/dataframe.R
```

For help on optimizing your programs, the [configuration](https://spark.apache.org/docs/latest/configuration.html) and [tuning](https://spark.apache.org/docs/latest/tuning.html) guides provide information on best practices. They are especially important for making sure that your data is stored in memory in an efficient format. For help on deploying, the [cluster mode overview](https://spark.apache.org/docs/latest/cluster-overview.html) describes the components involved in distributed operation and supported cluster managers.

Finally, full API documentation is available in [Scala](https://spark.apache.org/docs/latest/api/scala/#org.apache.spark.package), [Java](https://spark.apache.org/docs/latest/api/java/), [Python](https://spark.apache.org/docs/latest/api/python/) and [R](https://spark.apache.org/docs/latest/api/R/).