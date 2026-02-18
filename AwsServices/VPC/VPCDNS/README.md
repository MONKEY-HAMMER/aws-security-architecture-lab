# VPC DNS 

## Default VPC DNS

Every VPC automatically includes an AWS-provided DNS server.

Its IP address is always:

  * `Base of the VPC CIDR block + 2`

Example:

    
VPC CIDR: `192.168.15.0/24`

DNS server: 
`192.168.15.2`

This DNS server:
* Resolves public domains (e.g., google.com)
* Resolves AWS service endpoints (e.g., S3, SQS)
* Works automatically for EC2 instances
* No manual configuration required.
--- 

## Private Hosted Zones (Route 53)

You can create a Private Hosted Zone in Route 53 and associate it with your VPC.

This allows you to create private DNS names like:

`db.internal.company.local`
`api.internal.company.local`

These names: 

* Only resolve inside the VPC
* Are not publicly accessible
* Are resolved by the AWS DNS server

Common use cases:
* Internal services
* Microservices architectures
* Corporate internal domains

---

## Hybrid DNS (On-Prem Integration)

The VPC DNS system (Route 53 Resolver) can:

* Answer DNS queries from on-premises networks
* Forward DNS queries to on-prem DNS servers

This enables hybrid cloud DNS integration.

⚠️ Important security note:

`DNS can become a lateral movement path between cloud and on-prem environments.`

--- 

## Route 53 Resolver DNS Firewall

You can configure a DNS Firewall to:

- Block malicious domains
- Allow only approved domains
- Prevent DNS data exfiltration attacks

This adds an extra security control layer at the DNS level.

--- 
## DNS Query Logging

You can enable DNS query logging and send logs to:

Amazon CloudWatch

For example, to a log group like:

`VPCResolverLogs`

This allows you to:
* Monitor DNS activity
* Detect suspicious behavior
* Send logs to a SIEM for threat analysis

--- 

## Cost Awareness

Advanced features such as:
* DNS Firewall
* Query Logging
* Resolver endpoints

Have additional costs.

Security features in AWS are powerful but often not free.