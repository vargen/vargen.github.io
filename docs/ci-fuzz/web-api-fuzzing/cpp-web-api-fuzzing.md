---
layout: default
title: CPP Web API Fuzzing
parent: Web API Fuzzing
grand_parent: CI Fuzz
nav_order: 2
permalink: cpp-web-api-fuzzing
---
# **C++ Web API Fuzzing**
{: .no_toc }

CI Fuzz is capable of coverage-guided fuzzing for C++ based REST and GRPC APIs. This section will guide you through the initial setup of a C++ Web API fuzzing project. 

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
* Received the C++ bug detector framework from Code Intelligence.

## Web API Server

Your Web API can run anywhere as long as it is reachable from the server hosting CI Fuzz. Port details are provided below.

Typically, this would be one of the following:

* On the CI/CD server (gitlab/github/jenkins server)
* On the CI Fuzz server
* A separate server (e.g. your integration testing cluster)

If you choose to run your Web API on a separate server from the one hosting CI Fuzz, you need to make sure that:

1. The Web API is exposed on a network interface and the port on which it is running is reachable from CI Fuzz.
2. Ports 443 or 80 and 6777-7777 on the server running CI Fuzz are reachable from the server hosting your Web API.

## Create a Fuzz Test

You should have already created a fuzz test based on [the provided template]({{ site.baseurl }}{% link docs/ci-fuzz/web-api-fuzzing/index.md %}).

## Add a Web Service

CI Fuzz uses the idea of web services to associate fuzz tests to a given service when the fuzzing agent contacts CI Fuzz from the software under test. Web services can be created in the UI with the following steps:

1. Open CI Fuzz in your browser
2. From the left sidebar, select your project from the dropdown menu.
3. Click **Project Settings**
4. Click **WEB SEVICES**
5. Click the **ADD WEB SERVICE** button.
6. Add a name in the **Web Service Identifier** textbox. The name/identifier can be whatever you like, but it must match the name in the `.code-intelligence/web_services.yaml` file. If you used the template project, the name is `mywebservice` by default.
7. Click **NEXT**.
8. Select **C/C++**
9. If you have not already, you can download the fuzzing agent by clicking **DOWNLOAD NATIVE FUZZING AGENT**. You can also download the fuzzing agent at any time by running `cictl get nativeagent`.
10. If you have not already, you can download the static support library by clicking **DOWNLOAD STATIC SUPPORT LIBRARY**. You can also download the static support library at any by running `cictl get fuzzingsupportlib`.
11. Click **DONE**

## Compile C++ Bug Detector Framework

The C++ bug detector framework is a framework to hook into C and C++ code with the goal of detecting bugs via CI Fuzz. It is provided as a CMake project so that users can build it with their compiler, using their C++ runtime. The final artifact is a shared object, `libhooks.so`, which is meant to be preloaded into the
application under test. Currently, only Linux is supported.

### Requirements
{: .no_toc }

* C++ 17+ compiler toolchain.
* If detectors with external dependencies are activated during build an internet connection to download them is necessary.

### Air Gapped Environments


If you are building this framework in an airgapped environment, you will need to download the dependencies manually and place them in the folders where CMake expects them. Download all files referenced in CMakeLists.txt by `ExternalProject_Add` URL and place them in <build-dir>/<project-name>-prefix/src/external-project-url-downloaded-file>.

For the sql-parser dependency this would be build/sql-parser-prefix/src/f0faaad39904f7756d1a8becf798a96e444351d0.zip
downloaded from https://github.com/CodeIntelligenceTesting/sql-parser/archive/f0faaad39904f7756d1a8becf798a96e444351d0.zip

### Build 


```
mkdir build
cd build
cmake -DCMAKE_BUILD_TYPE=RELEASE ..
cmake --build .
```

**Note**: Optional detectors will be enabled depending on your build environment. The CMake configuration process reports for each optional detector whether it's
enabled, and why. For example, if you have a license for SQLAPI++, you can enable the SQL injection detector by pointing CMake to your SQLAPI.h using -DCMAKE_INCLUDE_PATH=/path/to/sqlapi/include.

External dependency for all detectors: sql-parser

* SQLAPI++
    * Deactivated by default; to enable set -DCMAKE_INCLUDE_PATH=/path/to/sqlapi/include
    * Tests require SQLAPI++, point CMake to it with -DCMAKE_LIBRARY_PATH=/path/to/sqlapi/lib
* SQLite
    * Enabled by default; to disable set -DENABLE_SQLITE=OFF
    * `libsqlit3` required on build system

After building the framework, you will have the `libhooks.so` file that you can preload with your application under test when you run it. 

## Instrument your Code for Fuzzing

Add the following flags to your compilation step:

```
-fprofile-instr-generate -fcoverage-mapping -fsanitize=fuzzer-no-link -Lpath/to/downloaded/agent -Wl,-rpath,path/to/downloaded/agent -lfuzzing_agent
```

