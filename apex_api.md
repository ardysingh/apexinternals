# Apex API

Apache Apex provides multiple ways to write an application.

## Low level API
This API allows specifying application logic as DAG using operators
and streams.

## High level API (Work in progress)
This API provides a way to write an application using functional way. This provides somewhat similar API
provided by *Apache Spark* and *Apache Flink*. Internally the operations are
mapped to Operators and computation is translated to Apex DAG using low level api before application
launch.

## SQL API (Work in progress)
Apex integrates with calcite to provide SQL like syntax to construct
an DAG. The relational algebra DAG is translated to Apex DAG before launch.
