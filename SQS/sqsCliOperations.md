# AWS SQS CLI Commands Summary
This note outlines common operations for managing AWS SQS via the command line.
## 1. List Queues
To view all available SQS queues in your account: `aws sqs list-queues`
Response: Returns a JSON object containing a list of `QueueUrls`.
```json
{
		"QueueUrls": [
			"https://sqs.ap-south-2.amazonaws.com/431451851278/order-notification-queue"
		]
	}
  ```
  

## 2. Receive a Message
To poll for messages from a specific queue: `aws sqs receive-message --queue-url=<QUEUE_URL>`
• MessageId: Unique identifier for the message.
• ReceiptHandle: Essential for deleting the message after processing.
• Body: The actual content of the message.
```json
	{
		"Messages": [
			{
				"MessageId": "ea2f573f-6da1-4e34-a425-7be72a62efe1",
				"ReceiptHandle": "AQEBNhx3m9ev2LA3CHp2eH5XlvDbdUIi9klxU5YFCEW66ysyeANkbWO5KR3qcxuTq0GZ91PmhXkA2IEgNCjRMMv+npafs7dTVdJ26tUjqt2MxqJTeMylw81H4puttP5ZowjsldCKJAeTMcn5uZeDJYqXyLB1/MqmoWV8ptDAfUMi2j72lKh7/TFt5ww1xRWXJK2dEXqkKzjG/HI1+5Blj/KyxoeEt2Lb/vXfqJm9vF74lYKXskEw1zKeaIqrupe6G+54LvypRi6OtOsPTAJBO+a1O7dWyWn0Xfc2RMOfLbU1AUHRZ8qMG7aCGI2poCbJy95Mkpl4pTVBiCu/cYIlgONovDIVTJ0MhCwXl5dUTMSq53y3OdQ9s7jL8FAJNws1SO+DevuqTxlLQDXKTygJt6pAyxPzh0C5WkZG2xOSgXpLnEk=",
				"MD5OfBody": "14857df459b7b5220ef0845358d9fad3",
				"Body": "third message"
			}
		]
	}
```
  
## 3. Delete a Message
After successfully processing a message, it must be deleted from the queue to prevent it from being received again: `aws sqs delete-message --queue-url=<QUEUE_URL> --receipt-handle=<RECEIPT_HANDLE>`

## 4. Send a Message

To add a new message to the queue: `aws sqs send-message --queue-url=<QUEUE_URL> --message-body=<MESSAGE_CONTENT>`
```json
{
		"MD5OfMessageBody": "86aff643e8d5dfe0c9759bf6cc8bbe84",
		"MessageId": "e9b918e3-356f-48f2-9d6b-d1f1d48f4f83"
	}
```

  
Response: Returns the `MessageId` and the `MD5OfMessageBody` for confirmation.
