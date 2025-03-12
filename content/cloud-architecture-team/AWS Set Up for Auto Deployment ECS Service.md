# Tutorial: Automating ECS Deployment with AWS Lambda, EventBridge, and S3

## Objective
This guide provides step-by-step instructions for setting up an automated deployment pipeline using AWS Lambda, EventBridge, and S3. The goal is to automatically redeploy an ECS service whenever a new model file is uploaded to an S3 bucket.

---

## 1️⃣ Overview of the Automation Workflow
✅ **Trigger:** A new model file (e.g., `production/latest.pkl`) is uploaded or created to S3.  
✅ **EventBridge:** Captures the S3 event and forwards it to the ECS account.  
✅ **Lambda Function:** Receives the event and triggers ECS service redeployment.  
✅ **ECS Service:** Pulls the updated model and redeploys the inference container.  

---

## 2️⃣ Enabling EventBridge Notifications for S3
To capture S3 file updates, configure EventBridge to listen for `Object Created` events.
```bash
aws s3api put-bucket-notification-configuration \
    --bucket <your-s3-bucket-name> \
    --notification-configuration '{
        "EventBridgeConfiguration": {}
    }' --profile <your-aws-profile>
```
🔹 **What this does:**
- Enables EventBridge integration for your S3 bucket.
- Any object creation event in S3 will now be sent to EventBridge.

---

## 3️⃣ Setting Up Cross-Account Event Forwarding (If Needed)
If your S3 bucket and ECS service are in different AWS accounts, allow cross-account event forwarding.
```bash
aws events put-permission \
    --event-bus-name default \
    --action "events:PutEvents" \
    --principal "<ECS_AWS_ACCOUNT_ID>" \
    --statement-id "AllowECSAccountToPutEvents" \
    --profile <your-aws-profile>
```
🔹 **What this does:**
- Grants permission for the ECS AWS account (deployment) to receive events from the S3 account (training).

---

## 4️⃣ Creating an EventBridge Rule to Capture S3 Upload Events
Define an EventBridge rule to trigger on `Object Created` events for a specific file (`production/latest.pkl`).

### **Rule JSON Configuration:**
```json
{
  "source": ["aws.s3"],
  "detail-type": ["Object Created"],
  "detail": {
    "bucket": {
      "name": ["<your-s3-bucket-name>"]
    },
    "object": {
      "key": ["production/latest.pkl"]
    }
  }
}
```

📌 **To create the rule via CLI:**
```bash
aws events put-rule \
    --name "S3ModelUpdateRule" \
    --event-pattern file://event-pattern.json \
    --state ENABLED \
    --profile <your-aws-profile>
```

---

## 5️⃣ Deploying a Lambda Function to Trigger ECS Deployment
Now, create an AWS Lambda function to restart the ECS service when a new model file is uploaded.

### **Python Code for Lambda (`lambda_function.py`)**
```python
import boto3
import json
import os

ecs_client = boto3.client("ecs")

CLUSTER_NAME = os.getenv("ECS_CLUSTER_NAME")
SERVICE_NAME = os.getenv("ECS_SERVICE_NAME")

def lambda_handler(event, context):
    print("Received event:", json.dumps(event, indent=2))

    response = ecs_client.update_service(
        cluster=CLUSTER_NAME,
        service=SERVICE_NAME,
        forceNewDeployment=True
    )

    print("ECS Service Update Response:", response)
    return {"statusCode": 200, "body": json.dumps("ECS service redeployed successfully.")}
```
🔹 **What this does:**
- Listens for EventBridge events.
- Calls `ecs.update_service()` to force a new deployment of the ECS service.

### **Deploying the Lambda Function**

#### 1️⃣ Create the Lambda function:
```bash
aws lambda create-function \
    --function-name ECSAutoRedeploy \
    --runtime python3.9 \
    --role <LAMBDA_EXECUTION_ROLE> \
    --handler lambda_function.lambda_handler \
    --code S3Bucket=<your-lambda-code-bucket>,S3Key=lambda_function.zip \
    --profile <your-aws-profile>
```
#### 2️⃣ Set Environment Variables for ECS Cluster & Service Name:
```bash
aws lambda update-function-configuration \
    --function-name ECSAutoRedeploy \
    --environment "Variables={ECS_CLUSTER_NAME='your-cluster',ECS_SERVICE_NAME='your-service'}"
```
#### 3️⃣ Grant Lambda permission to call ECS:
```bash
aws iam attach-role-policy \
    --role-name <LAMBDA_EXECUTION_ROLE> \
    --policy-arn arn:aws:iam::aws:policy/AmazonECS_FullAccess
```

---

## 6️⃣ Linking Lambda to the EventBridge Rule
Ensure EventBridge calls your Lambda function whenever an event is triggered.
```bash
aws events put-targets \
    --rule S3ModelUpdateRule \
    --targets "Id"="1","Arn"="arn:aws:lambda:<region>:<account-id>:function:ECSAutoRedeploy"
```
🔹 **What this does:**
- Links `S3ModelUpdateRule` to your `ECSAutoRedeploy` Lambda function.
- Ensures ECS redeploys when `production/latest.pkl` is uploaded.

---

## 7️⃣ Testing the Setup
### **Test 1: Upload a Model to S3**
Manually upload a new model file:
```bash
aws s3 cp latest.pkl s3://<your-s3-bucket>/production/latest.pkl
```
🔹 **Expected result:**
- EventBridge triggers Lambda
- Lambda calls ECS to redeploy
- ECS pulls the latest model and restarts the service

### **Test 2: Manually Invoke Lambda**
```bash
aws lambda invoke \
    --function-name ECSAutoRedeploy \
    --log-type Tail \
    output.log
```
🔹 **Expected output:**
```json
{
  "statusCode": 200,
  "body": "\"ECS service redeployed successfully.\""
}
```

---

## 8️⃣ Best Practices & Security Considerations
✅ **Use Least Privilege IAM Roles**
- Ensure Lambda has only ECS update permissions (`ecs:UpdateService`).
- Limit S3 event permissions to necessary objects.

✅ **Use Versioned S3 Buckets for Model Files**
- Prevent accidental overwrites by enabling S3 versioning.

✅ **Monitor EventBridge & Lambda Logs**
- Use AWS **CloudWatch Logs** for debugging event triggers and ECS updates.

---
