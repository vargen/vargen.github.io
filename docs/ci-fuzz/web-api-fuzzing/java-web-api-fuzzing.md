---
layout: default
title: Java Web API Fuzzing
parent: Web API Fuzzing
grand_parent: CI Fuzz
nav_order: 0
permalink: java-web-api-fuzzing
---
# **Java Web API Fuzzing**
{: .no_toc }

CI Fuzz is capable of coverage-guided fuzzing for Java-based REST and GRPC APIs. This section will guide you through the initial setup of a Java Web API fuzzing project. 

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

## Prerequisites

Before you proceed with this guide, you should have already:

* [Installed CI Fuzz]({{ site.baseurl }}{% link docs/ci-fuzz/installation.md %}) and the build tools.
* [Configured authentication]({{ site.baseurl }}{% link docs/ci-fuzz/authentication.md %}).
* [Created a project]({{ site.baseurl }}{% link docs/ci-fuzz/project-creation.md %}).
* [Configured your Web API using the template]({{ site.baseurl }}{% link docs/ci-fuzz/web-api-fuzzing/index.md %}).

## Web API Server

Your Web API can run anywhere as long as it is reachable from the server hosting CI Fuzz. Port details are provided below.

Typically, this would be one of the following:

* On the CI/CD server (gitlab/github/jenkins server)
* On the CI Fuzz server
* A separate server (e.g. your integration testing cluster)

If you choose to run your Web API on a separate server from the one hosting CI Fuzz, you need to make sure that:

1. The Web API is exposed on a network interface and the port on which it is running is reachable from CI Fuzz.
2. Ports 443 or 80 and 6777-7777 on the server running CI Fuzz are reachable from the server hosting your Web API.

## Configure Java Agent

The `javaagent` is responsible for instrumenting the Web API to collect coverage information. This coverage information is sent back to CI Fuzz to help it determine which code paths to follow and how to mutate the inputs to your Web API. The CI Fuzz Agent is deployed with your Web API.

CI Fuzz will provide the command line argument that is needed to deploy the agent alongside your target application:

1. Open CI Fuzz in your browser
2. From the left sidebar, select your project from the dropdown menu.
3. Click **Project Settings**
4. Click **WEB SEVICES**
5. Click the **ADD WEB SERVICE** button.
6. Add a name in the **Web Service Identifier** textbox. The name/identifier can be whatever you like, but it must match the name in the `.code-intelligence/web_services.yaml` file. If you used the template project, the name is `mywebservice` by default.
7. If the target application is not running on the same server as CI Fuzz, click **Use Custom Fuzzing Server Settings** to specify the host and port where the CI Fuzz Agent can reach CI Fuzz. The host
8. Click **NEXT**.
9. Select **JAVA**
10. Copy the appropriate command line argument (depending on if your application uses JAVA_OPTS or not).

Here is an example `javaagent` argument string:

```
-javaagent:path/to/downloaded/fuzzing_agent.jar=service_name=projects/webgoat-27096bd3/web_services/mywebservice,fuzzing_server_host=grpc.code-intelligence.com,fuzzing_server_port=443,api_token=$CI_FUZZ_API_TOKEN,cert_file=$CERT_FILE,tls=true
```

### Java Agent Parameters

The `javaagent` supports the following parameters:

* **path/to/downloaded/fuzzing_agent.jar** - set the path of the `javaagent`. The `javaagent` must be available to the Web API The recommended way to acquire the `javaagent` (fuzzing_agent.jar) is to use the command `cictl get javaagent` to download it. There is an example of this below.

* **service_name** - a name which should be unique to be distinguishable from other services e.g. `service_name=projects/example-9070daab/web_services/example`. This is typically generated for you by CI Fuzz during web service creation.

* **api_token** - the `javaagent` needs the CI Fuzz API token to authenticate to CI Fuzz. 

* **instrumentation_includes** - a `:` separated list of package glob patterns to include in the instrumentation e.g. `instrumentation_includes=com.google.**:com.example.**`. Only the feedback that is received for the specified package patterns will affect the fuzzing input generation and path discovery of the code. Though not technically required, failure to specify will result in the `javaagent` instrumenting all packages (including native Java packages). This typically results in the JVM running out of memory as well as the fuzzer spending time exploring packages that you may not be interested in.

* **instrumentation_excludes** - a `:` separated list of package glob patterns to exclude from the instrumentation. 

* **tls** - set to `true` if CI Fuzz has TLS enabled, otherwise set to `false`. When `true`, this will encrypt the communication between CI Fuzz and the `javaagent`. The gRPC communication between the fuzzer and `javaagent` is still unencrypted.

* **cert_file** - The location of the certificate of the CA that signed the TLS certificate of the CI Fuzz server. Only required if the cert is not signed by a public CA.

* **fuzzing_server_host** - use a host name that is listed in the server's TLS certificate, otherwise the TLS connection will fail. 

* **fuzzing_server_port** - port where CI Fuzz can be reached.

## Example CI/CD Script

This is an example Gitlab CI job that will start a Java Web API with `javaagent` on another server. In this example, the deployment server already has the cictl tool and uses it to download the `javaagent`. You can also download the java agent in the CI/CD pipeline and then copy it to the deployment server.

```  
deploy:  
  stage: deploy  
  image: cifuzz/ubuntu-ssh  
  before_script:  
    - eval $(ssh-agent -s)  
    - echo ""$SSH_PRIVATE_KEY"" | tr -d '\\r' | ssh-add -  
    - mkdir -p ~/.ssh  
    - chmod 700 ~/.ssh  
    - echo ""$SSH_KNOWN_HOSTS"" >> ~/.ssh/known_hosts  
    - chmod 644 ~/.ssh/known_hosts  
  variables:  
    DEPLOY_HOST: azureuser@on-prem-l000003  
  script:  
    - echo ""Kill currently running app...""  
    - ssh $DEPLOY_HOST pkill -f rest-service || true  
    - echo ""Copy app binary...""  
    - ls build/libs  
    - scp build/libs/rest-service-?.?.?.jar $DEPLOY_HOST:~  
    - echo ""Prepare CA certificate for connecting to CI Fuzz...""  
    - cat $CA_CERT  
    - scp $CA_CERT $DEPLOY_HOST:~/CA.pem  
    - echo ""download correct version of java agent from CI Fuzz server""  
    - ssh $DEPLOY_HOST ""echo $CI_FUZZ_API_TOKEN | ./cictl login -s $FUZZING_SERVER""  
    - ssh $DEPLOY_HOST ""./cictl get javaagent -s $FUZZING_SERVER""  
    - echo ""Run app...""  
    - FUZZING_SERVER_HOST=$(echo $FUZZING_SERVER|cut -d ':' -f 1)  
    - FUZZING_SERVER_PORT=$(echo $FUZZING_SERVER|cut -d ':' -f 2)  
    - echo "java -javaagent:fuzzing_agent.jar=fuzzing_server_host=$FUZZING_SERVER_HOST,fuzzing_server_port=$FUZZING_SERVER_PORT,service_name=$PROJECT/web_services/mywebservice,api_token=$CI_FUZZ_API_TOKEN,cert_file=CA.pem,instrumentation_includes=\\""com.cifuzzexample.\*\*\\""  -Dserver.port=$SUT_PORT -jar rest-service-\*.jar > output.log 2>&1 &" > javacommand  
    - scp javacommand $DEPLOY_HOST:~  
    - ssh $DEPLOY_HOST ""chmod u+x javacommand && ./javacommand""
```





