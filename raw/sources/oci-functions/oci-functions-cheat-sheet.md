---
title: OCI Queue Cheat Sheet
description: Quick reference for common OCI Functions and Queue pitfalls when converting SQS-style code.
metadata:
  type: guide
  tags:
    - oci-functions
    - queue
    - sqs
    - boto3
  languages:
    - python
  updated_at: 2026-04-28
---
## PutMessages queue

Use this guide when an AWS Lambda Python function uses `boto3.client("sqs")`.

OCI Queue is the target service for Amazon SQS usage. OCI Queue data-plane operations use a queue OCID and a queue-specific messages endpoint, not an SQS queue URL. The messages endpoint can be supplied as configuration or discovered with `QueueAdminClient.get_queue(queue_id).data.messages_endpoint`.

#### MCP routing

| Need | MCP tool | Arguments |
| --- | --- | --- |
| Queue data-plane client | `get_service_client` | `service_ref=queue`, `client_ref=oci.queue.QueueClient` |
| Queue admin client | `get_service_client` | `service_ref=queue`, `client_ref=oci.queue.QueueAdminClient` |
| Send messages | `get_document` | `doc_id=oci.queue.python.put_messages` |
| Receive messages | `get_document` | `doc_id=oci.queue.python.get_messages` |
| Delete one message | `get_document` | `doc_id=oci.queue.python.delete_message` |
| Delete messages in batch | `get_document` | `doc_id=oci.queue.python.delete_messages` |
| Change message visibility | `get_document` | `doc_id=oci.queue.python.update_message` |
| Get queue metadata | `get_document` | `doc_id=oci.queue.python.get_queue` |
| Publish request model | `get_service_model` | `service_ref=queue`, `model_ref=oci.queue.models.PutMessagesDetails` |
| Publish entry model | `get_service_model` | `service_ref=queue`, `model_ref=oci.queue.models.PutMessagesDetailsEntry` |
| Message metadata model | `get_service_model` | `service_ref=queue`, `model_ref=oci.queue.models.MessageMetadata` |

#### SQS operation mapping

| boto3 SQS operation | OCI Queue operation | Notes |
| --- | --- | --- |
| `send_message` | `QueueClient.put_messages` | Use one `PutMessagesDetailsEntry`. |
| `send_message_batch` | `QueueClient.put_messages` | Use multiple `PutMessagesDetailsEntry` values. |
| `receive_message` | `QueueClient.get_messages` | Preserve max count, wait time, and visibility behavior when clear. |
| `delete_message` | `QueueClient.delete_message` | Use the OCI message receipt from `get_messages`. |
| `delete_message_batch` | `QueueClient.delete_messages` | Use the batch delete model from Queue docs. |
| `change_message_visibility` | `QueueClient.update_message` | Map visibility timeout only when source intent is clear. |
| `get_queue_url` | Runtime configuration or queue lookup | OCI uses queue OCID and messages endpoint, not SQS queue URL. |
| `get_queue_attributes` | `QueueAdminClient.get_queue` or `QueueClient.get_stats` | Choose based on whether source needs metadata or runtime stats. |

#### Required configuration

| Value | Recommended config name |
| --- | --- |
| Queue OCID | `QUEUE_OCID` |
| Queue messages endpoint | `QUEUE_MESSAGES_ENDPOINT` |
| Optional consumer group identifier | `QUEUE_CONSUMER_GROUP_ID` |

For OCI Functions, prefer resource principal authentication. Do not generate tenancy, user, fingerprint, or private key values in migrated code.

#### Examples

```python
import json
import os

import oci

QUEUE_OCID = os.environ["QUEUE_OCID"]
QUEUE_MESSAGES_ENDPOINT = os.environ["QUEUE_MESSAGES_ENDPOINT"]

signer = oci.auth.signers.get_resource_principals_signer()

queue_client = oci.queue.QueueClient(
    config={},
    signer=signer,
    service_endpoint=QUEUE_MESSAGES_ENDPOINT,
)


def publish_message(payload: dict):
    response = queue_client.put_messages(
        queue_id=QUEUE_OCID,
        put_messages_details=oci.queue.models.PutMessagesDetails(
            messages=[
                oci.queue.models.PutMessagesDetailsEntry(
                    content=json.dumps(payload)
                )
            ]
        ),
    )
    return response.data.messages
```

```yaml
config:
  QUEUE_OCID: "<queue-ocid>"
  QUEUE_MESSAGES_ENDPOINT: "https://<cell>.queue.messaging.<region>.oci.oraclecloud.com"
```

#### Conversion rules

- Use `oci.queue.QueueClient` for message data-plane operations.
- Use `oci.queue.QueueAdminClient` for queue metadata and endpoint discovery.
- Construct `QueueClient` with `service_endpoint=QUEUE_MESSAGES_ENDPOINT` for data-plane calls.
- Use `PutMessagesDetails` and `PutMessagesDetailsEntry` for publishing.
- Read publish results from `response.data.messages`.
- Do not use OCI Notifications models such as `MessageDetails` for Queue.
- Do not invent unsupported FIFO, deduplication, delay, dead-letter queue, trigger, or event-source behavior.
- Add migration notes when SQS behavior has no direct generated-code equivalent.

#### Review notes

- Replace the source SQS queue URL with OCI Queue OCID and messages endpoint.
- SQS event source mappings are not automatically migrated by function code conversion.
- Visibility timeout, batching, consumer group, retry, and dead-letter queue behavior need human review unless explicitly modeled.
