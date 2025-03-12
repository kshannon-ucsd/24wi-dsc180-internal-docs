# Tutorial: Automating Internal Documentation Deployments with GitHub OIDC and AWS S3

## Objective
This guide provides step-by-step instructions on how to securely automate deployments from GitHub to AWS S3 using GitHub OpenID Connect (OIDC), eliminating static credentials while ensuring secure and controlled access to AWS resources.

### The setup includes:
- ✅ Using GitHub OIDC for authentication (instead of AWS access keys).
- ✅ Deploying an internal static site to a private S3 bucket.
- ✅ Serving the site via AWS CloudFront while keeping the S3 bucket private.

---

## Benefits of This Design
### ✅ Security
- Eliminates static credentials in GitHub Actions, reducing risk exposure.
- Restricts direct access to the S3 bucket, allowing access only via IAM roles and CloudFront.

### ✅ Automation
- Enables seamless deployment from GitHub to AWS on every code push.
- Ensures the latest internal documentation is always synchronized with the repository.

### ✅ Performance & Cost Optimization
- Uses CloudFront as a CDN to reduce direct S3 access costs.
- Provides caching, reducing latency for end users.

---

## Step-by-Step Setup

### 1️⃣ Set Up GitHub OIDC with AWS
GitHub OIDC allows workflows to securely assume an AWS IAM role without requiring long-lived credentials.

#### 1.1 Configure AWS to Trust GitHub as an Identity Provider
1. Navigate to **AWS IAM Console → Identity Providers**.
2. Click **Add Provider** and select:
   - **Provider Type:** OpenID Connect (OIDC)
   - **Provider URL:** `https://token.actions.githubusercontent.com`
   - **Audience:** `sts.amazonaws.com`
3. Save and verify that the new identity provider is added.

#### 1.2 Create an IAM Role for GitHub Actions
1. In AWS IAM, create a new IAM Role with:
   - **Trusted Entity:** GitHub OIDC
   - **Permissions:** Grant S3 Write Access to the specific bucket.
   - **Conditions:** Restrict access to your GitHub repository.

✅ **Use the following trust policy** to limit role assumption to your specific repo:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Federated": "arn:aws:iam::<AWS_ACCOUNT_ID>:oidc-provider/token.actions.githubusercontent.com"
      },
      "Action": "sts:AssumeRoleWithWebIdentity",
      "Condition": {
        "StringEquals": {
          "token.actions.githubusercontent.com:sub": "repo:<YOUR_ORG>/<YOUR_REPO>:ref:refs/heads/main"
        }
      }
    }
  ]
}
```

✅ **Attach a permissions policy granting access to S3:**
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "s3:PutObject",
        "s3:ListBucket"
      ],
      "Resource": [
        "arn:aws:s3:::<YOUR_BUCKET_NAME>",
        "arn:aws:s3:::<YOUR_BUCKET_NAME>/*"
      ]
    }
  ]
}
```

---

### 2️⃣ Configure GitHub Actions for Automated Deployment

✅ **Create a GitHub Actions Workflow (`.github/workflows/deploy.yml`)**:
```yaml
name: Deploy to AWS S3

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: ubuntu-latest
    permissions:
      id-token: write
      contents: read

    steps:
      - name: Checkout Code
        uses: actions/checkout@v3

      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v2
        with:
          role-to-assume: arn:aws:iam::<AWS_ACCOUNT_ID>:role/<YOUR_ROLE_NAME>
          role-session-name: GitHubActions
          aws-region: <YOUR_AWS_REGION>

      - name: Deploy to S3
        run: |
          aws s3 sync ./site s3://<YOUR_BUCKET_NAME> --delete
```

✅ **Why This Works?**
- GitHub Actions authenticates via OIDC and assumes the AWS IAM role.
- The workflow syncs the website files from the repo to S3.

---

### 3️⃣ Configure S3 for Secure Access

#### Make the Bucket Private
In **S3 Console**, enable:
- ✅ **Block all public access**
- ✅ **Disable ACLs**

This ensures that objects are only accessible via CloudFront.

#### Set S3 Bucket Policy for CloudFront
After CloudFront setup (next step), update the bucket policy:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "cloudfront.amazonaws.com"
      },
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::<YOUR_BUCKET_NAME>/*",
      "Condition": {
        "StringEquals": {
          "AWS:SourceArn": "arn:aws:cloudfront::<AWS_ACCOUNT_ID>:distribution/<CLOUDFRONT_DIST_ID>"
        }
      }
    }
  ]
}
```

---

### 4️⃣ Set Up CloudFront for Public Access
CloudFront serves the static website securely while keeping the S3 bucket private.

#### 4.1 Create CloudFront Distribution
1. In AWS Console, go to **CloudFront → Create Distribution**.
2. Configure:
   - **Origin Domain:** Select your S3 bucket.
   - **Origin Access:** Do NOT allow public access to the S3 bucket.
   - **Caching:** Enable Cache Invalidation for faster updates.
3. Copy the **CloudFront Distribution URL** as your website’s public URL.

#### 4.2 Configure Automatic Cache Invalidation
✅ **In GitHub Actions, add a step after deployment:**
```yaml
- name: Invalidate CloudFront Cache
  run: |
    aws cloudfront create-invalidation --distribution-id <CLOUDFRONT_DIST_ID> --paths "/*"
```

This ensures that every update reflects immediately.

---

## Final Summary

### ✅ What We Achieved
- **Secure Deployments:** Eliminated static credentials using GitHub OIDC.
- **Automated Workflow:** GitHub Actions updates S3 on every commit.
- **Private Storage:** S3 is fully private, accessed only via CloudFront.
- **Optimized Delivery:** CloudFront caches and serves the latest content globally.

### 📌 Next Steps
- Test deployment by pushing a change to `main`.
- Verify that S3 is private and only accessible via CloudFront.
- Monitor CloudFront logs to analyze access patterns.

### 🔗 Resources
- [How to Connect GitHub Workflows and AWS with OIDC (OpenID Connect)](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_roles_providers_create_oidc.html)
