---
layout: default
title: Running a Fuzz Test
parent: CI Fuzz
nav_order: 2
permalink: ci-fuzz-running-a-fuzz-test
---
# **Running a Fuzz Test with CI Fuzz**
{: .no_toc }

This page describes how to run a fuzz test on CI Fuzz, both manually and as part of a CI/CD pipeline.

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

## Create a Bundle with CI Fuzz CLI

If you're project is already configured with CI Fuzz CLI, you can create a bundle by running the following command from the root of the project directory:

```bash
cifuzz bundle
```

* This command will build and bundle the runtime artifacts needed to run your fuzz test(s) on a CI Fuzz server
* You can specify the name of one or more fuzz tests or don't specify any to build all the fuzz tests for a project
* You can specify various arguments that will tell CI Fuzz how to run your fuzz tests. See [this page]({{ site.baseurl }}{% link docs/ci-fuzz-cli/cli-remote-run.md %}) for additional details.
* The output is a tarball of the form `fuzz_tests.tar.gz` or `<name_of_specific_fuzz_test>.tar.gz` if you specified one.

### Drag and Drop Bundle

Once you have created your bundle, you can drag and drop it in the Web UI on the project card you want to run it under. You can either do this from the **Dashboard** or from the **Overview** after selecting your project.

### Import and Start with cictl

Once you have created a bundle, you can also use the `cictl` command (located in `ci-fuzz-install-directory/bin/cictl`) to upload the bundle to the CI Fuzz server and then start it. Here are the `cictl` commands you can use to accomplish this

```bash
FUZZING_SERVER="<address of fuzzing server>"
PROJECT="<project name>"
CICTL_CMD="cictl -s $FUZZING_SERVER"
ARTIFACT_NAME=$($CICTL_CMD import artifact fuzz_tests.tar.gz --project-name "$PROJECT")
$CICTL_CMD start ${ARTIFACT_NAME}
```

* Most `cictl` commands require the `--server, -s` flag to specify the location of the fuzzing server, so be sure to include it
* The project name can be obtained by running `cictl -s <fuzzing server> list projects`. The project name should be of the form `projects/my_project-391A177B`.

## CI Fuzz CLI Remote Run

You can use the CI Fuzz CLI remote-run command (`cifuzz remote-run`) to bundle, submit, and start your fuzz tests with one command. This method does not require `cictl`.

```bash
cifuzz remote-run --server <server address>
```

* You can authenticate to the server automatically by passing the environment variable `CIFUZZ_API_TOKEN` as part of your command (e.g. `CIFUZZ_API_TOKEN=<token> cifuzz remote-run --server "https://cifuzz.server.com"`).
* You can specify the name of one or more fuzz tests. If you or don't specify any, then `remote-run` will bundle and run all the fuzz tests for a project.
* You can specify various arguments that will tell CI Fuzz how to run your fuzz tests. See [this page]({{ site.baseurl }}{% link docs/ci-fuzz-cli/cli-remote-run.md %}) for additional details.
* You can pass an existing bundle by simply specifying the `--bundle` flag and the path to the bundle. When passing an existing bundle, CI Fuzz will ignore any runtime flags passed as part of the `remote-run` command (e.g. `--timeout`, `--docker-image`). 

## Running Fuzz Tests with CI/CD  

To create a CI/CD pipeline, you will need to install both [CI Fuzz CLI]({{ site.baseurl }}{% link docs/ci-fuzz-cli/cli-installation.md %}) and the `cictl` command line tool. CI Fuzz must be reachable from the CI/CD server.

The general workflow is

1. Build/bundle the artifacts that will run on the server
2. Upload the artifacts to the server and run them 
3. Monitor the results

Here is a script you can use to help write and test your CI/CD pipeline.

```bash
#! /usr/bin/bash

# how long to monitor the fuzz test
TIMEOUT=300
# this should match wherever CI Fuzz is currently reachable
FUZZING_SERVER="127.0.0.1:8080"
# address of the web application
WEB_APP_ADDRESS="http://127.0.0.1:8080"
# project name of the form: projects/c-cpp-demo-0b5fbe28
# this can be obtained by using cictl -s <server> list projects
PROJECT=<project name>
# access token generated in the CI Fuzz web app
CI_FUZZ_API_TOKEN=
# local directory where the project is located
LOCAL_REPO=

# create bundle from local repo
# you can specify additional bundle options at the command line, or include them in cifuzz.yaml
cd $LOCAL_REPO
cifuzz bundle 

CICTL_CMD="cictl -s $FUZZING_SERVER"
# login to the CI Fuzz server
echo $CI_FUZZ_API_TOKEN | $CICTL_CMD login
# upload the bundle to the CI Fuzz server
ARTIFACT_NAME=$($CICTL_CMD import artifact fuzz_tests.tar.gz --project-name "$PROJECT")
# start the fuzz tests in the bundle 
CAMPAIGN_RUN=$($CICTL_CMD start --application-base-url "${WEB_APP_ADDRESS}" "${ARTIFACT_NAME}")
# monitor the output from the campaign_run
$CICTL_CMD monitor_campaign_run --dashboard_address="${WEB_APP_ADDRESS}" --keep_running --duration="${TIMEOUT}" ${CAMPAIGN_RUN}
```
