---
layout: default
title: Quick Start C/C++
parent: CI Fuzz
nav_order: 0
permalink: ci-fuzz-quick-start-cpp
---
# **Quick Start C/C++**
{: .no_toc }

This guide is intended to quickly get a local installation of CI Fuzz installed, configured, and running fuzz tests. 

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc .no-bullets}

---

## 1. Setting Up

Before you start, you should have:

* The CI Fuzz installer and docker image bundle files from Code Intelligence.
* [Installed the CI Fuzz CLI]({{ site.baseurl }}{% link docs/ci-fuzz-cli/cli-installation.md %})
* Root/sudo privileges on a systemd enabled Linux system (Ubuntu/Debian recommended).
* Installed docker and docker compose on the system where CI Fuzz will be installed.
* Network access to github from the CI Fuzz system (this is not required, but is helpful).

## 2. Run Installer with Docker Image Bundle

Run the provided CI Fuzz installer with the docker image bundle. 

```bash
chmod +x ci-fuzz-on-prem-installer-<version>-linux
./ci-fuzz-on-prem-installer-<version>-linux --docker-images ci-fuzz-on-prem-images-<version>-linux
```

By default, this will create the directory `/opt/ci-fuzz-<version>`. This is the installation directory. We'll start the service after we configure the server.

## 3. Configure Password Login

