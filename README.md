# IM and APM Instrumentation Demo

### Abstract

### Requirements
1. AWS Account
2. Access to the playground environment in us1

### Setup
Create an EC2 Account which is accessable via the internet
1. Create a security group which allows inbound TCP traffic on ports 80, 22, and 9090 from anywhere (0.0.0.0/0)
2. Launch the wizard for creating an EC2 instance, and configure the following
- Operating System: Amazon Linux 64-bit (x86)
- Instance Type: t2.small (or larger instances, but I have found that t2.small works fine for me)
- Key Pair: Select your key pair, or create one if it does not exist. 
- VPC: Default VPC, should be called aws-controltower-VPC
- Subnet: Any default subnet of the VPC
- Auto-assign public IP: Yes
- Security Group: Select your existing security group we created 
- Storage: Keep as default
3. Launch the instance, and once it is launched, connect to it via ssh

Install and set up the application we will instrument
1. Update and upgrade yum
```
sudo yum update && sudo yum upgrade -y
```
2. Install java and git
```
sudo yum install java -y && sudo yum install git -y
```
3. Download the java app

# Run

```
java -jar build/libs/profiling-workshop-all.jar
```

The server listens on port 9090.

<<<INSERT SCRIPT AND SETUP INFO HERE>>>