Add the following flags to your linker step:

```
-fprofile-instr-generate -fsanitize=fuzzer-no-link -L${CIFUZZ_DIR}/lib/code-intelligence -Wl,-rpath,${CIFUZZ_DIR}/lib/code-intelligence -lfuzzing_agent
```

Link the static support library with your binary with the following flags:

```
-Wl,--whole-archive path/to/downloaded/libfuzzing_support_lib.a -Wl,--no-whole-archive
```

### Running the Instrumented Binary


Before you start your fuzz test, you need to start the instrumented binary you created. This binary requires some environment variables so that the agent (`libfuzzing_agent.so`) knows how to reach CI Fuzz. There are several environment variables you can provide when invoking the instrumented binary. Most of them are required:

* ASAN_OPTIONS=alloc_dealloc_mismatch=0 - deactivates allocation checks that would be triggered in the hook.
* LD_PRELOAD - path to the `libhooks.so` when you [built the bug detector framework](#build). This is required.
* CIFUZZ_AUTH_TOKEN - API token or password you configured when you [configured authentication]({{ site.baseurl }}{% link docs/ci-fuzz/authentication.md %}).
* CIFUZZ_SERVICE_NAME - the full [web service you added](#add-a-web-service). This can be obtained by running `cictl list webservices` and copying the appropriate one. Be sure to copy the full path. It should be of the form `projects/<project_name>/web_services/<web_service_name>`.
* CIFUZZ_FUZZING_SERVER_HOST - the IP or domain where CI Fuzz is reachable.
* CIFUZZ_FUZZING_SERVER_PORT - the port where CI Fuzz is reachable.
* CIFUZZ_TLS - if CI Fuzz is configured with TLS, this should be set to 1.
* CIFUZZ_TLS_CERT_FILE - if you are using a custom certificate, specify the path here.
* CIFUZZ_HALT_ON_ERROR - a discovered bug does not exit the application by default. If you want the application to halt on errors, set this variable to 1.

**Warning**: If TLS is enabled for CI Fuzz and `CIFUZZ_TLS` is not set to 1, the connection will fail.

Example of running the instrumented binary with appropriate environment variables:

```bash
ASAN_OPTIONS=alloc_dealloc_mismatch=0 \
LD_PRELOAD=/home/demo/cpp-bug-detector-framework/build/libhooks.so \
CIFUZZ_TLS=1 \
CIFUZZ_AUTH_TOKEN=d24573f2-b6d5-4180-9ce6-9a93872942ad \
CIFUZZ_SERVICE_NAME=projects/cpp-web-app-4d9d77bd/web_services/cpp-sql-service \
CIFUZZ_FUZZING_SERVER_PORT=8080 \
CIFUZZ_FUZZING_SERVER_HOST=127.0.0.1 \
/home/demo/repos/cpp-web-app/web-app-demo
```

## Example Run Script

The following is an example run script for running the fuzz tests for a gRPC project. This can also be used as the basis for a CI/CD script to automate the fuzzing process.

```bash
#! /usr/bin/bash

# duration to monitor the campaign run
TIMEOUT=120
# type of findings to report
FINDINGS_TYPE=CRASH,RUNTIME_ERROR
# [IP | DOMAIN]:PORT where CI Fuzz is currently reachable
FUZZING_SERVER=
# http[s]://[IP | DOMAIN]:PORT where CI Fuzz is currently reachable
WEB_APP_ADDRESS=
# project name obtained from: cictl list projects
PROJECT=
# API token (or password) you configured for connecting to CI Fuzz
CI_FUZZ_API_TOKEN=
# path to the root of the local repository
LOCAL_REPO=
# [IP | DOMAIN]:PORT where the target API is running
SOFTWARE_UNDER_TEST

# build the artifacts to submit to CI Fuzz
cd $LOCAL_REPO
ci-build fuzzers --directory=./
mkdir fuzzing_artifacts

# authenticate to the fuzzing server
CICTL_CMD="cictl -s $FUZZING_SERVER"
echo $CI_FUZZ_API_TOKEN | $CICTL_CMD login

# import the artifact bundle to CI Fuzz
ARTIFACT_NAME=$($CICTL_CMD import artifact $LOCAL_REPO/fuzzing-artifacts.tar.gz --project-name "$PROJECT")

# start the fuzz tests
TEST_COLLECTION_RUN=$($CICTL_CMD start --application-base-url $SOFTWARE_UNDER_TEST "${ARTIFACT_NAME}")

# monitor the output of the run
$CICTL_CMD monitor_campaign_run --dashboard_address="${WEB_APP_ADDRESS}" --duration="${TIMEOUT}" --findings_type="${FINDINGS_TYPE}" ${TEST_COLLECTION_RUN}
```