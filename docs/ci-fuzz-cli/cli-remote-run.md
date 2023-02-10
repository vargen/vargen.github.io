---
layout: default
title: Remote Run
parent: CI Fuzz CLI
nav_order: 8
permalink: cli-remote-run
---

# **Remote Run**
{:.no_toc}

The `cifuzz remote-run` command builds fuzz tests, packages all runtime artifacts into a bundle, and uploads that to a remote CI Fuzz server to start a remote fuzzing run. This section covers the components of the `remote-run` command that are necessary to make it work with your CI Fuzz installation.

**Note**: The `cifuzz remote-run` command currently only supports C/C++ projects. 

---

## Requirements

These are the items that must be specified to use `cifuzz remote-run` with CI Fuzz in a CI/CD pipeline.

### cifuzz.yaml

The project must have a `cifuzz.yaml` (created from `cifuzz init`) with the appropriate build-system files modified. See [how to initialize a project]({{ site.baseurl }}{% link docs/ci-fuzz-cli/cli-initialize-a-project.md %}) for details on how to do that.

`cifuzz.yaml` should have the correct `build-system` set (e.g. CMake, Bazel). If the `build-system` is other, a `build-command` must be specified. This can either be set in `cifuzz.yaml` or passed as a command line argument using the `--build-command` flag of `remote-run`.

### Docker Image

The bundle requires a docker image to run on CI Fuzz. The default image is "ubuntu:rolling", but you can also specify your own image by using the `--docker-image` flag. The docker image you specify should contain the necessary runtime libraries.

### Project

The project has to be specified, otherwise cifuzz will prompt for it. It is expected that the project was already created on the remote server. Creating a project from scratch using the cifuzz is not supported yet. Use the `--project` flag to specify the project name (e.g. c-cpp-demo-4c381320).

### Server

Specify where the CI Fuzz server is located using the `--server` flag (protocol://domain:port).

### Access Token

CI Fuzz requires an access token. Set an environment variable named `CIFUZZ_API_TOKEN` and assign it a token that was created in the CI Fuzz web application. This token will be stored in `~/.config/cifuzz/access_tokens.json` and used on subsequent executions of `cifuzz remote-run`.

### Fuzz Test

You must specify the name of 1 or more fuzz tests to be run. 





