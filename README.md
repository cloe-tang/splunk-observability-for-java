# splunk-observability-for-java-jboss
This guide is to walk you through the step by step setup for Splunk Observability on Java Application (running on jboss)

The following will be covered:
1. Setup a jboss server in AWS Environment
2. Deploy a Sample Java Application
3. Install Splunk OTEL collector
4. Instrument Sample Java Application to send metrics and traces
5. Enable Always On Profile

**Pre-requisite**
1. An AWS account
2. Sign up for Splunk Cloud Observability trial account

**Part 1 - Setup Jboss Server in AWS**

Step 1: Launch an instance using the "Red Hat JBoss Enterprise Application Platform v7.4 on Linux" image

<img width="1226" alt="image" src="https://user-images.githubusercontent.com/58005106/205944309-28f9bec7-5442-4a08-86e8-7f54a133ca2e.png">

Configure the instance accordingly: (Instance Type: M5 Large; Root Volume: 50) 

Step 2: execute the following script to change password

`sudo /opt/jboss-eap-7.4/bin/add-user.sh`

<img width="816" alt="image" src="https://user-images.githubusercontent.com/58005106/205948506-00f6434a-b8d1-46f5-b924-25786fa7ee28.png">

Step 3: Access Jboss server at http://<ip-address>:9990
  
<img width="932" alt="image" src="https://user-images.githubusercontent.com/58005106/205949002-f4de317c-fe0f-42df-8a9e-1d7c2b6e8b31.png">

**Part 2 - Deploy Sample Java Application**
