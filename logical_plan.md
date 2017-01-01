# Logical Plan

User specifies application logic as a `DAG` in populateDAG in StreamingApplication. The
DAG instance provided to populateDAG is `LogicalPlan` in the engine.

## Constructing Logical Plan
Helper objects to store meta data about `DAG` elements
### InputPortMeta
  - Name of the port
  - Attributes
  - Annotations on the port
  - Reference to owner OperatorMeta.
  
### OutputPortMeta
  - Name of the port
  - Attributes
  - Reference to owner OperatorMeta.
  - unifier meta (instance of OperatorMeta as unifier are operators in physical plan)
  - Annotations on the port (Optional)

### StreamMeta
  - Locality of the stream
  - OutputPortMeta for the source
  - List<InputPortMeta> for the sinks (destinations)
  - id of the stream (name)

### OperatorMeta
  - portMapping - this keeps the map form input port to InputPortMeta
    and outputPort to OutputPortMeta for an operator. `OperatorDescriptor` is
    used to extract this mapping. Currenty `OperatorDescriptor` use reflection
    to extract information about the port and populate the portMapping.
    
  - inputStreams - This is a map from InputPortMeta to the StreamMeta.
  - outputStreams - map from OutputPortMeta to StreamMeta
  - Attributes for the operator
  - Operator - actual operator instance used
  - LogicalOperatorStatus maintained at runtime.
    - totalTuplesProcessed
    - totalTuplesEmitted
    - failureCount
  - Name of the operator


### Fiels in LogicalPlan
- operators - a map from operator name to the OperatorMeta
- streams - a map from stream id to StreamMeta
- rootOperators - List of all input operators in the DAG.
- leafOperators - List of all output operators in the DAG.
- attributes - Attributes applicable to whole application.

#### addOperator(String name, Operator operator)
Adds operator to DAG. Construct the `OperatorMeta` from `Operator` and adds it into
the operators map. If operator with same name is already added then an error is thrown.

#### addStream(String name, OutputPort source, InputPort sinks ...)
Adds an stream to the DAG. A Stream is connection between one output port to multiple
input ports. This routine does basic validation such as stream with duplicate name, and
port specified in the argument belongs to the operators added into the DAG. Hence operators
needs to be added to DAG before they are connected using addStream.


### Validations
After DAG is constructed following validations are performed.
- No loop in the DAG. Loops are allowed only through `DelayOperator`
- The input port marks as non optional are connected to stream.
- Output ports marks as non options are connected to streams.
- Attributes set on the operators are Serializables.
- rootOperators are instance of InputOperators
- Validate affinity rules.
  - host locality options conflict with affinity rules.
  - stream locality options conflict with affinity rules.
- Validate processing modes,

### property injection
The properties injection is handled through `LogicalPlanConfiguration`. Which properties to 
applied on an operator/stream is determined by `LogicalPlanConfiguration` and applied on the
object using `BeanUtils.populate`.

### Construction from property file
`LogicalPlanConfiguration.createFromProperties` construct a LogicalPlan from property file.

### construction from json file
`LogicalPlanConfiguration.createFromJson` constructs a LogicalPlan from DAG specified
in *Json*. first it converts json to an property file and use `createFromProperties` to
construct final `LogicalPlan`
