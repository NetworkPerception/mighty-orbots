# Table of Contents
- [1.  Welcome to MightyOrbots](#1--welcome-to-mightyorbots)
- [2.  Problem Space](#2--problem-space)
- [3.  Solution Space](#3--solution-space)
- [4.  System Architecture: Components](#4--system-architecture-components)
- [5.  System Architecture: Data Flow](#5--system-architecture-data-flow)
- [6.  Detailed Architecture](#6--detailed-architecture)
- [7.  Architecture Decision Records](#7--architecture-decision-records)


# 1.  Welcome to MightyOrbots
Repository for MightyOrbots' solution to O'Reilly 2024 Architectural Katas Challenge.

# 2.  Problem Space
## 2.1  Functional Requirements
From the problem statement, we extracted the following core requirements to guide our proposed architecture for the MonitorMe system.

**Data Ingestion:**
The system should be able to read data from eight different patient-monitoring equipment.
Vital sign data sources include heart rate, blood pressure, oxygen level, blood sugar, respiration rate, electrocardiogram (ECG), body temperature, and sleep status.

**Data Consolidation and Display:**
The system should consolidate the data from multiple sources into a single stream.
It should display data on a consolidated monitoring screen at nurses' stations.
Each patient's vital signs should be displayed, rotating between patients every 5 seconds.
The monitoring screen should have a maximum capacity of displaying vital signs for 20 patients per screen.

**Data Recording and Storage:**
MonitorMe must record and store the past 24 hours of all vital sign readings for each patient.
Medical professionals should be able to review patient history, filtering on time range and vital signs.

**Data Analysis and Alerting:**
The system should analyze each patient's vital signs for abnormalities or preset thresholds.
It should alert medical professionals if it detects issues such as a decrease in oxygen level or reaching preset thresholds like a temperature of 104<sup>o</sup> F.
Alerts should be timely and configurable to meet the needs of medical staff.

**User Interface and Interaction:**
MonitorMe should provide an intuitive user interface for medical professionals to interact with the system.
It should allow medical staff to configure alert thresholds, view patient data, and access historical records.

## 2.2  Additional Requirements

**Fault Tolerance and High Availability:**
MonitorMe should remain operational even if individual vital sign devices or components fail.
It should implement high-availability mechanisms to minimize downtime and ensure continuous monitoring and alerting services.

**Security:**
Although not required in the first iteration, MonitorMe should meet high data security and privacy standards in the future.
It should ensure user authentication and authorization to maintain data security and privacy.
It should ensure secure data exchange between different components.

**Scalability:**
The system should be scalable to accommodate increasing number of patients and vital sign monitoring devices.
It should handle peak loads during periods of high patient activity without degradation in performance.

**Integration Capabilities:**
Support integration with existing healthcare systems and other medical software solutions.
Provide APIs or interfaces for interoperability with third-party systems and devices.


## 2.3  System Requirements
> [!NOTE]
> TBD
* Data acquisition may involve protocols like Bluetooth, Wi-Fi, or specialized medical device interfaces to communicate with the monitoring equipment.
* 

## 2.4  Users

| User Role  | Actions |
| ------------- | ------------- |
| Nurse | - Add patient<br>- Update patient<br>- Remove patient<br>- View specific patient<br>- View room status<br>- View sensor status<br>- View all patients<br>- Acknowledge notification<br>- Setup notification |
| Medical Professional | - Generate report<br>- Generate patient history |
| Doctor | - Acknowledge notification<br>- Setup notification  |
| Technical System Admin | - Review hardware status<br>- Update software version<br>- Review database health<br>- Review uptime |

## 2.5  Constraints and Assumptions
> [!NOTE]
> TBD

# 3.  Solution Space
In this section, we describe user personas and usage pattern analysis, which we used to help us make specific architectural choices. Notably, these analyses guided the particular components in our architecture. Subsequently, we specify the priority of order of architectural characteristics, followed by the proposed architectural style.

## 3.1  User Persona Analysis
It is essential to identify the needs of users who will directly use the MonitorMe system. We have come up with user personas early in architecture development so that they can guide a product that will serve users' best interests.

### Nancy, the Nurse

Nancy is a nurse who works with patients in her station. Her job consists of regular rounds with her patients and monitoring various situations that she can either act on or report to others for guidance.

Her day is spent walking between the rooms, and her time is at a premium. She prefers anything that can save her a trip to a room because by being with one patient, she can't quickly check on the others. But even with that, she must visit each room multiple times daily. So, her time at the Nurses' Station is at a premium because it won't be long until she has to head to a patient.

She understands the operation of a hospital room, so she values information that helps her do her job by giving her information she might only sometimes have.

### David, the Doctor

David works with many patients at many hospitals, so he is rarely at one location for long. He has to get in, work, and head off to another patient. Often, if he is a surgeon or other specialist, his time is spent practicing his procedures and is frequently uninterruptible.

Because he is often a contract-type worker at the hospital, his methods have to cross hospital systems, and he has to be well-versed in different configurations. David has to leave instructions for the day-to-day of his patients, and he comes back either for scheduled times or emergencies.

### Alice, the Nurse Administrator

Alice could be a nurse herself, but often, she is the one helping coordinate bringing patients in/out of different hospital sections. As a result, she is the one coordinating patient entry & leaving. She needs a global picture to make her decisions as possible.

Her work is done around the edges and usually is not horribly time-critical unless occupancy or staffing issues exist. When a patient arrives or leaves, usually, that doesn't require immediate attention.

### Sarah, the System Administrator

Sarah is responsible for the understanding and upkeep of all of the hospital systems. Since the patient's well-being is involved, she must be 1) available at a moment's notice to troubleshoot and 2) keep the systems working all the time. Her _five-nines_ goal means she is constantly trying to stay ahead of issues and perform maintenance at times that will not stress out the function of systems. She could be notified in a disaster, but she'd prefer to have that situation covered ahead of time and not have to scramble to bring a system back up.

## 3.2  Usage Patterns
> [!NOTE]
> TBD

## 3.3  Architecture Characteristics
### Availability

<ins>Reason:</ins>
As MonitorMe's primary purpose is to monitor patients' vital signs in real time, any system downtime could delay the detection of critical health issues, leading to potential harm or even fatalities for patients. High availability is also needed to ensure that alerts for abnormal conditions are delivered promptly, enabling healthcare providers to respond quickly in emergencies.

<ins>Impact on Architecture:</ins>
* We have added provision for load balancing in the data ingestion module.
* We have adopted Extract-Load-Transform (ELT) architecture.

### Data Integrity

<ins>Reason:</ins>
Healthcare decisions and diagnoses rely on accurate and reliable patient data. Therefore, MonitorMe needs high data integrity, meaning the data across the system must be free from incorrect modification and loss.

<ins>Impact on Architecture:</ins>
* We have adopted a central database that adheres to ACID properties.

### Data Consistency

<ins>Reason:</ins>
The MonitorMe system must also ensure that the vital sign readings ingested, stored, and displayed reflect the current state of the patient's health. Such consistency is also necessary for informed decision-making and patient safety.

<ins>Impact on Architecture:</ins>
* We have adopted a central database that adheres to ACID properties.

### Fault Tolerance

<ins>Reason:</ins>
The MonitorMe system should maintain service while facing failures. The primary failure scenario is when one or more vital sign devices or software components fail. It is essential for the MonitorMe system to still function for monitoring, recording, analyzing, and alerting based on the available data.

<ins>Impact on Architecture:</ins>
* We store and process each vital sign timeseries independently of other vital sign signals.
* We have designed for the ability to detect vital sign failures and the ability to alert a data/system administrator about the failures.
* We have also incorporated an ability to seamlessly ingest data after a failed component recovers, which essentially uses the ELT architecture.

> Note that for MonitorMe to tolerate failures of other hardware and software components, it must consider redundancy and replication at various levels of the system. This requirement could include deploying multiple instances of servers, databases, and other components across different physical locations or availability zones. However, these considerations have little impact on the software architecture. Therefore, we leave the discussion of such deployment considerations to a subsequent section of this document.

### Concurrency

<ins>Reason:</ins>
MonitorMe needs to read, store, and process data from multiple monitoring sources across multiple patients. Therefore, it is necessary to design components to handle concurrent operations and improve performance.

<ins>Impact on Architecture:</ins>
* We have adopted a microservice architecture for data ingestion and analysis components.

### Performance

<ins>Reason:</ins>
The system should process both on-demand and continuous requests very quickly. Specifically, as the problem statement states, MonitorMe should send "the data to a consolidated monitoring screen (per nurses station) with an average response time of 1 second or less".

<ins>Impact on Architecture:</ins>
* We have chosen a central database.
* We have adopted interactions to be asynchronous where a response is not needed.
* We have designed the most process intesive modules to handle concurrent requests.

> Given the Katas Challenge's objectives and our assumptions (discussed in previous sections), we have decided to deprioritize interoperability, responsiveness, and scalability in the first iteration of MonitorMe.


## 3.4  Architecture Style
We recommend **a combination of microservice and event-driven architecture styles**.
* Microservice architecture will allow keeping services of the system discrete, enabling fault tolerance and high availability.
* Event-driven architecture will enable real-time capabilities. Various components can subscribe to events and receive them as aynchronous messages. For instance, when vital sign data for a patient is ready for display at a nurse station, it can be delivered to output generation modules via an asynchronous message, allowing a non-blocking output handling.
* In order to meet the data consistency requirement and minimize data sharing among various microservices, we adopt shared special-purpose databases instead of a separate per-microservice database. These databases will store patient monitoring data, rules on medical readings, and alerts. The shared database style is suitable because MonitorMe needs to prioritize data integrity and consistency over data isolation and high scalability.

# 4.  System Architecture: Components

![Comonent Diagram.](/images/ComponentsDiagram.png)

Based on the [user persona](#31--user-persona-analysis) and [usage patterns](#32--usage-patterns) analyses, we have broken system architecture into four high-level components:
1. **Data Acquisition**:
     - Interfaces with the various monitoring devices to retrieve real-time data on vital signs.
     - Ensures reliable and continuous data acquisition from all sources, handling potential errors or disruptions gracefully.
2. **Data Processing**:
     - Receives the raw vital sign data collected by the data acquisition component, stores, and processes it for further analysis.
     - Includes tasks such as data normalization, validation, aggregation, and analysis to derive meaningful insights from the raw data.
     - Performs real-time analysis to detect abnormalities or threshold violations in vital sign readings, triggering alerts for medical professionals.
3. **Output Generation**:
     - Responsible for presenting the processed vital sign data to various output receivers in a user-friendly format.
     - Includes functionalities such as displaying vital signs on monitoring screens at nurses' stations, generating historical reports, and sending alerts to medical staff.
4. **Data Administration**:
     - Enter metadata such as thresholds and rules for generating alerts on vital sign signals.
     - Generate on-demand data snapshots and reports.

# 5.  System Architecture: Data Flow
![Data Flow Diagram.](/images/DataFlowDiagram.png)

## 5.1  Data Flow

1. Sensors produce data for their given type (HR, temp, etc.)
2. Sensors are physically connected to an in-room hub that displays data at the bedside, but they also have the ability to send sensor data along. However, they tag the sensor data with their hub ID and send it along. This way, anytime the hub receives updated data, it sends it along.
3. The receiving end has a load balancing setup with redundancy such that the data is always received in a timely manner and one connection will not take the data down.
4. Ingestion pool takes the sensor data and applies a canonical timestamp (and any other information we might think is non-mutable) and puts the data into the sensor database, with one entry per sensor, containing the timestamp and the hub ID.
5. A pool of rule monitors is watching the sensor database, querying repeatedly based on all of the rules in the rule database. When a complete new set of rule data is found, it creates a rule task.
6. Either on demand or from a pool, a rule alert processor takes the task and sees if the current sensor data triggers the rule. If it does, it triggers an alert.
7. Notification issuer is watching alert database. When one comes in, it looks up rule, gets notification and UI/text information and creates the needed output to cause events (fires data to nurses station, triggers notification on screen, etc.)
8. NOTE: under this setup, it is possible a rule could be created to just update the Coordinated Station screen.

## 5.2  Microservice Descriptions

**Patient Alert Monitor:**
- Monitors raw sensor pool to compare incoming data with existing rules, looking to check completely the set of sensor data for a given rule/patient. If a given rule for a patient has a changed set of sensor data, a task is created to verify the rule.

**Alert Rule Processor:**
- The processor takes a set of sensor data for a given patient and a rule and does the processing to determine if the rule is triggered. If it is, it generates an alert and puts it into the alert queue/database.

**Notification Monitors:**
- The monitor looks at a given alert, looks up the notification rules, and triggers the events based on the setup (i.e., send a message to the Consolidated Nurse Screen as well as a message to the StayHealthy app for a given doctor).


## 5.3  Considered Data Structures
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


# 6.  Detailed Architecture
## 6.1  Sensor Input
**Objective:**
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
| ECG | 1 second | depends on size, but approximately<br>`(5-10 sensors) * (update hz) * (each lead data field)` bytes | xml |

**The Data Ingestor microservice** accepts the sensor data from the In-Room Hub and adds some metadata, namely a canonical timestamp (to allow for comparison between all datasets) and the patient ID, looked up from the Patient->Room->Hub table. The Data Ingestor could pull the data from a queue and scale out in case the data needs/sizes come in faster.

The data is then pushed into the Vital-Sign Data Storage to be used by further steps.

## 6.2  Vital-Sign Data Model
### Schema

<p align="center"> <ins>Figure: Time Series Data Model</ins> </p>

![Vital Sign Data Model](/images/vital_sign_data_model.png)


As depicted in the schema diagram above, the vital signs are stored in a time series database using *tags* representing different sensor types. Specifically, a record in this database is comprised of one tag name, multiple data points, and multiple attributes. A data point constitutes a timestamp and measurement. Finally, the attributes are key-value pairs describing various sensor-specific details. Note that some sensors, like the ECG, are more complex than others and need additional attributes.

<p align="center"> <ins>Table: Time Series Data Schema</ins></p>

| Term | Required | Definition |
| ------------- | ---------- | ---------- |
| Tag | x | Name of a tag (e.g., temperature), which is non-unique. Howerver, a specific sensor's ID is stored as an attribute in the Attributes section. |
| Timestamp | x | The date and time in UNIX epoch time, in millisecond precision. |
| Measurement| x | The value produced by the sensor without a unit of measurement. |
|Attribute| | Key/Value pairs used to store data associated with the tag. Some mandatory ones are: hub ID, room ID, patient ID, and sensor ID. Other optional ones might be needed for more complex sensor types, such as ECG. |

  
### Supported Queries
The database will provide REST APIs for querying and aggregating time series data. Such query include:
- Read data for specific tags within a desired time window
- Filter data by attribute values
- Retrieve all available tags and their associated attribute keys
- Add aggregate values
- Interpolate data points in a given time window
- Read the latest values for a specific sensor type


## 6.3  Transformation
**Objective:** Transform stored monitoring data for display at nurse screens as continuous streams.

**Inputs:**

| Input | Retrieval Process |
| ------------- | ------------- |
| All nurse-screen IDs | Event-driven: subscribe to an event that is generated every time there is a change in active nurse screens in the database  |
| All patient IDs for a specific screen | Database query: a time-based query decided by its business logic |

**Processing available nurse-screens:**
* This module keeps a local copy of active nurse screen IDs, expected to change infrequently. The transformation module runs concurrent threads to independently read and transform data for each screen.
* Any change to the set of active nurse screens should be notified to this module. On such a notification, all the nurse-screen threads are restarted.

**Data transformation logic:**
* The primary goal of data transformation is to serve the most recent data to the output generation modules. For this reason, the transformation logic iterates over all patients associated with a specific screen to read and pass data onto the output generation modules.
* The response time of processing a patient's data is on the order of sub-second. At the same time, a given patient's data is displayed for five seconds. Therefore, it is sufficient to process all patients sequentially and even more suitable for providing the most recent data on screen.


![Sequence Diagram of the Transformation Component.](/images/Seq-Diagram-Transformation.jpg)

## 6.4  Filtering
> [!NOTE]
> TBD

## 6.5  Analysis
The analysis is split into two main microservices, utilizing the Rule and Vital-Sign data to create alerts when applicable:

**Patient Rule Monitor:**
Monitors Vital-Sign data to compare incoming data with existing rules, looking to check completely the set of sensor data for a given rule/patient. If a given rule for a patient has a changed set of sensor data, a task is created to verify the rule. The amount of microservice instances could be sliced any number of ways. First, there could literally be one per rule. Second, there could be one per patient. In either case, they'd need to verify both the patient rule and the applicable Vital-Sign data for that patient, accessing both data sets.

**Rule Alert Processor:**
Once a rule is determined to be checked, the Rule Alert Processor will analyze the data & the rule given, determining if conditions are met to trigger an alert. Once a rule is triggered, an alert is added to the Alert table, allowing further services to act on the alert. In addition, due to the complexity of the ECG data, this calculation could require some standalone processing vs merely a threshhold check. That's the reason to have these services in a pool or triggered per rule calculation.

In addition, should UX determine alerts need to be deduped or checked for confirmation, that logic would also be added to the Rule Alert Processor.

## 6.6  Alert Storage
> [!NOTE]
> TBD

## 6.7  Output Generation
> [!NOTE]
> TBD

# 7.  Architecture Decision Records
1. [Sensor Hub ADR](/adr/sensor_hub.md)
2. [Database Choice ADR](/adr/database_choice.md)
