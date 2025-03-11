# Tutorial: Deploying a Machine Learning Model Using AWS ECS, ECR, S3, and ALB

## Objective
This guide outlines the steps for deploying a machine learning model using AWS Elastic Container Service (ECS), Amazon Elastic Container Registry (ECR), Amazon S3, and an Application Load Balancer (ALB). This setup ensures scalability, flexibility, and high availability, making it more suitable for high-traffic and large-model deployments compared to AWS Lambda + API Gateway.

### The setup includes:
- ✅ Containerizing the model with Docker and pushing it to Amazon ECR.
- ✅ Deploying the model on ECS using Fargate (serverless) or EC2 (manual scaling).
- ✅ Using ALB to route external traffic to ECS tasks securely.
- ✅ Loading the ML model dynamically from S3 to keep containers stateless.

---

## 1️⃣ Preparing the Model and Uploading to S3
Since ECS containers are stateless, the ML model should be stored externally (not inside the container).

✅ **Upload the model to S3:**
```bash
aws s3 cp model.keras s3://your-bucket-name/model.keras
```

✅ **Modify the Flask app to load the model from S3 on startup:**
```python
import boto3
import os
from tensorflow.keras.models import load_model

S3_BUCKET = "your-bucket-name"
MODEL_FILE = "model.keras"
LOCAL_MODEL_PATH = f"/app/{MODEL_FILE}"

# Download model if not already present
if not os.path.exists(LOCAL_MODEL_PATH):
    s3 = boto3.client("s3")
    print(f"📥 Downloading model from S3: {S3_BUCKET}/{MODEL_FILE}")
    s3.download_file(S3_BUCKET, MODEL_FILE, LOCAL_MODEL_PATH)

# Load model
model = load_model(LOCAL_MODEL_PATH)
print("✅ Model loaded successfully.")
```

---

## 2️⃣ Creating the Docker Image
A Dockerfile is used to package the application with dependencies.

✅ **Dockerfile:**
```dockerfile
# Use an official lightweight Python image
FROM python:3.12-slim

# Set the working directory in the container
WORKDIR /app

# Copy the application files into the container
COPY . /app

# Install dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Expose Flask's default port
EXPOSE 8080

# Start the Flask app
CMD ["gunicorn", "-b", "0.0.0.0:8080", "app:app"]
```

---

## 3️⃣ Building and Testing Locally

✅ **Build the Docker image:**
```bash
docker build -t flask-ecs-app .
```
✅ **Run it locally:**
```bash
docker run -p 8080:8080 flask-ecs-app
```
✅ **Test the API:**
```bash
curl -X GET http://127.0.0.1:8080/
```

---

## 4️⃣ Pushing the Image to AWS Elastic Container Registry (ECR)

✅ **1. Create an ECR Repository:**
```bash
aws ecr create-repository --repository-name flask-ecs-app
```

✅ **2. Authenticate Docker with AWS:**
```bash
aws ecr get-login-password --region your-region | docker login --username AWS --password-stdin your-aws-account-id.dkr.ecr.your-region.amazonaws.com
```

✅ **3. Tag & Push Docker Image to ECR:**
```bash
docker tag flask-ecs-app:latest your-aws-account-id.dkr.ecr.your-region.amazonaws.com/flask-ecs-app:latest
docker push your-aws-account-id.dkr.ecr.your-region.amazonaws.com/flask-ecs-app:latest
```

---

## 5️⃣ Deploying the Model Using AWS ECS

✅ **1. Create an ECS Task Definition**
- Go to **ECS Console → Task Definitions → Create New Task Definition**.
- Choose **Fargate (serverless)** or **EC2 (manual scaling)**.
- Add a container definition:
  - **Image:** ECR URL (`your-aws-account-id.dkr.ecr.your-region.amazonaws.com/flask-ecs-app:latest`).
  - **Port Mapping:** 8080 (match Flask app).

✅ **2. Create an ECS Cluster**
- Go to **ECS Console → Clusters → Create Cluster**.
- Choose **Fargate (for serverless)** or **EC2 (for manual scaling)**.

✅ **3. Deploy an ECS Service**
- Go to **your Cluster → Create Service**.
- **Launch Type:** Choose **Fargate (recommended)**.
- **Task Definition:** Select the one created earlier.
- **VPC & Subnets:** Select AWS networking settings.
- **Auto Scaling:** Adjust based on expected traffic.

---

## 6️⃣ Configuring the Application Load Balancer (ALB) for Public Access
To expose the ECS service securely, we use ALB (Application Load Balancer) to route traffic.

✅ **Steps to Set Up ALB**
1. **Create an ALB** in **AWS Console → EC2 → Load Balancers**.
2. **Attach the ALB to a Public Subnet**.
3. **Create a Target Group**:
   - Register the **ECS service** as a target.
   - Use **port 8080** (Flask app).
4. **Configure Listeners**:
   - **Port 80** → Redirect to **8080** (for HTTP traffic).
   - **Port 443** → Enable **SSL** (optional for HTTPS).
5. **Set Up SSL Certificate**
   - **You cannot get a free SSL certificate directly for ALB.**
   - Instead, **purchase a domain from Route 53**.
   - **Use AWS Certificate Manager (ACM)** to generate an SSL certificate.
   - **Create subdomains** for different models (e.g., `resnet.example.com`, `catboost.example.com`). Details you can check documentation on how frontend and backend are connected

5. **Deploy the ECS Service with ALB integration**.

✅ **Get the ALB URL and test the API:**
```bash
curl -X POST -F "image=@test.jpg" http://your-alb-dns-name:8080/predict
```

---

## 7️⃣ Final Summary: Why ECS + ALB is Better Than Lambda + API Gateway

| Feature              | ECS + ALB                           | Lambda + API Gateway           |
|----------------------|----------------------------------|------------------------------|
| **Model Size Limit** | No limit (can pull large models from S3) | 50 MB (Lambda) |
| **Cold Start Latency** | Lower (Containers are always running) | Higher (Cold starts in Lambda) |
| **Scalability**      | Auto Scales with Load Balancer   | Auto Scales (but may hit throttling) |
| **Stateful Processing** | ✅ Supports persistent connections | ❌ Stateless only |
| **Custom Dependencies** | ✅ Full control via Docker | ❌ Limited to Lambda Layers |
| **Long-Running Inference** | ✅ Can handle batch & real-time inference | ❌ Hard limit of 15 minutes per execution |

📌 **Key Takeaways**
- ✅ **ECS + ALB** is more suitable for **large ML models** and **high-traffic applications**.
- ✅ **Lambda + API Gateway** is better for **small functions**, **low-latency**, and **event-driven workflows**.
- ✅ **Using ALB** improves performance and allows better scaling.
- ✅ **Keeping the model in S3** makes ECS containers stateless and flexible.

📌 **Next Steps**
- **Monitor ECS task performance** using CloudWatch Logs.
- **Set up Auto Scaling** for ECS tasks based on demand.
- **Implement HTTPS for ALB** with SSL certificates.
- **Optimize costs** by switching between **EC2 and Fargate** based on usage.

---

### 🔗 **Resource:**
- [GitHub Repository](https://github.com/kshannon-ucsd/24wi-dsc180-profile/tree/ECS-ResNet)
