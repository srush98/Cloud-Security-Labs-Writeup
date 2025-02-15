# CloudFoxable - First Flag Writeup

## Overview

**Challenge Name:** First Flag  
**Starting Point:** [CloudFoxable GitHub Repository](https://github.com/BishopFox/cloudfoxable)

CloudFoxable is a hands-on AWS security challenge where participants deploy an intentionally vulnerable cloud environment using Terraform. The objective is to practice security assessments by identifying misconfigurations and extracting sensitive data.

## Setup Instructions

### Step 1: Prepare Your AWS Environment

1. Created an AWS account.
2. Created a non-root IAM user with administrative access.
3. Generated an access key for the new user.
4. Installed the AWS CLI ([Download AWS CLI](https://aws.amazon.com/cli/)).
5. Configured the AWS CLI with my new IAM user credentials:
   ```sh
   aws configure
   ```
6. Verified AWS CLI configuration:
   ```sh
   aws sts get-caller-identity
   ```
   This returned my AWS account ID, user ARN, and principal ID.

### Step 2: Deploy CloudFoxable Using Terraform

1. Cloned the CloudFoxable repository:
   ```sh
   git clone https://github.com/BishopFox/cloudfoxable
   cd cloudfoxable/aws
   ```
2. Copied and modified the Terraform variables file:
   ```sh
   cp terraform.tfvars.example terraform.tfvars
   ```
3. Edited `terraform.tfvars` to set my AWS profile:
   ```hcl
   aws_local_profile = "terraform-admin-sru"
   ```
4. Initialized Terraform:
   ```sh
   terraform init
   ```
5. Checked the Terraform plan:
   ```sh
   terraform plan
   ```
6. Deployed the CloudFoxable environment:
   ```sh
   terraform apply
   ```
   Typed `yes` when prompted.

### Step 3: Install AWS Security Tools (Optional, but Recommended)

For better visibility and enumeration, I installed:

- CloudFox
- **Principal Mapper (Pmapper):**
- **Pacu (AWS Exploitation Framework):**

## Finding the First Flag

### Step 1: Read Terraform Output

After running `terraform apply`, the output will display critical information about the deployed environment. Look closely at the end of the Terraform output to find:

- A new AWS user (`ctf-starting-user`)
- Access keys for the new user
- A flag in the format: `FLAG{challengeName::CamelCaseText}`

### Step 2: Configure AWS CLI with CTF User

You will need to configure your AWS CLI to use the `ctf-starting-user` to proceed with further challenges:

```sh
echo "" >> ~/.aws/credentials && \
echo "[cloudfoxable]" >> ~/.aws/credentials && \
echo "aws_access_key_id = `terraform output -raw CTF_Start_User_Access_Key_Id`" >> ~/.aws/credentials && \
echo "aws_secret_access_key = `terraform output -raw CTF_Start_User_Secret_Access_Key`" >> ~/.aws/credentials && \
echo "region = us-west-2" >> ~/.aws/credentials
```

Alternatively, manually configure it by running:

```sh
aws configure --profile cloudfoxable
```

Enter the credentials from the Terraform output.

### Step 3: Verify CTF User Configuration

Check that the new profile works:

```sh
aws sts get-caller-identity --profile cloudfoxable
```

If configured correctly, this should return the IAM ARN of `ctf-starting-user`.

## Cleanup (Destroy CloudFoxable)

To remove all deployed AWS resources:

```sh
terraform destroy
```

Run this command from the `cloudfoxable/aws` directory and confirm the deletion when prompted.

## Conclusion

✅ Successfully set up CloudFoxable.  
✅ Extracted the first flag from Terraform output.  
✅ Configured the AWS CLI to use the `ctf-starting-user`.

By following these steps, I have a foundational understanding of interacting with AWS security challenges using Terraform.
