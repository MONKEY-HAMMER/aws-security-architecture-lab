# NACLs
NACLs are optional security layers for Amazon VPCs. It works like a firewall at the subnet to control inbound and outbound traffic. (Both incoming and outgoing traffic must be authorised. )

`By default, NACLs have a Deny All rule.`

`Unlike IAM, an explicit Deny does not override an Allow. In NACL, what matters is the numerical order.`



# Security Groups 
Unlike NACLs, security groups act on [ENIs](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/using-eni.html) and not on the entire subnet. 
Security Groups control both inbound and outbound ENIs traffic.  

`Security Groups do not support deny rules`

How to list ENIs in yout account:

```bash
aws ec2 describe-network-interfaces
``` 

--- 
# Resources Inside vs Outside the VPC

A VPC represents an isolated virtual network within AWS. However, not all AWS services run inside a VPC.

## Resources Inside a VPC

These resources require networking components such as subnets, route tables, and security groups:

* EC2 instances
* RDS databases
* Elastic Network Interfaces (ENIs)
* Application and Network Load Balancers
* ECS tasks (awsvpc mode)
* EFS

## Resources Outside a VPC

Some AWS services are managed at the regional or global level and do not reside inside your VPC:

* S3
* IAM
* Route 53
* CloudFront
* DynamoDB (public access)

