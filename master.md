
# Application Master

Important classes are
- ApplicationMasterService
- StreamingContainerManager
- StreamingContainerParent
- StreamWebService

# ApplicationMasterService

 Major Roles of ApplicationMasterService:
 
 Init:
 - Create recovery handlers from Logical Plan.
    - Read Logical Plan from HDFS (df-conf.ser).
    - Create recovery directories((FILE_LOG, FILE_LOG_BACKUP,FILE_SNAPSHOT, FILE_SNAPSHOT_BACKUP, FILE_HEARTBEATURI))
 - Create Plan context (StreamingContainerManager - manages the CheckpointState).
    - Load the CheckpointState from HDFS.
    - If CheckpointState is available:
        - Get CheckpointState from Physical plan and create StreamingContainerManager with this state.
        - Replay the log.
        - Restore the checkpoint info by Syncing and updating the checkpoints and get committedWindowId.
        - Create new container agents (StreamingContainerAgent) for existing containers.
        - Request new container if not present.
    - Else CheckpointState is unavailable:
        - Create StreamingContainerManager with logical dag.
    - Populate recoveryHandler and checkpoint the state.
 - Add service NMClientAsync (Communication between Node Manager and  Application Master).
 - Add service AMRMClient (Communication between Application Master and Resource Manager).
 - Add service HeartbeatListener(Communication Protocol between Containers and Application Master) 
 - Add service AppDataTransportAgent (Metrics, mainly Stats collections)

```
  execute
     - while !appDone
        refreshToken every expiryTime
        updateNodeReport every nodeReportUpdateTime
        process pendingTasks (currently pending tasks are submitting container start request to manager)
        sleep 1 second
        if manager.containerRequests not empty
            Prepare container request based on nodeReport and antiaffinity (add to containerRequests + pendingRequests)
        Try to prepare container requests which could be not sent before. (from pendingRequests to containerRequests if possible now)
        Check for blacklisted nodes.
        Send allocation reqquests to RM (containerRequests, releasedContainers, removedContainersRequests)
            
        for newly allocated containers
            if already allocated then continue (can happen as we could send multple requests to yarn for allocation)
            Remove from containerRequests
            StreamingContainerAgent sca = dnmgr.assignContainer(resource, null);
            launch container and raise an event.

        get completed containers
        for all completed containers
          if (exitStatus != 0) {
            if exitStatus != 1 put node in blacklist if number of consecutive failures are above threshold.
            schedule container restart.
          }
          dnmgr.removeContainerAgent(containerIdStr);
          raise event
          
       update black list at resource manager

       if (dnmgr.forcedShutdown)
         appDone = true;
          
        dnmgr.monitorHeartbeat();
}
```

### LaunchContainerRunnable
  launch allocated container on the node using node manager protocol.

### StreamingContainerParent
  This is  a RPC service, which start RPC server and hadles rpc requests.

  processHeartBeat
    print warning message if delay of timestamp is high
    dagManager.processHeartbeat(fmsg);


StreamingContainerManager

Imp variables
containerStopRequests <- containers which needs to be stoped.

```
processHeartbeat {
    get StreamingContainerAgent if not found or shutdown then send shutdown response.

    if (firstHeartbeat from container) {
        getBufferServer address
        set container state as active.
        log in container and operator file in a separate task.
    }

    if container has requested restart then put container to stop request.

    for each operator in heartbeat response {
        get PTOperator from operator id, log error and put in undeploy request.

        process operator responses

        process operator state (current state know to stram and state reported by the heartbeat)

        for each stats {
            if stats contains checkpoint update operator checkpoint
            process input port stats computing 
            - totalTuples, tuplesPMSMA, bufferServerBytesPMSMA, queueSizeMA
            process output port stats
            record operator last window change.
            update physical and logical operator stats.
        }
    }
}
```
