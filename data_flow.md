## Data Flow

1. Sensors produce data for their given type (HR, temp, etc.)
2. Sensors are physically connected to an in-room hub that displays data at the bedside, but they also have the ability to send sensor data along. However, they tag the sensor data with their hub ID and send it along. This way, anytime the hub receives updated data, it sends it along.
3. The receiving end has a load balancing setup with redundancy such that the data is always received in a timely manner and one connection will not take the data down.
4. Ingestion pool takes the sensor data and applies a canonical timestamp (and any other information we might think is non-mutable) and puts the data into the sensor database, with one entry per sensor, containing the timestamp and the hub ID.
5. A pool of rule monitors is watching the sensor database, querying repeatedly based on all of the rules in the rule database. When a complete new set of rule data is found, it creates a rule task.
6. Either on demand or from a pool, a rule alert processor takes the task and sees if the current sensor data triggers the rule. If it does, it triggers an alert.
7. Notification issuer is watching alert database. When one comes in, it looks up rule, gets notification and UI/text information and creates the needed output to cause events (fires data to nurses station, triggers notification on screen, etc.)
8. NOTE: under this setup, it is possible a rule could be created to just update the Coordinated Station screen.

## Microservice descriptions

**Patient Alert Monitor:**
- Monitors raw sensor pool to compare incoming data with existing rules, looking to check completely the set of sensor data for a given rule/patient. If a given rule for a patient has a changed set of sensor data, a task is created to verify the rule.

**Alert Rule Processor:**
- The processor takes a set of sensor data for a given patient and a rule and does the processing to determine if the rule is triggered. If it is, it generates an alert and puts it into the alert queue/database.

**Notification Monitors:**
- The monitor looks at a given alert, looks up the notification rules, and triggers the events based on the setup (i.e., send a message to the Consolidated Nurse Screen as well as a message to the StayHealthy app for a given doctor).


## Considered data structures
**Rule:**
- Patient to apply rule to
- List of sensors involved in rule
- Rule logic
- Rule notification settings
- Rule UI/text info (human readable alert, i.e. “Afib detected in ECG”)

**Alert:**
- Rule ID for rule that was triggered
- Timestamp when alert was triggered
- IDs of sensor data that were used to calculate alert
- Notification targets when rule was triggered
- Notified (bool)



