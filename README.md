# AWS CI/CD With Elastic Beanstalk

This will deploy a simple index.html website to Elastic Beanstalk using GitHub.

# PHASE 1 — Setup GitHub Repo

- Create a new GitHub repo: `eb-node-api-ci-cd`
- Add 2 files:
1. index.html
```
<h1>Hello from AWS CI/CD with Elastic Beanstalk (2025)</h1>
```
2. buildspec.yml (required by CodeBuild)
```
version: 0.2

phases:
  build:
    commands:
      - echo "Packaging static website..."

artifacts:
  files:
    - "**/*"
```
- Push to GitHub.


# PHASE 2 — Create IAM Roles

A) Create CodePipeline Role

Open AWS Console → IAM

Left sidebar → Roles

Click Create role

Trusted entity type:`Custom trust policy`

Custom trust policy Inline Code:
```
{
	"Version": "2012-10-17",
	"Statement": [
		{
			"Effect": "Allow",
			"Action": [
				"elasticbeanstalk:*",
				"cloudformation:*",
				"ec2:DescribeInstances",
				"ec2:DescribeImages",
				"ec2:DescribeKeyPairs",
				"ec2:DescribeSecurityGroups",
				"ec2:DescribeSubnets",
				"ec2:DescribeVpcs",
				"ec2:DescribeInstanceTypes",
				"ec2:DescribeLaunchTemplates",
				"ec2:DescribeLaunchTemplateVersions",
				"ec2:DescribeAvailabilityZones",
				"ec2:DescribeAddresses",
				"autoscaling:DescribeAutoScalingGroups",
				"autoscaling:DescribeLaunchConfigurations",
				"autoscaling:DescribeScalingActivities"
			],
			"Resource": "*"
		},
		{
			"Effect": "Allow",
			"Action": [
				"s3:GetObject",
				"s3:PutObject",
				"s3:ListBucket"
			],
			"Resource": "*"
		}
	]
}
```
Click Next

Permissions policies:

✔ AmazonS3FullAccess
✔ AWSCodeBuildAdminAccess
✔ AWSCodeDeployRole
✔ AWSCodePipeline_FullAccess

Role name:`CodePipeline-Role-2025`
Click Create role

B) Create CodeBuild Role

IAM → Roles → Create Role

Trusted entity: AWS Service

Use case: CodeBuild

Next

Attach policies:

✔ AWSCodeBuildAdminAccess
✔ AmazonS3FullAccess
✔ AmazonEC2ContainerRegistryFullAccess
✔ CloudWatchLogsFullAccess

Role name:`CodeBuild-EB-Role `
Create.

C) Elastic Beanstalk EC2 Instance Role

Check IAM → Roles search:

aws-elasticbeanstalk-ec2-role


If it does not exist → create:

IAM → Roles → Create Role

Use case → Elastic Beanstalk

Attach:

✔ AWSElasticBeanstalkWebTier
✔ AWSElasticBeanstalkWorkerTier
✔ AWSElasticBeanstalkMulticontainerDocker

Role name: `aws-elasticbeanstalk-ec2-role `


# PHASE 3 — Create Elastic Beanstalk Environment

1) Go to Elastic Beanstalk

AWS Console → Search → “Elastic Beanstalk”

2) Click Create Application

Application name:`Eb-web-app-env`

3) Platform

Select:

Platform: PHP 8.4 
Platform Branch: recommended latest version

Application name: eb-web-app

Environment name: something like:

eb-web-app-env


Environment URL:
http://Eb-web-app-env.eba-ajpmftbz.ap-south-1.elasticbeanstalk.com

4) Application code

Select:

Use sample application

5) Configure service access 

Service role:
`aws-elasticbeanstalk-service-role`

EC2 instance profile:
`aws-elasticbeanstalk-ec2-role`

6) Set up networking, configure instance traffic and scaling

You will see 3 fields:

- VPC
- Subnet
- Load balancer type

Instance subnets all selected.
Leave everything as default.

✔ VPC → default
✔ Subnets → default
✔ Load balancer → none / default

EC2 security groups: select security group which have HTTP and SSH inbound config.

7) Configure updates, monitoring, and logging

Monitoring:

System : Basic

Managed platform updates :

Managed updates: Disable

remaining everthing default

8) Review & Create.

<img width="2560" height="1756" alt="image" src="https://github.com/user-attachments/assets/d67f5b42-9a70-4867-9a73-f5cbe68936f9" />


# PHASE 4 — Create S3 Bucket for Build Artifacts

Go to S3 → Create Bucket

Bucket Name:`eb-cicd-artifacts-yash-2025`
Region: `us-east-1`

Keep “Block public access” ON.

Create.

<img width="2552" height="1020" alt="image" src="https://github.com/user-attachments/assets/06532107-53c0-4292-94cd-7fc75f08489f" />

# PHASE 5 — Create CodeBuild Project

Go to → CodeBuild → Build projects → Create build project

## STEP 1 — Project details

Name: `EB-Build-Project`

## STEP 2 — Source

Source Provider: GitHub

Connect to GitHub

Select your repo: [aws-eb-cicd-2025-demo](https://github.com/yashkanakiya/eb-node-api-ci-cd)

Branch: main

## STEP 3 — Environment

Environment image: Managed Image

Operating system: Amazon Linux 2023

Runtime: Standard

Image version: Always use the latest image

Privileged mode: OFF

Service role:
Choose existing → CodeBuild-EB-Role

## STEP 4 — Buildspec

Select: Use a buildspec file

(This will use buildspec.yml from GitHub)

## STEP 5 — Artifacts

Type: Amazon S3

Bucket: eb-cicd-artifacts-yash-2025

Name:build.zip

Artifacts packaging: None

Click Next → Create Build Project.

<img width="2560" height="5462" alt="image" src="https://github.com/user-attachments/assets/31159df0-15b5-4e4a-8c4d-90e77f9ee892" />

# PHASE 6 — Create CodePipeline

Go to CodePipeline → Create Pipeline → Build custom pipeline

## STEP 1 — Pipeline Settings

Name:

`EB-CICD-Pipeline`

Service Role:

Choose existing → CodePipeline-EB-Role

Click Next.

## STEP 2 — Source

Source Provider: GitHub

Connect GitHub

Select repo → yashkanakiya/eb-node-api-ci-cd

Branch: main

Detect changes → GitHub Webhook

Next.

## STEP 3 — Build

Provider: AWS CodeBuild

Project name: EB-Build-Project

Next.

## STEP 4 — Test

Click:

Skip stage

Next.

## STEP 5 — Deploy

Provider: Elastic Beanstalk

Application name: eb-web-app

Environment: eb-web-app-env

Input artifact: Select build.zip (automatically shown)

Next.

## STEP 6 — Review

Check values.

## STEP 7 — Create Pipeline

Click Create Pipeline

Pipeline will now run:

- Pull source from GitHub

- Build/zip with CodeBuild

- Upload to S3

- Deploy to Elastic Beanstalk

<img width="2552" height="786" alt="image" src="https://github.com/user-attachments/assets/3017c98b-bcbd-4426-ac6b-7e20d4709fb2" />


# PHASE 7 — Test Your CI/CD Pipeline

Open your GitHub repo → Edit index.html:

Change:

```<h1>Hello from AWS CI/CD with Elastic Beanstalk (2026)</h1>```


Commit & push.

Go to CodePipeline → It should auto-start → Deploy successfully.

Then open your Elastic Beanstalk URL → You will see updated content.

<img width="1357" height="1172" alt="image" src="https://github.com/user-attachments/assets/ac9a212b-42f3-4109-b534-74828aca42a7" />








