# RPC protocol

The Application Master (Stram) launches an RPC server using hadoop rpc `org.apache.hadoop.ipc.Server`  with protocol
defined in `StreamingContainerUmbilicalProtocol` which has following interface. The server is started as a
Service in `StreamingContainerParent`.

Once the service is started Stram writes the host name and port 
to HDFS file (apps/datatorrent/heartbeatUri), which is read by the StramChild to initialize
client for the RPC.

```java
  StreamingContainerContext getInitContext(String containerId) throws IOException;
  void log(String containerId, String msg) throws IOException;
  void reportError(String containerId, int[] operators, String msg);
  ContainerHeartbeatResponse processHeartbeat(ContainerHeartbeat msg);
```

### getInitContext
This is the first RPC call issued by StramChild upon launch. This RPC
provides global configuration for the container such as 
- startWindowMillis - timestamp of first window.
- HEARTBEAT_INTERVAL_MILLIS - Duration between heartbeat requests to Stream.
- STREAMING_WINDOW_SIZE_MILLIS - Duration of single streaming window. 
- CHECKPOINT_WINDOW_COUNT - duration between checkpoints.
- deployBufferServer - deploy buffer server in this container. This is set to tru
  for all containers.


### log
If StramChild wants to log any message in Stram's log file, it use this RPC
This is used mostly to log important debugging information.

### reportError
To report any exception to Stram.

### processHeartbeat

StramChild periodically send heartbeats to the Stram. The internal between two
heartbeat requests is configured through `HEARTBEAT_INTERVAL_MILLIS` DAG attribute.
following information is sent to the Stram.

HearBeat Request
- bufferServerHost - The host name at which bufferserver is launched.-
- bufferServerPort - Port at which buffer server is running.
- jvmName - name of the JVM. The format is pid@host
- memoryMBFree - Free memory at the container in MB, used for debugging
  and monitoring.
- restartRequested - If there is an issue with the container such as terminated eventlook,
  it can issue an request to `Stram` for relaunching the container.
- gcCollectionTime,gcCollectionCount  - gc stats for monitoring
- ContainerStats - The most important payload is stats for all the operators running in the container.
  - windowStats list of stats for each window. The number of windows sent in a request is roughly equal to
   `HEARTBEAT_INTERVAL_MILLIS` / `STREAMING_WINDOW_SIZE_MILLIS`.
    - windowId - window id for which these stats are reported.
    - checkpoint - last checkpointed window.
    - inputPorts; - Stats for each input port.
      - id - string id of the port
      - tupleCount - tuples processed in this window;
      - endWindowTimestamp - When end window was processed
      - bufferServerBytes - total bytes in bufferserver.
      - queueSize - queue size at input port;
    - outputPorts - stats for each output port.
      - structure is same as for input port.
    - cpuTimeUsed - cpu time used by an operator;
    - checkpointStats - checkpoint stats (time taken to checkpoint an operator);
    - Object counters - counter associated with this operator in a window; (deprecated)
    - Map<String, Object> metrics - The field with `@Autometric` annotations are sent as stats
      through this map.
  - nodeId - Id of the operator
  - generatedTms - When the data was collected from operator.
  - intervalMs - heartbeat interval millis (constant)
  - state - state of the operator one of the following
    - ACTIVE : Operator is running.
    - SHUTDOWN : Operator has shutdown successfully.
    - FAILED : Operator failed (mostly because of exception in operator code)
  - requestResponse;


The response to this request contains following information
- shutdown - shutdown this container.
- nodeRequests - of type `StramToNodeRequest`
- hasPendingRequests - This container has some pending requests.
- undeployRequest - undeploy these operators.
- deployRequest - deploy these operators.
  - id - unique id of the operator in the DAG;
  - type - type of the operator one of the following
    -  INPUT, UNIFIER, GENERIC, OIO
  - name - Logical name of the operator
  - checkpoint - `Checkpoint` from where to start processing. i.e if checkpoint
    is x, StramChild will restore the state of operator after processing of window x, and operator
    will start processing from (x+1) window.
  - inputs - deploy options for input streams.
    - Locality locality - 
    - portName - Port name matching the operator's port declaration
    - declaredStreamId - Name of stream declared in logical topology
    - sourceNodeId - If inline connection, id of source node in same container. For buffer server, upstream publisher node id.
    - sourcePortName - Port of the upstream node from where this input stream originates. Required to uniquely identify
      publisher end point.
    
    Following are buffer server specific fields. set only when upstream not in the same container.
    - bufferServerHost - 
    - bufferServerPort;
    - bufferServerToken;
    - Map<Integer, StreamCodec<?>> streamCodecs = new HashMap<>();
    - partitionKeys;
    - partitionMask;
    - contextAttributes - Attributes for the stream.
  - outputs - deploy options for output streams.
    - portName - Port name matching the node's port declaration
    - declaredStreamId - Name of stream declared in logical topology
    - bufferServerHost - 
    - bufferServerPort;
    - bufferServerToken;
    - streamCodecs - stream codec used while serializing data.
    - contextAttributes - Output port specific attributes.

- contextAttributes - operator specific attributes.
- committedWindowId - Committed window at `Stram`
- stackTraceRequired - stack trace required from this container.


deployRequest is returned for first heartbeat response. after that it is set only
when operator within container or upstream operators are killed and then
redeployed again.