## Presto 内存调优：

Presto是一个开源的分布式SQL查询引擎，适用于交互式分析查询，数据量支持GB到PB字节。Presto支持在线数据查询，包括Hive, Cassandra, 关系数据库以及专有数据存储。 一条Presto查询可以将多个数据源的数据进行合并，可以跨越整个组织进行分析。Presto以分析师的需求作为目标，他们期望响应时间小于1秒到几分钟。 Presto终结了数据分析的两难选择，要么使用速度快的昂贵的商业方案，要么使用消耗大量硬件的慢速的“免费”方案。

​        presto0.131开始对内存模型进行优化，直至当前EMRv2中上线版本0.188(包括EMRV1的0.161)都是使用的这个内存模型。使用的是一种称为内存池(memory-pool)的机制来管理presto中任务及presto本身的内存使用。

​       目标和设计思路可查看：https://github.com/prestodb/presto/issues/2624

​      当前presto已经实现了其设定的目标：

1. 更容易的让用户控制每个查询内存限制。
2. 预防内存溢出及由此引发的崩溃。
3. 充分利用集群内存，不被一个“大查询”引起整个集群内存资源的限制。    

# Presto内存调优参数

​      presto把每个worker节点可分配内存（jvm Xmx）分成三份，分别是系统内存池(SystemMemoryPool)，保留内存池(ReservedMemoryPool)和普通内存池(GeneralMemoryPool)。在Presto启动时，它们会随着worker节点初始化时被分配，然后通过服务发现各个worker节点上报给coordinator节点。

下图是presto worker节点的内存示意图：



![1607936321182](C:\Users\jinzhao01\AppData\Roaming\Typora\typora-user-images\1607936321182.png)



​       从示意图中可以看到，一个worker节点的内存堆大小可以最大分成两份：系统预留内存+查询内存。而查询内存又分为最大查询内存+其他查询内存。

**系统预留内存**：worker节点初始化和执行任务必要的内存，包括preto发现服务的定时上报、每个query中task管理数据结构等。使用**resources.reserved-system-memory**配置项配置，默认是worker节点堆大小的0.4。

除了系统预留内存，其余给woker 内存都会给查询使用：

**最大查询内存**：coordinator节点会定时调度查看每个query使用的时长和内存，在此过程中会找到耗用内存最大的一个query，并会为此query调度最大的内存使用。这个query可获得各个worker节点最大配置的**专用**最大内存量。使用**query.max-memory-per-node**配置项可以配置，默认是worker节点堆大小的0.1。这个值可根据query监控的peak Mem作为参考设定。

**其他查询内存**：worker节点堆中除了系统预留的内存和最大查询的内存就是其他查询内存。

worker节点的堆内存的配置跟用户使用两个场景关系最大：

1.用户查询数据量/复杂性

2.用户查询并发度

1.决定了改用多大的最大查询内存 2.决定了该用多大jvm堆。

​        举个简单的例子，如果有一批(n=5)查询同时提交有10个worker节点的presto集群，其中数据量/复杂度最高的一个query语句一共要使用20GB的内存，那么我们应该给每个worker节点最大查询内存即是20GB/10 = 2GB，而需要并发执行，那么留给查询内存的应该是2GB*5大约为10GB，此时可以推出应该给每个worker节点配置堆大小为10/0.6 =17。（预留暂用0.4，留给查询的为0.6）。

​      除了上述的三个配置，还有一个配置需要关注，即查询最大可支持内存：表示每个query最大可支持内存。配置项为：**query.max-memory**这个值可配置为 query.max-memory-per-node*worker数目，以上个例子来说就是2GB*10=20GB。

用这个几个参数就能基本解决在使用presto集群时碰到的大部分查询慢和OOM问题。当然，需要对一个presto集群做更多精细化内存管理：比如针对到用户的内存调度，比如使用排队限制进入集群而限制整个集群query使用内存的限额，比如coordinator的内存精细管理。

# Presto内存调优原理

​      看完上一部分可以直观的在emr配置下发控制台操作实践起来了，对于想了解其中原理和排查更深层原因可以继续往下看（开始从源码角度讲原理，因为源码才能了解一切细节）：

