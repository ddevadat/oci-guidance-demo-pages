---
title: OCI Functions Event Shape Cheat Sheet
description: Compact reference for converting AWS Lambda Python event handling to OCI Functions Python input handling.
metadata:
  type: guide
  tags:
    - oci-functions
    - aws-lambda
    - event-shapes
    - python
  languages:
    - python
  updated_at: 2026-04-28
---

## Event shape basics

Use this document when converting AWS Lambda Python handlers that inspect the `event` object.

Do not assume OCI Functions receives the same event envelope as AWS Lambda. Preserve source parsing intent where possible, and add migration notes when trigger wiring or event shape cannot be converted inside function code alone.

## Core MCP routing

| Need | MCP tool | Arguments |
| --- | --- | --- |
| OCI Functions Python guidance | `get_guidance_doc` | `doc_ref: guidance.oci-functions.oci-functions-python-function-cheat-sheet` |
| OCI Functions service docs | `get_service` | `service_ref: functions` |
| Object Storage guidance | `get_guidance_doc` | `doc_ref: guidance.oci-functions.oci-object-storage-cheat-sheet` |
| Queue guidance | `get_guidance_doc` | `doc_ref: guidance.oci-functions.oci-functions-cheat-sheet` |
| Notifications guidance | `get_guidance_doc` | `doc_ref: guidance.oci-functions.oci-notifications-cheat-sheet` |
| NoSQL guidance | `get_guidance_doc` | `doc_ref: guidance.oci-functions.oci-nosql-cheat-sheet` |

## Lambda handler mapping

| AWS Lambda shape | OCI Functions shape | Conversion guidance |
| --- | --- | --- |
| `def lambda_handler(event, context):` | `def handler(ctx, data=None):` | Rename handler and read request payload from `data`. |
| `event` dictionary | decoded request body or event payload | Parse from `data`; do not assume AWS envelope keys exist. |
| `context.aws_request_id` | `ctx` metadata if available | Preserve request ID usage as logging context when possible. |
| return `dict` | `response.Response(...)` or serializable body | Use the project OCI Functions response pattern. |

## Input parsing pattern

```python
import json


def read_json_payload(data):
    if data is None:
        return {}

    if isinstance(data, bytes):
        raw = data.decode("utf-8")
    elif isinstance(data, str):
        raw = data
    elif hasattr(data, "read"):
        body = data.read()
        raw = body.decode("utf-8") if isinstance(body, bytes) else body
    else:
        return data

    return json.loads(raw) if raw else {}
```

## Trigger and event mapping

| AWS source event | OCI migration behavior | Notes |
| --- | --- | --- |
| API Gateway proxy event | Convert request parsing only when source logic is clear | API Gateway routing and deployment wiring are outside generated function code. |
| S3 event | Convert object and bucket extraction to explicit payload assumptions | Pair with Object Storage cheat sheet for SDK calls. |
| SQS event | Convert record-loop logic only if payload shape is explicit | Pair with Queue cheat sheet. Queue trigger wiring needs review. |
| SNS event | Convert message extraction only if payload shape is explicit | Pair with Notifications cheat sheet. Subscription wiring needs review. |
| EventBridge schedule | Replace code-level event parsing with config and review notes | Scheduling is deployment wiring, not normal function code. |
| DynamoDB stream event | Review required | No direct code-only conversion. Preserve business logic and document trigger assumptions. |

## API Gateway proxy notes

Common Lambda API Gateway fields:

- `httpMethod`
- `path`
- `headers`
- `queryStringParameters`
- `body`
- `isBase64Encoded`

Generated OCI Functions code should:

- parse the incoming payload or body explicitly
- preserve HTTP method and path behavior only when those fields are still supplied by the caller
- add review notes for routing, auth, CORS, and API Gateway deployment behavior

## S3 or Object event notes

Common Lambda S3 fields:

- `event["Records"][0]["s3"]["bucket"]["name"]`
- `event["Records"][0]["s3"]["object"]["key"]`

Generated OCI Functions code should:

- avoid assuming the exact AWS `Records` envelope unless retained by the caller
- use clear placeholders such as `bucket_name` and `object_name`
- pair SDK conversion with OCI Object Storage guidance

## SQS event notes

Common Lambda SQS fields:

- `event["Records"]`
- `record["body"]`
- `record["messageId"]`
- `record["receiptHandle"]`

Generated OCI Functions code should:

- preserve message processing loops when source intent is clear
- avoid assuming OCI Queue trigger payload format unless explicitly provided
- add review notes for visibility timeout, delete or ack behavior, batching, and retry semantics

## SNS event notes

Common Lambda SNS fields:

- `event["Records"]`
- `record["Sns"]["Message"]`
- `record["Sns"]["Subject"]`
- `record["Sns"]["TopicArn"]`

Generated OCI Functions code should:

- preserve message extraction only when input shape is still supplied
- map publish operations separately using OCI Notifications guidance
- add review notes for subscription and delivery semantics

## Required generated code properties

Generated `func.py` must:

- define `handler(ctx, data=None)`
- avoid executable references to `lambda_handler` unless kept only as an internal helper
- parse `data` safely for bytes, strings, streams, and dictionaries
- avoid assuming AWS `event["Records"]` unless the source trigger shape is intentionally preserved
- return explicit responses or raise clear errors
- add migration notes for trigger wiring that cannot be converted in code

Generated `func.yaml` must:

- point to the OCI handler entrypoint
- include only non-sensitive config placeholders
- avoid embedding sample AWS event payloads as config

Generated notes must mention:

- original AWS trigger type when detected
- assumed OCI input payload shape
- trigger wiring that must be recreated outside function code
- any AWS event fields preserved for compatibility

## Common pitfalls

- Do not treat OCI Functions `data` as the same object as Lambda `event`.
- Do not silently keep AWS event envelope assumptions without notes.
- Do not convert deployment trigger configuration as executable function code.
- Do not hardcode sample bucket names, queue URLs, topic ARNs, account IDs, regions, or request IDs.
- Do not swallow malformed input errors; return or raise understandable failures.
