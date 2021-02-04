## Presto  任务调度

- ## 内存管理

  Presto是一款内存计算型的引擎，所以对于内存管理必须做到精细，才能保证query有序、顺利的执行，部分发生饿死、死锁等情况。

- ## **内存池**

  Presto采用逻辑的内存池，来管理不同类型的内存需求。

  Presto把整个内存划分成三个内存池，分别是System Pool ,Reserved Pool, General Pool。

  ![img](https://pic4.zhimg.com/80/v2-36393caca1aa8ec58284ef099445b8cb_720w.jpg)

  1. System Pool 是用来保留给系统使用的，默认为40%的内存空间留给系统使用。
  2. Reserved Pool和General Pool 是用来分配query运行时内存的。
  3. 其中大部分的query使用general Pool。 而最大的一个query，使用Reserved Pool， 所以Reserved Pool的空间等同于一个query在一个机器上运行使用的最大空间大小，默认是10%的空间。
  4. General则享有除了System Pool和General Pool之外的其他内存空间。

- ## 为什么要使用内存池

  System Pool用于系统使用的内存，例如机器之间传递数据，在内存中会维护buffer，这部分内存挂载system名下。

  那么，为什么需要保留区内存呢？并且保留区内存正好等于query在机器上使用的最大内存？

  如果没有Reserved Pool， 那么当query非常多，并且把内存空间几乎快要占完的时候，某一个内存消耗比较大的query开始运行。但是这时候已经没有内存空间可供这个query运行了，这个query一直处于挂起状态，等待可用的内存。 但是其他的小内存query跑完后，又有新的小内存query加进来。由于小内存query占用内存小，很容易找到可用内存。 这种情况下，大内存query就一直挂起直到饿死。

  所以为了防止出现这种饿死的情况，必须预留出来一块空间，共大内存query运行。 预留的空间大小等于query允许使用的最大内存。Presto每秒钟，挑出来一个内存占用最大的query，允许它使用reserved pool，避免一直没有可用内存供该query运行。

- ## 内存管理

  ![img](https://pic1.zhimg.com/80/v2-043d246b3d18a40b9939f8a3fb9767c0_720w.jpg)

  Presto内存管理，分两部分：

  - query内存管理

  - - query划分成很多task， 每个task会有一个线程循环获取task的状态，包括task所用内存。汇总成query所用内存。
    - 如果query的汇总内存超过一定大小，则强制终止该query。

  - 机器内存管理

  - - coordinator有一个线程，定时的轮训每台机
    - 器，查看当前的机器内存状态。

  当query内存和机器内存汇总之后，coordinator会挑选出一个内存使用最大的query，分配给Reserved Pool。

  内存管理是由coordinator来管理的， coordinator每秒钟做一次判断，指定某个query在所有的机器上都能使用reserved 内存。那么问题来了，如果某台机器上，，没有运行该query，那岂不是该机器预留的内存浪费了？为什么不在单台机器上挑出来一个最大的task执行。原因还是死锁，假如query，在其他机器上享有reserved内存，很快执行结束。但是在某一台机器上不是最大的task，一直得不到运行，导致该query无法结束。

- ▍1.1 简介 Presto是Facebook开源的MPP（Massive Parallel Processing）SQL引擎，其理念来源于一个叫Volcano的并行数据库，该数据库提出了一个并行执行SQL的模型，它被设计为用来专门进行高速、实时的数据分析。Presto是一个SQL计算引擎，分离计算层和存储层，其不存储数据，通过Connector SPI实现对各种数据源（Storage）的访问。
- [GC params introduction](https://blog.csdn.net/leo187/article/details/88920036)
- [Memory usage](https://prestosql.io/docs/current/admin/properties-memory-management.html)



- # 内存管理属性

  ## `query.max-memory-per-node`[＃](https://prestosql.io/docs/current/admin/properties-memory-management.html#query-max-memory-per-node)

  - **类型：** `data size`
  - **默认值：** `JVM max memory * 0.1`

  这是查询可在工作程序上使用的最大用户内存量。用户内存在执行期间分配给直接归因于用户查询或可由用户查询控制的事物。例如，执行期间建立的哈希表使用的内存，排序期间使用的内存等。当对任何工作程序的查询的用户内存分配达到此限制时，它将被杀死。

  ## `query.max-total-memory-per-node`[＃](https://prestosql.io/docs/current/admin/properties-memory-management.html#query-max-total-memory-per-node)

  - **类型：** `data size`
  - **默认值：** `JVM max memory * 0.3`

  这是查询可在工作程序上使用的最大用户和系统内存量。系统内存是在执行期间分配给与用户查询不直接相关或不可控制的内容的。例如，由读取器，写入器，网络缓冲区等分配的内存。当对任何worker的查询分配的用户和系统内存的总和达到此限制时，它将被杀死。的值`query.max-total-memory-per-node`必须大于 `query.max-memory-per-node`。

  ## `query.max-memory`[＃](https://prestosql.io/docs/current/admin/properties-memory-management.html#query-max-memory)

  - **类型：** `data size`
  - **默认值：** `20GB`

  这是查询可在整个群集中使用的最大用户内存量。用户内存在执行期间分配给直接归因于用户查询或可由用户查询控制的事物。例如，执行期间建立的哈希表使用的内存，排序期间使用的内存等。当查询在所有工作程序中的用户内存分配达到此限制时，它将被杀死。

  ## `query.max-total-memory`[＃](https://prestosql.io/docs/current/admin/properties-memory-management.html#query-max-total-memory)

  - **类型：** `data size`
  - **默认值：** `query.max-memory * 2`

  这是查询可在整个群集中使用的最大用户和系统内存量。系统内存是在执行期间分配给与用户查询不直接相关或不可控制的内容的。例如，由读取器，写入器，网络缓冲区等分配的内存。当查询在所有工作线程上分配的用户和系统内存的总和达到此限制时，它将被杀死。的值`query.max-total-memory`必须大于 `query.max-memory`。

  ## `memory.heap-headroom-per-node`[＃](https://prestosql.io/docs/current/admin/properties-memory-management.html#memory-heap-headroom-per-node)

  - **类型：** `data size`
  - **默认值：** `JVM max memory * 0.3`

  这是JVM堆中预留给Presto不能跟踪的分配的余量/缓冲区的内存量。

- [Presto内存调优及原理](https://cloud.tencent.com/developer/article/1156796)