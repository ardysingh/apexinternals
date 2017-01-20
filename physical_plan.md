# Physical Plan

Physical plan is contructed from LogicalPlan during start of the Application Master.
The most of the work in done in constructor of `PhysicalPlan`
. The steps to construct a plan is

### Classes 
Similar to LogicalPlan , PhysicalPlan maintans the physical DAG using following classes

- *PTOperator* - A partition of the Operator.
- *StreamMapping* - maintains physical representation of a StreamMeta. It maintains mapping from
  list of PTOutput to list of PTInput with unifiers.
- *PMapping* - Keep a mapping between logical operator and its physical partitions.
  It maintains following information
  - operatorMeta from LogicalPlan
  - partitions - all partitions of the operator.
  - a mapping from output port meta to StreamMapping.
- *PInput* - A physical input port of an PTOperator.
  - logicalStreamMeta - StreamMeta from LogicalPlan.
  - target - PTOperator
  - partitions - PartitionKeys for this input.
  - source - PTOutput
  - portName - name of the port.
- *POutput* - 
   - logicalStream;
   - source - operator for the port.
   - portName - name of the port
   - sinks - list of inputs where this port is connected 
- *PContainer* -


### Generation of Physical Plan from LogicalPlan
- From root to leaf operators call addLogicalOperator
  - construct a PMapping for operator.
  - for all input ports of the operator
    - if port is parallel partitioned add this node to set of parallelPartition
      of source.
    - check if all parallel partition port have common upstream, else throw an
      error.
    - if stream locality is CONTAINER_LOCAL or THREAD_LOCAL then update
      inlinePrefs.
  - update logicalToPTOperatorMap
  - initPartitions
    - get PARTITIONER for operator
    - call definePartitions with operator as single input partitioner
    - for each partition call addPTOperator.
      - for each outputPort add PTOutput to PTOperator
      - set partition keys from partition.
      - add partition into PMapping.
      - add the partition into newOpers.
  - call updateStreamMapping
  

### StreamMapping

Runtime activity
- Removing an partition
- Dynamic partitioning
- Dynamic logical plan change
- Checkpoint book-keeping
