---
layout: default
#title: CI/CD Integration
parent: CI Fuzz
nav_order: 3
---
# **Integrating CI Fuzz with CICD**
{: .no_toc }

<!-- TODO: add description of page -->

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---


### Architecture of CI Fuzz Integration

When running a pipeline, your CI/CD platform will to the following:

1.  Download necessary CI Fuzz tools (or retrieve them from cache)
2.  build your software with instrumentation (Unit fuzzing) or build your software, deploy it and start it with a fuzzing agent (Web fuzzing)
3.  build fuzz tests and upload them to the CI Fuzz server
4.  trigger fuzzing on the CI Fuzz server
5.  wait for findings and pass/fail depending or fuzzing results

Fuzzing begins with a regression test by checking previously generated inputs and crashes found in previous runs. When vulnerabilities or bugs are found, the pipeline is marked as failed and the detailed bug reports can be viewed and debugged in the CI Fuzz Web Interface.  

### API Token

Your CI/CD pipeline will need an API token, in order to authenticate to the CI Fuzz server. Click on your user  name in the lower left corner (Settings section):  
 then click generate:

Token name can be anything (for example the name of your CI/CD platform). Please set the expiration date. Not setting the date may result in unexpected expiration in the future.

Click generate token. We recommend saving it in a password manager.

Next, store this token into your CI/CD platform. Example with GitHub Actions:

Example with GitLab CI/CD:

The ""Key"" field should be CI\_FUZZ\_API\_TOKEN.

### CI/CD script

To enable continuous fuzzing of your projects, use the CI/CD configuration wizard in the CI Fuzz Web Interface.

Select your CI/CD platform and click generate script.

This will generate the parts that build fuzz tests, upload them to CI Fuzz server, start fuzzing and monitor findings, and for some platforms, obtain CI Fuzz tools.

In case of Web Fuzzing, the parts that you have to write yourself are:

*   Obtain the fuzzing agent:
    
    echo ""${CI\_FUZZ\_API\_TOKEN}"" | $CICTL\_CMD login  
    $CICTL\_CMD get javaagent
    
*   start your software with the fuzzing agent, explained here.
*   If you have not configured the URL of the Software Under Test in the yaml configuration files of your fuzz tests, add it when starting fuzzing:
    
    TEST\_COLLECTION\_RUN=$($CICTL\_CMD start --application-base-url <URL where your software is running> ""${ARTIFACT\_NAME}"")
    
*   optionally, stop fuzzing and stop your software, or wait for fuzzing to finish and then stop your software. Example:
    
    \# Wait until there are no more running fuzzing runs in this project  
    while $CICTL\_CMD list fuzzingruns -p $PROJECT |grep running , do sleep 30; done  
    \# stop our example web application which was deployed with docker compose  
    docker compose down
    

"