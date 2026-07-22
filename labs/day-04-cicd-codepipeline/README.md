# Lab 10 — CI/CD Pipeline with CodePipeline

**Date:** July 2026  
**Region:** ap-south-1 (Mumbai)  
**Status:** ✅ Complete  

## What I built
- Created GitHub repository with app code, Dockerfile and buildspec.yml
- Created ECR repository for Docker images
- Set up ECS Cluster and Service on Fargate
- Created CodeBuild project to build and push Docker images
- Created CodePipeline with 3 stages — Source, Build, Deploy
- Connected GitHub to CodePipeline via GitHub App
- Pushed code change and watched pipeline auto-trigger
- Fixed ALB security group to allow HTTP from internet

## Pipeline architecture
```
Developer pushes code to GitHub
    ↓ triggers automatically (within 30 seconds)
CodePipeline starts
    ↓
Stage 1 — Source
  pulls latest code from GitHub main branch
    ↓
Stage 2 — Build (CodeBuild)
  runs buildspec.yml
  builds Docker image
  pushes to ECR with commit SHA tag
  creates imagedefinitions.json
    ↓
Stage 3 — Deploy (Amazon ECS)
  reads imagedefinitions.json
  updates ECS service with new image
  rolling deployment — zero downtime
    ↓
Live app updated automatically ✅
```

## Files created

### buildspec.yml
```yaml
version: 0.2

phases:
  pre_build:
    commands:
      - echo Logging in to Amazon ECR...
      - aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com
      - REPOSITORY_URI=$AWS_ACCOUNT_ID.dkr.ecr.ap-south-1.amazonaws.com/hemant-web-app
      - IMAGE_TAG=$(echo $CODEBUILD_RESOLVED_SOURCE_VERSION | cut -c 1-7)

  build:
    commands:
      - echo Building Docker image...
      - docker build -t $REPOSITORY_URI:latest .
      - docker tag $REPOSITORY_URI:latest $REPOSITORY_URI:$IMAGE_TAG

  post_build:
    commands:
      - echo Pushing image to ECR...
      - docker push $REPOSITORY_URI:latest
      - docker push $REPOSITORY_URI:$IMAGE_TAG
      - echo Writing imagedefinitions.json...
      - printf '[{"name":"web-container","imageUri":"%s"}]' $REPOSITORY_URI:$IMAGE_TAG > imagedefinitions.json

artifacts:
  files:
    - imagedefinitions.json
```

### Dockerfile
```dockerfile
FROM nginx:alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

## Pipeline configuration
```
Pipeline name:    hemant-web-pipeline
Pipeline type:    V2
Execution mode:   SUPERSEDED
Service role:     auto-created by AWS

Source stage:
  Provider:       GitHub via GitHub App
  Repository:     hemantbhadke/hemant-web-app
  Branch:         main
  Auto-trigger:   true

Build stage:
  Provider:       AWS CodeBuild
  Project:        hemant-web-build
  Privileged:     true (required for Docker)

Deploy stage:
  Provider:       Amazon ECS
  Cluster:        hemant-web-cluster
  Service:        my-web-service
  Image file:     imagedefinitions.json
  Auto rollback:  enabled
```

## Troubleshooting encountered
- ALB security group only had self-referencing rule
- No HTTP port 80 rule from 0.0.0.0/0
- Fixed by adding HTTP inbound rule from internet
- Lesson: always verify ALB security group allows HTTP from 0.0.0.0/0

## Key insight — imagedefinitions.json
```json
[{"name":"web-container","imageUri":"726663954833.dkr.ecr.ap-south-1.amazonaws.com/hemant-web-app:abc1234"}]
```
This file is the handoff between Build and Deploy stages.
CodeBuild creates it, CodeDeploy reads it to know which image to deploy.

## Cost cleanup
```
☐ CodePipeline deleted
☐ CodeBuild project deleted
☐ ECS Service set to 0 tasks then deleted
☐ ECS Cluster deleted
☐ ALB deleted (~$16-20/month)
☐ Target Group deleted
☐ ECR Repository deleted
```

## Screenshot
<img width="1910" height="873" alt="041056PM072026" src="https://github.com/user-attachments/assets/389b956b-95e1-4685-92af-2a3d230aeaa1" />
<img width="1910" height="873" alt="041100PM072026" src="https://github.com/user-attachments/assets/a6988ffd-2cfa-4017-a582-8ba62fb9966d" />
<img width="1910" height="873" alt="041110PM072026" src="https://github.com/user-attachments/assets/acc67458-fc5e-40f0-89b2-07278ba3470a" />
<img width="1910" height="873" alt="041117PM072026" src="https://github.com/user-attachments/assets/22446524-444d-49fa-b69a-60a0a66c279b" />
<img width="1920" height="1080" alt="022501PM072026" src="https://github.com/user-attachments/assets/31d1dde2-bec0-42e2-8be5-c7b1bc8f3528" />
<img width="1910" height="873" alt="041038PM072026" src="https://github.com/user-attachments/assets/9d6d269c-147a-4032-9581-fad81290e2af" />
<img width="1910" height="873" alt="041044PM072026" src="https://github.com/user-attachments/assets/d987c9e4-1bc0-4511-bb24-aa87f331c56e" />
<img width="1910" height="873" alt="041051PM072026" src="https://github.com/user-attachments/assets/70313bc3-c848-4251-8a0d-4dfd17e0a020" />
