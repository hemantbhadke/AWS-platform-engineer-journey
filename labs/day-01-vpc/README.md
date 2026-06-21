# Lab 2 — VPC & Core Networking

**Date:** 21 June 2026  
**Region:** ap-south-1 (Mumbai)  
**Status:** Complete

## What I built
- Created a custom VPC with CIDR block `10.0.0.0/16`
- Built 2 public subnets and 2 private subnets across 2 Availability Zones
- Attached an Internet Gateway to the VPC
- Explored public and private route tables
- Created a Security Group `web-sg` allowing HTTP port 80 and SSH port 22

## Architecture diagram
Internet

↓
Internet Gateway
↓
[Public Subnet 10.0.1.0/24]    [Public Subnet 10.0.2.0/24]
Web Servers / Load Balancer
[Private Subnet 10.0.3.0/24]   [Private Subnet 10.0.4.0/24]
Databases / Backend Servers

## What I learned
- A subnet is public only if its route table has a route `0.0.0.0/0 → igw`
- The subnet name means nothing — the route table is the truth
- Internet Gateway is the door between VPC and the public internet
- Security Groups are firewalls attached to individual resources, not the VPC
- Route Table decides WHERE traffic goes, Security Group decides IF it's allowed
- Databases and backend services always go in private subnets

## Key concepts
| Concept | What it is |
|---|---|
| VPC | Your private isolated network inside AWS |
| CIDR Block | IP address range for your VPC — `10.0.0.0/16` means 65,536 possible IPs |
| Public Subnet | Subnet whose route table points to an Internet Gateway |
| Private Subnet | Subnet with no IGW route — internal communication only |
| Internet Gateway | Connects your VPC to the public internet |
| Route Table | Rules that decide where network traffic is directed |
| Security Group | Virtual firewall controlling allowed ports on individual resources |
| Availability Zone | Isolated data center — spread resources across AZs for high availability |

## Route table comparison
| Route Table | Destination | Target | Effect |
|---|---|---|---|
| Public | 0.0.0.0/0 | igw-xxxxxxxx | Traffic can reach internet |
| Public | 10.0.0.0/16 | local | Internal VPC traffic |
| Private | 10.0.0.0/16 | local | Internal VPC traffic only |

## Security Group rules configured
| Type | Port | Source | Reason |
|---|---|---|---|
| HTTP | 80 | 0.0.0.0/0 | Allow web traffic from anywhere |
| SSH | 22 | My IP only | Restrict server access to me only |

## Architecture decisions
- Web servers → public subnet (need internet access)
- Databases → private subnet (must never be publicly exposed)
- Only Load Balancer faces internet, app servers stay private
- Spreading across 2 AZs ensures high availability if one AZ goes down

## Real world relevance
Every production system at every company runs inside a VPC with this 
exact pattern — public subnets for internet-facing resources, private 
subnets for databases and internal services. Platform engineers design 
and maintain these networks. Putting everything in public subnets is a 
critical security risk and violates compliance regulations like GDPR 
and PCI-DSS.

## Screenshot
<img width="699" height="377" alt="image" src="https://github.com/user-attachments/assets/9904f5eb-5777-4609-a76c-459ea48e1e54" />
<img width="1366" height="599" alt="image" src="https://github.com/user-attachments/assets/d9c451e0-740d-480d-a432-dcfd1e7e9591" />
<img width="1366" height="599" alt="image" src="https://github.com/user-attachments/assets/5d583759-e9e7-4948-80ce-8c4dbb2f5a33" />
<img width="1366" height="599" alt="image" src="https://github.com/user-attachments/assets/4cf5047d-e2fa-4160-be66-6241d26966ff" />
<img width="1366" height="599" alt="image" src="https://github.com/user-attachments/assets/395478ac-c6a9-4b58-854f-aa7faeb1455e" />



