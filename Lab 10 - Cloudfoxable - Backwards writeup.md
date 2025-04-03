# Backwards Challenge Write-up

## **Overview**
The "Backwards" challenge in CloudFoxable is designed to simulate a real-world cloud penetration testing scenario where you begin with an interesting AWS resource and work backwards to determine who has access to it. In this case, we start with an AWS Secrets Manager ARN and investigate which IAM roles or users have permissions to access it. Once identified, we leverage those permissions to retrieve the secret.

## **Starting Point**
ARN: `arn:aws:secretsmanager:us-east-1:ACCOUNTID:secret:DomainAdministrator-Credentials-SUFFIX`

CloudFoxable Setup: None required.

## **Challenge Breakdown**
1. **Enumerating IAM Roles**
   - List all IAM roles in the environment:
     ```sh
     aws iam list-roles --profile cloudfoxable
     ```
   - Identified roles that can be assumed by `ctf-starting-user`.
   - From the output, determined that `terraform-admin-sru` is an assumable role.

2. **Checking Role Permissions**
   - Enumerated permissions assigned to the `Alexander-Arnold` role:
     ```sh
     aws iam list-attached-role-policies --role-name Alexander-Arnold --profile cloudfoxable
     ```
   - Retrieved the policy details:
     ```sh
     aws iam get-policy-version --policy-arn arn:aws:iam::ACCOUNT_ID:policy/corporate-domain-admin-password-policy --version-id v1 --profile cloudfoxable
     ```
   - The policy allows `secretsmanager:GetSecretValue` on the target secret.

3. **Identifying the Secret Name**
   - Use AWS CLI to list available secrets:
     ```sh
     aws secretsmanager list-secrets --profile cloudfoxable
     ```
   - Confirm that the secret name is `DomainAdministrator-Credentials`.

4. **Assuming the Role**
   - Configure an AWS CLI profile for `terraform-admin-sru`:
     ```ini
     [profile terraform-admin-sru]
     region = us-east-1
     role_arn = arn:aws:iam::ACCOUNT_ID:role/terraform-admin-sru
     source_profile = cloudfoxable
     ```
   - Assume the role and retrieve the secret value:
     ```sh
     aws secretsmanager get-secret-value --secret-id DomainAdministrator-Credentials --profile terraform-admin-sru
     ```

5. **Retrieving the Flag**
   - The flag is successfully retrieved by executing the final command.

## **Flag**
FLAG{backwards::IfYouFindSomethingInterstingFindWhoHasAccessToIt}

## **Key Takeaways**
- Cloud penetration testing often involves working backwards from an identified resource.
- Permissions are not always explicitly granted; role trust policies can introduce alternative access paths.
- CloudFox provides powerful enumeration capabilities for assessing access permissions and role assumptions.
- Understanding AWS IAM intricacies is crucial for effective security testing and privilege escalation.

## **Conclusion**

This challenge highlights the importance of understanding IAM roles, trust relationships, and permissions when performing cloud security assessments. By systematically enumerating roles and policies, we uncovered an assumable role with the necessary permissions to access a critical secret. This approach is a fundamental skill in cloud penetration testing and reinforces the need for strict access controls in cloud environments to prevent unauthorized access to sensitive resources.
