# Flow of operator being removed from the plan

An operator can request shutdown by throwing ShutdownException from any of operator lifecycle method. StramChild
terminate the operator thread also does some finalization if required such as calling `endWindow` or taking a
checkpoint for exactly once processing. From now on the operator stats sent by the container does
not contain any stats for this operator.

On master during heartbeat processing `processHeartbeat`, master checks for any missmatch between expected
number of operators in a container and actually reported operators by the heartbeat, master takes
following actions

- For extra operator reported, send undeploy request in heartbeat response.
- For missing operator, remove physical operator from the physical plan.

StreamingContianerManager `shutdownOperators` variabel maintains list of operators needs to be
shutdown. The operator can be safely removed if 
- operator window id is less than or equal to the committedWindowId.
- All downstream operators have windowId equal to the operator being removed.

  * processHeartbeat
    * processOperatorDeployStatus
      If operator state changes from ACTIVE to SHUTDOWN add it to the `shutdownOperators` map. `shutDownOperators` is a map
      from windowId to list of operators which have shutdown in this window.
 
  In processEvent after statshandlers are called `shutdownOperators` map is processed and these
  operators are removed from physical plan.
   
important functions to study

- removeTerminatedPartition - remove the physical partition also downstream operator if their input is empty after
removing current operator.

- removePTOperator : remove unifiers, remove inputs from downstream operators, remove from upstream
operator. remove from operator grouping.

- removePartition : remove parallel partitions recursively, update pamapping and call remotePTOperator at end.