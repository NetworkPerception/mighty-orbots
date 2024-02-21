# Sensor Hub Data Integration

## Status
Accepted

## Context

When taking sensor data in, we have to understand the methods by which the sensor data will be sent. This leads into 1) understanding of the data itself (and how to identify it), and 2) gives us a concept of what we can control around edge cases where sensors are not reporting data (either missing, damaged, disconnected, or malfunctioning). Also, the impact of this solution can greatly affect the business case for a given hospital, as well as the financial expectations of the company to support a given system.

The two major possibilities are:
- Create a full vertical integration, providing not only a device in each room to coordinate sensor data, but also maintaining the sensors themselves. This would involve replacements and understandings of the sensors, their reporting methods, the data formats, and the like.
- Integrate with an existing sensor setup. Companies like Medtronic already supply systems to hospitals that involve not only selling the sensors but also the hubs/screens for each bed.

## Decision (decision and justification - the "why")

As long as the integration is the same, we're punting this decision. If the integration is difficult (or the existing equipment can't distinguish between, say, unplugged and malfunctioning sensors), then our decision would be to completly vertically integrate into their systems.

## Consequences (tradeoff information and any other notable side effects, also impacts)

Now, if we vertically integrate, that gives us full confidence in giving the nurses complete status on the sensors (i.e., knowing the difference between sensors being unplugged and something going wrong). However the downside is that the hospital would have to adhere to our complete hardware solution. That would mean not only paying for & switching out new equipment, but also any staff training around differences between their current systems.

In addition, that adds overhead to our organization to be available to troubleshoot the individual hardware issues with both sensors and the screen displays at each bed.
