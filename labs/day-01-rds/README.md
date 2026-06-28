# Lab 6 — RDS & MySQL Database

**Date:** 28 June 2026  
**Region:** ap-south-1 (Mumbai)  
**Status:** ✅ Complete  
**Score:** 84% on quiz

## What I built
- Created a DB Subnet Group using private subnets across 2 AZs
- Created a dedicated Security Group `rds-sg` allowing port 3306 from EC2 only
- Launched a MySQL RDS instance `my-first-db` in a private subnet
- Launched a new EC2 instance inside `my-first-vpc` as a bastion host
- Connected to RDS from EC2 using MySQL client
- Created a database `myappdb`, a `users` table, inserted and queried real data

## Commands used
```bash
# Install MySQL client on Amazon Linux 2023
sudo yum install -y mariadb105

# Connect to RDS from EC2
mysql -h my-first-db.c18a2mucwv09.ap-south-1.rds.amazonaws.com -u admin -p
```

```sql
-- See all databases
SHOW DATABASES;

-- Switch to your database
USE myappdb;

-- Create a table
CREATE TABLE users (
  id INT AUTO_INCREMENT PRIMARY KEY,
  name VARCHAR(100),
  email VARCHAR(100),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Insert data
INSERT INTO users (name, email) VALUES ('Hemant', 'hemant@example.com');
INSERT INTO users (name, email) VALUES ('Platform Engineer', 'pe@example.com');

-- Query data
SELECT * FROM users;

-- Exit
EXIT;
```

## Output
```
+----+----------+--------------------+---------------------+
| id | name     | email              | created_at          |
+----+----------+--------------------+---------------------+
|  1 | Hemant   | hemant@example.com | 2026-06-28 12:03:52 |
|  2 | platform | pp@example.com     | 2026-06-28 12:04:10 |
+----+----------+--------------------+---------------------+
```

## Architecture built
```
Internet
    ↓
Internet Gateway
    ↓
Public Subnet (ap-south-1a)
  EC2 Bastion Host ← SSH from laptop
    ↓ port 3306 (allowed by rds-sg)
Private Subnet (ap-south-1b)
  RDS MySQL (my-first-db)
    └── myappdb
        └── users table
```

## Key concepts
| Concept | What it is |
|---|---|
| RDS | Managed relational database service — AWS handles maintenance |
| DB Subnet Group | Defines which subnets RDS can launch into — needs 2 AZs minimum |
| Multi-AZ | RDS standby in another AZ for automatic failover |
| Bastion Host | EC2 used as jump server to access private resources |
| Private Endpoint | RDS connection string — only reachable within VPC |
| mariadb105 | MySQL client package name on Amazon Linux 2023 |
| Port 3306 | MySQL default port — must be open in security group |
| rds-sg | Security group allowing port 3306 from web-sg only |

## Cost cleanup
- RDS stopped temporarily after lab ✅
- EC2 instances stopped ✅
- RDS free tier: 750 hours/month on db.t3.micro or db.t4g.micro

## Screenshot
<img width="874" height="344" alt="image" src="https://github.com/user-attachments/assets/27ff5233-9250-44d3-b6fa-c0c6da7dfc82" />
<img width="1910" height="873" alt="image" src="https://github.com/user-attachments/assets/2fbdd0ea-1f38-463f-8482-a23c6a0af351" />
<img width="1910" height="873" alt="image" src="https://github.com/user-attachments/assets/f04168d0-1da1-4df8-a383-07e7ec6795b0" />
<img width="1910" height="873" alt="image" src="https://github.com/user-attachments/assets/af6ea8eb-5ca7-4984-909d-037010639599" />
<img width="1910" height="873" alt="image" src="https://github.com/user-attachments/assets/bea8a3cc-e53a-4777-a955-a81fa29e4201" />
<img width="1910" height="873" alt="image" src="https://github.com/user-attachments/assets/a86c8914-85af-4336-9805-26c7e403972b" />
