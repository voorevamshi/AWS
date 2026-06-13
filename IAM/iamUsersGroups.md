Since you are running this from a Windows command prompt, here are the **exact copy-and-paste commands** to check all policies and groups assigned to you.
### Run the identity command

Open your terminal and type:
```
aws sts get-caller-identity

{
"UserId": "AIDAWI5EIZIHKSPNQ6WGQ",
"Account": "431451851278",
"Arn": "arn:aws:iam::431451851278:user/Vamshi_Dev"
}
```
## 1. Check Your IAM Groups

To see if you belong to any groups (and what those groups are named), run this command:

DOS

```
aws iam list-groups-for-user --user-name Vamshi_Dev
{
    "Groups": [
        {
            "Path": "/",
            "GroupName": "Developers",
            "GroupId": "AGPAWI5EIZIHL4WKHGWED",
            "Arn": "arn:aws:iam::431451851278:group/Developers",
            "CreateDate": "2026-05-30T04:32:31+00:00"
        }
    ]
}

```

### What to look for in the output:

If you belong to a group, you will see a JSON block containing a `GroupName` (for example, `"GroupName": "Developer-Group"`). Take note of that group name for the next steps!

## 2. Check Policies Attached to Your Groups

If the command above shows you belong to a group, copy the `GroupName` and plug it into this command to see what permissions that group has:

DOS

```
aws iam list-attached-group-policies --group-name YOUR_GROUP_NAME_HERE
aws iam list-attached-group-policies --group-name Developers
{
    "AttachedPolicies": [
        {
            "PolicyName": "AdministratorAccess",
            "PolicyArn": "arn:aws:iam::aws:policy/AdministratorAccess"
        }
    ]
}

```

(Replace `YOUR_GROUP_NAME_HERE` with the actual group name you found in Step 1)._

## 3. Check Policies Attached Directly to Your User

Sometimes permissions are attached directly to your user account instead of a group (known as Managed Policies). Run this to check:

DOS

```
aws iam list-attached-user-policies --user-name Vamshi_Dev
{
    "AttachedPolicies": []
}
```

## 4. Check for Inline Policies (Directly on Your User)

Inline policies are custom permissions embedded directly into your specific user profile rather than being a separate standalone policy. Check for them using this command:

DOS

```
aws iam list-user-policies --user-name Vamshi_Dev
{
    "PolicyNames": []
}

```

## 5. How to View the Actual Permissions (Read vs. Write)

The commands in steps 2, 3, and 4 will give you a list of policy names and **Policy ARNs** (which look like `arn:aws:iam::aws:policy/AmazonSQSFullAccess`).

To actually read what inside that policy to see if you have Read or Write access, grab the `PolicyArn` from the previous outputs and run this:

DOS

```
aws iam get-policy-version --policy-arn YOUR_POLICY_ARN_HERE --version-id v1
aws iam get-policy-version --policy-arn arn:aws:iam::aws:policy/AdministratorAccess --version-id v1
{
    "PolicyVersion": {
        "Document": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Action": "*",
                    "Resource": "*"
                }
            ]
        },
        "VersionId": "v1",
        "IsDefaultVersion": true,
        "CreateDate": "2015-02-06T18:39:46+00:00"
    }
}
```

> _(Replace `YOUR_POLICY_ARN_HERE` with the actual ARN string)._

### ⚠️ Note on Potential "Access Denied" Errors

Because you couldn't see groups in the console earlier, there is a strong chance that running these `aws iam` commands might give you an `Access Denied` error in your terminal.

If that happens, it means your company's security policy prevents developers from reading IAM configurations. If you get blocked by errors, your absolute best alternative is to test your specific service access using the simulation approach we discussed:

DOS

```
aws iam simulate-principal-policy --policy-source-arn arn:aws:iam::431451851278:user/Vamshi_Dev --action-names sqs:ListQueues

{
    "EvaluationResults": [
        {
            "EvalActionName": "sqs:ListQueues",
            "EvalResourceName": "arn:${Partition}:sqs:${Region}:${Account}:*",
            "EvalDecision": "allowed",
            "MatchedStatements": [
                {
                    "SourcePolicyId": "AdministratorAccess",
                    "SourcePolicyType": "IAM Policy",
                    "StartPosition": {
                        "Line": 3,
                        "Column": 17
                    },
                    "EndPosition": {
                        "Line": 8,
                        "Column": 6
                    }
                }
            ],
            "MissingContextValues": []
        } ...,
```
