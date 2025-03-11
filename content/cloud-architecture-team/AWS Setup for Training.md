# Tutorial: AWS SageMaker Setup for AI Engineering

## Objective
This guide provides step-by-step instructions on setting up AWS SageMaker Studio in a secure and structured manner. It covers networking, security, storage management, and compute selection to ensure efficient and scalable AI development on AWS.

### The setup includes:
- ✅ Creating a SageMaker Domain to manage users and applications.
- ✅ Configuring a secure VPC environment with proper access control.
- ✅ Using S3 & EFS for data and model storage while following best practices.
- ✅ Optimizing compute resources by choosing the right SageMaker instance types.

---

## 1️⃣ Setting Up SageMaker Studio

### 📌 Creating a SageMaker Domain
✅ **Domain Name:** `sagemaker-capstone-domain`
- This is the core environment for managing SageMaker Studio users and applications.
- Users created under this domain can access Jupyter notebooks, storage, and ML tools.

### 📌 Creating a VPC for SageMaker
✅ **Why use a VPC?**
- Ensures SageMaker runs in an isolated network for security and control.
- Allows custom network configurations, such as private access to AWS services.
- You define subnets and security groups (SGs) to manage inbound/outbound access.

### 📌 Configuring VPC Endpoints
| VPC Endpoint               | Purpose                                        |
|---------------------------|------------------------------------------------|
| **S3 Gateway Endpoint**    | Allows private S3 access without internet routing. |
| **SageMaker API & Runtime Endpoints (Interface)** | Enables SageMaker to communicate internally. |
| **CloudWatch Logs Endpoint** | Allows logging from SageMaker to CloudWatch for monitoring. |
| **S3 Interface Endpoint** | Direct connection to S3 for certain operations. |

🔹 **Issue encountered:**
- Incorrect inbound/outbound security group rules caused SageMaker to be unable to access AWS services like:
  - STS (`aws sts get-caller-identity` was hanging)
  - S3 (`aws s3 ls s3://your-bucket` was failing)
  - CloudWatch Logs (logs were not updating)

✅ **Solution:**
- Ensure **HTTPS (port 443) is open** and the security group allows traffic to necessary AWS services.

---

## 2️⃣ Managing Users & Workspaces

### 📌 Creating User Profiles
- You created two user profiles (yourself and a teammate).
- This allows multiple users to work in the same SageMaker Studio environment while maintaining workspace isolation.

### 📌 Creating & Managing SageMaker Spaces
- Spaces provide isolated work environments for different users.
- Managing spaces by deleting unused ones to optimize resources.

✅ **To delete a SageMaker space:**
```bash
aws sagemaker delete-space --domain-id <DOMAIN_ID> \
  --space-name "<SPACE_NAME>" \
  --region <AWS_REGION> --profile <AWS_PROFILE>
```
⚠ **Note:** Before switching instances, stop the running space first!

---

## 3️⃣ Uploading Data to S3

✅ **To upload training data from local to S3:**
```bash
aws s3 sync /path/to/local/files s3://<S3_BUCKET_NAME>/training_data \
  --exclude "*/.DS_Store" --profile <AWS_PROFILE>
```

### 📌 S3 Bucket Structure for Model Sharing
| **Account**          | **Purpose** |
|----------------------|------------|
| **Training Account** | Stores datasets, intermediate models, and final trained models. The final models are placed in the `production` folder, which is shared with the **Deployment Account**. |
| **Deployment Account** | Accesses trained models from the `production` folder in S3 and uses them for inference in production environments. |


---

## 4️⃣ Storage in SageMaker: EFS vs. S3

| Storage Type | Purpose | Temporary or Permanent? |
|-------------|---------|------------------------|
| **Amazon EFS (Elastic File System)** | Stores SageMaker Studio files (e.g., Jupyter notebooks, scripts). | **Permanent (until manually deleted)** |
| **Amazon S3** | Stores datasets, trained models, and logs. | **Permanent (unless lifecycle rules delete objects)** |

🔹 **EFS in SageMaker:**
- Used for **persistent storage** within SageMaker Studio.
- **Not temporary** – files remain even if instances are stopped.
- Default storage size: **5GB per user** but can be expanded if needed.
- SageMaker automatically creates an **EFS for Studio use** with a corresponding **Security Group (SG)**.

✅ **Best Practice:**
- **Use S3** for storing datasets & models, and **EFS** for code & working files.

---

## 5️⃣ Choosing the Right JupyterLab Instance Size

- **JupyterLab does not perform training** but is used for scripting, preprocessing, and job launching.

| Use Case | Suggested Instance Type | Reason |
|---------|-----------------|--------|
| **Light usage** (basic scripting, launching training jobs, small data manipulations) | `ml.t3.medium` or `ml.t3.large` | Low cost, good balance of CPU & RAM |
| **Moderate usage** (some preprocessing, moderate dataset size) | `ml.m5.large` | More memory for larger DataFrames |
| **Frequent visualization & large datasets** | `ml.m5.xlarge` or `ml.c5.xlarge` | More RAM & CPU for large data |

⚠ **Note:** GPU instances may require **manual quota increases** in new AWS accounts.

✅ **To request a quota increase for GPU instances:**
1. Go to **AWS Service Quotas**.
2. Search for **SageMaker GPU instance types**.
3. Click **Request Quota Increase** and submit a request.

---

## 6️⃣ Key Takeaways & Best Practices

### ✅ **Security & Access Control**
- Use **VPC Endpoints** to restrict internet access.
- Ensure **HTTPS (443) is open** to AWS services like S3, STS, CloudWatch.
- Limit SageMaker instance access with **IAM roles and least privilege policies**.

### ✅ **Storage Management**
- Use **EFS** for notebooks & scripts (**persistent storage**).
- Use **S3** for datasets & trained models (**scalable storage**).

### ✅ **Optimization**
- **Delete unused spaces** to free up resources.
- **Choose appropriate JupyterLab instance sizes** to reduce costs.
- **Request GPU quota increases manually** if needed.

---


### 🔗 **Resource:**
- [AI Engineering with AWS SageMaker: Crash Course for Beginners!](https://aws.amazon.com/sagemaker/)

---
