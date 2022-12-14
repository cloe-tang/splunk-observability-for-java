# splunk-observability-for-java-jboss
This guide is to walk you through the step by step setup for Splunk Observability on Java Application (running on jboss)

The following will be covered:
1. Setup a jboss server in AWS Environment
2. Deploy a Sample Java Application
3. Install Splunk OTEL collector
4. Instrument Sample Java Application to send metrics and traces
5. Enable Always On Profile
6. Send Always-On profiling logs to Splunk Observability and app logs to Splunk Cloud

## Pre-requisite
1. An AWS account
2. Sign up for Splunk Cloud Observability trial account

## Part 1 - Setup Jboss Server in AWS

Step 1: Launch an instance using the "Red Hat JBoss Enterprise Application Platform v7.4 on Linux" image

<img width="1226" alt="image" src="https://user-images.githubusercontent.com/58005106/205944309-28f9bec7-5442-4a08-86e8-7f54a133ca2e.png">

Configure the instance accordingly: (Instance Type: M5 Large; Root Volume: 50) 

Step 2: execute the following script to change password

```
sudo /opt/jboss-eap-7.4/bin/add-user.sh
```

<img width="816" alt="image" src="https://user-images.githubusercontent.com/58005106/205948506-00f6434a-b8d1-46f5-b924-25786fa7ee28.png">

Step 3: Access Jboss server at http://ip-address:9990
  
<img width="932" alt="image" src="https://user-images.githubusercontent.com/58005106/205949002-f4de317c-fe0f-42df-8a9e-1d7c2b6e8b31.png">

## Part 2 - Deploy Sample Java Application 

Step 1: Download the SampleWebApp.war 

Step 2: In the Jboss server UI, click on Deployments at the top navigation. Drag and drop the war file in the left panel

<img width="820" alt="image" src="https://user-images.githubusercontent.com/58005106/205953492-7c831095-3b7f-462d-84bc-9327e43f08d2.png">

Step 3: Access to http://ip-address:8080/SampleWebApp. You should see the following:

<img width="686" alt="image" src="https://user-images.githubusercontent.com/58005106/205953919-1cee8669-c6af-4199-b014-950ad15f75b4.png">

<img width="1299" alt="image" src="https://user-images.githubusercontent.com/58005106/205954011-54cc668a-5c84-4a81-bbb3-8f4adc029bf4.png">
  
## Part 3 - Install Splunk Otel Collector

Step 1: Navigate to Splunk Observability portal. On the left navigation, click on data management.

