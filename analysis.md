## Analysis

The analysis is split into two main microservices, utilizing the Rule and Vital-Sign data to create alerts when applicable:

Patient Rule Monitor:
Monitors Vital-Sign data to compare incoming data with existing rules, looking to check completely the set of sensor data for a given rule/patient. If a given rule for a patient has a changed set of sensor data, a task is created to verify the rule. The amount of microservice instances could be sliced any number of ways. First, there could literally be one per rule. Second, there could be one per patient. In either case, they'd need to verify both the patient rule and the applicable Vital-Sign data for that patient, accessing both data sets.

Rule Alert Processor:
Once a rule is determined to be checked, the Rule Alert Processor will analyze the data & the rule given, determining if conditions are met to trigger an alert. Once a rule is triggered, an alert is added to the Alert table, allowing further services to act on the alert.

In addition, should UX determine alerts need to be deduped or checked for confirmation, that logic would also be added to the Rule Alert Processor.
