# CloudFoxable challenges - Do this first Flag Writeup

## Overview

**Challenge Name:** Do this first!
**Starting Point:** [CloudFoxable GitHub Repository](https://github.com/BishopFox/cloudfoxable)

CloudFoxable is a hands-on AWS security challenge designed to simulate real-world cloud security misconfigurations. Participants deploy a vulnerable cloud environment using Terraform and work through challenges to identify security risks and extract sensitive data. The first step in this challenge is setting up CloudFoxable and retrieving the first flag by carefully analyzing Terraform’s output.

## Steps I followed to find the flag:

### Step 1: Preparing my AWS Environment

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

### Step 2: Deploying CloudFoxable Using Terraform

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

### Step 3: Installing AWS Security Tools (Optional, but Recommended)

For better visibility and enumeration, I installed:

- CloudFox
- Principal Mapper (Pmapper)
- Pacu (AWS Exploitation Framework)

## Finding the First Flag

### Step 1: Read Terraform Output

After running `terraform apply`, the output displayed critical information about the deployed environment. Looking closely at the end of the Terraform output, I found:

- A new AWS user (`ctf-starting-user`)
- Access keys for the new user
- A flag in the format: `FLAG{congrats_you_are_now_a_terraform_expert_happy_hunting}`

### Step 2: Configure AWS CLI with CTF User

Then, I configured my AWS CLI to use the `ctf-starting-user` to proceed with further challenges:

```sh
echo "" >> ~/.aws/credentials && \
echo "[cloudfoxable]" >> ~/.aws/credentials && \
echo "aws_access_key_id = `terraform output -raw CTF_Start_User_Access_Key_Id`" >> ~/.aws/credentials && \
echo "aws_secret_access_key = `terraform output -raw CTF_Start_User_Secret_Access_Key`" >> ~/.aws/credentials && \
echo "region = us-west-2" >> ~/.aws/credentials
```

### Step 3: Verify CTF User Configuration

Checking that the new profile works:

```sh
aws sts get-caller-identity --profile cloudfoxable
```

It was configured correctly, therefore it returned the IAM ARN of `ctf-starting-user`.

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
