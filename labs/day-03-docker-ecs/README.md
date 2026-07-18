# Lab 9 — Docker & ECS Container Deployment

**Date:** July 2026  
**Region:** ap-south-1 (Mumbai)  
**Status:** ✅ Complete  
**Score:** 65% on quiz (new topic — expected)

## What I built
- Installed Docker Desktop and ran hello-world container
- Wrote a Dockerfile for a custom nginx web application
- Built Docker image locally and tested on port 8080
- Created private ECR repository `my-web-app`
- Pushed Docker image to ECR with v1 and latest tags
- Created ECS Cluster `hemant-web-cluster` on AWS Fargate
- Created Task Definition `my-web-task` pointing to ECR image
- Created ECS Service with ALB and 2 running tasks
- Tested container self-healing by stopping a task
- Performed zero-downtime rolling update from v1 to v2

## Files created

### Dockerfile
```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

### index.html (v1)
```html
<!DOCTYPE html>
<html>
  <head><title>Hemant's Container App</title></head>
  <body>
    <h1>🐳 Running in a Docker Container!</h1>
    <p>Built by Hemant — Platform Engineer in training</p>
    <p>Deployed on Amazon ECS — ap-south-1 Mumbai</p>
  </body>
</html>
```

## Commands used

### Docker commands
```bash
# Verify installation
docker --version
docker run hello-world

# Pull and run nginx
docker pull nginx
docker run -d -p 8080:80 --name my-nginx nginx

# Build your own image
docker build -t my-web-app:v1 .

# Run locally and test
docker run -d -p 8080:80 --name web-app my-web-app:v1

# View running containers
docker ps

# View all containers including stopped
docker ps -a

# View logs
docker logs web-app

# Stop and remove
docker stop web-app
docker rm web-app

# View all local images
docker images
```

### ECR commands
```bash
# Authenticate Docker to ECR
aws ecr get-login-password --region ap-south-1 | docker login \
  --username AWS --password-stdin \
  726663954833.dkr.ecr.ap-south-1.amazonaws.com

# Tag image for ECR
docker tag my-web-app:v1 \
  726663954833.dkr.ecr.ap-south-1.amazonaws.com/my-web-app:v1

# Also tag as latest
docker tag my-web-app:v1 \
  726663954833.dkr.ecr.ap-south-1.amazonaws.com/my-web-app:latest

# Push to ECR
docker push 726663954833.dkr.ecr.ap-south-1.amazonaws.com/my-web-app:v1
docker push 726663954833.dkr.ecr.ap-south-1.amazonaws.com/my-web-app:latest
```

### Rolling update to v2
```bash
# Build new version
docker build -t my-web-app:v2 .

# Tag and push v2
docker tag my-web-app:v2 \
  726663954833.dkr.ecr.ap-south-1.amazonaws.com/my-web-app:v2
docker push 726663954833.dkr.ecr.ap-south-1.amazonaws.com/my-web-app:v2
```

## Architecture built
```
Internet
    ↓
Application Load Balancer (ecs-web-alb)
    ↓ distributes traffic
    ├── ECS Task 1 (Container running my-web-app:v1)
    └── ECS Task 2 (Container running my-web-app:v1)

ECR Repository (my-web-app)
    └── Images: v1, v2, latest

ECS Cluster (hemant-web-cluster) — Fargate
    └── ECS Service (my-web-service)
        └── Task Definition (my-web-task)
            └── Container: nginx serving index.html
```

## ECS Resources created
| Resource | Name | Purpose |
|---|---|---|
| ECR Repository | my-web-app | Stores Docker images privately |
| ECS Cluster | hemant-web-cluster | Groups ECS services and tasks |
| Task Definition | my-web-task | Blueprint for running container |
| ECS Service | my-web-service | Maintains 2 running tasks |
| Load Balancer | ecs-web-alb | Distributes traffic to containers |

## Task Definition configuration
```
Launch type:  Fargate
CPU:          0.25 vCPU
Memory:       0.5 GB
Container:    my-web-app:v1
Port:         80 (TCP)
```

## Troubleshooting encountered
- CannotPullContainerError — ECS tried to pull :latest but image tagged as :v1
- Fixed by pushing both :v1 and :latest tags to ECR
- Also updated Task Definition to explicitly use :v1 tag
- Lesson: always push both versioned tag AND latest tag to ECR

## Rolling update flow
```
Push v2 to ECR
    ↓
Create new Task Definition revision with v2 image
    ↓
Update ECS Service with new revision
    ↓
ECS starts new v2 container
    ↓
ALB health check passes on new container
    ↓
New container added to load balancer rotation
    ↓
Old v1 container stopped
    ↓
Repeat until all tasks running v2
    ↓
Zero downtime — users never saw interruption
```

## Cost cleanup
```
☐ ECS Service deleted
☐ ECS Cluster deleted  
☐ Load Balancer deleted (~$16-20/month)
☐ Target Group deleted
☐ ECR Repository deleted or kept (free under 500MB)
```

## Screenshot
<img width="1910" height="873" alt="014555PM072026" src="https://github.com/user-attachments/assets/bf15ddec-5ba0-4e73-bf0a-07348416208b" />
<img width="1910" height="873" alt="015439PM072026" src="https://github.com/user-attachments/assets/7e71bf70-d1d1-4fc6-adce-0c7b50b25b92" />
<img width="1910" height="873" alt="125623PM072026" src="https://github.com/user-attachments/assets/26ffc5d9-5f14-417b-9978-42f2a29fe031" />
<img width="1910" height="873" alt="013129PM072026" src="https://github.com/user-attachments/assets/21d56c48-6c32-46f1-b29b-2d45b6a60e75" />
<img width="1910" height="873" alt="013202PM072026" src="https://github.com/user-attachments/assets/8eba0d30-ae4e-497a-afdd-0ddcce90cf22" />
<img width="1910" height="873" alt="013205PM072026" src="https://github.com/user-attachments/assets/23a9e697-07a7-49d4-9ef9-6736b1bce09a" />
<img width="1910" height="873" alt="013212PM072026" src="https://github.com/user-attachments/assets/0738c1a2-dc18-40bd-a538-d4dd057295fd" />
<img width="1910" height="873" alt="013223PM072026" src="https://github.com/user-attachments/assets/e4c59dff-6c67-453f-ade9-2e26f685a06a" />
<img width="1910" height="873" alt="013225PM072026" src="https://github.com/user-attachments/assets/bdfaf965-a70b-4dba-81e6-189eb8a4ebc9" />
<img width="1910" height="873" alt="013230PM072026" src="https://github.com/user-attachments/assets/36e6f45c-db2a-4587-89fa-85d2a2cf5f69" />
<img width="1910" height="873" alt="013238PM072026" src="https://github.com/user-attachments/assets/6b03bc6d-2771-4e0d-a125-4a2cd4ce791a" />
<img width="1910" height="873" alt="013247PM072026" src="https://github.com/user-attachments/assets/af32981b-e32d-4810-b69a-431c73ec799e" />
<img width="1910" height="873" alt="013251PM072026" src="https://github.com/user-attachments/assets/06fc2f89-4797-4072-b53f-39144fc3e130" />
<img width="1910" height="873" alt="013258PM072026" src="https://github.com/user-attachments/assets/514ce7e1-78ef-4a8a-ab55-eefddbdd0878" />
<img width="1910" height="873" alt="013302PM072026" src="https://github.com/user-attachments/assets/5345189f-b3a3-4be2-bc0e-5918ba562dd8" />
