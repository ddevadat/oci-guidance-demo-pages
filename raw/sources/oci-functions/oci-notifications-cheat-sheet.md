---
title: OCI Notifications Cheat Sheet
description: Quick reference for common OCI Functions and Notifications pitfalls when converting SNS-style code.
metadata:
  type: guide
  tags:
    - oci-functions
    - notifications
    - ons
    - sns
  languages:
    - python
  updated_at: 2026-04-28
---

## PublishMessage basics

Use this document as a routing guide when converting AWS Lambda Python code that uses Amazon SNS through `boto3`.

Do not treat SNS request shapes as directly compatible with OCI Notifications. Use this sheet to choose focused MCP calls, then fetch the exact SDK operation and model docs needed for the source operations found in the Lambda.

## Core MCP routing

| Need | MCP tool | Arguments |
| --- | --- | --- |
| Notifications service overview | `get_service` | `service_ref: ons` |
| Data-plane client docs | `get_service_client` | `service_ref: ons`, `client_ref: oci.ons.NotificationDataPlaneClient` |
| Control-plane client docs | `get_service_client` | `service_ref: ons`, `client_ref: oci.ons.NotificationControlPlaneClient` |
| Available Notifications clients | `list_service_clients` | `service_ref: ons` |
| Available Notifications models | `list_service_models` | `service_ref: ons` |

## SNS operation routing

| boto3 SNS pattern | OCI Notifications target | Required MCP calls | Key guidance |
| --- | --- | --- | --- |
| `publish(...)` with `TopicArn` | `NotificationDataPlaneClient.publish_message(...)` | `get_document: oci.ons.python.publish_message`; `get_service_model: oci.ons.models.MessageDetails`; `get_document: oci.ons.python.get_topic` | Map topic ARN to topic OCID. Publish requires the topic API endpoint. |
| `publish(...)` with `Subject` and `Message` | `MessageDetails(title=..., body=...)` | `get_service_model: oci.ons.models.MessageDetails` | `body` is required. `title` is optional. Do not put confidential data in either field. |
| `publish(...)` with JSON `MessageStructure` | Review required | `get_document: oci.ons.python.publish_message`; `get_service_model: oci.ons.models.MessageDetails` | OCI publish supports message body text. Do not invent per-protocol payload mapping. Add review notes. |
| `create_topic(...)` | Usually out of generated function code | `get_document: oci.ons.python.create_topic`; `get_service_model` for create topic details | Resource provisioning is usually outside function conversion. Add review notes unless source creates topics at runtime. |
| `delete_topic(...)` | Usually out of generated function code | `get_document: oci.ons.python.delete_topic` | Do not generate destructive topic deletion unless source behavior clearly requires it. |
| `subscribe(...)`, `unsubscribe(...)` | Usually out of generated function code | `search_documents: ons subscription python`; `get_service_client: oci.ons.NotificationDataPlaneClient` | Subscription lifecycle is usually outside function conversion. Add review notes. |
| `list_topics(...)` | `NotificationControlPlaneClient.list_topics(...)` | `get_document: oci.ons.python.list_topics` | Requires compartment context. Prefer config placeholders. |
| `get_topic_attributes(...)` | `NotificationControlPlaneClient.get_topic(...)` | `get_document: oci.ons.python.get_topic`; `get_service_model: oci.ons.models.NotificationTopic` | Use `api_endpoint`, `topic_id`, and metadata from the returned topic. |

## Topic endpoint rule

Publishing requires the topic API endpoint.

Generated OCI Functions should either:

- read `NOTIFICATIONS_TOPIC_ENDPOINT` from function config, or
- use `NotificationControlPlaneClient.get_topic(topic_id).data.api_endpoint` before constructing the data-plane client.

Do not construct `NotificationDataPlaneClient` with a Queue messages endpoint or a generic region endpoint when publishing to a topic.

## Authentication rule

Generated OCI Functions should use resource principal authentication:

```python
import oci

signer = oci.auth.signers.get_resource_principals_signer()
```

Do not generate local config-file authentication for OCI Functions unless the source specifically targets local tooling.

## Correct publish example

```python
import json
import os

import oci

TOPIC_OCID = os.environ["NOTIFICATIONS_TOPIC_OCID"]
TOPIC_ENDPOINT = os.environ["NOTIFICATIONS_TOPIC_ENDPOINT"]

signer = oci.auth.signers.get_resource_principals_signer()
notifications_client = oci.ons.NotificationDataPlaneClient(
    config={},
    signer=signer,
    service_endpoint=TOPIC_ENDPOINT,
)


def publish_notification(subject: str, payload: dict):
    response = notifications_client.publish_message(
        topic_id=TOPIC_OCID,
        message_details=oci.ons.models.MessageDetails(
            title=subject,
            body=json.dumps(payload),
        ),
    )
    return response.data.message_id
```

## func.yaml config placeholders

```yaml
config:
  NOTIFICATIONS_TOPIC_OCID: "<topic-ocid>"
  NOTIFICATIONS_TOPIC_ENDPOINT: "https://<topic-endpoint>"
```

## Required generated code properties

Generated `func.py` must:

- use `oci.ons.NotificationDataPlaneClient` for publishing
- use `oci.ons.NotificationControlPlaneClient` only for topic metadata or endpoint discovery
- use resource principal auth in OCI Functions
- remove executable `boto3` SNS client code
- use `topic_id` with an OCI topic OCID, not an SNS topic ARN
- construct publish payloads with `oci.ons.models.MessageDetails`
- use `body` for the message payload and `title` for the optional subject
- avoid Queue models such as `PutMessagesDetails` for Notifications
- return explicit non-2xx errors or raise on OCI SDK failures; do not silently swallow exceptions

Generated `requirements.txt` must:

- include `oci`
- remove `boto3` when SNS was the only AWS SDK dependency

Generated notes must mention:

- SNS topic ARN to OCI topic OCID replacement
- topic API endpoint requirement
- IAM/resource principal policy requirement
- message size, protocol fanout, subscription, retry, and delivery semantics that need human review
- unsupported or review-required SNS features such as platform endpoints, SMS-specific behavior, subscription management, and per-protocol JSON payloads

## Common pitfalls

- Do not pass `TopicArn`, `Subject`, or `Message` directly to OCI SDK methods.
- Do not use `oci.queue.QueueClient` for SNS publish conversion.
- Do not use Queue `PutMessagesDetails` for OCI Notifications.
- Do not confuse topic OCID with topic endpoint.
- Do not create, delete, or subscribe topics unless the source Lambda explicitly performs that work at runtime.
- Do not include secrets, tenancy identifiers, user OCIDs, fingerprints, or private keys in generated code or examples.
