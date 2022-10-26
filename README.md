# IM and APM Instrumentation Demo

## Requirements
1. AWS Account
2. Access to the [playground environment in us0](https://app.signalfx.com/#/home)
3. Access to a set up [sfdemo backend demo](https://docs.google.com/document/d/1ldNG7qC5xUBNXA6NP9S5ueTihYBlBE57ZsEReZD80dQ/edit)

## Setup
#### 1. Create an EC2 Account which is accessable via the internet

- [Create a security group](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_SecurityGroups.html#working-with-security-groups) which allows inbound TCP traffic on ports 80, 4317, 22, and 9090 from anywhere (0.0.0.0/0) 
- Launch the wizard for creating an EC2 instance, and configure the following <br><br>

<div align="center">
            
| Config                | Value                                                    |
|-----------------------|----------------------------------------------------------|
| Operating System      | Amazon Linux 64-bit (x86)                                |
| Instance Type         | t2.small (or larger)                                     |
| Key Pair              | Select your key pair, or create one if it does not exist |
| VPC                   | Default VPC, should be called aws-controltower-VPC       |
| Subnet                | Any default subnet of the VPC                            |
| Auto-assign public IP | Yes                                                      |
| Security Group        | Select the security group you just created               |
| Storage               | Keep as default                                          |

</div>

- Launch the instance, and once it is launched, connect to it via ssh. Note the instance ID and the public IP, as we will need it later

<br><br>
#### 2. Install and set up the application we will instrument
- Update and upgrade yum
```
sudo yum update -y && sudo yum upgrade -y
```
- Install java and git
```
sudo yum install java -y && sudo yum install git -y
```
- Download the java app from this repository, and navigate into the directory 
```
git clone https://github.com/ylowe-splunk/InstrumentationDemo.git
```
```
cd InstrumentationDemo
```
- Set execution permissions for the gradlew file
```
chmod +x gradlew
```
- Build the application
```
./gradlew shadowJar
```
<br><br>
#### 3. [OPTIONAL] Prepare transition 
- If you plan on transitioning to the backend demo, make sure you have [deployed the backend scripts for sfdemo](https://docs.google.com/document/d/1ldNG7qC5xUBNXA6NP9S5ueTihYBlBE57ZsEReZD80dQ/edit)
<br><br><br>

## Talk Track
I understand that having a short implementation time, out of the box metrics, and being able to do root cause analysis are all important technical qualifiers for Splunk. My objective today is to show this by instrumenting a blank EC2 instance, which I have installed a basic java app on. 

Our goal first is to integrate with metrics. To do that we just go to our integration page, add integration, click Linux, and follow the steps to download the collector. We will specify our access token (Default_2022), keep it in agent mode, and keep log collection. Now, we can switch to our machine and run the script to curl and install the collector

```
curl -sSL https://dl.signalfx.com/splunk-otel-collector.sh > /tmp/splunk-otel-collector.sh && \
sudo sh /tmp/splunk-otel-collector.sh --realm us0 -- kzD15S9Cr9aTHhWgMQEeeA --mode agent
```

And our collector shoud now be installing. This process can be very easily automated either via custom scripts or terraform. As I mentioned before, Splunk utilizes our own fork of the otel collector, which has some defaults built into the configuration file that makes integrating with our cloud platform easier. That said, most of the automatic and manual integration steps listed on the opentelemetry website should still work with our collector. Now as the collector installs, lets open the floor to any questions we might have up until now.  

<br>

> Allocate at most five minutes for questions - the collector usually does not take longer than that
> 
<br>

Now that it is installed, we can grab our AWS Unique ID and navigate to our platform. 

<br>

> Go to [Infrastructure Monitoring](https://app.signalfx.com/#/infra?endTime=now&startTime=-3h), and open two tabs of [EC2](https://app.signalfx.com/#/infra/entity/AWS%20instances?groupBy=aws_region&endTime=now&startTime=-3h). On one of them, add a filter for `AWSUniqueId = Instance ID`
> 
<br>

Out of the box, we have all of these dashboards built out by default. These dashboards are malleable, and you can also easily edit the charts that make up the dashboards. Now based on some of the charts, we can tell this is an aggregate dashboard, which we can see to great effect once we loop in other instances.

- Switch to the other EC2 tab to show the aggregate instances and every chart filled out. Then, switch back to the tab with the filter

We can click into our instance name also, to see our metadata and to see more specific charts that pertain directly to our instance. 

This instrumentation works for applications too - we are able to do everything form grabbing application metrics, coorelating it with the host, to collecting stacktrace data and application memory usage over time. But before I do that, are there any questions about infrastructure monitoirng?

...

- Switch back to the terminal

Okay, so lets say I want to instrument a simple standalone Java application. All we need to do is go to data manager and add a Java application monitor

- Data management --> Add integration --> Sort by use case --> Monitor applications --> Java (traces)

Lets set our service name as Integration Demo, our environment as workshop, and we will enable code profiling and application metrics

- Service Name: IntegrationDemo-\[INITIALS\]
- Collector Endpoint: http://localhost:4317
- Environment: Workshop-\[INITIALS\]
- AlwaysOn Profiling: CPU and memory profiling
- Application Metrics: Yes
- Kubernetes: No
- Legacy Agent: No

Now we just curl the Java agent

```
curl -L https://github.com/signalfx/splunk-otel-java/releases/latest/download/splunk-otel-javaagent.jar -o splunk-otel-javaagent.jar
```

And run the application with the environment variables and the flags specified. I find it personally easier to define it all with flags, so I will use that command

```
java -javaagent:"splunk-otel-javaagent.jar" \
    -Dsplunk.profiler.enabled=true \
    -Dsplunk.profiler.memory.enabled=true \
    -Dsplunk.profiler.call.stack.interval=1000 \
    -Dotel.resource.attributes=deployment.environment=workshop \
    -Dotel.service.name=IntegrationDemo-YL \
    -jar build/libs/profiling-workshop-all.jar
```

Now, the app is written to run a monty hall game on port 9090, so lets go to that. 

- Public IP:9090

I'll post the link in the chat, and please do go on and play like five to ten rounds - it'll help me generate data a little bit faster. But as we are doing that, lets open the floor up to any questions we may have so far

...

Alright, so we've generated some traffic, and we also might have noticed that door 3 takes a lot more time to load than door 1 and 2. Keeping that in mind, lets look at the applications. 

- Navigate to APM

We can see our single service instrumented here, and some general metrics about latency and errors on the main page. 

- Three white dots --> View Dashboard

Digging in, we can look at the dashboard for our service, see application metrics, and then on the same page, also see the host metrics which we also just instrumented. If we want to look at traces, tags, and code profiling, we can choose to troubleshoot our time window. 

- Three dots on any of the application metrics --> Troubleshoot this time window

We can see our service here again, and then we can navigate to the right and see code profiling

- Go to code profiling

This is where we can see the callstack of what is going on in that node, as well as see application memory usage data. For Java, this manifests as heap memory usage, activity over time, and garbage collector activity. The flame graph is interactive, which lets us visually navigate the callstack in the case that we are looking for the root cause of any application errors

We can also look at the callstack in relation to individual traces

- Go back to the distributed graph, and click Traces. Find one with a duration of over 5 seconds

Here, we can see the spans of each trace, and should see that the innermost span DoorGame.getOutcome is responsible for a slowdown. We can click into this span to gain more detail, and view our sampled call stacks

In our case, we can quickly see that DoorChecker is called as we are getting the outcome of the game. but, that issues a precheck function that triggers a long series of sleep functions. We can view the other call stacks to verify that this is indeed the case for all call stacks. Once we know that every sample is held up in the same sleep call, we have a good lead for where we should start looking at our code.

```
cd ~/InstrumentationDemo/src/main/java/com/splunk/profiling/workshop
```
```
vim DoorChecker.java
```

Scrolling down to the precheck function, We can see that the wait time increases exponentially to the index of the door, which means that door 3 is going to be much slower than door one. We dont want this feature, so we can just change this to `sleep(300)`

Now, we can save and close the code (```[esc] --> :wq```), and rerun the app
```
cd ~/InstrumentationDemo
```
```
java -javaagent:"splunk-otel-javaagent.jar" \
    -Dsplunk.profiler.enabled=true \
    -Dsplunk.profiler.memory.enabled=true \
    -Dsplunk.profiler.call.stack.interval=1000 \
    -Dotel.resource.attributes=deployment.environment=workshop \
    -Dotel.service.name=IntegrationDemo-YL \
    -jar build/libs/profiling-workshop-all.jar
```
and we should see that the latency with door 3 is now gone. 

So in a nutshell, we wanted to see a tool that can easily get insight into your environment, and we saw how Splunk does that. We saw how easy it is to set up, the out of the box metrics it comes with, and how it is able to provide a clear view into an environment by coorelating both infrastructure and application data. Then we looked at how we can dig deeper into that application code, and saw how code profiling can collect stacktrace and memory data, and speed up the debugging process by coorelating issues such as errors or latency with code insights.


### Transition to Backend Demo
Now, all this is great for getting insight on a single instance, but a key differentiator for Splunk's platform is how we do root cause analysis in a distributed environment, which we will take a look at now. Before we do though, let's take a quick pause for any questions. 














