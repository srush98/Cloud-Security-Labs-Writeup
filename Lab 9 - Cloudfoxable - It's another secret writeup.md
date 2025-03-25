
# CloudFoxable - It's Another Secret Write-Up

## Overview

This write-up details the step-by-step approach taken to solve the **CloudFoxable - It's Another Secret** challenge. The challenge revolves around uncovering hidden secrets within AWS services using IAM role permissions, AWS CLI commands, and CloudFox enumeration.

---

## Challenge Setup

### Step 1. **Confirm AWS Identity**

After assuming the IAM role `Ertz`, I verified my identity with:

```bash
aws --profile ertz sts get-caller-identity
```

This confirmed that my profile was successfully linked to the `Ertz` role.

---

## Enumerating AWS Resources

### Step 2. **List Policies Attached to the Role**

To understand the role's permissions, I listed the attached policies:

```bash
aws iam list-attached-role-policies --role-name ertz --profile cloudfoxable
```

I identified the policy `its-another-secret-policy`, which was crucial for accessing the flag.

### Step 3. **Analyze Policy Permissions**

I retrieved the details of the `its-another-secret-policy`:

```bash
aws iam get-policy-version --policy-arn arn:aws:iam::390844766599:policy/its-another-secret-policy --version-id v1 --profile cloudfoxable
```

This policy allowed access to the **SSM Parameter Store** with the action `ssm:GetParameter` for the resource:

```
arn:aws:ssm:us-west-2:390844766599:parameter/cloudfoxable/flag/its-another-secret
```

### Step 4. **Retrieve the Secret**

Given the permissions in the policy, I accessed the secret stored in **AWS Systems Manager (SSM)** using the following command:

```bash
aws ssm get-parameter --name /cloudfoxable/flag/its-another-secret --profile ertz --with-decryption
```

This successfully returned the flag:

```
FLAG{ItsAnotherSecret::ThereWillBeALotOfAssumingRolesInThisCTF}
```

---

## Understanding IAM Permissions

### Step 5. **Analyze Why Access Was Granted**

To understand why the `Ertz` role had access to this secret, I analyzed the attached policy using:

```bash
aws iam get-policy-version --policy-arn arn:aws:iam::390844766599:policy/its-another-secret-policy --version-id v1 --profile cloudfoxable
```

This policy granted the role permissions to access specific SSM parameters, confirming why the `Ertz` role could retrieve the flag.

---

## Approach

- Verified authentication and role assumption to `Ertz` using the AWS CLI.
- List the policies attached to the `Ertz` role and analyzed permissions.
- Retrieved the secret from AWS Systems Manager Parameter Store using the `ssm:GetParameter` permission.
- Decrypted and captured the flag.

---

## Challenges Faced

- **Understanding IAM Permissions:** The key challenge was understanding why the `Ertz` role had the necessary permissions. By inspecting the policy attached to the role, I clarified the access control.

- **Retrieving the Flag:** The flag was stored as a SecureString in the SSM Parameter Store, requiring decryption, which was successfully done using the `--with-decryption` option.

---

## Key Takeaways

- **IAM policies should be reviewed thoroughly** to ensure least privilege access to sensitive resources.
- **SecureString parameters** in SSM need decryption to reveal their content.
- **Role assumption** is a critical part of cloud penetration testing, and understanding the permissions granted by roles is essential.

---

## Conclusion

This challenge demonstrated:

- Using **AWS CLI** to assume roles, list policies, and retrieve secrets.
- The importance of understanding **IAM permissions** to evaluate access control.
- The need for **proper secret management and decryption** in cloud environments.

**Final Thought:** Cloud security assessments must include proper **role-based access control (RBAC)** and **secure parameter management** to prevent unauthorized access to sensitive information.

---
