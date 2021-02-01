# Section-6-Deploying-Sample-App-on-AWS-Without-CI-CD


# Deploy Sample App on single ec2 from CodeCommit- Lab

**Step 1.Launch an Instance with default settings**

**Step 2.Ec2>Actions>Connect>connect**

**Step 3.Run the following commands**
```sh
$ sudo su 
$ yum update -y
$ curl -sL https://rpm.nodesource.com/setup.lts.x | bash -
$ yum install nodejs -y
$ node -v
$ yum install git -y
$ git --version
$ git clone https://git.codecommit.ap.south-1.amazonaws.com/v1/repos/Sample-Node-App
# Provide User name & Password
$ Clear
$ ls 
$ cd Sample-Node-App/
$ npm install
$ node app.js
```
**Step 4.Copy the INSTANCE Public ip address and paste it in browser**
- Not coming up due to no port 3000 open

**Step 5.Ec2>Security Groups>select your SG>Edit inbound rules**
- Add rule 
   - Type - Custom TCP 
   - Port - 3000 
   - Source - 0.0.0.0/0
  
Click to Save rules

**Step 5.Copy the INSTANCE Public ip address and paste it in browser**
Now it is Up and running.

### End of Lab

# Deploy Sample App on single ec2 from S3 - Lab

**Step 1. AWS Management Console>Services>S3>Create Bucket**

**Step 2. Give following details**
- Bucket name - teacheramitk
- Region

Click on Create Bucket

**Step 3. Click on bucket teacheramitk>Upload**

**Step 4. Open Visual Studio Code**
- Open new-node-project Folder
- Open the Terminal
- Type the following commands
```sh
$ zip -r new-node-project.zip . -x ".*" -x "_MACOSX"
```

**Step 5.Click on bucket teacheramitk>Upload>Add files>new-node-project.zip**
- Click on Upload

**Step 6. Click on new-node-project.zip**
- Copy S3 URI

**Step 7.Create Ec2 Instance**
- Use Default settings to create Ec2
- Security group with 80,22,300 Port Open

**Step 8.EC2>Instances>Instance-xxx>Actions>Connect>connect**
```sh
$ sudo su
$ yum update -y
$ curl -sL https://rpm.nodesource.com/setup_lts.x | bash -
$ yum install nodejs -y
$ aws s3 cp s3://teacheramitk/new-node-project.zip
# fatal Error
```
**Step 9. Goto AWS Management Console>Services>IAM>Roles>EC2S3FullAccess**
- Click on Permissions to check the policy

**Step 10. Goto Ec2Dashboard>Actions>Security>Modify IAM role**
- IAM role - EC2S3FullAccess

Click on Save

**Step 11. Goto Step 8 and continue..**
```sh
$ aws s3 cp s3://teacheramitk/new-node-project.zip
$ ls
$ unzip new-node-project
$ ls
$ npm install
$ node app.js
```
**Step 11. Copy the Public Ip address and paste in the browser**
- Application is working fine


# Create Auto Scaling Lifecycle Hook - Lab

**Step 1. Goto AWS Management Console>Services>Ec2>Ec2Dashboard>Auto Scaling>Launch Configurations**

**Step 2. Click on Create Launch configuration**
        - Give name - test-lc
        - AMI - Copy AMI-Id from launch Instance
        - Instance type- choose t2.micro
        - IAM Instance profile - EC2S3FullAccess

**Step 3. Click on Advance details>user data>As text**
Type the following commands-
```sh
#!/bin/bash
yum update -y
curl -sL https://rpm.nodesource.com/setup_lts.x | bash -
yum install nodejs -y
npm install -g pm2
pm2 update
cd /home/ec2-user
aws s3 cp s3://teacheramitk/new-node-project.zip
unzip new-node-project
npm install
pm2 start -f app.js
sleep 10
INSTANCE_ID=$(curl -s http://169.254.169.254/latest/meta-data/instance-id)
aws autoscaling complete-lifecycle-action --lifecycle-action-result CONTINUE --instance-id
$INSTANCE_ID --lifecycle-hook-name test-hook--auto-scaling-group-name test-asg --region ap-south-1 || \
aws autoscaling complete-lifecycle-action --lifecycle-action-result ABANDON --instance-id
$INSTANCE_ID --lifecycle-hook-name test-hook --auto-scaling-group-name test-asg --region ap-south-1
```
**Step 4. Select Security Group and key pair**

