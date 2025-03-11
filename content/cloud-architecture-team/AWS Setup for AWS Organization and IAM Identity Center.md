# Tutorial: Setting Up AWS Organization and IAM Identity Center

## Objective
This guide provides a step-by-step approach to configuring AWS Organizations and IAM Identity Center for secure access management across multiple AWS accounts. The setup ensures least privilege access, centralized authentication, and role-based control for different teams.

### The setup includes:
- ✅ Organizing AWS accounts (Internal Note, Training, Deployment) within AWS Organizations.
- ✅ Configuring IAM Identity Center for secure authentication and role-based access.
- ✅ Using AWS SSO (`aws configure sso`) to eliminate static credentials.
- ✅ Enhancing security with MFA, access monitoring, and logging.

---

## 1️⃣ AWS Account Structure & Organization Setup
To efficiently manage multiple AWS environments, accounts are structured under AWS Organizations.

### 🔹 AWS Accounts Created
- **Internal Note Account** → Manages internal documentation and resources.
- **Training Account** → Dedicated for model training and data processing.
- **Deployment Account** → Hosts deployed models, inference endpoints, and APIs.

### 🔹 AWS Organizations Setup
✅ Steps to Set Up AWS Organizations:
1. Go to **AWS Organizations Console** and create a new organization.
2. Invite existing AWS accounts or create new ones under the organization root.

---

## 2️⃣ Configuring IAM Identity Center for User Management
IAM Identity Center allows centralized authentication and authorization for AWS accounts.

### 🔹 Creating User Groups
✅ Steps:
1. Go to **AWS IAM Identity Center Console**.
2. Create user groups based on roles:
   - **Admins** → Full access for AWS account management.
   - **Data Scientists** → Limited access for model training and deployment.
3. Assign teammates to their respective groups.

### 🔹 Mapping Groups to AWS Accounts
✅ Steps:
1. Go to **IAM Identity Center → Assign Users**.
2. Select the AWS account (**Internal Note, Training, Deployment**).
3. Assign groups with corresponding permissions.

✅ **Example IAM Identity Center Group Access:**

| Group               | Assigned AWS Accounts  | Permissions                          |
|--------------------|----------------------|--------------------------------------|
| **adminTraining**    | Training Account      | Full admin access to training workloads. |
| **adminDeployment**  | Deployment Account    | Full admin access to deployment resources. |
| **adminInternalNote** | Internal Note Account | Full admin access to internal documentation. |
| **dataScientists**   | Training  | Limited access to model training. |

---

## 3️⃣ Implementing Secure Authentication via AWS SSO
AWS SSO eliminates static credentials, ensuring secure role-based authentication.

✅ Steps to Set Up AWS SSO:
```bash
aws configure sso
```
1. Follow the interactive prompt to authenticate with AWS Identity Center.
2. Select the appropriate AWS account and role.
3. Use temporary credentials for authentication instead of long-lived access keys.

✅ Example to Assume a Role in a Specific Profile:
```bash
aws sso login --profile training
aws sso login --profile deployment
```

### ✅ Why Use AWS SSO?
- Eliminates the need for static IAM credentials.
- Provides temporary session-based access.
- Ensures role-based authentication.

---

## 4️⃣ Enforcing Multi-Factor Authentication (MFA) for Extra Security

### 🔹 Why MFA?
- Prevents unauthorized access in case of compromised credentials.
- Adds an extra layer of security to AWS logins.

✅ Steps to Enforce MFA in AWS Identity Center:
1. Go to **IAM Identity Center → Settings → Authentication**.
2. Enable MFA for all users.
3. Require MFA at login for all team members.

---

## 5️⃣ Future Improvements & Best Practices

### ✅ Apply Service Control Policies (SCPs) for Stronger Restrictions
- Prevent accidental deletion of AWS resources.
- Restrict who can modify IAM policies.
- Enforce mandatory logging and security configurations.

✅ Example SCP to Restrict S3 Bucket Public Access:
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Deny",
      "Action": "s3:PutBucketAcl",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "s3:x-amz-acl": "public-read"
        }
      }
    }
  ]
}
```

### ✅ Regularly Audit IAM Identity Center Permissions
- Review user access periodically.
- Remove inactive users from AWS accounts.
- Ensure least privilege access is maintained over time.

✅ Steps:
1. Use **AWS IAM Access Analyzer** to identify overly permissive roles.
2. Review **IAM Identity Center activity logs in CloudTrail**.
3. Adjust user roles based on project needs.

### ✅ Enable Logging & Monitoring for Access Management
Monitoring access logs helps detect unauthorized access and security anomalies.

✅ Tools to Enable:
- **AWS CloudTrail** → Tracks all AWS API calls.
- **AWS Security Hub** → Monitors security posture.

✅ Steps to Enable AWS CloudTrail for Identity Monitoring:
1. Go to **AWS CloudTrail → Create Trail**.
2. Enable logging for **IAM Identity Center events**.
3. Store logs in **Amazon S3** for analysis.
4. Use **AWS CloudWatch** to set alerts for suspicious activities.

---

### ✅ Conclusion
By implementing **AWS Organizations** and **IAM Identity Center**, you can centralize and secure access management across multiple AWS accounts while enforcing **least privilege policies** and best security practices. Additionally, using AWS Cost Explorer, Budgets, and **Cost Anomaly Detection**, you can monitor spending patterns, set alerts for unusual costs, and quickly identify anomalies at the **account level** across your organization.
