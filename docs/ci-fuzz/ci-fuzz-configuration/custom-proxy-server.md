---
layout: default
#title: Custom Proxy Server
parent: Advanced Configuration
grand_parent: CI Fuzz
nav_order: 2
---
# **Custom Proxy Server**
{: .no_toc }


## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

## Using your own reverse proxy/load balancer

If you want to terminate TLS yourself, comment out CIFUZZ_CERT_FILE and CIFUZZ_CERT_KEY. Change CIFUZZ_SERVER_PORT to some other port, but keep port 443 (or other port where users will be accessing CI Fuzz) in CIFUZZ_SERVER_ORIGIN.

Set your reverse proxy or load balancer to direct HTTP and HTTP2/GRPC traffic to CIFUZZ_SERVER_PORT. Nginx example:  

```  
server {  
 listen 443 ssl;  
  ssl_certificate      /home/azureuser/tls/cifuzz.onprem.lab_cert.pem;  
  ssl_certificate_key  /home/azureuser/tls/cifuzz.onprem.lab_key.pem;  
 location / {  
  proxy_pass http://localhost:80/;  
 }  
}  
server {  
 listen 6443 http2 ssl;  
  ssl_certificate      /home/azureuser/tls/cifuzz.onprem.lab_cert.pem;  
  ssl_certificate_key  /home/azureuser/tls/cifuzz.onprem.lab_key.pem;  
 client_max_body_size 500M;  
 location / {  
   grpc_pass grpc://localhost:80;  
 }  
}
```

Attention! This will work well for Code Intelligence App and CI Fuzz CLI (HTTP 1 traffic) and cictl (GRPC traffic), but Java Agents and possibly native fuzzing agents will have a problem connecting to fuzzers.

For Web API fuzzing, please disable TLS and use CI Server's gateway port (80 by default) in the agent options. Java agent example:
```
java -javaagent:fuzzing_agent.jar=fuzzing_server_host=$FUZZING_SERVER_HOST,fuzzing_server_port=80,tls=false,service_name=$PROJECT/web_services/mywebservice,api_token=$CI_FUZZ_API_TOKEN,instrumentation_includes=\"com.myexample.restapi.**\"
```
