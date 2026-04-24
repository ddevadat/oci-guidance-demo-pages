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

For OCI NoSQL row writes, prefer:

- `oci.nosql.NosqlClient`
- `NosqlClient.update_row(table_name_or_id, update_row_details, **kwargs)`
- `oci.nosql.models.UpdateRowDetails`

When converting DynamoDB-style code, do not assume the AWS request shape maps directly to OCI NoSQL.

## Compartment rule

If `table_name_or_id` is a table name rather than an OCID, include `compartment_id` in `UpdateRowDetails`.

If `table_name_or_id` is already an OCID, compartment context may not be required.

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

print(response.data)
```

#### Incorrect example

```python
client.update_row(
    table_name_or_id="<table-name>",
    compartment_id="<compartment-ocid>",
    update_row_details=update_details,
)
```

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

Do not invent undocumented fields.

## Common mistakes to avoid

- Do not use `oci.nosql.NoSQLClient`; use `oci.nosql.NosqlClient`
- Do not add `is_upsert` to `UpdateRowDetails`
- Do not put `compartment_id` on `update_row(...)`
- Do not omit compartment context when the table reference is a plain table name
- Do not carry DynamoDB-specific helper names or request models into OCI NoSQL code

## Suggested function config

For OCI Functions, common config keys may look like:

- `ORDERS_TABLE`
- `ORDERS_TABLE_COMPARTMENT`

If the table reference is an OCID instead of a table name, the compartment variable may not be needed.

## Conversion hint

When translating DynamoDB writes to OCI NoSQL, keep the generated code focused on:

- resource principal authentication
- `NosqlClient.update_row(...)`
- `UpdateRowDetails(...)`
- placing `compartment_id` inside the model when the table reference is not an OCID
- preserving only the minimum compatibility env vars required by the source application
