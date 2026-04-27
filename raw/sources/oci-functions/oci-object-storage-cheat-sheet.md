---
title: OCI Object Storage Cheat Sheet
description: Quick reference for common OCI Functions and Object Storage pitfalls when converting S3-style code.
metadata:
  type: guide
  tags:
    - oci-functions
    - object-storage
    - s3
  languages:
    - python
  updated_at: 2026-04-28
---

## Object Storage basics

Use this document as a routing guide when converting AWS Lambda Python code that uses Amazon S3 through `boto3`.

Do not treat S3 request shapes as directly compatible with OCI Object Storage. Use this sheet to choose focused MCP calls, then fetch the exact SDK operation and model docs needed for the source operations found in the Lambda.

## Core MCP routing

| Need | MCP tool | Arguments |
| --- | --- | --- |
| Object Storage service overview | `get_service` | `service_ref: object_storage` |
| Object Storage client docs | `get_service_client` | `service_ref: object_storage`, `client_ref: oci.object_storage.ObjectStorageClient` |
| Available Object Storage clients | `list_service_clients` | `service_ref: object_storage` |
| Available Object Storage models | `list_service_models` | `service_ref: object_storage` |

## S3 operation routing

| boto3 S3 pattern | OCI Object Storage target | Required MCP calls | Key guidance |
| --- | --- | --- | --- |
| `get_object(...)` | `ObjectStorageClient.get_object(...)` | `get_document: oci.object_storage.python.get_object`; `get_document: oci.object_storage.python.get_namespace` | OCI requires `namespace_name`, `bucket_name`, and `object_name`. Read response body from `response.data`. |
| `put_object(...)` | `ObjectStorageClient.put_object(...)` | `get_document: oci.object_storage.python.put_object`; `get_document: oci.object_storage.python.get_namespace` | Map `Bucket` to `bucket_name`, `Key` to `object_name`, and `Body` to `put_object_body`. Preserve content type only with documented parameter names. |
| `delete_object(...)` | `ObjectStorageClient.delete_object(...)` | `get_document: oci.object_storage.python.delete_object`; `get_document: oci.object_storage.python.get_namespace` | Map bucket/key directly. Add notes for delete marker/versioning behavior if source uses `VersionId`. |
| `head_object(...)` | `ObjectStorageClient.head_object(...)` | `get_document: oci.object_storage.python.head_object`; `get_document: oci.object_storage.python.get_namespace` | Use for metadata/ETag checks. Do not replace with `get_object` unless the source needs the body. |
| `list_objects(...)`, `list_objects_v2(...)` | `ObjectStorageClient.list_objects(...)` | `get_document: oci.object_storage.python.list_objects`; `get_document: oci.object_storage.python.get_namespace` | Map prefix, delimiter, and pagination carefully. Add review notes for continuation token differences. |
| `copy_object(...)` | `ObjectStorageClient.copy_object(...)` | `get_document: oci.object_storage.python.copy_object`; `get_service_model: oci.object_storage.models.CopyObjectDetails` | Use documented copy details model. Do not invent source URI formats. |
| `create_bucket(...)` | `ObjectStorageClient.create_bucket(...)` | `get_document: oci.object_storage.python.create_bucket`; `get_service_model: oci.object_storage.models.CreateBucketDetails`; `get_document: oci.object_storage.python.get_namespace` | Bucket creation usually needs compartment context. Prefer config/env for compartment OCID. |
| `delete_bucket(...)` | `ObjectStorageClient.delete_bucket(...)` | `get_document: oci.object_storage.python.delete_bucket`; `get_document: oci.object_storage.python.get_namespace` | Bucket must be empty. Add a review note if source assumes recursive deletion. |
| `upload_file(...)`, `download_file(...)` | Usually `put_object(...)` / `get_object(...)` | Fetch operation docs for `put_object` or `get_object`; search docs for multipart if large files are implied | boto3 transfer helpers are not direct OCI SDK calls. Convert simple cases; mark large file/multipart behavior for review. |
| `generate_presigned_url(...)` | Usually pre-authenticated request, review required | `search_documents: object storage preauthenticated request python`; `get_service_model` as needed | S3 presigned URLs are not a direct API shape match. Do not silently generate a different security model. |
| multipart upload APIs | Multipart upload operations, review required | `search_documents: object storage multipart upload python`; `list_service_models: object_storage` | Fetch exact create/upload/commit docs. Do not invent multipart model fields. |
| ACL, bucket policy, tagging, notification config | Review required | `search_documents` with the exact source operation name | Do not invent OCI IAM, policy, tag, or event behavior. Add explicit migration notes. |

## Namespace rule

Most Object Storage object operations require `namespace_name`.

Generated OCI Functions should either:

- call `client.get_namespace().data` once per invocation or cached module-level initialization, or
- read `OBJECT_STORAGE_NAMESPACE` from function config when explicitly provided.

If the source code only has S3 bucket and key values, do not confuse bucket name with namespace.

## Authentication rule

Generated OCI Functions should use resource principal authentication:

```python
import oci

signer = oci.auth.signers.get_resource_principals_signer()
client = oci.object_storage.ObjectStorageClient(config={}, signer=signer)
