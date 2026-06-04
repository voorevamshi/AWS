### Real-time Example: "The Forbidden Bucket"

Imagine you are a Developer (the **Identity**) trying to delete a file in a critical S3 bucket, but you keep getting an `AccessDenied` error. Here is how you troubleshoot the layers:

#### Layer 1: The SCP (The "Corporate Guardrail")

-   **Scenario:** Your company has an organization-wide rule that no one (not even Administrators) can delete S3 buckets or objects in the "Production" account.
    
-   **Result:** Even if your IAM user has "Admin" rights, the **SCP denies the action at the account level.**
    
-   **Troubleshoot:** Check if your AWS Account is part of an Organization that has an SCP denying `s3:DeleteObject`.
 ```
 {
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyS3DeleteInProduction",
      "Effect": "Deny",
      "Action": "s3:DeleteObject",
      "Resource": "arn:aws:s3:::production-data-bucket/*"
    }
  ]
}
```    

#### Layer 2: The Permissions Boundary (The "Max Potential")

-   **Scenario:** Your manager wants to ensure you don't accidentally create high-cost resources. They set a Permissions Boundary on your IAM user that only allows EC2 and Lambda actions.
    
-   **Result:** You have an "S3 Access" policy attached to your user, but the **Permissions Boundary restricts your maximum capability to only EC2 and Lambda.** The S3 action is effectively ignored.
    
-   **Troubleshoot:** Check if a Permissions Boundary is attached to your IAM user/role that restricts the permissions you are trying to use.
   

 ```
    {
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowOnlyEC2AndLambda",
      "Effect": "Allow",
      "Action": [
        "ec2:*",
        "lambda:*"
      ],
      "Resource": "*"
    }
  ]
}
```
    
#### Layer 3: Identity-Based Policies (The "Job Description")

-   **Scenario:** This is your standard IAM policy attached to your user. It says: _"Allow: s3:PutObject, s3:GetObject"_.
    
-   **Result:** You notice your policy is missing `s3:DeleteObject`.
    
-   **Troubleshoot:** This is the most common place to find the issue. You simply need to update your IAM policy to include the missing action.
    
```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowS3ReadWrite",
      "Effect": "Allow",
      "Action": [
        "s3:GetObject",
        "s3:PutObject"
      ],
      "Resource": "arn:aws:s3:::my-developer-bucket/*"
    }
  ]
}
```

### Summary Checklist for Troubleshooting:
**1. SCP Layer:** Does the Organization block this action for the _entire account_?
**2. Boundary Layer** Does the user's _Maximum Allowed_ limit exclude this action?
**3. IAM Policy Layer** Does the user's _Policy_ explicitly list `Allow` for this action?
**4. Resource Policy Layer** Does the _S3 Bucket Policy_ explicitly block my IAM user?
