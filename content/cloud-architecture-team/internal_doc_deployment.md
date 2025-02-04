---
title: "Cloud Arichtechture Team Documentation"
date: 2025-01-30
---

# Steps for Setting Up the Web Team Infrastructure

## 1. Set Up OIDC
1. **Create Trust Relationship**:
   - Establish a trust relationship between AWS and GitHub by configuring an OpenID Connect (OIDC) identity provider.
   - This allows GitHub workflows to securely assume IAM roles to access AWS resources without requiring long-term credentials.

2. **Create IAM Role**:
   - Create an IAM role that can be assumed by GitHub workflows.
   - Specify the allowed GitHub repository and workflow in the trust policy.
   - Define permissions for accessing the S3 bucket and other services with proper scoping (e.g., read/write access for specific buckets).

3. **Configure Local YAML File**:
   - Write a YAML file locally that includes:
     - The S3 bucket location.
     - The IAM role to assume.
     - Any necessary configurations for GitHub workflows to deploy updates to the S3 bucket.

4. **Set S3 Bucket Policy**:
   - Update the S3 bucket policy to allow access only from CloudFront and the IAM role created for GitHub workflows.
   - Deny all public access to the S3 bucket.

---

## 2. Create a CloudFront Distribution
1. **Set Up CloudFront Distribution**:
   - Use CloudFront to serve website content from the private S3 bucket securely.
   - Ensure the S3 bucket is configured to block public access and is accessible only via the CloudFront distribution.

2. **Enable CloudFront Logging**:
   - Configure CloudFront to log access requests and store them in an S3 bucket named **dsc180-internal-docs-log**.

3. **Update S3 Bucket Policy**:
   - Copy the CloudFront-generated bucket policy and apply it to the S3 bucket. This ensures that only the CloudFront distribution can access the bucket.

4. **Set Up Cache Invalidation**:
   - Create a process to invalidate the CloudFront cache when the S3 bucket content is updated. This ensures that the latest content is served to end users.

---

## Answers to Key Questions

### **Q1: Why Use CloudFront for a Website Instead of Public Access for S3?**
- **Improved Performance**:
  - CloudFront caches content at edge locations globally, reducing latency for users.
- **Enhanced Security**:
  - S3 bucket access is restricted to CloudFront, preventing direct public access.
- **Logging & Monitoring**:
  - CloudFront logs can be stored in the **dsc180-internal-docs-log** S3 bucket for tracking and analysis.
- **Scalability**:
  - CloudFront can handle high traffic volumes and scale automatically.

### **Q2: Why Use GitHub Actions Instead of Directly Pushing to S3?**
- **Automation**:
  - GitHub Actions automates deployment, ensuring consistent and reliable updates.
- **Security**:
  - By using OIDC, GitHub workflows assume a temporary IAM role, avoiding long-term credentials.
- **CI/CD Integration**:
  - Workflows integrate seamlessly into the development pipeline, enabling testing, building, and deployment in one process.
- **Efficiency**:
  - GitHub Actions allows for conditional deployments and multi-step workflows.
