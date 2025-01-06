# Setting Up S3, Lambda, and SNS with the AWS Management Console

Cloud computing has revolutionized the way we build and deploy applications, particularly with event-driven architectures. Imagine this: you upload a file to an S3 bucket, and instantly, a notification is sent to your email or mobile device. This seamless integration of services, like S3, Lambda, and SNS, simplifies workflows and automates tasks, saving you valuable time and effort.

In this guide, I’ll show you how to set up a fully functional serverless solution using the AWS Management Console. We’ll create an S3 bucket to store files, a Lambda function to process events, and an SNS topic to send notifications. Best of all, you don’t need to write extensive boilerplate code or configure complex setups—just a few clicks, and you’re ready to go. Whether you're a seasoned developer or new to AWS, this walkthrough will help you unlock the potential of serverless technology.

## Overview
This guide demonstrates how to create a serverless architecture using AWS services:
- **S3**: For file storage.  
- **SNS**: For sending notifications.  
- **IAM**: For role-based access control.
- **Lambda**: For event-driven processing.

By the end of this tutorial, you’ll have a working setup that sends an email notification whenever a file is uploaded to an S3 bucket.

---

## Prerequisites
- An AWS account.
- Basic familiarity with the AWS Management Console.

---

### Step 1: Create an S3 Bucket
1. Log in to the **AWS Management Console**.
2. Navigate to the **S3 Console**.
3. Click **"Create bucket"**.
4. Provide the following details:
   - **Bucket name**: Choose a unique name, e.g., `my-unique-bucket-name`.
   - **AWS Region**: Select your preferred region.
5. Leave the default settings (or customize them as needed) and click **"Create bucket"**.

---

### Step 2: Create an SNS Topic
1. Go to the **SNS Console** by searching for "SNS" in the AWS Console.
2. Click **"Topics"** in the left sidebar and then **"Create topic"**.
3. Configure the topic:
   - **Type**: Standard.
   - **Name**: `S3UploadNotifications`.
4. Click **"Create topic"**.
5. Add a subscription:
   - Click your topic name, then **"Create subscription"**.
   - **Protocol**: Email.
   - **Endpoint**: Enter your email address.
6. Check your email inbox and confirm the subscription.

---

### Step 3: Create an IAM Role
1. Log in to the **AWS Management Console**.
2. Navigate to the **IAM Console**.
3. Click **"Policy"** in the left sidebar and then **"Create policy"**.
4. Choose the **JSON tab** and replace the default policy with the following:
   ```json
    {
      "Version": "2012-10-17",
      "Statement": [
         {
            "Effect": "Allow",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::my-unique-bucket-name/*"
         },
         {
            "Effect": "Allow",
            "Action": "sns:Publish",
            "Resource": "<YOUR_SNS_TOPIC_ARN>"
         }
      ]
    }
    ```
5. Replace `<YOUR_SNS_TOPIC_ARN>` with the ARN of the SNS topic you will create later.
6. Click **"Review policy"**.
7. Provide a **Name** (for example `s3-lambda-sns-policy`) and **Description** for the policy, then click **"Create policy"**.
8. Click **"Roles"** in the left sidebar and then **"Create role"**.
9. Choose **"Lambda"** as the service that will use this role and click **"Next: Permissions"**.
10. Search for the policy you created earlier (`s3-lambda-sns-policy`) and select it.
11. Click **"Next: Tags"** (optional) and then **"Next: Review"**.
12. Provide a **Role name** (e.g., `S3LambdaSNSTriggerRole`) and **Description** for the role, then click **"Create role"**.
13. Note the **Role ARN** for later use.

---

### Step 4: Create a Lambda Function
1. Navigate to the **Lambda Console**.
2. Click **"Create function"**.
3. Choose **"Author from scratch"** and provide the following details:
   - **Function name**: `S3EventToSNS`.
   - **Runtime**: Python 3.x.
   - **Execution role**: Select **"Use an existing role"** and choose the role you created earlier (`S3LambdaSNSTriggerRole`).
4. Click **"Create function"**.
5. Replace the default function code with:
   ```python
   import json
   import boto3

   sns_client = boto3.client('sns')
   SNS_TOPIC_ARN = '<Your-SNS-Topic-ARN>'  # Replace with your SNS Topic ARN

   def lambda_handler(event, context):
    # Extract details from the S3 event
    bucket = event['Records'][0]['s3']['bucket']['name']
    key = event['Records'][0]['s3']['object']['key']

    print(f"New object uploaded: {key} in bucket {bucket}")

    # Create a notification message
    message = f"File uploaded: {key} in bucket {bucket}"

    # Publish the message to the SNS topic
    sns_client.publish(
        TopicArn=SNS_TOPIC_ARN,
        Message=message,
        Subject="New S3 Upload Notification"
    )

    print(f"Notification sent for {key}")
    return {
        'statusCode': 200,
        'body': json.dumps('Notification sent successfully!')
    }

    ```
6. Replace `<Your-SNS-Topic-ARN>` with the ARN of the SNS topic you created earlier.

---

### Step 5: Configure S3 to Trigger the Lambda Function

1. Navigate to your **S3 bucket** in the S3 Console.
2. Open the **"Properties"** tab.
3. Scroll to **"Event notifications"** and click **"Create event notification"**.
4. Provide the following details:
      * Name: `S3UploadTrigger`.
      * **Event types**: Select "All object create events".
      * Destination: Choose **"Lambda function"** and select the `S3EventToSNS` function.
5. Save the configuration.

---

### Step 6: Test the Setup
1. Upload a file to your S3 bucket.
2. Check your email inbox for the notification..
3. Check lambda logs for the execution details.
   - Navigate to the **Lambda Console** and check the **Monitoring** tab for invocation metrics.
   - You should see an increase in the invocation count.
4. You can also view the **CloudWatch Logs** for the function to see the execution details.

### Final Thoughts
Congratulations! You’ve successfully created a serverless application using S3, Lambda, and SNS. This solution is scalable, cost-efficient, and automates workflows effectively.
