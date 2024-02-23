# Analysis and Output Generation are Event-Driven

## Status 

Accepted

## Context 

We had a basic understanding that the screen had to update according to the most recent data. At first we considered that the client 
(the screen) could send queries to the sensor database directly. In this design, any functionality that needed any sensor data would 
query the database, making a sunburst structure facing the input database. The more we thought about how the analysis and the output 
generation fit in the overall architecture, we realized the whole process should be event driven. 

## Decision (decision and justification - the "why")

This is a more optimal and refined design than our initial implementation. We see a data streamline that starts from the sensor input, and
different analysis can happen along the way. The data is straightforward, and a transformation to make the data useful 
(adding metadata and patient association) is inexpensive. The new sensor data is always wanted, the data always takes the same path.
Our core functionality is based on reading data the same way each time, transforming it in the same way, and 
creating alerts of an expected type. We are not imagining much functionality to be needed outside of this main update pipeline.

## Consequences (tradeoff information and any other notable side effects, also impacts)
This is a great way to handle systems that have a well defined data flow. But, if there is a major change to the design of the product, and 
a lot of secondary functionality is requested, it will be difficult to make the data flow more general.
