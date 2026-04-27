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
| `put_object(...)` | `ObjectStorageClient.put_object(...)` | `get_document: oci.object_storage.python.put_object`; `get_document: oci.object_storage.python.get_namespace` | Map `Bucket` to `bucket_name`, `Key` to `object_name`, and `Body` to `put_object_body`. |
| `delete_object(...)` | `ObjectStorageClient.delete_object(...)` | `get_document: oci.object_storage.python.delete_object`; `get_document: oci.object_storage.python.get_namespace` | Map bucket/key directly. Add notes for delete marker/versioning behavior if source uses `VersionId`. |
| `head_object(...)` | `ObjectStorageClient.head_object(...)` | `get_document: oci.object_storage.python.head_object`; `get_document: oci.object_storage.python.get_namespace` | Use for metadata and ETag checks. Do not replace with `get_object` unless the source needs the body. |
| `list_objects(...)`, `list_objects_v2(...)` | `ObjectStorageClient.list_objects(...)` | `get_document: oci.object_storage.python.list_objects`; `get_document: oci.object_storage.python.get_namespace` | Map prefix, delimiter, and pagination carefully. Add review notes for continuation token differences. |
| `copy_object(...)` | `ObjectStorageClient.copy_object(...)` | `get_document: oci.object_storage.python.copy_object`; `get_service_model: oci.object_storage.models.CopyObjectDetails` | Use documented copy details model. Do not invent source URI formats. |
| `create_bucket(...)` | `ObjectStorageClient.create_bucket(...)` | `get_document: oci.object_storage.python.create_bucket`; `get_service_model: oci.object_storage.models.CreateBucketDetails`; `get_document: oci.object_storage.python.get_namespace` | Bucket creation usually needs compartment context. Prefer config or environment for compartment OCID. |
| `delete_bucket(...)` | `ObjectStorageClient.delete_bucket(...)` | `get_document: oci.object_storage.python.delete_bucket`; `get_document: oci.object_storage.python.get_namespace` | Bucket must be empty. Add a review note if source assumes recursive deletion. |
| `upload_file(...)`, `download_file(...)` | Usually `put_object(...)` / `get_object(...)` | Fetch operation docs for `put_object` or `get_object`; search docs for multipart if large files are implied | boto3 transfer helpers are not direct OCI SDK calls. Convert simple cases; mark large file or multipart behavior for review. |
| `generate_presigned_url(...)` | Usually pre-authenticated request, review required | `search_documents: object storage preauthenticated request python`; `get_service_model` as needed | S3 presigned URLs are not a direct API shape match. Do not silently generate a different security model. |
| multipart upload APIs | Multipart upload operations, review required | `search_documents: object storage multipart upload python`; `list_service_models: object_storage` | Fetch exact create, upload, and commit docs. Do not invent multipart model fields. |
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
```

Do not generate local config-file authentication for OCI Functions unless the source specifically targets local tooling.

## Correct get/put example

```python
import json
import oci

signer = oci.auth.signers.get_resource_principals_signer()
client = oci.object_storage.ObjectStorageClient(config={}, signer=signer)

namespace = client.get_namespace().data

get_response = client.get_object(
    namespace_name=namespace,
    bucket_name="<source-bucket>",
    object_name="<source-object>",
)

payload = json.loads(get_response.data.read().decode("utf-8"))
payload["processed"] = True
payload_bytes = json.dumps(payload).encode("utf-8")

client.put_object(
    namespace_name=namespace,
    bucket_name="<target-bucket>",
    object_name="<target-object>",
    put_object_body=payload_bytes,
    content_length=len(payload_bytes),
)
```

## Required generated code properties

Generated `func.py` must:

- use `oci.object_storage.ObjectStorageClient`
- use resource principal auth in OCI Functions
- remove executable `boto3` S3 client code
- use `namespace_name`, `bucket_name`, and `object_name` explicitly
- read object body from `get_object(...).data`
- avoid S3-style parameters such as `Bucket`, `Key`, and `Body` in final OCI calls
- avoid S3 compatibility endpoints unless explicitly required by the migration plan
- return explicit non-2xx errors or raise on OCI SDK failures; do not silently swallow exceptions

Generated `requirements.txt` must:

- include `oci`
- remove `boto3` when S3 was the only AWS SDK dependency

Generated `func.yaml` should define placeholders only for non-sensitive config such as:

- `SOURCE_BUCKET`
- `TARGET_BUCKET`
- `OBJECT_STORAGE_NAMESPACE`, if not using `get_namespace()`
- `OBJECT_STORAGE_COMPARTMENT_ID`, only when bucket creation or listing requires compartment context

Generated notes must mention:

- S3 bucket to OCI bucket assumption
- Object Storage namespace requirement
- IAM/resource principal policy requirement
- event payload shape assumptions
- unsupported or review-required S3 features such as ACLs, presigned URLs, multipart transfers, bucket notifications, and tagging

## Common pitfalls

- Do not pass `Bucket`, `Key`, or `Body` to OCI SDK methods.
- Do not treat bucket name as namespace.
- Do not assume S3 presigned URLs map directly to OCI calls.
- Do not invent bucket policy or ACL behavior.
- Do not use local OCI config-file auth for generated OCI Function runtime code.
- Do not leave `boto3` imports in executable function code when S3 is fully migrated.
