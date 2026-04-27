---
title: OCI NoSQL Cheat Sheet
description: Quick reference for common OCI Functions and NoSQL pitfalls when converting DynamoDB-style code.
metadata:
  type: guide
  tags:
    - oci-functions
    - nosql
    - dynamodb
  languages:
    - python
  updated_at: 2026-04-25
---
## UpdateRow basics

Use this document as a routing guide when converting AWS Lambda Python code that uses DynamoDB through `boto3`.

Do not treat DynamoDB request shapes as directly compatible with OCI NoSQL. Use this sheet to choose focused MCP calls, then fetch the exact SDK operation and model docs needed for the source operations found in the Lambda.

## Core MCP routing

| Need | MCP tool | Arguments |
| --- | --- | --- |
| NoSQL service overview | `get_service` | `service_ref: nosql` |
| NoSQL client docs | `get_service_client` | `service_ref: nosql`, `client_ref: oci.nosql.NosqlClient` |
| Available NoSQL clients | `list_service_clients` | `service_ref: nosql` |
| Available NoSQL models | `list_service_models` | `service_ref: nosql` |

## DynamoDB operation routing

| DynamoDB pattern | OCI NoSQL target | Required MCP calls | Key guidance |
| --- | --- | --- | --- |
| `put_item(...)` | `NosqlClient.update_row(...)` | `get_document: oci.nosql.python.update_row`; `get_service_model: oci.nosql.models.UpdateRowDetails` | Use `UpdateRowDetails(value=...)`. For `attribute_not_exists(...)`, consider `option: IF_ABSENT` and add a note for unsupported condition semantics. |
| `get_item(...)` | `NosqlClient.get_row(...)` | `get_document: oci.nosql.python.get_row` | Convert DynamoDB typed key maps to OCI NoSQL primary-key values. Verify key order and table schema. |
| `delete_item(...)` | `NosqlClient.delete_row(...)` | `get_document: oci.nosql.python.delete_row` | Convert DynamoDB key maps to OCI NoSQL key values. Include compartment context when docs or table-name usage require it. |
| `update_item(...)` | Usually `NosqlClient.update_row(...)` | `get_document: oci.nosql.python.update_row`; `get_service_model: oci.nosql.models.UpdateRowDetails` | DynamoDB `UpdateExpression` is not a direct shape match. Prefer explicit row-value updates and add a review note. |
| `query(...)` | `NosqlClient.query(...)` | `get_document: oci.nosql.python.query`; `get_service_model: oci.nosql.models.QueryDetails` | DynamoDB `KeyConditionExpression`, indexes, and expression attributes need careful translation. Add a review note. |
| `scan(...)` | Review required | `search_documents: nosql scan query python`; `get_document: oci.nosql.python.query` if applicable | Do not invent full-table scan behavior. Mark unclear mappings as review-required. |
| `batch_write_item(...)` | Review required | `search_documents: nosql update row batch python`; `list_service_models: nosql` | Do not invent batch API calls. Prefer explicit limitation notes unless docs confirm a target. |
| `batch_get_item(...)` | Review required | `search_documents: nosql get row batch python`; `list_service_models: nosql` | Do not invent batch API calls. Prefer explicit limitation notes unless docs confirm a target. |

## Compartment rule

If `table_name_or_id` is a table name rather than an OCID, include `compartment_id` inside `UpdateRowDetails` for row writes.

If `table_name_or_id` starts with `ocid1.`, compartment config may be omitted unless the exact operation docs require it.

This is a common conversion pitfall for generated OCI Functions.

## Correct placement of compartment_id

Put `compartment_id` inside `UpdateRowDetails`, not on `update_row(...)`.

#### Correct example

```python
import oci

signer = oci.auth.signers.get_resource_principals_signer()
client = oci.nosql.NosqlClient(config={}, signer=signer)

table_name_or_id = "<table-name-or-ocid>"
row_value = {
    "id": "<row-id>",
    "status": "UPDATED",
}

details_kwargs = {
    "value": row_value,
}

if not table_name_or_id.startswith("ocid1."):
    details_kwargs["compartment_id"] = "<compartment-ocid>"

update_details = oci.nosql.models.UpdateRowDetails(**details_kwargs)

response = client.update_row(
    table_name_or_id=table_name_or_id,
    update_row_details=update_details,
)
```

#### Incorrect example

```python
client.update_row(
    table_name_or_id="<table-name>",
    compartment_id="<compartment-ocid>",
    update_row_details=update_details,
)
```

## Required generated code properties

Generated `func.py` must:

- use `oci.nosql.NosqlClient`
- use resource principal auth in OCI Functions
- remove executable `boto3` DynamoDB client code
- avoid `oci.nosql.NoSQLClient`
- avoid DynamoDB typed request maps like `{"S": ...}` in final OCI calls
- return explicit non-2xx errors or raise on OCI SDK failures; do not silently swallow exceptions

Generated `requirements.txt` must:

- include `oci`
- remove `boto3` when DynamoDB was the only AWS SDK dependency

Generated notes must mention:

- table name vs table OCID assumption
- DynamoDB condition/update expression limitations
- primary key and table schema assumptions

## Supported UpdateRowDetails fields

Use documented fields such as:

- `value`
- `option`
- `compartment_id`
- `is_get_return_row`
- `timeout_in_ms`
- `ttl`
- `is_ttl_use_table_default`
- `identity_cache_size`
- `is_exact_match`

Do not invent undocumented fields such as `is_upsert`.
