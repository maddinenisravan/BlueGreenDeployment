# EKS Blue/Green Deployment with Jenkins

## Prerequisites
- AWS CLI configured
- Terraform installed
- Python 3 installed
- kubectl installed
- Docker installed
- Docker Hub account

## Steps

### 1. Full Setup

python scripts/setup_all.py

This script will perform below actions:

1. Create the EKS cluster with Terraform
2. Deploy Jenkins to Kubernetes
3. Build & push initial Blue and Green images
4. Deploy Blue & Green deployments
5. Point service to Blue initially


### 2. CI-CD

Jenkins job to automatically update deployment and switchover. Below is the flow:

Two identical environments:

	Blue: live (serving users)

	Green: idle (ready for updates)


On each deployment:

	Detects which version is live

	Deploys new code to the idle version

	Switches service traffic to idle version (making it live)

	Keeps the old live version running for instant rollback