**Step 5. Click Create Launch Configurations**
Successfully created launch configuration test-lc

### End of lab


# Deploy Sample App on EC2 Fleet - Lab

**Step 1 Goto AWS Management Console>Services>Ec2>Ec2Dashboard>Auto Scaling>Auto Scaling groups>test-asg>edit**
- Desired capacity - 1
- Minimum capacity - 1
- Maximum capacity - 1

Click on Update

**Step 2.Click Refresh on EC2>Auto Scaling Groups>Instances**
- Lifecycle shows stages as following 
  - Pending 
  - Pending Wait 
Pending wait is not changing. 
  
**Step 3.Copy the Instance Ip and paste it in the browser to see is it Running or not**
- It is Running 
 
**Step 4.Ec2Dashboard>Connect & see the instance**
```sh
$ ls
$ aws autoscaling complete-lifecycle-action --lifecycle-action-result CONTINUE --instance-id
# Error-Access denied
```
**Step 5.Goto IAM>Roles>EC2S3FullAccess>Permissions>Attach policies**
- Search for AutoScalingFullAccess
- Attach policy AutoScalingFullAccess

**Step 6.Ec2>Instances>Instance-id**
- See that Instance is Terminated
- New instance is created

**Step 7.Goto AWS Management Console>Services>Ec2>Ec2Dashboard>Auto Scaling>Auto Scaling groups>test-asg>edit**
- Desired capacity - 0
- Minimum capacity - 0
- Maximum capacity - 0

Click on Update

**Step 8.EC2>Auto Scaling groups>test-asg>Instance management**
- See lifecycle shows Pending>Pending:Wait>InService

**Step 9. Click on Refresh and Instance us Terminated**

**Step 10. Edit test-asg and do following**
- Desired capacity - 3
- Minimum capacity - 3
- Maximum capacity - 3

Click on Update

**Step 11.EC2>Auto Scaling groups>test-asg>Instance management**
- See lifecycle shows Pending>Pending:Wait>Pending:Proceed>InService

**Step 12.Goto Ec2Dashboard and copy Ip address of each Instance and check that they are running in browser**

### End of Lab


# Deploy Sample App on EC2 Fleet with ALB : Part 1 - Lab

**Step 1.Ec2>Auto Scaling groups>Load Balancing>Target Groups**

**Step 2.Click on Create target group**
- target type - Instances
- Target group name - test
- Protocol HTTP & port 80

Click on Next

**Step 3. In Register targets**
- Select from the available instances
- Click Include as pending below
- Click on Create Target group

**Step 4.Click on Target group test>Group details>edit Attributes**
- change Deregistration delay - 120 seconds

**Step 5.Ec2>Auto Scaling groups>Load Balancing>Load Balancers>Create Load Balancer>Application Load Balancer**
- Name - test
- Scheme - internet-facing
- IP address type - ipv4
- Listeners - HTTP,Port 80
- Availablity zones - select all

Click Next:Configure Security Groups

**Step 5.Select security group in Configure Security Groups**
Click on Next:Configure Routing

**Step 6.In Configure Routing>Target group**
- Target group - Existing target group
-  Name - test
- Protocol - HTTP
- Port - 80

Click on Next:Register Targets

**Step 7.Click on Next:Review**

**Step 8.Click on Create**

**Step 9.Click on Close**

**Step 7.Ec2>Auto Scaling groups>Load Balancing>Target Groups>test**
- Click to open targets
- wait for load balancer to become active
- showing unhealthy instances due to Port 80 
- we need to Change port 80 to 3000

