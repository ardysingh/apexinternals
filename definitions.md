# Terms used in the documents

- **Operator** -
  Operators are independent units of logical operations which can contribute in executing the business logic of a use case.
  
- **Input Port** -
  This is a port through which an operator accepts input tuples from an upstream operator. An operator can have
  multiple input ports. Input port can be marked as optional, an optional port need
  not be connected in the DAG.
  
- **Output Port** -
  This is a port through which an operator passes on the processed data to downstream operators. An operator
  can have multiple output ports. By default these ports are optional.
  
- **Stream** -
  A stream is connection between output port and one or more input ports. The data emitted
  on output port is sent to all the input ports connected through a stream.

- **Input Operator** - An operator without any input ports. These operators mostly read tuples from external
  systems.
  
- **Output Operators** - An operator without any output ports. These operators are mostly
  communicate with external systems to store the result.

- **Upstream operators** - These are the operators from which there is a directed path to opr in the application DAG.

- **Downstream operators** - These are the operators to which there is a directed path from opr in the application DAG.

- **Attribute** -
  attribute are meta data associated with operator/stream which is used by
  platform for resource allocation.
  
- **Partition** -
  Apex can scale application by creating mulitiple instance of an operator. Each instance 
  of an operator is called as partition of the operator. Apex supports specifying parallelism
  for an operator during launch time as well as parallelism can be changed
  during runtime based on the load.
  
- **Unifier** -
  Unifier can perform additional computation on result from multiple partition of
  same operator before delivering data to next operator. Unifiers are added by
  platform automatically during partitioning.
  
- **Stram** -
  Stram is master process which monitors the application, and carry out recovery in case of failure.
  
- **Stram Child** -
  Stream child are java process in which one or more operators are deployed. Stram launch these
  processes through yarn.
  
- **Buffer Server** -
  Communication between operators deployed in different process/node goes through
  buffer server. Buffer server is in memory publisher/subscriber messaging queue used
  for communication.
  
- **Container** -
  Container is an jvm process launch by yarn to run Apex process (stram/stram child).
  
See []Apex user documentation](http://apex.apache.org/docs/apex/operator_development/) for detail
description for above terms.
