# Tutorial: Deploying a Machine Learning Model Using AWS Lambda, S3, ECR, and API Gateway

## Objective
This guide provides a step-by-step approach to deploying a machine learning model using AWS Lambda, Amazon S3, Elastic Container Registry (ECR), and API Gateway. This setup eliminates AWS Lambda’s size limitations by packaging the model and dependencies inside a Docker container, enabling flexible and scalable deployment.

### The setup includes:
- ✅ Training and serializing a model locally for deployment.
- ✅ Packaging the model in a Docker container and storing it in Amazon ECR.
- ✅ Deploying a Lambda function using the Docker image.
- ✅ Setting up API Gateway to expose the model as a web API.

---

## 1️⃣ Develop the Model Locally
Before deploying, you need a trained model saved in a format that can be loaded later.

✅ **Steps:**
- Train the model using **scikit-learn**, **TensorFlow**, or **PyTorch**.
- Save the model as a serialized file (`.pkl` for scikit-learn, `.pt` for PyTorch, etc.).

✅ **Example (scikit-learn model save):**
```python
import pickle
from sklearn.ensemble import RandomForestClassifier

# Train model
model = RandomForestClassifier(n_estimators=100)
model.fit(X_train, y_train)

# Save model
with open("model.pkl", "wb") as f:
    pickle.dump(model, f)
```

---

## 2️⃣ Write the AWS Lambda Handler Function
AWS Lambda needs a Python script to load the model, process requests, and return predictions.

✅ **Example (`app.py`):**
```python
import pickle
import boto3
import json
import os

# Load model from S3
s3_client = boto3.client('s3')
bucket_name = os.getenv("S3_BUCKET")
model_key = "model.pkl"

def load_model():
    obj = s3_client.get_object(Bucket=bucket_name, Key=model_key)
    model = pickle.loads(obj['Body'].read())
    return model

model = load_model()

def lambda_handler(event, context):
    body = json.loads(event['body'])
    features = body['features']
    prediction = model.predict([features]).tolist()
    return {
        'statusCode': 200,
        'body': json.dumps({'prediction': prediction})
    }
```
✅ **What this does:**
- Downloads the model from **S3** when Lambda starts.
- Reads input data from the HTTP request.
- Runs predictions and returns results in JSON format.

---

## 3️⃣ Create a Docker Image for AWS Lambda
Since ML dependencies like **scikit-learn** exceed Lambda’s 50MB limit, we use **Docker** for packaging.

✅ **Create a Dockerfile:**
```dockerfile
# Use AWS Lambda base image for Python
FROM public.ecr.aws/lambda/python:3.9

# Install dependencies
RUN pip install boto3 scikit-learn pandas numpy

# Copy the model script
COPY app.py .  

# Set the Lambda function handler
CMD ["app.lambda_handler"]
```

---

## 4️⃣ Upload Resources to AWS

### 🔹 Upload Model to S3
✅ **Before deploying Lambda, upload the serialized model to Amazon S3:**
```bash
aws s3 cp model.pkl s3://your-bucket-name/
```

### 🔹 Build and Push the Docker Image to ECR
✅ **Authenticate with ECR:**
```bash
aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com
```
✅ **Create an ECR repository:**
```bash
aws ecr create-repository --repository-name ml-model-api
```
✅ **Build and tag the image:**
```bash
docker build -t ml-model-api .
docker tag ml-model-api:latest <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/ml-model-api:latest
```
✅ **Push the image to ECR:**
```bash
docker push <AWS_ACCOUNT_ID>.dkr.ecr.us-east-1.amazonaws.com/ml-model-api:latest
```

---

## 5️⃣ Deploy the Lambda Function Using the ECR Image
✅ **Create a Lambda function in AWS Console:**
1. Go to **AWS Lambda → Create Function**.
2. Select **Container Image** as the deployment package.
3. Choose the **Docker image** from Amazon ECR.
4. Set **memory and timeout** according to the model complexity.
5. Grant **S3 access permissions** via an **IAM role**.

✅ **IAM Policy for Lambda to access S3:**
```json
{
    "Effect": "Allow",
    "Action": ["s3:GetObject"],
    "Resource": "arn:aws:s3:::your-bucket-name/*"
}
```

---

## 6️⃣ Configure API Gateway

✅ **Steps to Expose the Model via API Gateway:**
1. Go to **AWS API Gateway → Create API**.
2. Select **HTTP API** and click **Build**.
3. Set the **Integration Type** to Lambda Function.
4. Attach the **Lambda function**.
5. Enable **API Key usage**:
   - Navigate to **API Gateway → Settings**.
   - Enable **API Key Requirement**.
   - Create an **API Key & Usage Plan**.
6. Deploy the API and obtain the **endpoint URL**.

✅ **Example request (using API key):**
```bash
curl -X POST "https://your-api-id.execute-api.us-east-1.amazonaws.com/predict" \
-H "x-api-key: YOUR_API_KEY" \
-H "Content-Type: application/json" \
-d '{"features": [5.1, 3.5, 1.4, 0.2]}'
```

---

## 7️⃣ Test the Deployment
✅ **Steps:**
- Deploy API Gateway and copy the **API endpoint**.
- Use **Postman, curl, or Python requests** to test the endpoint.
- Validate responses and fix any **permission errors**.

✅ **Example Test with Python:**
```python
import requests

url = "https://your-api-id.execute-api.us-east-1.amazonaws.com/predict"
headers = {"x-api-key": "YOUR_API_KEY", "Content-Type": "application/json"}
data = {"features": [5.1, 3.5, 1.4, 0.2]}

response = requests.post(url, json=data, headers=headers)
print(response.json())
```

---

## Why Use AWS ECR and Lambda Together?

### 1️⃣ Traditional AWS Lambda Deployment Constraints
✅ **AWS Lambda ZIP limitations:**
- **50MB compressed**, **250MB uncompressed**.
- Not ideal for **machine learning models requiring heavy dependencies**.

### 2️⃣ Docker Containers with AWS Lambda
✅ **Advantages of Docker for ML Model Deployment:**
- Bypasses the **50MB ZIP file limit** (Docker images can be up to **10GB**).
- Custom runtime environments (**install ML libraries like TensorFlow, PyTorch**).
- Preloaded dependencies, **eliminating the need for Lambda Layers**.

### 3️⃣ Using API Gateway for Secure Model Access
✅ **API Gateway provides:**
- A **secure HTTP endpoint** for model inference.
- Authentication via **API keys** or **IAM policies**.
- Integration with **Lambda** for serverless execution.

---
