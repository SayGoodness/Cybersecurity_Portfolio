# AWS Cloud Security & Forensic Investigation

## Objective
Design and implement a real-time AWS security monitoring and alerting system to detect unauthorized access to sensitive secrets, then build an automated active-defense mechanism to immediately revoke access from a compromised user. Two detection approaches were built and compared: a metric-based approach using CloudWatch, and an event-driven approach using EventBridge.

## Tools & Technologies
- AWS CloudTrail (multi-region)
- AWS Secrets Manager
- Amazon CloudWatch (Logs, Metric Filters, Alarms)
- Amazon EventBridge
- Amazon SNS
- Amazon S3 (log storage)
- AWS CLI
- IAM

## Architecture

**Conceptual flow:**
Sensitive Access → CloudTrail → Detection Logic → Alerting → Automated Response

**Flow 1 (CloudWatch):** CloudTrail → CloudWatch Logs → Metric Filter → Alarm → SNS Email

**Flow 2 (EventBridge):** CloudTrail → EventBridge Rule → SNS Email + Automated Kill-Switch

## What I Did

**Phase 1 — Infrastructure & Logging**
- Configured a multi-region AWS CloudTrail to capture all read and write management events, integrated with CloudWatch Logs for real-time detection
- Created a monitored honeytoken secret (`Database_Credentials`) in AWS Secrets Manager to trigger alerts on access
- Set up an S3 bucket with Server Access Logging enabled to store CloudTrail logs and support data residency requirements

**Phase 2 — Detection Logic (Flow 1: CloudWatch)**
- Built a CloudWatch Logs metric filter to detect `GetSecretValue` API calls targeting the honeytoken secret
- Configured a CloudWatch Alarm to trigger on any single access attempt (threshold ≥ 1 within a 1-minute period)
- Linked the alarm to an SNS topic with a confirmed email subscription for alert delivery

**Phase 3 — Detection Logic (Flow 2: EventBridge)**
- Created an EventBridge rule to detect sensitive data access events directly from CloudTrail, matching management API calls related to secret retrieval
- Configured a separate SNS topic for Flow 2 to independently evaluate detection speed and alert context against Flow 1
- Simulated a breach using the AWS CLI (`aws secretsmanager get-secret-value`) to trigger both detection flows

**Phase 4 — Active Defense (Kill-Switch)**
- Implemented an automated remediation mechanism triggered by EventBridge that disables the IAM access key of the offending user on detection, stripping all permissions
- Configured the EventBridge rule with dual targets (SNS + Kill-Switch) so detection and response happen within seconds
- Ran a trap test with a "Victim" IAM user (read-only Secrets Manager access): accessed the secret via CLI, triggered detection, and confirmed subsequent CLI commands failed due to revoked access

## Forensic Evidence
- CloudTrail log entry confirming the breach (event time, user identity, source IP)
- Follow-up CloudTrail log confirming remediation, showing an `AccessDenied` error on the Victim user's next command

## Screenshots
<img width="553" height="161" src="https://github.com/user-attachments/assets/37d21d23-0953-43ee-b966-ea5f6ea7f50d" />

*Fig 1: CloudTrail multi-region configuration*

<img width="554" height="159" src="https://github.com/user-attachments/assets/992a01ce-7376-4179-9e7d-a8f1fa3df9fb" />

*Fig 2: Secrets Manager honeytoken*

<img width="440" height="314" src="https://github.com/user-attachments/assets/4976b554-7e86-43e7-9165-81c384623ede" />

*Fig 3: CloudWatch metric filter*

<img width="553" height="177" src="https://github.com/user-attachments/assets/121e8e14-ada5-4e1c-829d-b34615c01fc0" />

*Fig 4: CloudWatch alarm configuration*

<img width="553" height="177" src="https://github.com/user-attachments/assets/ea398da3-fad2-4def-bcf5-debff0f7415c" />

*Fig 5: SNS topic with email subscription*

<img width="554" height="230" src="https://github.com/user-attachments/assets/7a9c3979-e2c2-4bfe-95f3-e226e8c77330" />

*Fig 6: EventBridge rule configuration*

<img width="554" height="240" src="https://github.com/user-attachments/assets/d611daf3-4eb8-4797-b78b-bb3834d2bd2b" />

*Fig 7: Kill-Switch implementation*

<img width="554" height="234" src="https://github.com/user-attachments/assets/026f986d-7295-4ca4-ab97-3c087d3cae22" />

*Fig 8: EventBridge dual targets*

<img width="554" height="225" src="https://github.com/user-attachments/assets/28d6b761-bca0-4dea-a6c4-3236bf366155" />

*Fig 9: IAM Victim user with revoked access*

<img width="536" height="231" src="https://github.com/user-attachments/assets/4472327f-5c9c-4b76-a684-7cba5af368da" />

*Fig 10: CloudTrail log showing GetSecretValue event*

<img width="553" height="128" src="https://github.com/user-attachments/assets/b00ef6ca-7024-4a62-a223-36534100c8c4" />
<img width="543" height="234" src="https://github.com/user-attachments/assets/7ebe5266-fd70-4099-8b2b-1164a0939047" />

*Fig 11: CloudTrail log showing AccessDenied*

## Key Takeaways
- **Speed:** EventBridge (Flow 2) delivered notifications faster than CloudWatch (Flow 1), which introduced a short delay through metric evaluation
- **Context:** EventBridge alerts carried richer contextual data (event structure, identity details), making it the stronger choice when paired with Lambda or SSM Automation for automated blocking
- **Compliance:** This lab satisfies the Continuous Monitoring requirement in frameworks like NIST, through ongoing visibility, automated detection, and timely response to sensitive access events
- **Real-world takeaway:** In a banking environment, EventBridge would be the preferred trigger for a Kill-Switch mechanism, given its lower latency, richer context, and native integration with automated remediation workflows
