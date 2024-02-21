# 1 Welcome to MightyOrbots
Repository for NP's O'Reilly 2024 Architectural Kata Challenge

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
It should alert medical professionals if it detects issues such as a decrease in oxygen level or reaching preset thresholds like a temperature of 104 degrees F.
Alerts should be timely and configurable to meet the needs of medical staff.

**Fault Tolerance and High Availability:**
MonitorMe should remain operational even if individual vital sign devices or components fail.
It should implement high-availability mechanisms to minimize downtime and ensure continuous monitoring and alerting services.


## 2.2 Additional Requirements
**User Interface and Interaction:**
MonitorMe should provide an intuitive user interface for medical professionals to interact with the system.
It should allow medical staff to configure alert thresholds, view patient data, and access historical records.

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
TBD

## 2.4 Users
TBD

## 2.5 Constraints and Assumptions
TBD

# 3 Solution Space
## 3.1 Identifying Components through User Scenarios
TBD

## 3.2 Architecture Charactersitics
TBD

## 3.3 Architecture Style
TBD
