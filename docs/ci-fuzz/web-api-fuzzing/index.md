---
layout: default
title: Web API Fuzzing
parent: CI Fuzz
nav_order: 6
has_children: true
has_toc: true
permalink: web-api-fuzzing
---

# **Web API Fuzzing**
{: .no_toc }

CI Fuzz is capable of coverage-guided fuzzing for Java- and Go-based REST and GRPC APIs. This section contains the initial steps you will need to perform to prepare your API for fuzzing.

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

## Download the Template

Download the following template to the system where you will be building your fuzz tests:

* <a href="/assets/files/web-api-fuzzing-template.tar">CI Fuzz Web API Fuzzing Template</a>

The template contain a directory structure that CI Fuzz needs to start fuzz tests. Depending on the type of API you intend to fuzz, REST or gRPC, will determine the type of modifications you need to make to the configuration files in the template.

## Template Overview

Here is an overview of the key files and directories in the template.

### .code-intelligence/web_services.yaml
{: .no_toc }

The `web_services.yaml` file lists the different web services you want to fuzz. Each fuzz test must be associated with a web service. If your application consists of several microservices, add one web service entry for each microservice and name them appropriately. Web services can be named however you want as long as they are consistent between the `web_services.yaml` and `<fuzz_test>.yaml` files.

### .code-intelligence/fuzz_targets
{: .no_toc }

This folder contains the API fuzz tests. A fuzz test for an API consists of two things:

* A fuzz test configuration file (`<fuzz_test>.yaml`). 
* HTTP requests necessary to authenticate to the application.

The fuzz test configuration file contains several options that can be configured to adjust the behavior of your fuzz test. Some options are specific to the type of fuzz test you are performing (HTTP vs gRPC).

**Note**: the directory containing the fuzz tests can be configured in `.code-intelligence/project.yaml` if needed by setting the `web_app_target_configs` option.

### .code-intelligence/fuzz_test_seed_corpus
{: .no_toc }

The `<fuzz_test>_seed_corpus` directory should contain manual seeds that you add to improve the fuzz test. 

## Configure the Template 

Configure the template files as follows depending on if you are fuzzing HTTP or gRPC.

### HTTP / REST

#### **.code-intelligence/web_services.yaml**
{: .no_toc }

Change the name of the web_service if desired. A web service can have any name as long as it matches that used by the fuzzing agent and listed in `.code-intelligence/project.yaml`.

If your application has an OpenAPI / swagger specification file, add the path to it in this file on the line containing `open_api_spec`. The path is relative to the root directory of the git repository. CI Fuzz will parse this file and automatically generate seeds to use for fuzzing. An OpenAPI specification corresponds to a specific web service.

#### **.code-intelligence/fuzz_targets/fuzz_test.yaml**
{: .no_toc }

Configure the following in your `<fuzz_test>.yaml` file:

* Set `protocol` to http.
* Set `base_url` to the address where the application you are testing will be running. Include the protocol when specifying the address: `http(s)://`
* Enable and set `allowed_requests` to whichever HTTP paths (endpoints) the fuzzer should test. This is optional.
* Enable and set `denied_requests` to whicheer HTTP paths (endpoints) the fuzzer should not attempt to test. This is optional.
* Enable and set `health_check_url` to an endpoint that CI Fuzz can use to verify the target application is reachable and running. This is optional.

#### **.code-intelligence/project.yaml**
{: .no_toc }

Set the `web_app_target_configs` to the path to the `<fuzz_test>.yaml` files in `.code-intelligence/fuzz_targets/`.

### gRPC

#### **.code-intelligence/web_services.yaml**
{: .no_toc }

Change the name of the web_service if desired. A web service can have any name as long as it matches that used by the fuzzing agent and listed in `.code-intelligence/project.yaml`.

#### **.code-intelligence/fuzz_targets/fuzz_test.yaml**
{: .no_toc }

Configure the following in your `<fuzz_test>.yaml` file:


* Set `protocol` to grpc.
* Set `base_url` to the address where the application you are testing will be running. Specify the address as `[DOMAIN | IP]:[PORT]`.
* Enable and set `run_extra_args` to be `"--proto_stub_path=./libproto_stub.so"`.
* Enable and set `health_check_url` to an endpoint that CI Fuzz can use to verify the target application is reachable and running. This is optional.

#### **.code-intelligence/project.yaml**
{: .no_toc }

Set the `web_app_target_configs` to the path to the `<fuzz_test>.yaml` files in `.code-intelligence/fuzz_targets/`.
