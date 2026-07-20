# Continuous Compliance & Change Management (AWS Config)

## Objective
Implement a continuous compliance and change management solution in AWS to demonstrate how cloud resources are monitored automatically, how security misconfigurations are detected, and how real-time alerts are generated when a compliance violation occurs — using S3 public access as the test scenario.

## Tools & Technologies
- AWS Config
- Amazon SNS
- Amazon S3
- IAM

## What I Did

**Setup**
- Logged in via an IAM user with administrative permissions (not root), following AWS security best practice
- Worked within a single region (US East, N. Virginia) to keep all services aligned

**Configuring AWS Config**
- Enabled AWS Config to record all supported resource types in the region
- Allowed AWS Config to auto-create the required IAM role
- Created an S3 bucket to store configuration history and compliance snapshots

**Configuring Amazon SNS for Alerts**
- Created a standard SNS topic (`SecurityAlertsTopic`) as the central channel for security notifications
- Set up and confirmed an email subscription to receive real-time alerts

**Adding the Compliance Rule**
- Added the AWS-managed Config rule `s3-bucket-public-read-prohibited` to evaluate S3 buckets for public read access
- Linked the AWS Config delivery channel to the `SecurityAlertsTopic` SNS topic, so compliance status changes trigger notifications

**Triggering a Non-Compliant Event (Break Phase)**
- Created a test S3 bucket (`test-public-bucket-amarachi`) and intentionally disabled "Block all public access" to violate the compliance rule
- Confirmed AWS Config detected the change and flagged the resource as **Non-compliant**

**Investigating the Violation**
- Used the AWS Config Resource Inventory and Resource Timeline to trace exactly when the bucket became public and when the compliance rule flagged it
- Demonstrated how AWS Config supports auditing and investigation through detailed resource change visibility

## Screenshots

<img width="458" height="110" src="https://github.com/user-attachments/assets/5f6f714c-1a8c-4261-82b6-224c850816c0" />
<img width="342" height="53" src="https://github.com/user-attachments/assets/91504af8-01a7-4cad-a130-8c01fa67504f" />

*Fig 1: AWS Config recording setup* 

<img width="186" height="43" src="https://github.com/user-attachments/assets/627c13f1-4410-41cf-92d0-009225c10bf2" />
<img width="193" height="48" src="https://github.com/user-attachments/assets/f075f180-7f01-4b5d-9997-e42db871e3cd" />

*Fig 2: SNS topic and email subscription*

<img width="227" height="107" src="https://github.com/user-attachments/assets/57d893a8-a7a2-4a9c-b7ec-0e407958f068" />

*Fig 3: Compliance rule added and linked to SNS*

<img width="335" height="115" src="https://github.com/user-attachments/assets/45836053-08c8-4c47-9be2-6da7e9fb2b38" />

*Fig 4: S3 bucket public access violation triggered*

<img width="421" height="234" src="https://github.com/user-attachments/assets/bd6547d5-4e75-4675-836d-d3442081dd9a" />

*Fig 5: SNS email alert for non-compliant bucket*

<img width="261" height="123" src="https://github.com/user-attachments/assets/188d70b3-3f14-4c53-8f06-c747baaec757" />

*Fig 6: AWS Config resource timeline investigation*

## Key Takeaways
- AWS Config provides continuous, automated visibility into configuration drift, catching risky changes (like a bucket flipping to public) without manual review
- Pairing AWS Config with SNS turns passive monitoring into real-time alerting, closing the gap between misconfiguration and detection
- The Resource Timeline is valuable for incident investigation, giving a clear before/after view of exactly when a resource became non-compliant
