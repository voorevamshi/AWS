# AWS Identity and Access Management (IAM) Study Notes

This document serves as a comprehensive summary of AWS IAM concepts, tailored for the **AWS Certified Developer – Associate** exam.

## 1. IAM Roles
An IAM role is an identity that defines a set of permissions for making AWS service requests.

* **Key Characteristics:**
    * **No Long-term Credentials:** Roles are not associated with a specific user or group; they provide temporary credentials.
    * **Trust Entities:** Define who can assume the role (e.g., AWS Services like EC2, Lambda).
    * **EC2 Instances:** An EC2 instance can only have **ONE** IAM role attached at a time (via an Instance Profile).

## 2. JSON Policy Structure
Most policies are stored in AWS as JSON documents.

### Anatomy of a Policy
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "ExampleStatement",
      "Effect": "Allow",
      "Action": "s3:ListBucket",
      "Resource": "arn:aws:s3:::example_bucket"
    }
  ]
}
```

* **Version:** Usually "2012-10-17".
* **Statement:** The main block (can be a single object or an array of objects).
* **Effect:** `Allow` or `Deny` (Explicit Deny overrides everything).
* **Action:** The specific service/API action.
* **Resource:** The target ARN.

## 3. Amazon Resource Names (ARNs)
ARNs uniquely identify AWS resources.
* **Format:** `arn:partition:service:region:account-id:resource-id`
* **Example:** `arn:aws:iam::891377051037:role/EC2-53`

## 4. Policy Types
1.  **Identity-based:** Attached to users, groups, or roles.
2.  **Resource-based:** Attached directly to resources (e.g., S3 Bucket Policy).
3.  **Permissions Boundaries:** Defines the maximum permissions an identity can have.
4.  **Organizations SCPs:** Guardrails for an entire AWS account.
5.  **Session Policies:** Restricts permissions for a temporary session.
6.  **ACLs:** Legacy access control lists.

## 5. Managed vs. Inline Policies
* **AWS Managed Policies:** Created and administered by AWS. Cannot be modified.
* **Customer Managed Policies:** Standalone policies that you create and manage. Best practice: Copy an AWS Managed policy and modify it.
* **Inline Policies:** Embedded directly into a single user, group, or role. Deleting the identity deletes the policy.

#### Customer Managed Policy Example (S3 CreateBucket Deny)
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "VisualEditor0",
      "Effect": "Deny",
      "Action": [
        "s3:CreateBucket",
      ],
      "Resource": "arn:aws:s3:::example_bucket"
    }
  ]
}
```
#### Customer Managed by importing EC2 fulll access and modifying (EC2, CloudWatch, ELB, ASG, and IAM)
This policy allows EC2/CloudWatch access while explicitly denying ELB/AutoScaling.
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Action": "ec2:*",
      "Effect": "Allow",
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Action": "elasticloadbalancing:*",
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "cloudwatch:*",
      "Resource": "*"
    },
    {
      "Effect": "Deny",
      "Action": "autoscaling:*",
      "Resource": "*"
    },
    {
      "Effect": "Allow",
      "Action": "iam:CreateServiceLinkedRole",
      "Resource": "*",
      "Condition": {
        "StringEquals": {
          "iam:AWSServiceName": "autoscaling.amazonaws.com"
        }
      }
    }
  ]
}
```

#### To manage Inline Policies effectively

-   **Creation**: Navigate to the User, select **Permissions** > **Add permissions** > **Create inline policy**.
-   **Assignment**: These policies are embedded directly into a specific user, group, or role.
-   **Constraint**: The policy remains subject to the identity's **Permissions Boundary**.    
-   **Lifecycle**: Deleting the user or role automatically deletes the attached inline policy.


## 6. Policy Evaluation Logic (Exam Critical)
When a request is made, AWS evaluates all applicable policies:

1.  **Default Deny:** If no policy explicitly allows an action, it is denied.
2.  **Explicit Deny:** If any policy contains a `Deny`, the request is rejected immediately.
3.  **Explicit Allow:** If no `Deny` exists and there is an `Allow`, access is granted.

### Troubleshooting Order: [TroubleShoot RealTime IAM Role Example](TroubleShoot_RealTime_IAM_Role_Example.md)
1.  **SCP** (Organization level)
2.  **Permissions Boundary** (Identity level)
3.  **Identity/Resource-based Policies** (Permission level)