![image](https://user-images.githubusercontent.com/58005106/205959448-b0cff3c4-de63-4e13-a898-d767d7354ffa.png)

Step 2: Click on +Add Integration. Search for Linux. Follow through the wizard. It will provide the instruction to install the OTEL collector
  
![Screenshot 2022-12-07 at 1 48 55 PM](https://user-images.githubusercontent.com/58005106/206099076-9b222f6f-6393-4d36-9f61-3adcb664d207.png)

Step 3: Go to your JBoss server. Execute the step by step instruction in #Step 2. If successfully install, you should see that splunk-otel-collector service is running
  
<img width="766" alt="image" src="https://user-images.githubusercontent.com/58005106/206099638-895ca08a-b13f-4d19-a512-d8cac36b9238.png">

Step 4: Navigate to Splunk Observability Portal. Under Infrastructure Monitoring, you should see the instance being monitored
  
![image](https://user-images.githubusercontent.com/58005106/206099906-e3bcf4b1-9f98-4601-b710-bb74b463b560.png)

Step 5: Click on the instance and you will be able to see the resources being monitored
  
![image](https://user-images.githubusercontent.com/58005106/206100180-045578f9-6725-400c-91c0-1a3fbd6f294c.png)

## Part 4 - Instrument Sample Java Application to send traces and metrics

Step 1: Go to the JBoss server. Download the java otel agent jar file by executing the following
  
```
curl -L https://github.com/signalfx/splunk-otel-java/releases/latest/download/splunk-otel-javaagent.jar \
-o splunk-otel-javaagent.jar
```
  
Step 2: Navigate to /opt/jboss-eap-7.4/bin. Edit the standalone.conf file. Add the following configuration at the end of the file and restart jboss

```
JAVA_OPTS="$JAVA_OPTS -javaagent:/home/ec2-user/splunk-otel-javaagent.jar -Dotel.resource.attributes=deployment.environment=lab,service.name=sampleApp -Dsplunk.profiler.enabled=true -XX:StartFlightRecording"
```
- javaagent:/home/ec2-user/splunk-otel-javaagent.jar -> Load the Java Otel Agent
- deployment.environment=lab -> Define environment
- service.name=sampleApp -> Define service name
- Dsplunk.profiler.enabled=true -> Enable AlwaysOn Profiling
- -XX:StartFlightRecording -> Start JFR record. (If this is not set, no data will be sent to AlwaysOn Profiling)

```  
sudo systemctl restart jboss-eap` -> restart jboss
```

Step 3: Go to Splunk Observability Cloud. Click on APM at the left navigation bar. You should see your environment created and the services monitored

![image](https://user-images.githubusercontent.com/58005106/206112539-d9aa6412-3e44-4aee-a0ad-9dc77759dacb.png)

Step 4: Click on Explore at the right panel. You should see you Service flow. Currently there is only one service. 

![image](https://user-images.githubusercontent.com/58005106/206112819-064ed999-233d-466b-9615-27fb0ccbb655.png)

Step 5: Go back to APM page. Click on AlwaysOn Profiling at the bottom right panel. Select the sampleApp service. You should see the following
  
![image](https://user-images.githubusercontent.com/58005106/206113262-0aab1d92-18a5-4fae-8a75-06757c54d888.png)

## Part 5 - Send Always-On profiling logs to Splunk Observability and Application Logs to Splunk Cloud (Optional)

Step 1: Navigate to /etc/otel/collector. Edit agent_config.yaml.

Under receivers, add in the following configuration. The profiling name can be change to any other names.
```
  otlp/profiling:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4319
```
After adding, the configuration should look something like the following
```
receivers:
  fluentforward:
    endpoint: 127.0.0.1:8006
  hostmetrics:
    collection_interval: 10s
    scrapers:
      cpu:
      disk:
      filesystem:
      memory:
      network:
      # System load average metrics https://en.wikipedia.org/wiki/Load_(computing)
      load:
      # Paging/Swap space utilization and I/O metrics
      paging:
      # Aggregated system process count metrics
      processes:
      # System processes metrics, disabled by default
      # process:
  jaeger:
    protocols:
      grpc:
        endpoint: 0.0.0.0:14250
      thrift_binary:
        endpoint: 0.0.0.0:6832
      thrift_compact:
        endpoint: 0.0.0.0:6831
      thrift_http:
        endpoint: 0.0.0.0:14268
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
      http:
        endpoint: 0.0.0.0:4318
  otlp/profiling:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4319
```

Step 2: In the same file, add in the exporters configuration

Look for exporters section and add in the following configuration. The "olly" exporter name can be any other names.
```
  splunk_hec/olly:
    token: "${SPLUNK_ACCESS_TOKEN}"
    endpoint: "https://ingest.${SPLUNK_REALM}.signalfx.com/v1/log"
```
After adding, the configuration should look like the following:
```
exporters:
  # Traces
  sapm:
    access_token: "${SPLUNK_ACCESS_TOKEN}"
    endpoint: "${SPLUNK_TRACE_URL}"
  # Metrics + Events
  signalfx:
    access_token: "${SPLUNK_ACCESS_TOKEN}"
    api_url: "${SPLUNK_API_URL}"
    ingest_url: "${SPLUNK_INGEST_URL}"
    # Use instead when sending to gateway
    #api_url: http://${SPLUNK_GATEWAY_URL}:6060
    #ingest_url: http://${SPLUNK_GATEWAY_URL}:9943
    sync_host_metadata: true
    correlation:
  # Logs
  splunk_hec:
    token: "b1f58197-8ce3-43df-8e98-879b124cd1a4"
  #  endpoint: "${SPLUNK_HEC_URL}"
    endpoint: "https://prd-p-t9icu.splunkcloud.com:8088/services/collector/event"
    source: "otel"
    sourcetype: "otel"
  # Send to gateway
  splunk_hec/olly:
    token: "${SPLUNK_ACCESS_TOKEN}"
    endpoint: "https://ingest.${SPLUNK_REALM}.signalfx.com/v1/log"
  otlp:
    endpoint: "${SPLUNK_GATEWAY_URL}:4317"
    tls:
      insecure: true
  # Debug
  logging:
    loglevel: debug
```

Step 3: In the same file, add in the service configuration

Look for service section and add in the following configuration. The "olly" service name can be any other names.
```
logs/olly:
      receivers: [otlp/profiling]
      processors:
      - memory_limiter
      - batch
      - resourcedetection
      #- resource/add_environment
      exporters: [splunk_hec/olly]
 ```
After adding, the configuration should look something like the following:
```
service:
  extensions: [health_check, http_forwarder, zpages, memory_ballast]
  pipelines:
    traces:
      receivers: [jaeger, otlp, smartagent/signalfx-forwarder, zipkin]
      processors:
      - memory_limiter
      - batch
      - resourcedetection
      #- resource/add_environment
      exporters: [sapm, signalfx]
      # Use instead when sending to gateway
      #exporters: [otlp, signalfx]
    metrics:
      receivers: [hostmetrics, otlp, signalfx, smartagent/signalfx-forwarder]
      processors: [memory_limiter, batch, resourcedetection]
      exporters: [signalfx]
      # Use instead when sending to gateway
      #exporters: [otlp]
    metrics/internal:
      receivers: [prometheus/internal]
      processors: [memory_limiter, batch, resourcedetection]
      # When sending to gateway, at least one metrics pipeline needs
      # to use signalfx exporter so host metadata gets emitted
      exporters: [signalfx]
    logs/signalfx:
      receivers: [signalfx, smartagent/processlist]
      processors: [memory_limiter, batch, resourcedetection]
      exporters: [signalfx]
      # Use instead when sending to gateway
      #exporters: [otlp]
    logs/olly:
      receivers: [otlp/profiling]
      processors:
      - memory_limiter
      - batch
      - resourcedetection
      #- resource/add_environment
      exporters: [splunk_hec/olly]
    logs:
      receivers: [fluentforward, otlp]
      processors:
      - memory_limiter
      - batch
      - resourcedetection
      #- resource/add_environment
      exporters: [splunk_hec]
      # Use instead when sending to gateway
      #exporters: [otlp]
```
** Important: If service configuration is not added, the newly created port will not be listening.

Step 4: Restart the splunk-otel-collector service

```
sudo systemctl restart splunk-otel-collector
```

Step 5: Navigate to /opt/jboss-eap-7.4/bin. Edit standalone.conf

Add in the following configuration
```
-Dsplunk.profiler.logs-endpoint=http://localhost:4319
```
After adding, the configuration should look something like that following:
```
JAVA_OPTS="$JAVA_OPTS -javaagent:/home/ec2-user/splunk-otel-javaagent.jar -Dotel.resource.attributes=deployment.environment=cloe-demo-env,service.name=cloe-sampleJavaApp -Dsplunk.profiler.enabled=true -Dsplunk.profiler.logs-endpoint=http://localhost:4319 -XX:StartFlightRecording"
```

Step 5: Restart JBOSS service

```
sudo systemctl restart jboss-eap
```

Step 6: Send application logs to Splunk Enterprise via Fluentd

Navigate to /etc/otel/collector/fluentd/conf.d and create a file. In my case, my filename is jboss.conf as I am sending jboss logs (server and audit). Do the same if you are sending other application logs.

```
<source>
  @type tail
  @label @SPLUNK
  <parse>
    @type none
  </parse>
  path /opt/jboss-eap-7.4/standalone/log/server.log
  pos_file /var/log/td-agent/jboss-server.pos
  tag jboss-server
</source>

<source>
  @type tail
  @label @SPLUNK
  <parse>
    @type none
  </parse>
  path /opt/jboss-eap-7.4/standalone/log/audit.log
  pos_file /var/log/td-agent/jboss-audit.pos
  tag jboss-audit
</source>
```
Step 7: Restart the td-agent

```
sudo systemctl restart td-agent
```

Step 8: Navigate to the Splunk Enterprise. You should see the logs streaming in with log source "otel".

![image](https://user-images.githubusercontent.com/58005106/208832537-10d6bb80-922a-404e-b55e-f1c5ea820f7f.png)

## Troubleshooting
  
Following are some logs that you can lookout for when troubleshooting
1. /var/log/messages -> Check if the java agent is running
2. /opt/jboss-eap-7.4/standalone/log/server.log -> Check if there is any errors 
