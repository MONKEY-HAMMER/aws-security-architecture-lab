# Elastic Compute Cloud (EC2)
Amazon EC2 provides scalable virtual servers in the cloud, allowing users to provision compute capacity on demand and configure CPU, memory, storage, and networking according to workload requirements.



## Instance Creation Flow

When launching an EC2 instance, AWS provisions a virtual machine inside a Virtual Private Cloud (VPC). During creation, the following components must be defined:

 - AMI (Amazon Machine Image) → Defines the OS and base configuration.

 - Instance Type → Defines CPU, memory, and network capacity.

 - VPC and Subnet → Determines networking scope (public or private).

 - Security Group → Acts as a stateful firewall controlling inbound/outbound  traffic.

 - Key Pair → Used for SSH authentication (Linux instances).

 - Public IP (if required) → Needed for direct internet access.

At launch time, AWS allocates compute resources, attaches networking interfaces, applies the Security Group rules, and boots the instance using the selected AMI.

## Public vs Private Access

If deployed in a `public subnet` (route to Internet Gateway + public IP), the instance can be accessed directly from the internet.

If deployed in a `private subnet`, access requires:
 - Bastion Host
 - VPN
 - Or Session Manager (SSM)



## IMDS and EC2 Credential Retrieval

EC2 instances have a built-in mechanism to obtain temporary IAM credentials through the Instance Metadata Service (IMDS).

IMDS is a local API accessible from within the instance that provides metadata information (such as instance ID and networking details) as well as temporary credentials associated with the IAM Role attached to the instance.

`Historically, IMDS was vulnerable to SSRF (Server-Side Request Forgery) attacks. In a misconfigured application scenario, an attacker could exploit an SSRF vulnerability to access the metadata endpoint and retrieve temporary role credentials, potentially leading to privilege escalation within the AWS account.`

To mitigate this risk, IMDSv2 was introduced.
With IMDSv2, a session-oriented approach is required: the client must first send an HTTP PUT request to obtain a session token. This token must then be included in the headers of subsequent metadata requests, adding an additional layer of protection against SSRF-based credential theft.


### How it works in IMDS: 
```bash
curl -v http://169.254.169.254/latest/meta-data/iam/security-credentials/
 ```
This command returns the name of the IAM Role associated with the EC2 instance.

Now that you have the IAM Role name, you can get session credentials for the role: 

```bash 
curl -v http://169.254.169.254/latest/meta-data/iam/security-credentials/{role_name}
``` 

### Now activating ICMSv2 
First you will need the instance Id.
```bash
instance_id=$( curl -s http://169.254.169.254/latest/meta-data/instance-id )
``` 
With this command, you store the value returned from the API call in the variable "instance_id" created.
```bash
aws ec2 modify-instance-metadata-options --instance-id $instance_id --http-tokens required --region us-east-1
```

Now that ICMSv2 is active , to get credentials you will need to get the token:

```bash
TOKEN=`curl -s -X PUT "http://169.254.169.254/latest/api/token" -H "X-aws-ec2-metadata-token-ttl-seconds: 21600"`
```
The token is being stored in the variable `TOKEN`.

`X-aws-ec2-metadata-token-ttl-seconds: 21600`: 
This means that the token will be valid for up to 6 hours.

With a valid token, now you can get the credentials. 

```bash
role_name=$( curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/iam/security-credentials/ )
```

```bash
curl -s -H "X-aws-ec2-metadata-token: $TOKEN" http://169.254.169.254/latest/meta-data/iam/security-credentials/${role_name}
```
--- 

# Elastic Network Interface (ENI)

An Elastic Network Interface (ENI) is a virtual network interface that can be attached to an EC2 instance within a VPC.

Each ENI contains:
 - A private IPv4 address (primary)

 - Optional secondary private IPs

 - An optional public IP or Elastic IP

 - One or more associated Security Groups

 - A MAC address

An EC2 instance can have multiple ENIs attached, depending on the instance type. This allows separation of network traffic (e.g., management, application, database) across different interfaces.

Security Groups are attached at the ENI level, meaning traffic filtering is applied per network interface.


```bash
 aws ec2 describe-network-interfaces 
``` 

