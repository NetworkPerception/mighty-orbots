# The data administration component is an input to analysis

## Status

Superseded

## Context

The data administration component takes user input like notification settings, thresholds, rules, new patient info. We need some of this data
during analysis. The rule processor definitely needs input from the user's rules

## Decision (decision and justification - the "why")

We decided that it would be appropriate to draw an arrow from the data administration component to the analysis component, indicating that 
data from data administration is an input to analysis.

## Consequences (tradeoff information and any other notable side effects, also impacts)
We so far had a unidirectional data flow, and there were no loops in our data flow, which we viewed as spots where data can lose consistency.
Adding the data administration component as an input is a complication to this design.
