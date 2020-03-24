# gamesday
Docker 101
1) write a dockerfile to containerise php app 
2) build image
3) run docker image

Docker exercises
Language
GitHub
Go
https://github.com/callicoder/go-docker
Python
https://github.com/lvthillo/python-flask-docker
PHP
https://github.com/aws-samples/ecs-demo-php-simple-app
Node.js
https://github.com/fwaibel/nodejs-docker-webapp.git

Docker cheatsheet
https://cheatsheet.dennyzhang.com/cheatsheet-docker-A4

Sample SQL Syntax
create database myDBName;
grant all on myDBName.* to 'master' identified by 'password';
use myDBName;
CREATE TABLE mytable (id VARCHAR(100),hash VARCHAR(200));

Kinesis Firehose + Athena
===============
https://aws.amazon.com/blogs/big-data/analyzing-vpc-flow-logs-with-amazon-kinesis-firehose-amazon-athena-and-amazon-quicksight/
Docker
===========
docker build -t phpapp .
docker run -d -p 80:80 phpapp
--to list--
docker ps -a
--to stop--
docker stop {container name}
--to start--
docker start {container name}
--to remove--
docker rm {container name}
--to manage images--
docker image ls
docker image rm {image name}
--to tag images--
docker tag {image name} {tag}
--push multiple times to get multiple tags--
Code Deploy for EC2
===========
set EC2 S3 IAM Role
install CodeDeploy agent through User Data on deployment EC2
--for github--
go to app root directory
git config --global user.email "{emailhere}"
git remote add origin {something.git}
git add .
git commit -m "Your Desc"
git push -u origin master
Code Deploy for ECS
===============
create ALB
create ECS task definition and service (specify blue green, create two target groups)
--to update deployment--
create new taskdef.json (note ARN, containerName and Port)
Code Pipeline for ECS (No Blue/Green)
================
must add buildspec.yml to root directory
use codebuild image for environment and privileged mode
add ecr poweruser permissions in codebuild iam role
ecs deployment must be rolling update & must specify image definition.json file
Code Pipeline + Deploy(Blue/Green)
===================
change taskdef.json as needed ( cpu and memory min. 256 and 512 respectively)
remember to change original termination to 5 mins
Include appspec.yml also
Code Commit
===========
Use https url. needs iam role w/ policies
ECS EC2 Cluster Scaling (Capacity Provider)
=============================
must setup capacity provider before creating service etc
Use non managed scaling :D
Logging Container logs
======================
use symlink e.g.
ln -s /dev/stdout ./access.log
Must include logging definition and execution task role in taskdef.json for CD Blue/Green
Refund
===============
Post command must be JSON type

Ryan (Small Notes)
CodePipeline
https://ap-southeast-1.console.aws.amazon.com/codesuite/codepipeline/pipelines?region=ap-southeast-1
Create pipeline
New service role
 
Add source provider
CodeCommit
S3
GitHub
CodeBuild
BuildSpec
Use Amazon Linux 2 image
Example BuildSpec
Add '["name":"simple-app", "imageUri":"%s"]' "$ECR_REPO" > images.json to post build
Add images.json as an artifact
Environment variables
Region and ECR repository
Requires EC2ContainerRegistryPowerUser policy attached to IAM role
 
Deploy stage (Skip deploy stage until ECS is deployed)
ECS
Requires images.json
ECS Blue/Green
Requires appspec.yml and taskdef.json
https://docs.aws.amazon.com/codepipeline/latest/userguide/tutorials-ecs-ecr-codedeploy.html
Make both an artifact of the build stage
Extract appspec.yml and taskdef.json from BuildArtifact
**Remember to change default termination from 1hr to 5mins**

ECS
 Create cluster
 
Create service provider (Optional | Can be replaced with manual scaling)
 
ECR repository
 
Task definition
 
Create service
 
**Note**
Fargate launch type requires IP target groups instead of instance
Create 2 service providers to speed up scaling
For EC2 task definition use port 0 for host
 
Log Driver (Containers)
Remember to put symbolic link from log file to stdout:
https://serverfault.com/questions/599103/make-a-docker-application-write-to-stdout
 
ECS-EC2 normal deployment
Create taskdef in console with logging enabled
Then continue to pipeline
AWS imports previous configurations from taskdef into new taskdef
 
ECS-EC2-Blue/Green
Create taskdef in console with logging enabled
Create taskdef.json and appspec.yaml in repository
Copy taskdef from taskdefinition to repository
 
