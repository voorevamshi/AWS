# AWS EKS Cluster Creation Guide: Permissions & Setup

This document outlines the minimum IAM permissions required for a user to create and manage an Amazon EKS cluster manually via the AWS Console or CLI.

---

## 1. User Permissions
Attach the following managed policies and inline configuration to the **IAM User** performing the creation.

### AWS Managed Policies
Search for and attach these to the user:
* **AmazonEKSClusterPolicy**: Provides permissions to create and describe clusters.
* **AmazonEC2FullAccess**: Required to manage VPCs, Subnets, and Security Groups.
* **IAMReadOnlyAccess**: (Optional) Helps in the console to list and select roles.

### Required Inline Policy (PassRole)
You must create an **Inline Policy** for the user to allow them to "pass" the service role to the EKS service. 
* *Note: If you don't see "Create Inline Policy", create a "Customer Managed Policy" and attach it.*

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "AllowPassRoleToEKS",
            "Effect": "Allow",
            "Action": "iam:PassRole",
            "Resource": "*",
            "Condition": {
                "StringEquals": {
                    "iam:PassedToService": "eks.amazonaws.com"
                }
            }
        }
    ]
}
