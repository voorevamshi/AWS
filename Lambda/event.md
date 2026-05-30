# Lambda Event Summary

The **Event** object serves as the input payload sent to your Lambda function. It contains the data describing what triggered the execution.

### Key Attributes

-   **Event Source:** `aws:s3`
-   **Event Name:** `ObjectCreated:Put`
-   **Triggered by:** S3 Bucket `demos3triggerambda`
-   **File Key:** `download6.jpg`

```json
{
    "Records": [
        {
            "eventVersion": "2.1",
            "eventSource": "aws:s3",
            "awsRegion": "ap-south-2",
            "eventTime": "2026-05-30T15:10:15.585Z",
            "eventName": "ObjectCreated:Put",
            "userIdentity": {
                "principalId": "AWS:AIDAWI5EIZIHKSPNQ6WGQ"
            },
            "requestParameters": {
                "sourceIPAddress": "49.43.232.164"
            },
            "responseElements": {
                "x-amz-request-id": "S073DKYE9XP6J5TV",
                "x-amz-id-2": "JnM/FUQPaNlW2wiIAKFjHYtaS+l4p6b3S+1Ez1cyTJjN862pliXmsHmsImtnRGydOR8XIbZMjFK5lYNZtyUZAmKOHIhpcGBUowvwSwef2eE="
            },
            "s3": {
                "s3SchemaVersion": "1.0",
                "configurationId": "demo-s3-event-trigger-lambda",
                "bucket": {
                    "name": "demos3triggerambda",
                    "ownerIdentity": {
                        "principalId": "A3I5S1OXXDS5T9"
                    },
                    "arn": "arn:aws:s3:::demos3triggerambda"
                },
                "object": {
                    "key": "download6.jpg",
                    "size": 5677,
                    "eTag": "b9490ff311bf8588c75a91c8bec37cd6",
                    "sequencer": "006A1AFDD78E464CC0"
                }
            }
        }
    ]
}
```
