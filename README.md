# 1 Welcome to MightyOrbots
Repository MightyOrbots' solution to O'Reilly 2024 Architectural Kata Challenge.

# 2 Problem Space
## 2.1 Functional Requirements
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

## 2.2 Additional Requirements

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


## 2.3 System Requirements
> [!NOTE]
> TBD
* Data acquisition may involve protocols like Bluetooth, Wi-Fi, or specialized medical device interfaces to communicate with the monitoring equipment.
* 

## 2.4 Users

| User Role  | Actions |
| ------------- | ------------- |
| Nurse | - Add patient<br>- Update patient<br>- Remove patient<br>- View specific patient<br>- View room status<br>- View sensor status<br>- View all patients<br>- Acknowledge notification<br>- Setup notification |
| Medical Professional | -Generate report<br>- Generate patient history |
| Doctor | - Acknowledge notification<br>- Setup notification  |
| Technical System Admin | -Review hardware status<br>- Update software version<br>- Review database health<br>- Review uptime |

## 2.5 Constraints and Assumptions
> [!NOTE]
> TBD

# 3 Solution Space
In this section, we describe user personas and usage pattern analysis, which we used to help us make specific architectural choices. Notably, these analyses guided the particular components in our architecture. Subsequently, we specify the priority of order of architectural characteristics, followed by the proposed architectural style.

## 3.1 User Persona Analysis
> [!NOTE]
> TBD

## 3.2 Usage Patterns
> [!NOTE]
> TBD

## 3.2 Architecture Charactersitics
**Availability**

<ins>Reason:</ins>
As MonitorMe's primary purpose is to monitor patients' vital signs in real time, any system downtime could delay the detection of critical health issues, leading to potential harm or even fatalities for patients. High availability is also needed to ensure that alerts for abnormal conditions are delivered promptly, enabling healthcare providers to respond quickly in emergencies.

<ins>Impact on Architecture:</ins>
* We have added provision for load balancing in the data ingestion module.
* We have adopted Extract-Load-Transform (ELT) architecture.

**Data Integrity**

<ins>Reason:</ins>
Healthcare decisions and diagnoses rely on accurate and reliable patient data. Therefore, MonitorMe needs high data integrity, meaning the data across the system must be free from incorrect modification and loss.

<ins>Impact on Architecture:</ins>
* We have adopted a central database that adheres to ACID properties.

**Data Consistency**

<ins>Reason:</ins>
The MonitorMe system must also ensure that the vital sign readings ingested, stored, and displayed reflect the current state of the patient's health. Such consistency is also necessary for informed decision-making and patient safety.

<ins>Impact on Architecture:</ins>
* We have adopted a central database that adheres to ACID properties.

**Fault Tolerance**

<ins>Reason:</ins>
The MonitorMe system should maintain service while facing failures. The primary failure scenario is when one or more vital sign devices or software components fail. It is essential for the MonitorMe system to still function for monitoring, recording, analyzing, and alerting based on the available data.

<ins>Impact on Architecture:</ins>
* We store and process each vital sign timeseries independently of other vital sign signals.
* We have designed for the ability to detect vital sign failures and the ability to alert a data/system administrator about the failures.
* We have also incorporated an ability to seamlessly ingest data after a failed component recovers, which essentially uses the ELT architecture.

> Note that for MonitorMe to tolerate failures of other hardware and software components, it must consider redundancy and replication at various levels of the system. This requirement could include deploying multiple instances of servers, databases, and other components across different physical locations or availability zones. However, these considerations have little impact on the software architecture. Therefore, we leave the discussion of such deployment considerations to a subsequent section of this document.

**Concurrency**

<ins>Reason:</ins>
MonitorMe needs to read, store, and process data from multiple monitoring sources across multiple patients. Therefore, it is necessary to design components to handle concurrent operations and improve performance.

<ins>Impact on Architecture:</ins>
* We have adopted a microservice architecture for data ingestion and analysis components.

**Performance**

<ins>Reason:</ins>
The system should process both on-demand and continuous requests very quickly. Specifically, as the problem statement states, MonitorMe should send "the data to a consolidated monitoring screen (per nurses station) with an average response time of 1 second or less".

<ins>Impact on Architecture:</ins>
* We have chosen a central database.
* We have adopted interactions to be asynchronous where a response is not needed.
* We have designed the most process intesive modules to handle concurrent requests.

> Given the Katas Challenge's objectives and our assumptions (discussed in previous sections), we have decided to deprioritize interoperability, responsiveness, and scalability in the first iteration of MonitorMe.


## 3.3 Architecture Style
We recommend a combination of microservice and event-driven architecture styles.
* Microservice architecture will allow keeping services of the system discrete, enabling fault tolerance and high availability.
* Event-driven architecture will enable real-time capabilities. Various components can subscribe to events and receive them as aynchronous messages. For instance, when vital sign data for a patient is ready for display at a nurse station, it can be delivered to output generation modules via an asynchronous message, allowing a non-blocking output handling.
* In order to meet the data consistency requirement and minimize data sharing among various microservices, we adopt a single shared database to store patient monitoring data. The central database is also suitable because MonitorMe does not need data isolation or very high scalability.

# 4 System Architecture
Based on the [user persona](./README.md#31-user-persona-analysis) and [usage patterns](./README.md#32-usage-patterns) analyses, we have broken system architecture into four high-level components:
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


![Comonent Diagram.](/images/Components-Diagram.jpg)

# 5 Detailed Architecture
## 5.1 Sensor Input
> [!NOTE]
> TBD

## 5.2 Vital-Sign Data Storage
> [!NOTE]
> TBD

## 5.3 Transformation
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

## 5.4 Filtering
> [!NOTE]
> TBD

## 5.5 Analysis
> [!NOTE]
> TBD

## 5.6 Alert Storage
> [!NOTE]
> TBD

## 5.7 Output Generation
> [!NOTE]
> TBD

# 6 Architecture Decision Records
> [!NOTE]
> TBD
