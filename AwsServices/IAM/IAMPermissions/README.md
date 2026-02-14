# IAM Permissions

A principal IAM can have multiple policies, which determine what the principal is authorized to do in your account.

```bash
aws iam list-policies
```
This command list all IAM Policies available in the AWS account, including both AWS-managed and customer-managed policies.

___
The most powerful AWS managed policy is the AdministratorAccess policy:

```bash 
aws iam get-policy --policy-arn arn:aws:iam::aws:policy/AdministratorAccess                                                                          
{
    "Policy": {
        "PolicyName": "AdministratorAccess",
        "PolicyId": "ANPAIWMBCKSKIEE64ZLYK",
        "Arn": "arn:aws:iam::aws:policy/AdministratorAccess",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 2,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "Description": "Provides full access to AWS services and resources.",
        "CreateDate": "2015-02-06T18:39:46Z",
        "UpdateDate": "2015-02-06T18:39:46Z",
        "Tags": []
    }
}
```
```bash
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
        "CreateDate": "2015-02-06T18:39:46Z"
    }
}
```

### In Statement part, we can see that it allows all action on all resources: 
```json 
   "Statement": [
                {
                    "Effect": "Allow",
                    "Action": "*",
                    "Resource": "*"
                }
            ]
```

___


## Conditions
It is also possible to use conditions in policies: 
```json 

{
  "Effect": "Deny",
  "Principal": "*",
  "Action": "s3:PutObject",
  "Resource": "arn:aws:s3:::my-logs-bucket/AWSLogs/AccountNumber/*",
  "Condition": {
    "StringNotEquals": {
      "aws:SourceVpc": "vpc-abcdef2",
      "aws:PrincipalServiceName": "glue.amazonaws.com"
    }
  }
}
```

This policy denies the `s3:PutObject` action (uploading objects to an S3 bucket) on the resource:

arn:aws:s3:::my-logs-bucket/AWSLogs/AccountNumber/*

The deny rule is triggered when the request does not originate from the specified VPC (`vpc-abcdef2`) and is not made by the AWS Glue service (`glue.amazonaws.com`).

In practice, this means that only requests coming from the approved VPC or from the AWS Glue service are allowed to upload objects to this bucket path. All other attempts are explicitly denied.

In addition to `StringNotEquals` there are others conditions operators that you can see more about it [here](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_elements_condition_operators.html)4. 