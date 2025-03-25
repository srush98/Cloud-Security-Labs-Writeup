# CloudFoxable - It's another Secret Write-Up

## Overview

This write-up details the step-by-step approach taken to solve the CloudFoxable - It's a Secret challenge and retrieve the flag. The challenge revolves around uncovering hidden secrets within AWS services using IAM role permissions, AWS CLI commands, and CloudFox enumeration.

---

## Challenge Setup

### Step 1. **Confirm AWS Identity**

To verify authentication as the `ctf-starting-user`, I ran:

```bash
aws --profile cloudfoxable sts get-caller-identity
```

This confirmed that my profile was correctly set up and linked to `arn:aws:iam::ACCOUNT_ID:user/ctf-starting-user`.

## Enumerating AWS Resources

### Step 2. **Run CloudFox to Identify Secrets**

The challenge hint suggested using **CloudFox** to identify secrets stored in AWS Secrets Manager:

```bash
cloudfox aws -p cloudfoxable secrets -v2
```

I noted the **SSM parameter** `/cloudfoxable/flag/its-a-secret` was a likely candidate for the flag.

### Step 3. **Extracting Secrets from AWS Systems Manager (SSM)**

Since I identified a potential flag stored as an **SSM Parameter**, I attempted to retrieve it using the AWS CLI:

```bash
aws --profile cloudfoxable ssm get-parameter --name "/cloudfoxable/flag/its-a-secret" --with-decryption
```

## Retrieving the Secret

The flag was successfully retrieved!

    FLAG{ItsASecret::IsASecretASecretIfTooManyPeopleHaveAccessToIt?}

---

## Understanding IAM Permissions

### Step 4. **Analyze Why Access Was Granted**

To understand why `ctf-starting-user` had access to this secret, I used CloudFox to inspect its permissions:

```bash
cloudfox aws -p cloudfoxable permissions --principal ctf-starting-user -v2
```

This showed that the `AdministratorAccess` policy granted full access, confirming why I could retrieve all secrets without restrictions, allowing access to retrieve the flag.

---

## Approach

- Verified AWS authentication as `ctf-starting-user`.
- Used **CloudFox** to enumerate secrets stored in AWS Secrets Manager.
- Retrieved and analyzed IAM permissions to understand access control.
- Successfully retrieved the flag.

---

## Challenges Faced

- **Locating the Secret:** CloudFox significantly streamlined the discovery process.
- **Understanding IAM Access:** Running permission analysis provided clarity on why the secret was accessible.

---

## Key Takeaways

- **AWS Secrets Manager entries should have strict IAM permissions.**
- **CloudFox is an essential tool** for AWS security enumeration and privilege auditing.
- **Understanding IAM role policies is crucial** for cloud security assessments.

---

## Conclusion

This challenge demonstrated:

- Using **CloudFox** and AWS CLI to discover secrets and analyze IAM permissions.
- Diagnosing and verifying why access to secrets was granted.
- The importance of **least privilege IAM policies** to prevent unauthorized access.

**Final Thought:** This challenge reinforces the need for continuous **IAM security auditing** to ensure that secrets remain protected.
