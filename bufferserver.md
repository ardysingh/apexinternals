# Buffer Server

- Deploy
  Buffer Server is started on all containers by default. The port and host
  where buffer server is running is sent to the master in heartbeat rpc. Application
  Master use this address to setup routing between operators deployed in different
  containers. The deploy request for downstream operator's input port contains buffer server
  address of upstream operator depending on how operator is partitioned.
  
- Publish 
  The publish request contains following information.
  - identifier : The identifier as a stream. The identifier is constructed as
  ```
    operatorId + "." + outputPortName + "." + streamCodecId
  ```
  Where
  - operator Id is physical operator id
  - outputPortName is name of the output port
  - streamCodecId is id of the stream codec. The stream codec is appended because multiple input
  port could connect to an output port with different stream codec. If stream codec are different
  then a separate data channel is created for them as their serialization format and hashcode could
  be different.
  
  Each publisher is represented by DataList. The list of all publishers to a buffer server is kept in `publisherBuffers`
  
- Subscribe
  This functionality is implemented through *Server.handleSubscriberRequest*
  The subscribe request contains following information.
  - identifier - Input port identifier
  - windowId - start windowId from where to start reading.
  - streamType - No idea
  - upstreamIdentifier - upstream stream identifier. topic to listen to.
  - mask - for subscription mask
  - partitions - for subscription partitions.
  - bufferSize -
  
  mask and partitions is used for routing of the user tuples. The subscribe request is sent during deployment
  of input ports of downstream operators in`StreamingContainer.deployInputStreams`. A `BufferServerSubscriber`
  object is created to fetch data from upstream operators publish channel and attached to the operator's input
  port.
  

- adding tuple
  - what happens when memory is full
- stream codec/partitions keys
- purge
- reset
- BufferServer publisher/Subscriber/Reservior