**Objectives:**
* The system must support multiple sensors, be able to submit the information to a central system, and be able to detect & report on various sensor failures.
* In addition, as many sensor types must be supported.
* All of the requested data is simple, other than ECG.

| Sensor Type | Update Interval | Data Size | Data Type |
| ------------- | ------------- | ------------- | ------------- |
| Heart rate | .5 seconds | 1 byte | int |
| Blood pressure | 3600 seconds | 2 bytes | int/int |
| Oxygen level | 5 seconds | 1 byte | int |
| Blood sugar | 120 seconds | 4 bytes | float |
| Respiration | 1 second | 1 byte | int |
| Body temp | 300 seconds | 1 byte | int |
| Sleep status | 60 seconds | 1 byte | bool |
| ECG | 1 second | depends on size, but 5-10 sensors * update hz * each lead data field | xml |

**The Data Ingestor microservice** accepts the sensor data from the In-Room Hub and adds some metadata, namely a canonical timestamp (to allow for comparison between all datasets) and the patient ID, looked up from the Patient->Room->Hub table. The Data Ingestor could pull the data from a queue and scale out in case the data needs/sizes come in faster.

The data is then pushed into the Vital-Sign Data Storage to be used by further steps.
