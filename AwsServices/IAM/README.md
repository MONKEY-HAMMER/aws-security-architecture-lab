# IAM (Identity and Access Management)

IAM (Identity and Access Management) is a web service that helps securely manage access to AWS resources. It allows you to control which AWS resources users can access and what actions they are allowed to perform.
[For a detailed explanation of IAM, see the official AWS documentation.](https://docs.aws.amazon.com/iam/)

You can see what users exists in your AWS accout with:

 - AWS CONSOLE: [AWS Identity and Access Managment](https://us-east-1.console.aws.amazon.com/iam#/users)
 - AWS CLI: 
  ```bash
   aws iam list-users 
  ```


## AWS PRINCIPAL
In the context of Amazon Web Services (AWS), a Principal is an entity that makes a request to access an AWS resource.

## Types of Principals:

- ### IAM User (A user created within an AWS account)
Example: 
```json
arn:aws:iam::123456789012:user/MONKEY-HAMMER
```
___

- ### IAM Role 

   Often used by:  
    * EC2
    * Lambda
    * AWS Services
    * External Accounts 


Example:

```json 
arn:aws:iam::123456789012:role/AdminRole 
```
___        
- ### AWS Service 
An AWS Service can act like a PRINCIPAL.

Example: 
```python 
lambda.amazonaws.com
ec2.amazonaws.com
```
___
- ### An entire AWS account 
You can allow access to another entire AWS account

Example: 
```json 
"Principal": {
  "AWS": "arn:aws:iam::999999999999:root"
} 
```
---

- ### Federated identity 
   
   Users authenticated via: 
    * SAML
    * OICD 
    * Cognito 

---

## Where do they appear ? 

### Resource-based policies 
Classic example: S3 Bucket policy 
```json 
{
  "Effect": "Allow",
  "Principal": {
    "AWS": "arn:aws:iam::999999999999:role/ExternalRole"
  },
  "Action": "s3:GetObject",
  "Resource": "arn:aws:s3:::my-bucket/*"
}
```
___

# Root User

The AWS Root User is a special type of principal that represents the account owner. It has full administrative privileges over the AWS account, limited only by Service Control Policies (SCPs).

The root user is associated with the email address used during account creation. Because it has unrestricted access, securing and monitoring the root email account is critical. If an attacker gains access to the root email inbox, they may be able to reset the password and take control of the entire AWS account.

It's not possible an IAM User to identify the root email address, but if the AWS Account is member of an AWS organization, you can discover the email address of the Organization's Master Account: 

```bash 
aws organizations describe-organization
```
___

# IAM Groups 
IAM Groups are collections of IAM users. Groups allow administrators to attach policies to multiple users at once, ensuring that all members inherit the assigned permissions.


You can see the IAM Groups in your account via [AWS Console](https://us-east-1.console.aws.amazon.com/iam/home#/groups) and AWS CLI:

```bash 
aws iam list-groups 
```

More details with: 

```bash 
aws iam get-group --group-name "<GroupName>"
```
