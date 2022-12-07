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

Step 1: Download the SampleWebApp.war 

Step 2: In the Jboss server UI, click on Deployments at the top navigation. Drag and drop the war file in the left panel

<img width="820" alt="image" src="https://user-images.githubusercontent.com/58005106/205953492-7c831095-3b7f-462d-84bc-9327e43f08d2.png">

Step 3: Access to http://<ip-address>:8080/SampleWebApp. You should see the following:

<img width="686" alt="image" src="https://user-images.githubusercontent.com/58005106/205953919-1cee8669-c6af-4199-b014-950ad15f75b4.png">

<img width="1299" alt="image" src="https://user-images.githubusercontent.com/58005106/205954011-54cc668a-5c84-4a81-bbb3-8f4adc029bf4.png">
  
**Part 3 - Install Splunk Otel Collector**

Step 1: Navigate to Splunk Observability portal. On the left navigation, click on data management.

![image](https://user-images.githubusercontent.com/58005106/205959448-b0cff3c4-de63-4e13-a898-d767d7354ffa.png)

Step 2: Click on +Add Integration. Search for Linux

