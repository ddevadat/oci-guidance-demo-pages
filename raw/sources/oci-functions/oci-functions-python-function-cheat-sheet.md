---
title: OCI Functions Python Function Cheat Sheet
description: Compact reference for generating Python OCI Functions artifacts from AWS Lambda handlers.
metadata:
  type: guide
  tags:
    - oci-functions
    - fn-project
    - python
    - aws-lambda-migration
  languages:
    - python
  updated_at: 2026-04-27
---

# OCI Functions Python Function Cheat Sheet

Use this when generating Python OCI Functions artifacts.

## Minimum files

| File | Purpose |
| --- | --- |
| `func.py` | Python function handler |
| `func.yaml` | Fn/OCI Functions metadata |
| `requirements.txt` | Python dependencies |
| `MIGRATION_NOTES.md` | Migration assumptions, warnings, and deferred work |

## Typical `func.py`

```python
import io
import json

from fdk import response


def handler(ctx, data: io.BytesIO = None):
    body = {}

    if data is not None:
        try:
            raw = data.getvalue()
            if raw:
                body = json.loads(raw)
        except Exception:
            body = {}

    result = {
        "message": "hello from OCI Functions",
        "input": body,
    }

    return response.Response(
        ctx,
        response_data=json.dumps(result),
        headers={"Content-Type": "application/json"},
    )
```

## Handler signature

```python
def handler(ctx, data: io.BytesIO = None):
```

| Parameter | Meaning |
| --- | --- |
| `ctx` | Function invocation context from the Python FDK |
| `data` | Request body stream; parse with `data.getvalue()` when JSON input is expected |

## Typical `func.yaml`

```yaml
schema_version: 20180708
name: generated-function
version: 0.0.1
runtime: python
entrypoint: /python/bin/fdk /function/func.py handler
memory: 256
config:
  EXAMPLE_SETTING: "<value>"
  RESOURCE_OCID: "<resource-ocid>"
```

## `func.yaml` fields

| Field | Guidance |
| --- | --- |
| `schema_version` | Use `20180708` unless project conventions say otherwise |
| `name` | Function name; keep lowercase and deployment-safe |
| `version` | Start with `0.0.1` for generated bundles |
| `runtime` | Use `python` for Python OCI Functions |
| `entrypoint` | Points the FDK to `func.py` and `handler` |
| `memory` | Start with `256`; adjust only when source behavior needs more |
| `config` | Function configuration values exposed as environment variables |

## Function config

OCI Functions config values in `func.yaml` are available to Python code through `os.environ`.

```yaml
config:
  BUCKET_NAME: "<bucket-name>"
  QUEUE_OCID: "<queue-ocid>"
  TABLE_NAME: "<table-name>"
```

```python
import os

bucket_name = os.environ.get("BUCKET_NAME")
queue_ocid = os.environ.get("QUEUE_OCID")
table_name = os.environ.get("TABLE_NAME")
```

When converting AWS Lambda environment variables, preserve only the values the function actually needs and rename AWS-specific keys when an OCI-specific resource is required.

## Typical `requirements.txt`

```text
fdk
oci
```

Add only dependencies required by the converted function.

## AWS Lambda to OCI Functions hints

| AWS Lambda concept | OCI Functions equivalent |
| --- | --- |
| `lambda_handler(event, context)` | `handler(ctx, data)` |
| `event` dict | JSON parsed from `data.getvalue()` |
| Lambda `context` object | `ctx` from the FDK; not API-compatible with Lambda context |
| Return dict | `response.Response(...)` with serialized JSON |
| Environment variables | OCI Functions `config` values read through `os.environ` |
| IAM role credentials | OCI Resource Principal signer |
| `boto3` clients | OCI Python SDK clients |

## Conversion rules

- Do not keep the Lambda handler signature as the OCI entrypoint.
- Do not assume the Lambda `event` and OCI `data` shapes are identical.
- Do not assume the Lambda `context` object exists in OCI Functions.
- Do not return raw Python dictionaries unless the project explicitly supports that pattern.
- Do not hard-code OCIDs, tenancy names, compartment names, regions, endpoints, or credentials.
- Use placeholders such as `<compartment-ocid>`, `<bucket-name>`, `<queue-ocid>`, and `<region>` when needed.
- Prefer OCI Resource Principal authentication for OCI Functions runtime code.
- Keep `func.py`, `func.yaml`, and `requirements.txt` small and reviewable.
- Put migration assumptions, unsupported behavior, and validation gaps in `MIGRATION_NOTES.md`.
