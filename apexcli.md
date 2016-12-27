# Application Statup Sequence

1. apex cli launch command
2. get yarn application id
3. copy resources to hdfs (jar, configuration files)
4. Ask yarn to start master process
5. Master on launch construct physicalPlan + runtimePlan
6. Master asks for slave containers from RM
7. Master launch slave containers
8. One slave container is launched it sends first heartbeat request to master.
9. Master notes down bufferserver address of new container and submits
   operator deploy requests to slave.
10. Master keeps on monitoring slave using heartbeat protocol.

![launch seq](/home/tushar/work/apex/internals/images/initiallaunchseq.png)
The details are below

### Apex cli launch

#### Format of application package
Application package is nothing but a jar file with following structure.

app/*.jar
lib/*.jar
conf/*.xml

Apex cli opens up the application package and looks for application defined using
following formats.
The application can be specified as 
- Java application (implementing StreamingApplication)
- Json file 
- Property file 


Application Factory is used to prepare an DAG for submission to yarn.
![AppFactory](/home/tushar/work/apex/internals/images/AppFactory.jpg)


### Master launch


### Slave launch
