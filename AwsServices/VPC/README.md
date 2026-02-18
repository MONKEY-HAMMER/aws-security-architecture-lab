# Virutal Private Cloud (VPC)
A Virtual Private Cloud (VPC) is a logically isolated virtual network within Amazon Web Services. It allows you to define your own IP address range, subnets, route tables, and network gateways.

A VPC enables:


 - Network isolation between environments (e.g., Dev, Staging, Production)

 - Separation of public and private resources

 - Fine-grained traffic control using route tables and security groups

 - Secure connectivity to on-premises networks via VPN or Direct Connect

By controlling traffic flow and segmentation, VPC improves the overall security posture of cloud architectures.

--- 
You can see your VPCs on the [VPC Dashboard](https://console.aws.amazon.com/vpc/).

## CIDR
When you create a VPC , you define a principal CIDR block. For example:

`VPC CIDR: 10.0.0.0/16`

It means that your VPC can use IPs from:

`10.0.0.0 → 10.0.255.255`


## Subnets
 
Subnets in AWS are a range of IP addresses (a logical subdivision) within a Virtual Private Cloud (VPC) used to organize and isolate resources, such as EC2 instances, into public or private networks

* Each Subnet exists only in one Availability Zone (AZ) 
* Each subnet in your VPC must be associated with a route table

### Route tables
Subnet route tables are routing rule sets associated with a subnet inside a VPC.

They define where network traffic is sent based on the destination IP address.

Each subnet must be associated with one route table, which determines:

* Whether traffic stays inside the VPC (local route)
* Goes to the internet (via an Internet Gateway)
* Goes out through a NAT Gateway
* Goes to a VPN, peering connection, or other network target
* Route tables do not filter traffic — they only decide the next hop for packets.

### Gateways:

#### IGW: 
 * Typical subnet: `Public`
 * Allowed traffic: `Internet in/out`
 * Typical Use: `Public Servers`


#### NAT:
* Typical subnet: `Private subnet`
* Allowed traffic: `Outbound to internet only`
* Typical Use: `Updates, downloads, external APIs`



#### VGW:
* Typical subnet: `Any`
* Allowed traffic: `Private network (VPN, Direct Connect)`

* Typical Use: `Private network (VPN, Direct Connect)`