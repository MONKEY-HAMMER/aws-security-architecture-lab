# Amazon Machine Images (AMI)

Amazon Machine Images (AMI)

EC2 instances are launched from an Amazon Machine Image (AMI), which defines the operating system, storage configuration, and initial state of the instance.

### Listing AMIs

To list images owned by a specific AWS account:

```bash
aws ec2 describe-images --owners <AWS-ACCOUNT>
```

### UserData
Retrieving UserData allows inspection of the initialization script associated with a specific EC2 instance. While commonly used for bootstrapping and configuration, UserData may expose sensitive information if not properly managed.

#### How to get the user-data via EC2 Instance Connect or the SSM Session Manager console (Using IMDS):

```bash
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/user-data
```

### How to get the user-data via AWS CLI

You will need of the Instance Id:
```bash
aws ec2 describe-instance-attribute --attribute userData --instance-id $instance_id --region us-east-1 --query UserData --output text  | base64 -d
``` 