**Step 8.Delete the Listeners in ALB**

**Step 9. Delete the target group**

**Step 10. Ec2>Auto Scaling groups>Load Balancing>Load Balancers>Create Load Balancer>Application Load Balancer**
- Name - test
- Scheme - internet-facing
- IP address type - ipv4
- Listeners - HTTP,Port 3000
- Availablity zones - select all

**Step 11. Ec2>Auto Scaling groups>Load Balancing>Load Balancers>testlb** 
- Click on Add listener
  - Protocol:Port - HTTP:80
     - Default action - Forward to test Target Group

- Save this

**Step 12.See the targets now**
- All are showing Healthy

**Step 13.Copy the Load Balancer of DNS and paste it in browser**
- Refresh it 3 times to see serving the request on all 3 instances

### End of lab

# Deploy Sample App on EC2 Fleet with ALB : Part 2 - Lab

**Step 1. Goto AWS Management Console>Services>Ec2>Ec2Dashboard>Auto Scaling>Auto Scaling groups>test-asg>edit**
- - Scroll down to Load balancing step
    - select Application or Network load balancer target groups
      -Select test|HTTP
	  
Click on Update

**Step 2.Click on Target group test>Group details>edit Attributes**
- change Deregistration delay - 120 seconds

**Step 3.In Targets select 1 instance and change security group with no Port open**
- Instance-1>Action>Security>change security group
- SG has no port open for incoming

**Step 4. See in Auto Scaling groups>test-asg**
- Instance-1 is unhealthy now
- Lifecycle is showing "Terminating"
- New instance is in Pending:Wait>Pending:Proceed>Inservice


### End of Lab



# Deploy Sample App on Elastic Beanstalk - Lab

**Step 1.AWS Management Console>Services>Elastic Beanstalk>Create Application**

**Step 2.In Create a web app give details as following**

Application name - node-app
Platform - Node.js
Application>Upload your code>choose file
Click on Create application

**Step 3.Elastic Beanstalk>Environments>NodeApp-env**

Click on URL and see it is running
**Step 4.Elastic Beanstalk>Applications>node-app>Application version**

**Step 5.Check for the resources created**
- Ec2>Launch Configuration>awseb-xxxx created

- Ec2>Auto Scaling groups>select auto scaling group>edit
  - See Load balancer is created 
  
- Ec2>Target groups>awseb-xxx>Targets
  - Port is showing 80 in Registered targets

**Step 5.Copy this Instance public ip and paste it in browser**
- Not coming up

**Step 7.Ec2>Security Groups>select your SG>Edit inbound rules**
- See it is only accepting traffic on port 80 from only Load balancer
- Add rule 
  - Type - HTTP
  - Port - 80 
  - Source - 0.0.0.0/0
  
click on save rules

**Step 8.Copy this Instance public ip and paste it in browser**
- Now it is Up and running

### End of lab

# Section Clean-up - Lab

**Step 1. AWS Management Console>Services>ElasticBeanStalk>Applications>node-app>Actions>Delete Application**
- Enter the name in Confirm Application Deletion
- Click on Delete

**Step 2. Goto AWS Management Console>EC2 Dashboard>Auto Scaling Groups>test-asg**
- Ensure that all the ASG group have following settings by Editing-
  - Desired - 0
  - Minimum - 0
  - Maximum - 0
  
**Step 3. AWS Management Console>EC2 Dashboard>Load Balancing>load Balancers>testlb>Actions>Delete**
- Click on Yes,Delete

**Step 4. AWS Management Console>EC2 Dashboard>Load Balancing>Target groups>test>Actions>Delete**
- Click on Yes,Delete  

**Step 5. Goto AWS Management Console>EC2 Dashboard>Auto Scaling Groups>test-asg>delete>confirm delete**

**Step 6. Goto AWS Management Console>EC2 Dashboard>AUTO SCALING>Launch configurations>test-lc>Actions>Delete>Confirm delete**

### End of lab