ECS-Fargate normal deployment
Same as ECS-EC2 normal deployment
 
ECS-Fargate Blue/Green
Same as ECS-EC2-Blue/Green
 
**Note make sure there is a log group created beforehand | Deployment may be stuck in install loop if not**
**AWS console automatically creates a log group if taskdefinition is done in console**
 
Normal EC2 logs agent
https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/QuickStartEC2Instance.html

https://rollout.io/blog/building-minimal-docker-containers-for-go-applications/ (Build binary and run app)

Creating a golang docker image and running the binary
Building containers
![test1](https://184723g-public-tmp.s3-ap-southeast-1.amazonaws.com/1new.png)

Lambda refund
![test2](https://184723g-public-tmp.s3-ap-southeast-1.amazonaws.com/3.png)

2 stage Pipeline
Codecommit -> CodeDeploy
 
Supports EC2 and EC2-ASG deployments with/without B/G deployment
Required file:
appspec.yaml
![test3](https://184723g-public-tmp.s3-ap-southeast-1.amazonaws.com/new3.png)

Dockerfile
https://github.com/callicoder/go-docker/blob/master/Dockerfile
https://github.com/aws/aws-sdk-go

Buildspec https://github.com/awslabs/ecs-refarch-continuous-deployment/blob/master/templates/deployment-pipeline.yaml

For B/G
Buildspec
- https://github.com/awslabs/ecs-refarch-continuous-deployment/blob/master/templates/deployment-pipeline.yaml
Taskdef
- https://aws.amazon.com/blogs/devops/use-aws-codedeploy-to-implement-blue-green-deployments-for-aws-fargate-and-amazon-ecs/
- Copy from ECS Task Definition
Appspec
- https://docs.aws.amazon.com/codedeploy/latest/userguide/reference-appspec-file-example.html#appspec-file-example-ecs

ECR
Write own docker file and add 
RUN go get -u github.com/aws/aws-sdk-go
RUN ln -sf /dev/stdout ./access.log

docker build -t goapp:v1 .
docker run -p 80:80 goapp:v1

aws ecr get-login-password --region ap-southeast-1 | docker login --username AWS --password-stdin 043504510473.dkr.ecr.ap-southeast-1.amazonaws.com
docker images
docker tag 043504510473.dkr.ecr.ap-southeast-1.amazonaws.com/ecr_main:latest
docker push 043504510473.dkr.ecr.ap-southeast-1.amazonaws.com/ecr_main:latest
To push to ECR
https://docs.aws.amazon.com/AmazonECR/latest/userguide/Registries.html#registry_auth
https://docs.aws.amazon.com/AmazonECR/latest/userguide/docker-push-ecr-image.html
Note: if tag is omitted, it will be the latest

ECS
https://docs.aws.amazon.com/codedeploy/latest/userguide/tutorial-ecs-deployment.html
Task Def
Task role: access to other services
Fargate
Target group: ip
0.5, 0.25
Port mapping: 80
EC2
Memory: 350
Port mapping: 0:80
Task Execution Role (Policy: AmazonECSTaskExecutionRolePolicy)
Container: simple-app (image pushed to ECR)
Auto Scaling
https://aws.amazon.com/blogs/compute/automatic-scaling-with-amazon-ecs/
https://docs.aws.amazon.com/AmazonECS/latest/developerguide/tutorial-cluster-auto-scaling-console.html
ECS container instance IAM role
ECS service-linked IAM role
Auto Scaling service-linked IAM role

Step 1: Create ECS cluster

Step 2: Create Auto Scaling Resources
1) Create Launch Config
- AMI: Amazon ECS-Optimized Amazon Linux 2 AMI
- IAM Role: ECS container instance IAM role
- User data
2) Create 2 ASG
- Group size: 0 (ECS managed scaling so there is no need to have the Auto Scaling group launch any initial instances)
- Instance Protection: Protect From Scale In (managed termination protection: prevents container instances that contain tasks from being terminated during scale-in)
- Keep this group at its initial size (ECS managed scaling so there is no need to create a scaling policy)
- Max: 100

Step 3: Create 2 Capacity Providers
- Use both ASGs
- Managed scaling: Enabled (default)
- Target capacity %: 100
- Managed termination protection: Enabled (default)

Step 4: Set a Default Capacity Provider Strategy for the Cluster
- Update cluster > Add provider
- Default: Base value: 0, Weight value: 1
- Add 2 providers

Step 5: Register a Task Definition
- Create new Task Definition > EC2
- Configure via JSON

Step 6: Run a Task
- Actions > Run task
- Number of tasks: 5
- Placement Templates: BinPack
B/G
https://aws.amazon.com/blogs/devops/use-aws-codedeploy-to-implement-blue-green-deployments-for-aws-fargate-and-amazon-ecs/

CodePipeline
https://docs.aws.amazon.com/codepipeline/latest/userguide/actions-invoke-lambda-function.html#actions-invoke-lambda-function-add-action
https://docs.aws.amazon.com/codepipeline/latest/userguide/tutorials-cloudwatch-sns-notifications.html
https://docs.aws.amazon.com/codepipeline/latest/userguide/approvals.html
https://aws.amazon.com/blogs/compute/continuous-deployment-to-amazon-ecs-using-aws-codepipeline-aws-codebuild-amazon-ecr-and-aws-cloudformation/
https://github.com/awslabs/ecs-refarch-continuous-deployment
- Deploy: ECS B/G
Artifacts from Source (taskdef, appspec) & Build (buildspec: images.json)

CodeDeploy
- Create App
- Create Deployment Group (Service role, Deployment type: In-place, Deployment config: CodeDeployDefault.AllatOnce)
- Environment config: Key and Value, enter the instance tag key and value that was applied to your instance as part of Step 4: Provision an Instance

Refund
1) Write own docker file and add this line (!IMPORTANT)
- RUN ln -sf /dev/stdout ./access.log
2) ECS Task definition, CloudWatch
- Ensure awslog driver is used to send logs to CloudWatch
3) Lambda
- Python
- Trigger: CloudWatch Metric filter
- https://stackoverflow.com/questions/50295838/cloudwatch-logs-stream-to-lambda-python

