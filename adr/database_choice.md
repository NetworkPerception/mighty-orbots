# Database Choice

## Status
Proposed
  
## Context
Raw vital signs are received from sensors and need to be stored at very high throughput. 
Sensor history is only kept for 24hrs but sampled constantly, several times per second in order to identify anomalies and alert the medical staff in real-time.
Standard relational database management systems can be used but are not optimized for time series data and tend to be slower for inserting and retrieving time series data 

## Decision
Considering the temporal constraint and distribution of the sensor data, choosing a time series database came naturally. 
Time series databases are optimized for collecting, storing, retrieving and processing of time series data. 
They are unique in a sense they support very high throughput, regular and irregular writes and have native support for summaries, aggregation and rollups.

## Consequences
Two database management systems will be used here:
- a time series database for the vital sign data storage
- a standard relational database system with distinct tables for rules, alerts, patient to room and hub to room and other associations.
