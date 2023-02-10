---
layout: default
title: Go Web API Fuzzing
parent: Web API Fuzzing
grand_parent: CI Fuzz
nav_order: 1
permalink: go-web-api-fuzzing
---
# **Go Web API Fuzzing**
{: .no_toc }

CI Fuzz is capable of coverage-guided fuzzing for Go-based REST and GRPC APIs. This section will guide you through the initial setup of a Go Web API fuzzing project. 

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
* Go version 1.18 or newer

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
8. Select **GO**
9. If you have not already, you can download the fuzzing agent by clicking **DOWNLOAD NATIVE FUZZING AGENT**. You can also download the fuzzing agent at any time by running `cictl get nativeagent`.
10. Click **DONE**

## Instrument your code for fuzzing

In order to fuzz your application, you first need to instrument the code for fuzzing. This includes adding bug detection 
capabilities and code coverage instrumentation. Please note that to use the tools we provide for instrumentation, your code 
should be organized either in the `GOPATH` directory layout, or using Go modules. If you are using Bazel, you can use 
the `go_path` [rule here](https://github.com/bazelbuild/rules_go/blob/master/docs/go/core/rules.md#go_path) to do that. 


### Instrument your code with bug detectors

We have developed an extensible framework for Go bug detectors that can catch higher-level bugs such as command or 
template injection while fuzzing (https://github.com/CodeIntelligenceTesting/gofuzz). **gofuzz** provides a CLI tool 
to add bug detection capabilities into your Go code. It transforms the source code and replaces calls to functions/methods 
of interest by calls to corresponding hooks in the `github.com/CodeIntelligenceTesting/gofuzz/sanitizers` module. 
By default, **gofuzz** does not change the code in-place, but generates the instrumented source code in a temporary directory.
It produces an [overlay file](https://go.dev/doc/go1.16) that can be used by Go's standard build tools. Three steps 
are needed to instrument bug detection capabilities into your code:

1. (Once) Install the **gofuzz** CLI

   ```shell
   go install github.com/CodeIntelligenceTesting/gofuzz/cmd/gofuzz@latest
   ```

2. (Once) Add the **sanitizers** package as a dependency for the code base you want to test.
   This package contains the implementation of the hooks inserted by **gofuzz** into your code,
   and therefore must be available when the instrumented code is being compiled.

   ```shell
   cd <my project>
   go get -u github.com/CodeIntelligenceTesting/gofuzz/sanitizers@latest
   ```

   This command also adds the **sanitizers** package as a dependency in the `go.mod` file.
3. Instrument your code using the **sanitize** subcommand

   ```shell
   gofuzz sanitize <package> -o <overlay.json>
   ```

   This instruments the specified package and writes the instrumented file into a temporary
   directory. The corresponding file replacements are stored in the <overlay.json> file.
   By default, **gofuzz** writes a file named overlay.json in the current directory.

### Instrument your code for code coverage

In order to fuzz your application, you will need to instrument the code for code coverage. This instrumentation is used
to guide the fuzzer so that it generates inputs that maximize code coverage. CI Fuzz provides a command line tool to help
with this called `ci-gofuzz`. The `ci-gofuzz` tool has several options you can view with `ci-gofuzz -h`, but here are 
the recommended options to use:

* **-cifuzz_agent_lib** - this is the path to the fuzzing agent library (libfuzzing_agent.so). The fuzzing agent is 
responsible for communicating with the fuzzing server. It can be obtained using the command 
`cictl -s <address of fuzzing server> get nativeagent`. This will download the fuzzing agent to the current 
directory where `cictl` was executed from. 
* **-cover** - this enables source-based coverage instrumentation. This is needed to get a coverage report similar to 
what you get with `go cover`
* **-include** - this is a comma separated list of import paths that should be instrumented. If this is not specified, 
then CI Fuzz will attempt to fuzz all packages with their dependencies. This might result in a loss of performance and
generally fuzzing code you may not be interested in. It is recommended you set this to the specific packages you want to fuzz. 
* **-o** - name of the output file of the instrumented binary.
* **-preserve** - a comma-separated list of import paths not to instrument for fuzzing.
* **-race** - enable data race detection
* **-exclude_from_coverage** - a comma-separated list of regexes of file paths that should not be instrumented for coverage (e.g. mocks, test helpers).

You must also specify the package path you want to instrument. Here is an example of running `ci-gofuzz` against the [helloworld grpc server](https://github.com/grpc/grpc-go/tree/master/examples/helloworld)

```bash
ci-gofuzz -cifuzz_agent_lib libfuzzing_agent.so -cover -include 'google.golang.org/grpc/examples/helloworld*' -o greeter_server ./helloworld/greeter_server
```

After running the above command, it will produce an instrumented binary (called `greeter_server` in this instance).

### Running the Instrumented Binary

Before you start your fuzz test, you need to start the instrumented binary you created with `ci-gofuzz`. This binary requires some environment variables so that the agent (`libfuzzing_agent.so`) knows how to reach CI Fuzz. You should provide the following environment variables when invoking the instrumented binary:

* LD_LIBRARY_PATH - path to the directory containing `libfuzzing_agent.so`
* CIFUZZ_AUTH_TOKEN - API token or password you configured when you [configured authentication]({{ site.baseurl }}{% link docs/ci-fuzz/authentication.md %}).
* CIFUZZ_SERVICE_NAME - the full [web service you added](#add-a-web-service). This can be obtained by running `cictl list webservices` and copying the appropriate one. Be sure to copy the full path. It should be of the form `projects/<project_name>/web_services/<web_service_name>`.
* CIFUZZ_FUZZING_SERVER_HOST - the IP or domain where CI Fuzz is reachable.
* CIFUZZ_FUZZING_SERVER_PORT - the port where CI Fuzz is reachable.
* CIFUZZ_TLS - if CI Fuzz is configured with TLS, this should be set to 1.
* CIFUZZ_TLS_CERT_FILE - if you are using a custom certificate, specify the path here.

**Warning**: If TLS is enabled for CI Fuzz and `CIFUZZ_TLS` is not set to 1, the connection will fail.

Example of running the instrumented binary with appropriate environment variables:

```bash
CIFUZZ_TLS=1 \
CIFUZZ_AUTH_TOKEN=d24573f2-b6d5-4180-9ce6-9a93872942ad \
LD_LIBRARY_PATH=/home/demo/repos/grpc-go \
CIFUZZ_SERVICE_NAME=projects/grpc-go-4d9d77bd/web_services/go-grpc-server \
CIFUZZ_FUZZING_SERVER_PORT=8080 \
CIFUZZ_FUZZING_SERVER_HOST=127.0.0.1 \
/home/demo/repos/grpc-go/greeter_server
```

## Compile Application Protocol Buffers

An essential step of the fuzzing setup of gRPC applications is to compile the protocol buffer files of the application. The most commonly used Interface Definition Language of gRPC applications are Protobuffers. Fuzzing of gRPC applications will target the mutation and transmission of protobuffers. For that reason, the proto buf definitions will be used to generate stubs that can be called by the fuzzer and be used for the mutation of the application's input data.  The protobuffer description files (.proto) of the target application can be compiled with ci-protoc of Code Intelligence:

```bash
ci-protoc STUB_OUT_PATH PROTOC_ARGS...
```

`STUB_OUT_PATH` will be the name of a shared object file that can be used afterwards by the CI-daemon to generate reasonable gRPC input data for the target application.

### Adding Field Hints

Some applications have authorization or authentication in place which requires to have fixed values to be set in protobuffers sent by client applications. For example, assume there is an access token called "letmein" for a gRPC target service, then the following example ci-protoc command can be used to generate a stub that covers authorized messages during fuzzing.

```bash
ci-protoc libproto_stub.so -Iproto proto/target_service.proto --field_hint=access_token=let_me_in
```

`Field hints` will be used as hints during fuzzing. This means the fuzzer will also fuzz the access_token hint, but sometimes will use the set hint to cover authorized code paths in addition. 

**Note**: It is possible to set multiple hint values for the same field if it is reasonable, for example to cover different authorization level of an application.

**Note**: When CI Fuzz uses `field hints` is essentially non-deterministic.

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
# add the libproto_stub.so file to the artifact bundle
mkdir fuzzing_artifacts
cd fuzzing_artifacts
tar -xf ../fuzzing-artifacts.tar.gz
cp ../libproto_stub.so work_dir
tar -czf ../fuzzing-artifacts.tar.gz *

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