​      presto把每个worker节点可分配内存（jvm Xmx）分成三份，分别是系统内存池(SystemMemoryPool)，保留内存池(ReservedMemoryPool)和普通内存池(GeneralMemoryPool)。在Presto启动时，它们会随着worker节点初始化时被分配，然后通过服务发现各个worker节点上报给coordinator节点。具体初始化是通过构造LocalMemoryManager类时完成的：

```js
public final class LocalMemoryManager
{
    public static final MemoryPoolId GENERAL_POOL = new MemoryPoolId("general");
    public static final MemoryPoolId RESERVED_POOL = new MemoryPoolId("reserved");
    public static final MemoryPoolId SYSTEM_POOL = new MemoryPoolId("system");

    private final DataSize maxMemory;
    private final Map<MemoryPoolId, MemoryPool> pools;

    @Inject
    public LocalMemoryManager(NodeMemoryConfig config, ReservedSystemMemoryConfig systemMemoryConfig)
    {
        requireNonNull(config, "config is null");
        requireNonNull(systemMemoryConfig, "systemMemoryConfig is null");
        long maxHeap = Runtime.getRuntime().maxMemory();
        checkArgument(systemMemoryConfig.getReservedSystemMemory().toBytes() < maxHeap, "Reserved memory %s is greater than available heap %s", systemMemoryConfig.getReservedSystemMemory(), new DataSize(maxHeap, BYTE));
        maxMemory = new DataSize(maxHeap - systemMemoryConfig.getReservedSystemMemory().toBytes(), BYTE);

        ImmutableMap.Builder<MemoryPoolId, MemoryPool> builder = ImmutableMap.builder();
        checkArgument(config.getMaxQueryMemoryPerNode().toBytes() <= maxMemory.toBytes(), format("%s set to %s, but only %s of useable heap available", QUERY_MAX_MEMORY_PER_NODE_CONFIG, config.getMaxQueryMemoryPerNode(), maxMemory));
        builder.put(RESERVED_POOL, new MemoryPool(RESERVED_POOL, config.getMaxQueryMemoryPerNode()));
        DataSize generalPoolSize = new DataSize(Math.max(0, maxMemory.toBytes() - config.getMaxQueryMemoryPerNode().toBytes()), BYTE);
        builder.put(GENERAL_POOL, new MemoryPool(GENERAL_POOL, generalPoolSize));
        builder.put(SYSTEM_POOL, new MemoryPool(SYSTEM_POOL, systemMemoryConfig.getReservedSystemMemory()));
        this.pools = builder.build();
    }
```

​      reservatedMemoryPool的大小根据config.properties文件配置query.max-memory-per-node决定。这个值的默认值是jvm(xmX)的0.1。

```js
public class NodeMemoryConfig
{
    public static final String QUERY_MAX_MEMORY_PER_NODE_CONFIG = "query.max-memory-per-node";

    private DataSize maxQueryMemoryPerNode = new DataSize(Runtime.getRuntime().maxMemory() * 0.1, BYTE);

    @NotNull
    public DataSize getMaxQueryMemoryPerNode()
    {
        return maxQueryMemoryPerNode;
    }

    @Config(QUERY_MAX_MEMORY_PER_NODE_CONFIG)
    public NodeMemoryConfig setMaxQueryMemoryPerNode(DataSize maxQueryMemoryPerNode)
    {
        this.maxQueryMemoryPerNode = maxQueryMemoryPerNode;
        return this;
    }
}
```

​       systemMemoryPool大小则根据配置resources.reserved-system-memory决定，这个值的默认值是jvm(xmX)的0.4。

```js
public class ReservedSystemMemoryConfig
{
    private DataSize reservedSystemMemory = new DataSize(Runtime.getRuntime().maxMemory() * 0.4, BYTE);

    @NotNull
    public DataSize getReservedSystemMemory()
    {
        return reservedSystemMemory;
    }

    @Config("resources.reserved-system-memory")
    public ReservedSystemMemoryConfig setReservedSystemMemory(DataSize reservedSystemMemory)
    {
        this.reservedSystemMemory = reservedSystemMemory;
        return this;
    }
}
```

