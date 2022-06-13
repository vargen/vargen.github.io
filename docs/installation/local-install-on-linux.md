---
layout: default
title: Local Installation on Linux
parent: Installation
nav_order: 1
---

# Local Installation on Linux

## How to Install CI Fuzz Locally for Initial Project Setup

### Overview

CI Fuzz is a software system including a fuzzing backend as well as a user interface.

Its fuzzing backend relies on docker to encapsulate different tasks such as building, running, and monitoring components. The user can interact with the system through a command line interface (CLI) or via the CI Fuzz extension for Visual Studio Code.

**CI Fuzz contains three main components:**

* CI-Daemon: The CI-Server is responsible for backend tasks such as project compilation, managing fuzz-targets and operating the docker infrastructure and interacts with the CI-Client and the UI.
* User-Interface: The CI Fuzz extension for Visual Studio Code helps the user to create and manage fuzz-targets and reproduce crash.
* CI-Client: The CI-Client is the command line interface to initialize, build, and run fuzzers.


