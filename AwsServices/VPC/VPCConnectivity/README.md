# VPC Connectivity Overview

AWS provides multiple ways to connect VPCs to on-premises networks, other VPCs, and end users.

Cloud-to-Ground connectivity can be achieved using:

* Direct Connect – a dedicated physical fiber connection from on-premises infrastructure to AWS. It offers high bandwidth but does not encrypt traffic by default.

* Site-to-Site VPN – an IPSec-encrypted tunnel between AWS and an on-premises router or firewall. It requires a Customer Gateway and a Virtual Private Gateway (VGW).

Both methods require proper route table configuration and represent potential pivot paths between cloud and on-prem environments. From a security standpoint, VPN traffic is encrypted, while Direct Connect traffic is not unless additional encryption is implemented.

Cloud-to-Cloud connectivity between VPCs can be done using:

* VPC Peering – a direct one-to-one connection between two VPCs (non-transitive and no overlapping CIDRs allowed).

* Transit Gateway – a centralized routing hub that simplifies multi-VPC connectivity and can integrate VPN and Direct Connect connections.

`VPC Peering is not transitive, meaning traffic cannot automatically pass through an intermediary VPC.`


Client VPN allows end-user access to AWS environments using an AWS-managed OpenVPN service. Authentication can be done via certificates, SSO, or managed Active Directory. While convenient, misconfiguration can introduce security and compliance risks.

Overall, these connectivity options enable flexible architectures but also introduce potential attack paths if not properly secured.