## 最大内存调度策略

​        Presto一条query语句宏观上来说是把它转换成基于stage的逻辑执行计划。然后再通过逻辑执行计划转成物理执行计划在worker节点间分成并行的task，每个task内又转换成具体的Operator实际执行。在真正执行物理计划前，内存需求都来自于systemMemoryPool，包括临时数据结构，传输buffer等。执行物理计划时，不同的Operator类型都根据需要申请内存，比如aggregationOperator使用getEsctimatedSize()方法预估需要的内存。这里获取的内存来自于reservatedMemoryPool或者generalMemoryPool，究竟使用哪个pool取决于当前查询是否耗用内存最大。

​       在查询创建时，默认使用generalMemoryPool:

```js
public SqlTaskManager(
            LocalExecutionPlanner planner,
            LocationFactory locationFactory,
            TaskExecutor taskExecutor,
            QueryMonitor queryMonitor,
            NodeInfo nodeInfo,
            LocalMemoryManager localMemoryManager,
            TaskManagementExecutor taskManagementExecutor,
            TaskManagerConfig config,
            NodeMemoryConfig nodeMemoryConfig,
            LocalSpillManager localSpillManager,
            NodeSpillConfig nodeSpillConfig)
    {     

...      

  queryContexts = CacheBuilder.newBuilder().weakValues().build(CacheLoader.from(

                queryId -> new QueryContext(
                        queryId,
                        maxQueryMemoryPerNode,
                        localMemoryManager.getPool(LocalMemoryManager.GENERAL_POOL),
                        localMemoryManager.getPool(LocalMemoryManager.SYSTEM_POOL),
                        taskNotificationExecutor,
                        driverYieldExecutor,
                        maxQuerySpillPerNode,
                        localSpillManager.getSpillSpaceTracker())));
```

​        presto会定时检查所有查询消耗的内存，这个定时器在presto初始化时被构造，实现如下：

```js
queryManagementExecutor.scheduleWithFixedDelay(() -> {
    ...
    enforceMemoryLimits();
    ...
}, 1, 1, TimeUnit.SECONDS);
```

​        其中的updateAssignments方法，其作用是找出最耗费内存的查询，并放入RESERVED_POOL： 

```js
if (reservedPool.getAssignedQueries() == 0 && generalPool.getBlockedNodes() > 0) {
    QueryExecution biggestQuery = null;
    long maxMemory = -1;
    for (QueryExecution queryExecution : queries) {

        if (resourceOvercommit(queryExecution.getSession())) {     
            continue;
        }

        long bytesUsed = queryExecution.getTotalMemoryReservation();

        if (bytesUsed > maxMemory) {
            biggestQuery = queryExecution;
            maxMemory = bytesUsed;
        }
    }

    if (biggestQuery != null) {
        biggestQuery.setMemoryPool(new VersionedMemoryPoolId(RESERVED_POOL, version));
    }

}
```

​       再看外层的updateNodes方法，可以发现RESERVED POOL的这种分配策略会应用到每个节点。也就是说这个查询在每个节点都会独占RESERVED POOL的空间。

```js
for (RemoteNodeMemory node : nodes.values()) {
    node.asyncRefresh(assignments);
}
```

# Presto内存监控

   可以通过presto提供的WebUi来即时追踪整个集群或者每个查询执行的内存的使用情况。presto的WebUI后台使用的是DI框架Guice，前台是jquery+react实现。



![1607936416151](C:\Users\jinzhao01\AppData\Roaming\Typora\typora-user-images\1607936416151.png)

Presto 集群overview

![1607936440935](C:\Users\jinzhao01\AppData\Roaming\Typora\typora-user-images\1607936440935.png)

presto 某个query监控图

​       web页面中reservedMemmory即是ReservedMemoryPool的大小。而在查询资源汇总中，peakMemory表示当前查询使用memory峰值。memory pool表示当前使用的pool(reserved或者genaral)。

[ref link](https://cloud.tencent.com/developer/article/1156796)