There are multiple ways to [configure authentication]({{ site.baseurl }}{% link docs/ci-fuzz/authentication.md %}) for CI Fuzz. For this installation, we will use the [password authentication](ci-fuzz-authentication#password) approach. If the directory `/etc/cifuzz` does not exist yet, create it and the file `cifuzz.env` now.

```bash
mkdir /etc/cifuzz
touch /etc/cifuzz/cifuzz.env
```

Open `/etc/cifuzz/cifuzz.env` with your preferred text editor and add the following values to it:

```
CIFUZZ_ENABLE_PASSWORD_LOGIN=1
DEMO_ORG_ADMIN_TOKEN=<your_password_here>
```

By default CI Fuzz runs on port 8080, but you can change this by adding the following line to `/etc/cifuzz/cifuzz.env`:

```
CIFUZZ_SERVER_PORT=<port_number>
```

This will enable us to login to the web application with just a password. This same password will be used as the access token by command line tools.

Now start the CI Fuzz service:

```bash
sudo systemctl start cifuzz
```

## 4. Project Setup

Once CI Fuzz is running, you can access the web application at `http://127.0.0.1:8080` in your browser. If you set a different port number in `/etc/cifuzz/cifuzz.env` be sure to use that port number instead.

### 4.1 Login to CI Fuzz
{: .no_toc }

When you open the web application:

1. You should see a button saying **PASSWORD LOGIN**. Click it.
2. Type the password you specified in `/etc/cifuzz/cifuzz.env` from [Step 3](#3-configure-password-login) and click **LOGIN**.
3. This will take you to the dashboard.

### 4.2 Add New Project to CI Fuzz
{: .no_toc }

Now you can add a project. You will need a repo to start fuzzing. Code Intelligence provides several demo projects which are a good way to start fuzzing and ensure everything is working correctly. 

We will use the [c-cpp-demo project for this](https://github.com/CodeIntelligenceTesting/c-cpp-demo).

To add the project to CI Fuzz, you should be at the dashboard in the web application:

1. From the sidebar on the left where it says **Select Project**, click the dropdown and select **Add Project**.
2. Set the project as **Personal**.
3. Enter the name of the project: **c-cpp-demo**
4. Copy and paste the Git URL for the c-cpp-demo project ([this one](https://github.com/CodeIntelligenceTesting/c-cpp-demo.git)) in the text box and click **ADD**. If the system where you are installing CI Fuzz cannot reach github, you can skip this step.

### 4.3 Clone Repository Locally
{: .no_toc }

This C/C++ demo already contains two fuzz tests and the necessary configuration information to work with CI Fuzz. Clone or download the project to the host where CI Fuzz is running.

```bash
git clone https://github.com/CodeIntelligenceTesting/c-cpp-demo.git
```

## 5. Running the Fuzz Tests

You will use CI Fuzz CLI to run the fuzz tests from the command line. Run the following command from the root of the project directory where you cloned/copied the c-cpp-demo project:

```bash
CIFUZZ_API_TOKEN=<your_password> cifuzz remote-run -v --server "http://127.0.0.1:8080" --timeout=120s
```

**Note**: use the password you specified in `/etc/cifuzz/cifuzz.env` and be sure to adjust the IP and port if necessary.

This command will `bundle` the c-cpp-demo project and submit it to the server. It will run each of the fuzz tests for 2 minutes (or until a crashing input is found). When you run the command above, it will ask you to select which project this fuzz test belongs to. Select the one you created earlier in [section 4.2](#42-add-new-project-to-ci-fuzz). Once the fuzz test is started, you can monitor the running fuzz tests in the UI:

1. Open the browser and navigate to CI Fuzz. 
2. Select your project from the dropdown in the top of the left sidebar.
3. Click one of the two fuzz tests (heap_buffer_overflow_test or stack_buffer_overflow_test) to view the metrics.


![](/assets/images/ci-fuzz-quick-start-cpp/running-fuzz-tests.png)

* **Coverage Metrics** - this is the coverage level of CI Fuzz over time. In this case you can see the coverage increased from the beginning of the test and stayed relatively constant. This makes sense because this is a small demo application and CI Fuzz quickly discovered several interesting paths.
* **Performance Execution** - the number of executions / second at a given time. This will vary based on your system resources, the fuzz test, and the code you are testing itself.
* **Unique Corpus Inputs** - every time the fuzzer discovers an input that leads to a new code path (increases coverage), it adds it to a set of unique corpus inputs. These are stored and can be used again later as a way to check for regressions in your code as they will cover previously discovered code paths. 

### 5.1 Review Fuzz Test

If you select **Overview** from the left sidebar, you will see the current overview of the project updated with results from the fuzz tests you just ran. This overview shows:

* that you have 2 total findings with 2 of them being new.
* the job status of the recent fuzz test run.
* the overall coverage for the project.

![](/assets/images/ci-fuzz-quick-start-cpp/overview.png)

### 5.2 Finding Details

To get additional details about the findings, you can either click on the project overview pane, or select **Findings** from the left sidebar. From there you can select one of the findings to view.

![](/assets/images/ci-fuzz-quick-start-cpp/findings-pane.png)

In this guide, we will focus on some key points about the finding, but if you want additional information, check out the [findings section]({{ site.baseurl }}{% link docs/ci-fuzz/ci-fuzz-findings.md %})

This view shows the status of each finding, it's type and ID, a link to the source code where the finding was discovered (if you added the Git URL of the repo and it is reachable from CI Fuzz).

CI Fuzz also provides information to help remediate the finding. When you expand a finding in the bottom pane, it will contain 3 tabs with different information: `Debug`, `Description`, and `Log`. These tabs contain several pieces of information that can help you determine the root cause of the finding. 

* The `debug` tab contains the fuzz test responsible for the finding, the source line, the stack trace (if available), and the crashing input. 
* The `description` tab contains the `severity score`, a short description of the finding, and possibly some links to additional information about this type of finding.
* The `log` tab content depends on the type of fuzzing that discovered the finding. If the finding is from unit fuzzing, then the output will be output directly from the fuzzer. If the finding is from Web API fuzzing, then the output will contain the API request responsible for triggering the finding.

### 5.3 Examine Coverage

To view additional details about coverage, just click **Code Coverage** on the left sidebar. This will show you the overall coverage and a breakdown of the coverage by file.

![](/assets/images/ci-fuzz-quick-start-cpp/code-coverage.png)