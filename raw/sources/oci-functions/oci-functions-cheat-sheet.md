---
title: OCI Functions Cheat Sheet
description: Quick reference for common OCI Functions and Queue pitfalls.
metadata:
  type: guide
  tags:
    - oci-functions
    - queue
  languages:
    - python
  updated_at: 2026-04-24
---
## PutMessages queue

Puts messages into the queue. You must use the messages endpoint e.g https://cell-1.queue.messaging.<region>.oci.oraclecloud.com  to produce messages. The messages endpoint may be different for different queues. 


#### Examples

```python
# This is an automatically generated code sample.
# To make this code sample work in your Oracle Cloud tenancy,
# please replace the values for any parameters whose current values do not fit
# your use case (such as resource IDs, strings containing ‘EXAMPLE’ or ‘unique_id’, and
# boolean, number, and enum parameters with values not fitting your use case).

import oci

# Create a default config using DEFAULT profile in default location
# Refer to
# https://docs.cloud.oracle.com/en-us/iaas/Content/API/Concepts/sdkconfig.htm#SDK_and_CLI_Configuration_File
# for more info
config = oci.config.from_file()

# Initialize service client with default config file
queue_client = oci.queue.QueueClient(config)

# Send the request to service, some parameters are not required, see API
# doc for more info
put_messages_response = queue_client.put_messages(
    queue_id="ocid1.test.oc1..<unique_ID>EXAMPLE-queueId-Value",
    put_messages_details=oci.queue.models.PutMessagesDetails(
        messages=[
            oci.queue.models.PutMessagesDetailsEntry(
                content="EXAMPLE-content-Value",
                metadata=oci.queue.models.MessageMetadata(
                    channel_id="ocid1.test.oc1..<unique_ID>EXAMPLE-channelId-Value",
                    custom_properties={
                        'EXAMPLE_KEY_mCLgv': 'EXAMPLE_VALUE_VrQNarWSyXlBWIRs1OvC'}))]),
    opc_request_id="KXVYQFUUMUBO3XEBK4H8<unique_ID>")

# Get the data from response
print(put_messages_response.data)
```
