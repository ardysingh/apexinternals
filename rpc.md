# RPC protocol

Explain

- Log
- ReportError
- HeartBeat
- HeartBeatResponse
- OperatorCommands
- OperatorResponse
- DeployRequest
- InitContext (getInitContext)

```java
  StreamingContainerContext getInitContext(String containerId) throws IOException;
  void log(String containerId, String msg) throws IOException;
  void reportError(String containerId, int[] operators, String msg);
  ContainerHeartbeatResponse processHeartbeat(ContainerHeartbeat msg);
```
