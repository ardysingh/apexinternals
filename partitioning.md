# Partitioning

### Important Interfaces

- Paritioner - How to partition state.
```java
  Collection<Partition<T>> definePartitions(Collection<Partition<T>> partitions, PartitioningContext context);
  void partitioned(Map<Integer, Partition<T>> partitions);
```

- StatsListener - When to trigger partition.
```java
  Response processStats(BatchedOperatorStats stats);
```

- StreamCodec - How data is distributed.
```java
  Object fromByteArray(Slice fragment);
  Slice toByteArray(T o);
  int getPartition(T o);
```

- PartitionKeys - What each partition accepts.
```java
  public final int mask;
  public final Set<Integer> partitions;
```

- Unifier - How to combine output results from multiple partitions.
```java
  void process(T tuple);
```
