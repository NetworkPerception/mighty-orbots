# Sensor Input Database is ELT

## Status

Accepted

## Context

There is lots of sensor input data coming at a continuous rate. We have a choice - should we transform the data as it comes in, and then 
push to the database, or the opposite?

## Decision (decision and justification - the "why")

We decided that this is a good situation to push the data as it is generated, and to avoid transforming it before storing it. We chose ELT 
over ETL. It is important that the sensors are able to push data with as little delay as possible. The integrity and consistency of the data
is based on the assumption that the data in the sensor database is up to date. We can transform data after the fact, and if the system needs 
to process some data for a while, it will not block newer data from being written. The transformation is always performed on the most current
data.

## Consequences (tradeoff information and any other notable side effects, also impacts)
There are some steps we could take to optimize the organization of the data before we store it. This potentially could improve performance, 
or save some memory. But we decided that data consistency was of a higher priorty.
