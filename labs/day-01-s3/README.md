# Lab 4 — S3 Static Website Hosting

**Date:** 22 June 2026  
**Region:** ap-south-1 (Mumbai)  
**Status:** ✅ Complete  
**Score:** 75% on quiz

## What I built
- Created an S3 bucket `hemant-aws-lab-2026`
- Disabled Block Public Access
- Uploaded a custom HTML webpage
- Enabled Static Website Hosting
- Added a Bucket Policy to allow public read access
- Hosted a live website on the internet with no server

## Bucket details
- **Bucket name:** hemant-aws-lab-2026
- **Region:** ap-south-1 (Mumbai)
- **Endpoint:** http://hemant-aws-lab-2026.s3-website.ap-south-1.amazonaws.com

## Bucket Policy used
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "PublicReadGetObject",
      "Effect": "Allow",
      "Principal": "*",
      "Action": "s3:GetObject",
      "Resource": "arn:aws:s3:::hemant-aws-lab-2026/*"
    }
  ]
}
```

## Policy explained line by line
| Field | Value | Meaning |
|---|---|---|
| Version | 2012-10-17 | Policy language version — always this date |
| Effect | Allow | Granting permission, not denying |
| Principal | * | Everyone — any person or service on the internet |
| Action | s3:GetObject | Read and download only — no upload, no delete |
| Resource | arn:aws:s3:::bucket/* | Every file inside this bucket |

## What I learned
- S3 is object storage — like Google Drive, accessible from anywhere
- EBS is block storage — like a hard drive, attached to one EC2 instance
- S3 bucket is private by default — Block Public Access must be disabled first
- Bucket Policy defines who can do what with files inside the bucket
- Static Website Hosting creates a public HTTP endpoint for HTML files
- S3 websites are HTTP only — need CloudFront for HTTPS in production
- Pre-signed URLs give temporary access to private files without making bucket public
- Lifecycle Policies automatically move files to cheaper storage tiers over time

## S3 vs EBS comparison
| Feature | S3 | EBS |
|---|---|---|
| Type | Object storage | Block storage |
| Access | From anywhere via URL | Only from attached EC2 |
| Speed | Slower | Fast — good for databases |
| Storage limit | Virtually unlimited | Limited to volume size |
| File types | Any file type | Any file type |
| Existence | Independent of EC2 | Tied to EC2 instance |
| Use case | Files, assets, backups, websites | OS, databases, app storage |
| Cost model | Pay per GB stored | Pay per GB provisioned |

## 403 Forbidden debugging checklist
If you get 403 on S3 check in this order:
1. **Block Public Access** — must be disabled, overrides all bucket policies
2. **Bucket Policy** — must have `s3:GetObject` with `Principal: *`
3. **Static Website Hosting** — must be enabled with correct index document
4. **File name** — index.html must match exactly what's set as index document

## Real world S3 uses for platform engineers
| Use case | Details |
|---|---|
| Static website hosting | Serve HTML/CSS/JS with no server needed |
| Media and asset storage | Images, videos served via CloudFront |
| Log and compliance archiving | Store logs, move to Glacier after 90 days |
| Terraform remote state | Store infrastructure state for team collaboration |

## Key concepts
| Concept | What it is |
|---|---|
| S3 | Simple Storage Service — object storage for any file type |
| Bucket | Container for files in S3 — globally unique name |
| Object | A file stored in S3 — has a key (name) and value (content) |
| Bucket Policy | JSON document defining who can do what with bucket files |
| Block Public Access | Master switch that overrides all bucket policies |
| Static Website Hosting | Feature that serves HTML files via HTTP endpoint |
| Pre-signed URL | Temporary time-limited link to a private S3 file |
| S3 Glacier | Archival storage tier — very cheap, slow retrieval |
| Lifecycle Policy | Auto-moves files between storage tiers based on age |
| CloudFront | CDN that adds HTTPS and global caching in front of S3 |
| Terraform Remote State | Storing Terraform state file in S3 for team use |
| ARN | Amazon Resource Name — unique identifier for any AWS resource |

## Architecture of what I built
```
Browser
    ↓
S3 Static Website Endpoint
    ↓
hemant-aws-lab-2026 bucket
    ↓
index.html (public read via bucket policy)
```

## Production architecture (what comes next)
```
Browser
    ↓
CloudFront (HTTPS + global caching)
    ↓
S3 Bucket (private — only CloudFront can read it)
    ↓
index.html
```

## Real world relevance
S3 is one of the most used AWS services at every company. Platform engineers
use it daily for storing application assets, hosting static frontends,
archiving logs for compliance, and storing Terraform state files. S3 bucket
misconfigurations are the number one cause of data breaches in AWS —
understanding Block Public Access, bucket policies, and least privilege
on S3 is a must for any platform engineering role.

## Cost notes
- S3 Free tier: 5GB storage, 20,000 GET requests, 2,000 PUT requests per month
- Static website hosting: no extra charge
- Data transfer out: first 100GB free per month
- Glacier: ~$0.004 per GB per month (very cheap for archiving)

## Screenshot
<img width="1920" height="869" alt="image" src="https://github.com/user-attachments/assets/b2ebf4fc-45b6-472d-8124-6ea11477635a" />
<img width="1920" height="869" alt="image" src="https://github.com/user-attachments/assets/4c67b415-403f-433f-95c6-e8251aedd035" />
<img width="1920" height="869" alt="image" src="https://github.com/user-attachments/assets/b7499557-673c-497e-b4e2-21dea60eb283" />
<img width="1920" height="869" alt="image" src="https://github.com/user-attachments/assets/ef8ad61d-e472-496a-9df0-8fd7f6e29866" />

