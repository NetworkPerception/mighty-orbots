# Table of Contents
- [1.  Welcome to MightyOrbots](#1--welcome-to-mightyorbots)
- [2.  Problem Space](#2--problem-space)
   - [2.1 Functional Requirements](#21--functional-requirements)
   - [2.2 Additional Requirements](#22--additional-requirements)
   - [2.3 Users and Roles](#23--users-and-roles)
   - [2.4 Constraints and Assumption](#24--constraints-and-assumptions)
- [3.  Solution Space](#3--solution-space)
   - [3.1 User Persona Analysis](#31--user-persona-analysis)
   - [3.2 Usage Patterns](#32--usage-patterns)
   - [3.3 Architecture Characteristics](#33--architecture-characteristics)
   - [3.4 Architecture Style](#34--architecture-style)
- [4.  System Architecture: Components](#4--system-architecture-components)
- [5.  System Architecture: Data Flow](#5--system-architecture-data-flow)
   - [5.1 Data Flow](#51--data-flow)
   - [5.2 Microservice Descriptions](#52--microservice-descriptions)
   - [5.3 Considered Data Structures](#53--considered-data-structures)
- [6.  Detailed Architecture](#6--detailed-architecture)
   - [6.1 Sensor Input](#61--sensor-input)
   - [6.2 Vital Sign Data Model](#62--vital-sign-data-model)
   - [6.3 Transformation](#63--transformation)
   - [6.4 Analysis](#64--analysis)
   - [6.5 Rules Processing](#65--rules-processing)
   - [6.6 Output Generation](#66--output-generation)
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

## 2.3  Users and Roles

<p align="center"> <ins><strong>Table: Users and Roles</strong></ins></p>

| User Role  | Actions |
| ------------- | ------------- |
| Nurse | - Add patient<br>- Update patient<br>- Remove patient<br>- View specific patient<br>- View room status<br>- View sensor status<br>- View all patients<br>- Acknowledge notification<br>- Setup notification |
| Medical Professional | - Generate report<br>- Generate patient history |
| Doctor | - Acknowledge notification<br>- Setup notification  |
| Technical System Admin | - Review hardware status<br>- Update software version<br>- Review database health<br>- Review uptime |

## 2.4  Constraints and Assumptions
* We have developed a fault-tolerant and high-availability architecture. At the same time, fully realizing these two properties would require special deployment considerations. For instance, redundancy mechanisms, such as active-active or active-passive configurations, providing failover capabilities to maintain service availability in case of component failures, and business continuity strategies using response and recovery are necessary. However, they are out of scope for the presented architecture.
* Our architecture has put patient safety at the forefront, which we have realized through data integrity and performance. Additionally, the desired scale of users and data is not in the same order as the Internet scale. These facts and assumptions have led us to deprioritize scalability compared to patient safety.
We have assumed that the monitoring terminals used by nurses (e.g., for performing real-time tasks) and those used by medical professionals (e.g., for reviewing patient history) will be different.
* Although we have briefly discussed system administration, it is not the central topic in our architecture. System administration includes reviewing software and hardware health, performing updates, and backing up data.
* We have assumed that certain data records related to patient, nurse, and medical professional registration will be available to our system through hospital management systems. We have provided specific descriptions of such data records and related assumptions in later sections of this document.

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

When we started designing our solution, we found it important to thoroughly understand how this system would be used. We invested some time to mock up basic UI wireframes. These gave us tangible images that helped us delineate the system's inputs and outputs, clarify data flows, and weigh the impacts of our architecture decisions. Thinking thoroughly about the usage patterns led us to some surprising architecture choices. 

### Monitoring Screen

We started by imagining the consolidated monitoring screen that is located at each nurse's station. As we designed this image, a few key requirements to our data organization became illuminated. 
* A **patient** and a **bed** are two distinct data types. The bed represents a physical device that the sensors get plugged into, so we can also call it the in-room **hub**.
* It was important that a nurse can see the status of each sensor, and the status of one must not affect the availability of any other. This would influence us towards a **microservice** architecture.
* The alerts must be dismissible, to keep the screen clear. We also felt that a nurse should be able to manually notify a doctor (in addition to automatic notifications). But this got us thinking, what if a nurse hit the wrong button by accident? What if someone wanted to see a previous alert? We realized we needed to keep a **notification history**
* This screen is quite busy already, and we still have a lot of functionality to support. We decided that the key functionality of this screen is that nurses to get current, helpful information *at a glance*.

![Consolidated Monitoring Screen At Nurses Station.](/images/ui/0-Main_nurses_screen.png)


## Adding the Admin Screen

It is vital that the monitoring screen stays clear and easy to read. But we still have a lot of requirements support: viewing history, setting thresholds, and managing patients. We realized that we needed a second screen at the nurse's station.

With this decision, we were able to start organizing our architecture. We have several requirements that need user input, and we were having a hard time envisioning this in our components diagram. By splitting the screen, we can treat the monitoring screen as a read-only display of data. The new admin screen fields user inputs and pushes them for analysis. We can define the "data administration" component as an input into the analysis component. 

### Notification History

We saw the need for a notification history from the monitoring screen, and this is the next screen we created. We imagined some useful notifications, and found that they fell into two categories:
* administrative alerts, which indicate simple status updates: devices disconnecting, patient management
* threshold alerts, which are complex to compute, and involve multiple reads from the sensor data

This helped us figure out a key requirement for the sensor database. It is critical that nothing impedes the sensors from writing to the sensor input database. We decided to create a separate **alerts database**. We felt that if both sensor input and notifications are trying to write to a database, we could lose data consistency.

Splitting the database allows us to silo the sensor input database to its core purpose: serving many concurrent writes, without a break. All other functionality reads from here, processes the data, and sends it along. This split refined the design of the overall architecture. We created a clear delineation between the sensor database, and the analysis that comes after. 

![Notification History on Admin Screen.](/images/ui/1-admin_screen_notification_history.png)


### Setting Thresholds

After seeing the different notifications,  we started thinking about the functionality required to set thresholds, and the computation needed to generate notifications. We tried to come up with a flexible way to set up thresholds, and called them **rules**. 

Each patient has their own set of rules (there can be a default set). One patient can have many rules, and each patient may have a different set. We realized that there can be thousands of rules at play. We also realized that this is a core functionality that drives the rest of the system forward. We decided to store rules away from sensor input, adding a new table to the alerts database. We decided to create a **rule alert processor**. This processor is multi-threaded and able to run many rule calculations in parallel. 

For us, the idea of rules as a data structure was very powerful. This rule processor is constantly running, ingesting sensor data and creating notifications. We even thought of the screen update to the nurse's monitoring screen as a "notification" created by a rule (When the sensor data has new data, update the screen).
We realized this is the core of notifying the rest of the system of changes. This led us to accept an **event-driven architecture** for this portion of the system. 

> [!NOTE]
> The image shows rules based on one sensor at a time, but we will actually need rules that involve multiple sensors. 
> For example:  Alert the Doctor when heart rate is below 60 bpm, and the patient is not asleep

![Notification Settings.](/images/ui/5-admin_screen_patient_thresholds.png)


### Patients and Beds

We decided that a nurse should be able to add or remove a patient. We imagined that a nurse can get a patient's ID by scanning a hospital bracelet, and assigning them to a bed from this screen. 

A medical professional can create and upload a patient snapshot from here, as well as link to view the patient's history in the 24-Hour History screen. 

While creating these frameworks, we realized it was intuitive to refer to bed numbers in the UI. It is a physical location for a nurse to walk to, and it is easy to read on the screen. It also makes sense with the data: sensors are assigned to beds, and typically stay with those beds. Patients are also assigned to beds. If a patient needs to move to a new bed, we can update that from this screen. 

> [!NOTE]
>  We refer to **beds** as "in-room **hubs**" elsewhere in the architecture. We assume a hardware hub is kept with a bed, and **bed number** can be thought of as **hub ID.**

Seeing this usage pattern, we made an important architecture decision. Initially, we had considered giving a separate database system for each nurse's station. We considered the scenario where an existing patient moves to a different bed, and realized that the patient might be moved to a different nurse's station. So, we need a central database system, to aggregate a patient's history across any bed they had, and share settings across all stations. 

![Manage Patients.](/images/ui/2-admin_screen_manage_patients.png)

### Beds and Devices

> [!NOTE]
>  "Device" is another name for "sensor"

We decided that it will be useful to display the status of all of the sensors that belong to the nurses station. We wanted a screen that communicated both the location of a device, and its availability. From this screen a nurse can see when a device is online, assigned to an unoccupied bed, not assigned, or malfunctioning. 

Devices are assigned to a bed, and  incoming sensor data is tagged by the bed number. If we keep track of the patients assigned to the bed, we can associate the patient to the sensor data. 
Since we chose an ELT data pipeline, we decided that linking sensor data to a patient is a transformation that happens in analysis. 

![Manage Devices.](/images/ui/3-admin_screen_manage_linked_devices.png)


### Patient History

By the time we got to creating the Patient History wireframe, we had made a lot of important decisions. This usage pattern fit well with the architecture and data flow we designed. The analysis can associate patients to sensor data, and the rules can filter by time range, or vital sign.

However, this did help us refine our components diagram.
This usage pattern is outside of the event-driven flow that generates output. A doctor or nurse can initiate queries to get aggregated sensor data for their patients. Unlike the notification or screen updates, this is a pull request initiated by the user. 

Unlike the unidirectional flow of data that serves most of the output generation, we need to indicate a component that has a bidirectional flow of data. We called this portion of the architecture the "data administration" and initially pictured it as an input to the analysis. Thinking through this usage pattern, we realized that the component was both input and output. 

![Patient History.](/images/ui/7-admin_screen_view_history.png)


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
* We have adopted shared databases that adhere to ACID properties.

### Data Consistency

<ins>Reason:</ins>
The MonitorMe system must also ensure that the vital sign readings ingested, stored, and displayed reflect the current state of the patient's health. Such consistency is also necessary for informed decision-making and patient safety.

<ins>Impact on Architecture:</ins>
* We have adopted shared databases that adhere to ACID properties.

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

<p align="center"><ins><strong>Figure: Components Diagram</strong></ins></p>

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
     - Includes functionalities such as displaying vital signs on monitoring screens at nurses' stations and sending alerts to medical staff.
4. **Data Administration**:
     - Enter metadata such as thresholds and rules for generating alerts on vital sign signals.
     - Generate on-demand data snapshots and reports.
     - Generate historical reports.

# 5.  System Architecture: Data Flow

<p align="center"><ins><strong>Figure: Data Flow Diagram</strong></ins></p>

![Data Flow Diagram.](/images/DataFlowDiagram.png)

## 5.1  Data Flow

1. Sensors produce data for their given type (HR, temp, etc.)
2. Sensors are physically connected to an in-room hub that displays data at the bedside, but they also have the ability to send sensor data along. However, they tag the sensor data with their hub ID and send it along. This way, anytime the hub receives updated data, it sends it along.
3. The receiving end has a load balancing setup with redundancy such that the data is always received in a timely manner and one connection will not take the data down.
4. Ingestion pool takes the sensor data and applies a canonical timestamp (and any other information we might think is non-mutable) and puts the data into the sensor database, with one entry per sensor, containing the timestamp and the hub ID.
5. A pool of rule monitors is watching the sensor database, querying repeatedly based on all of the rules in the rule database. When a complete new set of rule data is found, it creates a rule task.
6. Either on demand or from a pool, a rule alert processor takes the task and sees if the current sensor data triggers the rule. If it does, it triggers an alert.
7. Notification issuer is watching alert database. When one comes in, it looks up rule, gets notification and UI/text information and creates the needed output to cause events (fires data to nurses station, triggers notification on screen, etc.)

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

<p align="center"> <ins><strong>Table: Sensor Input Structure</strong></ins></p>

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

## 6.2  Vital Sign Data Model
### Schema

<p align="center"> <ins><strong>Figure: Time Series Data Model</strong></ins> </p>

![Vital Sign Data Model](/images/vital_sign_data_model.png)


As depicted in the schema diagram above, the vital signs are stored in a time series database using *tags* representing different sensor types. Specifically, a record in this database is comprised of one tag name, multiple data points, and multiple attributes. A data point constitutes a timestamp and measurement. Finally, the attributes are key-value pairs describing various sensor-specific details. Note that some sensors, like the ECG, are more complex than others and need additional attributes.

<p align="center"> <ins><strong>Table: Time Series Data Schema</strong></ins></p>

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

<p align="center"> <ins><strong>Figure: Sequence Diagram of the Transformation Component</strong></ins></p>

![Sequence Diagram of the Transformation Component.](/images/Seq-Diagram-Transformation.jpg)

**Objective:** Transform stored monitoring data for display at nurse screens as continuous streams.

**Processing available monitoring screens:**
* This module keeps a local copy of active monitoring screen IDs. This data is expected to change infrequently and is read in an event-driven manner: the module subscribes to an event that is generated every time there is a change in active nurse screens in the database.
* The transformation module runs concurrent threads to read and transform data for each screen independently.
* Any change to the set of active monitoring screens should be notified to this module. On such a notification, all the nurse-screen threads are restarted.

**Data transformation logic:**
* The primary goal of data transformation is to serve the most recent data to the output generation modules. For this reason, the transformation logic iterates over all patients associated with a specific screen to read and pass data onto the output generation modules.
* The patient IDs for a specific monitoring screen are retrieved using a time-based query decided by its business logic.
* The response time of processing a patient's data is in the order of sub-seconds. At the same time, a given patient's data is displayed for five seconds. Therefore, it is sufficient to process all patients sequentially and even more suitable for providing the most recent data on screen.


## 6.4  Analysis
The analysis is split into two main microservices, utilizing the rule and vital-sign data to create alerts when applicable.

**Patient Rule Monitor:**
Monitors vital-sign data to compare incoming data with existing rules, looking to check completely the set of sensor data for a given rule/patient. If a given rule for a patient has a changed set of sensor data, a task is created to verify the rule. The amount of microservice instances could be sliced any number of ways. First, there could literally be one per rule. Second, there could be one per patient. In either case, they'd need to verify both the patient rule and the applicable vital-sign data for that patient, accessing both data sets.

**Rule Alert Processor:**
Once a rule is determined to be checked, the Rule Alert Processor will analyze the data & the rule given, determining if conditions are met to trigger an alert. Once a rule is triggered, an alert is added to the Alert table, allowing further services to act on the alert. In addition, due to the complexity of the ECG data, this calculation could require some standalone processing vs merely a threshhold check. That's the reason to have these services in a pool or triggered per rule calculation.

In addition, should UX determine alerts need to be deduped or checked for confirmation, that logic would also be added to the Rule Alert Processor.

## 6.5  Rules Processing
We have provisioned three types of rules on sensor readings that would trigger alerts. Each of these rules is also associated with the patient's sleep/awake status.

### State Trigger
The state trigger performs an alert when a sensor status changes. Three possible rules are available in a state trigger as depicted below:

![State Trigger](/images/rule_state.png)

**Note**: In addition to the sensor state, there might also be a message or image accompanying the change. This extra data, when available, will be attached to the alert sent.

| Rule Setting | Description|
|---|---|
|When sensor state is [...]|Select the status that triggers an alert: <li>**Down**: Trigger an alert if a sensor changes to the Down status.</li><li>**Warning**: Trigger an alert if a sensor changes to the Warning status.</li><li>**Unusual**: Trigger an alert if a sensor changes to the Unusual status.</li><li>**Up**: Trigger an alert if a sensor changes to the Up status.</li><li>**Unknown**: Trigger an alert if a sensor changes to the Unknown status</li>|
|for at least [...] seconds|Enter the time in seconds before the trigger sends an alert (latency). This can avoid false alarms if, for example, a sensor changes to the Down status for only a few seconds. Enter an integer. Do not define a latency that is shorter than the scanning interval of a sensor that uses this trigger.|
|perform [...]|Select the alert sent if the sensor is in the selected status and if the defined latency is over.|
|When sensor state is [...] for at least [...] seconds|Enter the escalation latency in seconds to wait before sending an escalation alert. Use this to escalate an alert automatically if a problem exists for a longer time. Enter an integer.|
|and repeat every [...] minute|Enter the interval in minutes after which send the escalation alert again. Enter an integer.|
|When the sensor state is no longer [...] , perform [...]|Select the alert to send if the sensor is no longer in the selected status and the defined latency is over. |



### Threshold Trigger
The threshold trigger performs an alert when a sensor reading reaches a specific value.

![Threshold Trigger](/images/rule_threshold.png)

| Rule Setting | Description|
|---|---|
|When [sensor data X]|Select the sensor data available for the type of sensor this trigger is associated with. Note: the exact name of the sensor data will be retrieved from the database. 
|is [...]|Select the condition that triggers the alert from the fixed values below: <li>**Above**: Trigger the alert if the value of the selected sensor data exceeds a defined value.</li><li>**Below**: Trigger the alert if the value of the selected sensor data falls below a defined value.</li><li>**Equal to**: Trigger the alert if the value of the selected sensor data is the same as a defined value.</li><li>**Not equal to**: Trigger the alert if the value of the selected sensor data is different from a defined value.</li>|
|[value]|Enter the value that compares the sensor data. |
|for at least [...] seconds|Enter the time in seconds to wait before sending an alert (latency). |
|perform [...]|Select the alert to send if the defined sensor data condition is true and the defined latency is over. |
|When the condition clears, perform [...]|Select the alert to send if the defined sensor data condition is no longer true and the defined latency is over. |


### Change Trigger
The change trigger performs an alert when a compatible sensor's value changes.

**Note**: The ECG sensor might be linked to a more elaborate system computing anomalies. These anomalies could be seen as a change triggering an alert.

![Change Trigger](/images/rule_change.png)


| Rule Setting | Description|
|---|---|
|When the sensor changes, perform [...]|Select the alert to send if a compatible sensor triggers a change alert. |

## 6.6  Output Generation
The system generates three types of outputs:

* alerts and notifications
* updates to the monitoring screen
* patient history and reports

> Unlike the first two, patient history is initiated by a user, and dependent on input from the user. These requests come from the admin screen at a nurse's station. We decided to consider this as part of the data adminitstration component. Let's consider notifications, and updates to the screen as the output generated. 

Alerts and updates are important events that serve the main functionality of the system. The screen is constantly updating, and alerts must reported as soon as possible. We decided to make this component in the architecture event-driven. As soon as newer sensor data is availiable, the monitoring screen will display it.

Consider how a notification makes it to the nurse's screen. The notification issuer, that is in charge of displaying the notification, should poll the alerts database, to retrieve the newest alerts added. The rule processor is in charge of applying the rules to sensor data, and adding alerts to the alerts database. The rule processor should poll the sensor input database, to retrieve the newest sensor data added. Each event is pushed by the process before it.


# 7.  Architecture Decision Records
1. [Sensor Hub ADR](/adr/sensor_hub.md)
2. [Database Choice ADR](/adr/database_choice.md)
3. [Event Driven Architecture ADR](/adr/event_driven_architecture.md)
4. The data administration component is an input to analysis ADRs
   4.1 [Superseded](/adr/data_admin_isinput.md)
   4.2 [Accepted](/adr/data_admin_isinput_and_output.md)
