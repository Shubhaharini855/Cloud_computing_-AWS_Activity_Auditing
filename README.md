##Centralized AWS Activity Auditing and Forensic Investigation Report
#Project Overview
RetailCorp stores critical business assets in Amazon S3 and operates multiple cloud workloads. When a production file is deleted from an S3 bucket, determining the responsible identity, the execution timestamp, the exact API call, and the affected resource is essential for incident response.

This project implements centralized activity auditing on AWS using CloudTrail, Amazon S3, IAM, and CloudWatch Logs. It captures S3 object-level data events and provides forensic discovery capabilities via CloudWatch Logs Insights, along with automated deployment through a CI/CD pipeline.

#Architecture Components and Specifications
Production S3 Bucket (retailcorp-production-storage)

Hosts production business data. Monitored for read and write API calls. Default encryption and public access blocks are enabled.

Audit Log S3 Bucket (retailcorp-central-trail-logs)

Serves as the isolated, long-term archive for compressed CloudTrail audit logs. Protected with a service access policy.

AWS IAM Service Role (CloudTrailToCWLogsRole)

Delegates permission to the CloudTrail service to assume the role and execute logs:CreateLogStream and logs:PutLogEvents into the log group.

Amazon CloudWatch Log Group (/aws/cloudtrail/retailcorp-audit)

Receives real-time log streams from CloudTrail for indexing and analytical queries.

AWS CloudTrail (retailcorp-security-trail)

Records management events and S3 object data events (Write actions) targeting the production bucket.

Implementation Procedure and Documentation
Step 1: Create Production S3 Bucket
Provision the primary storage bucket, retailcorp-production-storage, to store critical business assets. Enforce boundary security by blocking all public access and applying default server-side encryption with Amazon S3 managed keys (SSE-S3).

Step 2: Provision Dedicated Central Audit Log Bucket
Deploy a dedicated storage repository, retailcorp-central-trail-logs, in the same region. This bucket isolates compliance log archives from application data, ensuring compliance integrity and preventing accidental or malicious log tampering.

Step 3: Configure CloudTrail Delivery Policy on Audit Bucket
Attach an S3 bucket policy to the central audit bucket granting permissions to the CloudTrail service principal (cloudtrail.amazonaws.com). The policy authorizes s3:GetBucketAcl and s3:PutObject under the account prefix with the bucket-owner-full-control condition.

Step 4: Create CloudWatch Log Group for Real-Time Ingestion
Configure an Amazon CloudWatch Log Group named /aws/cloudtrail/retailcorp-audit with a retention policy of 30 days. Direct log group streaming allows rapid query capabilities without waiting for batched S3 archive deliveries.

Step 5: Configure IAM Role for CloudTrail-to-CloudWatch Delivery
Create an IAM service role, CloudTrailToCWLogsRole, with a trust policy allowing cloudtrail.amazonaws.com to assume the role. Attach an inline permissions policy that restricts logs:CreateLogStream and logs:PutLogEvents strictly to the destination log group ARN.

Step 6: Deploy AWS CloudTrail with S3 Data Events
Create a multi-service trail named retailcorp-security-trail. Configure the storage destination to the central audit S3 bucket and enable CloudWatch Logs integration using the IAM role created in Step 5. Add an S3 Data Event selector targeting the production bucket for all Write operations.

Step 7: Upload Critical Business Assets to Production Storage
Upload the test business file, q4_confidential_financials.csv, to the production bucket to establish an operational baseline. This action generates a PutObject data event within CloudTrail.

Step 8: Simulate Accidental Deletion Incident
Simulate the security incident by executing an unversioned delete operation on q4_confidential_financials.csv through an IAM user identity. Confirm that the object is permanently deleted from the S3 storage bucket.

Step 9: Execute Forensic Log Investigation in CloudWatch Logs Insights
Access CloudWatch Logs Insights, choose /aws/cloudtrail/retailcorp-audit, and run the following query to extract the deletion record:

Step 10: Extract and Verify Complete Forensic Audit Record
Select and inspect the full JSON log event in the query result table. Verify the identity ARN, user name, event timestamp, API action name, and target S3 key parameters.
