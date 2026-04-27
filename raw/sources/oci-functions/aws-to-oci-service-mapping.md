---
title: AWS to OCI Service Mapping
description: Compact AWS-to-OCI service mapping for serverless migration guidance.
metadata:
  type: guide
  tags:
    - aws
    - oci
    - service-mapping
    - serverless-migration
  updated_at: 2026-04-27
---

# AWS to OCI Service Mapping

Use this table as a quick service-name mapping during migration. It is not an API compatibility guarantee.

| AWS service | OCI service |
| --- | --- |
| AWS Lambda | OCI Functions |
| Amazon API Gateway | OCI API Gateway |
| Amazon EventBridge / CloudWatch Events | OCI Events |
| Amazon SQS | OCI Queue |
| Amazon SNS | OCI Notifications |
| Amazon Kinesis | OCI Streaming |
| Amazon S3 | OCI Object Storage |
| Amazon S3 Glacier | OCI Archive Storage |
| Amazon EFS | OCI File Storage |
| Amazon EBS | OCI Block Volumes |
| AWS Secrets Manager | OCI Vault |
| AWS Key Management Service (KMS) | OCI Vault |
| AWS IAM | OCI IAM |
| AWS CloudTrail | OCI Audit |
| Amazon CloudWatch | OCI Monitoring |
| Amazon CloudWatch Logs | OCI Logging |
| AWS X-Ray | OCI Application Performance Monitoring / OpenTelemetry-compatible tracing |
| AWS CloudFormation | OCI Resource Manager |
| Amazon DynamoDB | OCI NoSQL Database Cloud Service |
| Amazon RDS | OCI Base Database / Autonomous Database / MySQL HeatWave |
| Amazon Aurora | OCI Autonomous Database / MySQL HeatWave |
| Amazon Redshift | OCI Autonomous Data Warehouse |
| Amazon ElastiCache | OCI Cache |
| Amazon VPC | OCI Virtual Cloud Network (VCN) |
| AWS Direct Connect | OCI FastConnect |
| AWS Site-to-Site VPN | OCI VPN Connect |
| Amazon Route 53 | OCI DNS / Traffic Management |
| Elastic Load Balancing | OCI Load Balancer |
| AWS WAF | OCI Web Application Firewall |
| AWS Shield | OCI DDoS Protection |
| Amazon SES | OCI Email Delivery |
| Amazon ECR | OCI Container Registry |
| Amazon EKS | Oracle Container Engine for Kubernetes |
| Amazon EC2 | OCI Compute Virtual Machine Instances |
| Amazon EC2 Bare Metal | OCI Bare Metal Instances |
| Amazon EC2 Dedicated Instances | OCI Dedicated Virtual Machine Hosts |
| AWS Batch | OCI Data Flow / OCI Batch-style worker pattern |
| Amazon EMR | OCI Big Data Service / OCI Data Flow |
| AWS Glue Data Catalog | OCI Data Catalog |
| Amazon Athena | OCI Query Service / query over Object Storage patterns |
| Amazon QuickSight | Oracle Analytics Cloud |
| Amazon SageMaker | OCI Data Science |
| Amazon Inspector | OCI Cloud Guard |
| AWS Organizations | OCI Organization Management |
| AWS Resource Groups | OCI Compartments |
| AWS Tags | OCI Defined Tags / Free-form Tags |

## Serverless migration hints

| AWS pattern | OCI migration hint |
| --- | --- |
| Lambda handler | Convert to OCI Functions `func.py` handler shape |
| Lambda environment variables | Use OCI Functions configuration |
| Lambda IAM role | Use OCI Resource Principal / OCI IAM policy |
| Lambda + SQS | OCI Functions + OCI Queue |
| Lambda + SNS | OCI Functions + OCI Notifications |
| Lambda + EventBridge | OCI Functions + OCI Events |
| Lambda + DynamoDB | OCI Functions + OCI NoSQL Database Cloud Service |
| Lambda + S3 | OCI Functions + OCI Object Storage |
| Lambda + Secrets Manager / KMS | OCI Functions + OCI Vault |
| Lambda logs in CloudWatch Logs | OCI Logging / structured application logs |
| Lambda metrics in CloudWatch | OCI Monitoring / Prometheus-compatible local metrics |
