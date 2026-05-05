
## Security Controls Applied
- **Least-privilege IAM** — dedicated service account with read-only access to CloudTrail, S3, and SQS only
- **MFA** — enabled on all users including root account
- **KMS encryption at rest** — applied to S3 bucket storing CloudTrail logs
- **CloudTrail log file validation** — digest files ensure logs cannot be tampered with
- **S3 versioning** — protects against accidental deletion and ransomware
- **Block public access** — enabled at account level on all S3 buckets
- **AWS Budgets alert** — zero spend threshold to prevent unexpected charges
- **Dedicated service account** — Splunk credentials scoped to minimum permissions required

## Pipeline Components

### AWS CloudTrail
- Enabled across all regions
- Captures all management events (read and write)
- Captures S3 data events
- Log file validation enabled for tamper detection
- KMS encryption enabled

### Amazon S3
- Stores CloudTrail log files
- Server-side encryption with KMS
- Versioning enabled
- All public access blocked
- S3 event notifications configured to trigger SQS

### Amazon SQS
- Receives S3 event notifications when new log files arrive
- Access policy configured to allow S3 to send messages
- Eliminates need for Splunk to poll S3 directly

### Splunk Cloud
- Splunk Add-on for AWS installed and configured
- SQS-based S3 input pulling CloudTrail logs automatically
- Sourcetype: aws:cloudtrail
- Real-time ingestion of API activity across the AWS account

## Detections Built in Splunk

### Root Account Usage
Detects any API activity from the root account — a critical risk indicator in any AWS environment.
index=* sourcetype="aws:cloudtrail" userIdentity.type=Root
| stats count by userIdentity.arn, eventName, sourceIPAddress

### IAM Privilege Escalation
Detects attempts to escalate privileges through IAM policy modifications or access key creation.
index=* sourcetype="aws:cloudtrail" eventName IN ("AttachUserPolicy","CreateAccessKey","PutUserPolicy")
| stats count by userIdentity.userName, eventName, sourceIPAddress

### S3 Public Access Setting Changed
Detects when public access settings on S3 buckets are modified — a common misconfiguration risk.
index=* sourcetype="aws:cloudtrail" eventName="PutBucketPublicAccessBlock"
| table eventTime, userIdentity.userName, requestParameters

## Key Risk Findings
- Successfully detected root account activity generated during lab setup — demonstrating why root usage is a critical risk indicator
- Pipeline confirmed working with real CloudTrail events flowing into Splunk
- Detections validated against live AWS account activity

## Lessons Learned
- IAM permissions must be scoped carefully — initial lab-user account was missing SQS permissions which caused Splunk ingestion to fail. Fixed by adding AmazonSQSFullAccess and removing AmazonSQSReadOnlyAccess
- SQS access policy must explicitly allow S3 to send messages before S3 event notifications will work
- CloudTrail data events incur additional AWS charges — monitor with AWS Budgets

## Tools and Services Used
- AWS CloudTrail
- Amazon S3
- Amazon SQS
- AWS IAM
- AWS KMS
- AWS Budgets
- Splunk Cloud (14 day trial)
- Splunk Add-on for Amazon Web Services

## Frameworks Referenced
- NIST Cybersecurity Framework — Security pillar (Identify, Protect, Detect)
- MITRE ATT&CK — Detections mapped to Privilege Escalation and Defense Evasion tactics
- AWS Well-Architected Framework — Security pillar
