# IM and APM Instrumentation Demo

### Abstract

### Requirements
1. AWS Account
2. Access to the playground environment in us0
3. Access to a set up sfdemo backend demo

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
3. Launch the instance, and once it is launched, connect to it via ssh. Note the instance ID and the public IP, as we will need it later

Install and set up the application we will instrument
1. Update and upgrade yum
```
sudo yum update -y && sudo yum upgrade -y
```
2. Install java and git
```
sudo yum install java -y && sudo yum install git -y
```
3. Download the java app from this repository, and navigate into the directory 
```
git clone https://github.com/ylowe-splunk/InstrumentationDemo.git
```
```
cd InstrumentationDemo
```
4. Set execution permissions for the gradlew file
```
chmod +x gradlew
```
5. Build the application
```
./gradlew shadowJar
```

Deploy the backend scripts for sfdemo

### Talk Track
I understand that having a short implementation time, out of the box metrics, and being able to do root cause analysis are all important technical qualifiers for Splunk. My objective today is to show that, via instrumenting a blank EC2 instance, which I have installed a basic java app on. 

Our goal first is to integrate with metrics. To do that we just go to our integration page, add integration, click Linux, and follow the steps to download the collector. We will specify our access token (Default_2022), keep it in agent mode, and keep log collection. Now, we just run

```
curl -sSL https://dl.signalfx.com/splunk-otel-collector.sh > /tmp/splunk-otel-collector.sh && \
sudo sh /tmp/splunk-otel-collector.sh --realm us0 -- kzD15S9Cr9aTHhWgMQEeeA --mode agent
```

And our collector shoud now be installing. This process can be very easily automated either via custom scripts or terraform. As I mentioned before, Splunk utilizes our own fork of the otel collector, which has some defaults built into the configuration file that makes integrating with our cloud platform easier. That said, most of the automatic and manual integration steps listed on the opentelemetry website should still work with our collector. Now as the collector installs, lets open the floor to any questions we might have up until now.  

...

Now that it is installed, we can grab our AWS Unique ID and navigate to our platform. 

- Infrastructure Monitoring --> open two tabs of EC2, on one of them, filter for AWSUniqueId = Instance ID

Out of the box, we have all of these dashboards built out by default. These dashboards are malleable, and you can also easily edit the charts that make up the dashboards. Now based on some of the charts, we can tell this is an aggregate dashboard, which we can see to great effect once we loop in other instances.

- Switch to the other EC2 tab to show the aggregate instances and every chart filled out. Then, switch back to the tab with the filter

We can click into our instance name also, to see our metadata and to see more specific charts that pertain directly to our instance. 

This instrumentation works for applications too - we are able to do everything form grabbing application metrics, coorelating it with the host, to collecting stacktrace data and application memory usage over time. But before I do that, are there any questions about infrastructure monitoirng?

...

Okay, so lets say I want to instrument a simple standalone java application. All we need to do is curl the java agent, and run the application with the flags specified. Now, the app is written to run a monty hall game on port 9090, so lets go to that. I'll post the link in the chat, and please do go on and play like five to ten rounds - it'll help me generate data a little bit faster. But as we are doing that, lets open the floor up to any questions we may have so far