CloudWatch Agent
https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/install-CloudWatch-Agent-commandline-fleet.html
https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/create-cloudwatch-agent-configuration-file-wizard.html

Lambda
https://blog.shikisoft.com/send-sms-with-sns-aws-lambda-python/

RDS
https://www.technlg.net/aws/create-mysql-user-aws-rds/

WordPress
https://docs.aws.amazon.com/codedeploy/latest/userguide/tutorials-wordpress.html
Create app and push to s3 using aws deploy cli (ensure EC2 has permission to create app)
aws deploy push   --application-name WordPress_App   --s3-location s3://tjx123/WordPressApp.zip   --ignore-hidden-files --region ap-southeast-1
aws deploy create-deployment --application-name WordPress_App --s3-location bucket=tjx123,key=WordPressApp.zip,bundleType=zip,eTag=0a3edb55541480245d4cdedc2293185c-3  --deployment-group-name WordPress_Deploy --deployment-config-name CodeDeployDefault.OneAtATime --region ap-southeast-1
{
    "deploymentId": "d-SWA4E4SE2"
}

https://aws.amazon.com/blogs/compute/continuous-deployment-to-amazon-ecs-using-aws-codepipeline-aws-codebuild-amazon-ecr-and-aws-cloudformation/
https://github.com/awslabs/ecs-refarch-continuous-deployment

1) VPC
2 Public subnets

2) Load Balancer
ALB Security Group: 80 from 0.0.0.0/0
1 target group, 1 listener
Target type: Instance
Deregistration delay: 30s
Health check (success codes 200-299)

3) ECS
- Instance Profile: ECS Role for EC2 (Policy: AmazonEC2ContainerServiceforEC2Role)
- Security Group: All traffic from ALB Security Group
- Cluster
-- EC2 Linux + Networking
-- Container instance IAM role: Instance Profile: ECS Role for EC2
- Launch Config
- Create cluster with autoscaling group

4) Service
- Task Execution Role (Policy: AmazonECSTaskExecutionRolePolicy)
- Task definition
-- Compatibility: EC2
-- Task Execution Role (Policy: AmazonECSTaskExecutionRolePolicy)
-- 2 containers, 1 volume
- EC2 Service
- Task points to a log group

5) CodePipeline
- Create S3 bucket
- Create ECR repo
- CodePipeline service role
Create Pipeline with app: GitHub, Build: CodeBuild and Deploy: ECS
b) Build
-- Create CodeBuild project
-- CodeBuild Service role
-- Point to ECR
c) Deploy
-- ECS

6) CodeCommit Repository
- Clone the repository
- Add files, make commit and push to master
sudo pip install git-remote-codecommit
git clone codecommit::ap-southeast-1://Repo1
cd Repo1/
git add *
git commit -m "1st commit"
git push -u origin master
- Pipeline: use CloudWatch events to take note of changes
