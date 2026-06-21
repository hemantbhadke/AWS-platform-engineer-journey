# Lab 3 — EC2 & Web Server Setup

**Date:** 21 June 2026  
**Region:** ap-south-1 (Mumbai)  
**Status:** ✅ Complete  
**Score:** 89% on quiz

## What I built
- Launched an EC2 t2.micro instance (Amazon Linux 2023)
- Created a Key Pair for SSH access
- Configured Security Group with HTTP port 80 and SSH port 22
- SSH'd into the server from my local terminal
- Installed Apache web server (httpd)
- Hosted a live webpage accessible from the internet

## Commands used
```bash
# Fix key pair permissions before SSH
chmod 400 my-aws-key.pem

# SSH into EC2 instance
ssh -i my-aws-key.pem ec2-user@YOUR_PUBLIC_IP

# Update all packages
sudo yum update -y

# Install Apache web server
sudo yum install -y httpd

# Start web server
sudo systemctl start httpd

# Enable on reboot
sudo systemctl enable httpd

# Check status
sudo systemctl status httpd

# Create webpage
echo "<h1>My First Web Server</h1><p>Built by Hemant</p>" > /var/www/html/index.html
```

## What I learned
- EC2 is a virtual server running in AWS data centre
- Key pairs use public/private key cryptography — AWS keeps public key, you keep private key
- AWS never stores your private key — zero knowledge security
- Stop = paused, data intact, no compute charge but EBS still costs
- Terminate = permanent deletion, cannot recover
- Public IP changes every stop/start — Elastic IP fixes this
- Always use IAM Role not access keys for EC2 to AWS service communication

## Key concepts
| Concept | What it is |
|---|---|
| EC2 | Virtual server in AWS |
| AMI | Amazon Machine Image — the OS template used to launch EC2 |
| Instance type | Hardware size — t2.micro is free tier (1 CPU, 1GB RAM) |
| Key Pair | Public/private key for SSH — replaces password |
| Security Group | Virtual firewall controlling ports on the instance |
| Public IP | Temporary IP — changes on every stop/start |
| Elastic IP | Reserved static IP — stays fixed, costs money if unused |
| EBS | Elastic Block Store — the hard drive attached to EC2 |
| EC2 Instance Connect | Browser-based SSH — useful for debugging without key |
| IAM Instance Profile | Mechanism to attach IAM Role to EC2 |
| httpd | Apache web server package on Amazon Linux |

## Stop vs Terminate
| Action | Data | Compute charge | EBS charge |
|---|---|---|---|
| Stop | Preserved | Stopped | Continues |
| Terminate | Deleted permanently | Stopped | Stopped (root volume) |

## SSH debugging checklist
If SSH times out check in this order:
1. Key pair permissions — chmod 400 on Mac/Linux, properties on Windows
2. Security Group — port 22 allowed from your IP
3. Subnet — is EC2 in a PUBLIC subnet with IGW route?
4. Public IP — using public IP not private IP, get fresh IP after restart
5. Use EC2 Instance Connect in browser to isolate if problem is key or network

## Public IP vs Elastic IP
Without Elastic IP
Stop/Start EC2  →  New IP assigned  →  DNS breaks  →  App down ❌
With Elastic IP
Stop/Start EC2  →  Same IP always   →  DNS intact  →  App up  ✅
Elastic IP is free when attached to running instance.
Costs ~$3.65/month if allocated but not in use — always release when done.

## Key pair cryptography
Public Key  🔓  →  AWS installs on EC2 at launch
stored in ~/.ssh/authorized_keys on server
Private Key 🔑  →  You keep in .pem file on your laptop
AWS never stores this — zero knowledge security

If you lose your .pem file — create new key pair, AWS cannot recover it.

## IAM Role vs Access Keys on EC2
- Never put access keys on a server — permanent credentials, disaster if hacked
- Always use IAM Role attached as Instance Profile
- Role gives temporary credentials that auto-expire every few hours
- Scope policy to minimum — s3:GetObject on specific bucket only, not s3:* on *

## Architecture of what I built
Internet

↓

Internet Gateway

↓

Public Subnet (ap-south-1a)

↓

EC2 Instance (t2.micro)

├── Security Group: port 80 (HTTP), port 22 (SSH)

├── Apache web server running on port 80

├── Webpage at /var/www/html/index.html

└── Key pair: my-aws-key.pem

## Real world relevance
Platform engineers launch, configure, and maintain EC2 instances constantly.
SSH access, web server setup, key pair management, and IAM roles on EC2
are tested in every platform engineering and DevOps interview. The Elastic IP
vs public IP distinction is critical for production systems where DNS must
always resolve correctly.

## Cost cleanup
- Instance stopped after lab ✅
- No Elastic IPs allocated ✅
- EBS volume preserved with stopped instance (small cost)

## Screenshot
<img width="1920" height="869" alt="image" src="https://github.com/user-attachments/assets/681c017d-5524-4db8-b77f-3d18c972a2fe" />
<img width="1920" height="869" alt="image" src="https://github.com/user-attachments/assets/3761fcb5-3c48-4be6-9ef4-be9f74076a86" />
<img width="1920" height="869" alt="image" src="https://github.com/user-attachments/assets/789796f8-3338-479e-a79a-e6c59d2b8900" />
<img width="1920" height="869" alt="image" src="https://github.com/user-attachments/assets/a74c704b-cb2e-48cb-b464-2905a3b9f828" />
