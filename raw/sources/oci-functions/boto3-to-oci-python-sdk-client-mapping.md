---
title: Boto3 to OCI Python SDK Client Mapping
description: Compact boto3 client to OCI Python SDK client mapping for AWS Lambda to OCI Functions migration guidance.
metadata:
  type: guide
  tags:
    - boto3
    - oci-python-sdk
    - aws-lambda
    - oci-functions
    - serverless-migration
  languages:
    - python
  updated_at: 2026-04-27
---

# Boto3 to OCI Python SDK Client Mapping

Use this table to identify likely OCI Python SDK clients when converting Python AWS Lambda code that uses `boto3.client(...)`.

This is a service/client mapping only. Do not assume request shapes, auth models, pagination, waiters, exceptions, or response objects are compatible.

| boto3 client | Common AWS service | OCI Python SDK client |
| --- | --- | --- |
| `boto3.client("s3")` | Amazon S3 | `oci.object_storage.ObjectStorageClient` |
| `boto3.client("dynamodb")` | Amazon DynamoDB | `oci.nosql.NosqlClient` |
| `boto3.client("sqs")` | Amazon SQS | `oci.queue.QueueClient` |
| `boto3.client("sns")` | Amazon SNS | `oci.ons.NotificationControlPlaneClient` / `oci.ons.NotificationDataPlaneClient` |
| `boto3.client("events")` | Amazon EventBridge / CloudWatch Events | `oci.events.EventsClient` |
| `boto3.client("cloudwatch")` | Amazon CloudWatch metrics | `oci.monitoring.MonitoringClient` |
| `boto3.client("logs")` | Amazon CloudWatch Logs | `oci.logging.LoggingManagementClient` / `oci.loggingingestion.LoggingClient` |
| `boto3.client("secretsmanager")` | AWS Secrets Manager | `oci.secrets.SecretsClient` |
| `boto3.client("kms")` | AWS KMS | `oci.key_management.KmsManagementClient` / `oci.key_management.KmsCryptoClient` |
| `boto3.client("iam")` | AWS IAM | `oci.identity.IdentityClient` |
| `boto3.client("sts")` | AWS STS | OCI Resource Principal signer / `oci.identity.IdentityClient` |
| `boto3.client("lambda")` | AWS Lambda | `oci.functions.FunctionsManagementClient` / `oci.functions.FunctionsInvokeClient` |
| `boto3.client("apigateway")` | Amazon API Gateway | `oci.apigateway.ApiGatewayClient` |
| `boto3.client("kinesis")` | Amazon Kinesis Data Streams | `oci.streaming.StreamClient` / `oci.streaming.StreamAdminClient` |
| `boto3.client("firehose")` | Amazon Data Firehose | OCI Streaming / Service Connector Hub patterns |
| `boto3.client("ses")` | Amazon SES | `oci.email.EmailClient` |
| `boto3.client("ssm")` | AWS Systems Manager Parameter Store | OCI Vault / OCI Functions configuration |
| `boto3.client("cloudformation")` | AWS CloudFormation | `oci.resource_manager.ResourceManagerClient` |
| `boto3.client("ec2")` | Amazon EC2 | `oci.core.ComputeClient` / `oci.core.VirtualNetworkClient` |
| `boto3.client("ecr")` | Amazon ECR | `oci.artifacts.ArtifactsClient` |
| `boto3.client("eks")` | Amazon EKS | `oci.container_engine.ContainerEngineClient` |
| `boto3.client("rds")` | Amazon RDS | `oci.database.DatabaseClient` |
| `boto3.client("redshift")` | Amazon Redshift | OCI Autonomous Data Warehouse / `oci.database.DatabaseClient` |
| `boto3.client("glue")` | AWS Glue / Data Catalog | `oci.data_catalog.DataCatalogClient` |
| `boto3.client("athena")` | Amazon Athena | OCI query-over-object-storage patterns; no direct drop-in client |
| `boto3.client("sagemaker")` | Amazon SageMaker | `oci.data_science.DataScienceClient` |
| `boto3.client("inspector")` | Amazon Inspector | `oci.cloud_guard.CloudGuardClient` |
| `boto3.client("cloudtrail")` | AWS CloudTrail | `oci.audit.AuditClient` |

## Lambda conversion hints

| boto3 pattern | OCI Functions conversion hint |
| --- | --- |
| `boto3.client(...)` inside handler | Prefer module-level OCI client creation when safe, or lazy initialization |
| Lambda execution role credentials | Use OCI Resource Principal signer |
| AWS region from environment | Use OCI region/configuration from Resource Principal or runtime config |
| AWS ARNs | Replace with OCI OCIDs or configured resource names |
| S3 bucket/key | Object Storage namespace, bucket name, object name |
| DynamoDB table name | OCI NoSQL table name or OCID; include compartment ID when required |
| SQS queue URL | OCI Queue OCID plus messages endpoint |
| SNS topic ARN | OCI Notifications topic OCID |
| Secrets Manager secret ID | OCI Vault secret OCID |
| KMS key ID/ARN | OCI Vault key OCID |
| CloudWatch logs/metrics | OCI Logging and OCI Monitoring |
| EventBridge event pattern | OCI Events rule pattern |

## Safe generation rules

- Do not copy AWS request objects directly into OCI SDK calls.
- Do not invent OCI SDK model fields.
- Do not put credentials, tenancy IDs, compartment OCIDs, secret OCIDs, or private endpoints in generated examples.
- Use placeholders such as `<compartment-ocid>`, `<bucket-name>`, `<queue-ocid>`, `<topic-ocid>`, and `<region>`.
- Prefer OCI Resource Principal authentication for OCI Functions runtime code.
