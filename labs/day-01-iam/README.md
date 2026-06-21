# Lab 1 — IAM & Root Account Security

**Date:** 21 June 2026  
**Region:** ap-south-1 (Mumbai)  
**Status:** Complete

## What I built
- Enabled MFA on the root account using Google Authenticator
- Created an IAM admin user (`admin-hemant`) with AdministratorAccess policy
- Set up a billing budget alert at $10/month
- Logged out of root and switched to IAM user for all future work

## What I learned
- Root account has unrestricted access and should almost never be used
- IAM users are individual human identities — one per person
- IAM policies define what actions are allowed on which resources
- Billing budgets protect your account from unexpected charges

## Key concepts
| Concept | What it is |
|---|---|
| Root account | Master account — full unrestricted access, use only when absolutely necessary |
| IAM User | Individual identity for a human — has its own login credentials |
| IAM Policy | JSON document that defines permissions attached to users/roles |
| MFA | Multi-factor authentication — second layer of security on top of password |
| Billing Budget | Alert that notifies you when AWS spend crosses a threshold |

## IAM best practices I applied
- MFA enabled on root
- No root access keys created
- IAM admin user created for daily work
- Least privilege principle understood
- Billing alert configured

## Real world relevance
Every company that uses AWS locks down root on day one. Platform engineers 
are responsible for setting up this foundation for the entire organization. 
IAM is tested in every AWS interview — understanding Users, Roles, and 
Policies is non-negotiable.

## Commands used
```bash
# SSH into EC2 (practiced in Lab 3)
chmod 400 my-key.pem
ssh -i my-key.pem ec2-user@YOUR_IP
```

## Screenshot
<img width="1347" height="502" alt="image" src="https://github.com/user-attachments/assets/a8ee093c-1c4b-4f8b-92f1-4c0e01c2e834" />
