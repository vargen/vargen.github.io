---
layout: default
title: Installation
parent: CI Fuzz
nav_order: 1
permalink: ci-fuzz-installation
---
# **Installing CI Fuzz**
{: .no_toc }

This page will walk you through installing CI Fuzz. Before you begin, you should have received:

* the CI Fuzz on-prem installer (`ci-fuzz-on-prem-installer-<version>-linux`)
* a docker image bundle (`ci-fuzz-on-prem-images-<version>-linux`)

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---

## Prerequisites

In order to use the provided install mechanism and the resulting docker-compose setup, some prerequisites have to be met. We recommend using a server with at least:

*   32 GB RAM
*   16 or more CPU cores
*   100 GB storage
*   A Linux operating system
*   ssh access (or equivalent)
*   Docker and Docker-Compose
*   root privileges (if you want to use port 80/443)
*   Open ports. See [Ports and Network Settings](#ports-and-network-settings) for details
*   Access to a docker registry.

### Linux
{: .no_toc }

The only supported operating system is Linux. Most linux distributions will be fine, while the most comfort can be achieved by using one that supports systemd. If in doubt, 
we recommend using Ubuntu Server.

### TLS
{: .no_toc }

Operating system's certificate store must be in the following location:
```bash
/etc/ssl/certs/ca-certificates.crt
```
This is the default on Ubuntu and Debian. On Centos, you need to create a hard link here:
```bash
ln /etc/pki/ca-trust/extracted/openssl/ca-bundle.trust.crt /etc/ssl/certs/ca-certificates.crt
```

### Docker
{: .no_toc }

Docker-Compose relies on docker being installed on the host system. To do so, follow the provided instructions for your distribution as depicted in the docker engine documentation.

Additionally, CI Fuzz runs fuzz tests inside specific Docker containers. These docker containers are specified as part of the fuzz tests themselves, but the CI Fuzz server must have access to these docker images. They can either be pulled from a docker registry (internal or external) or otherwise placed on the CI Fuzz server.

### Docker-Compose
{: .no_toc }

In addition to the docker engine, the setup uses docker-compose to orchestrate its components. To setup docker-compose, follow the installation instructions.

The minimum supported docker-compose version is 1.27.0. CI Fuzz also supports the Compose plugin for Docker (`docker compose`).

### Systemd
{: .no_toc }

Having docker-compose allows to easily start and stop the service. But to be (re)started at boot, additional work has to be done. The installer provides a shortcut by setting up a systemd system service.

### Ports and Network Settings
{: .no_toc }

This table provides some details about potential port requirements that may be needed. 

**Note** AUT is the Application Under Test and applies to Web API fuzzing only.


| Port    	|Protocol|Source    |Destination|Purpose   | Comments |
|:---------:|:------:|:--------:|:---------:|:--------:|:--------:|
| 443     	| TCP | Browser     | CI Server | Access CI Fuzz web application to view and manage findings | Port is configurable in CI Fuzz |
| OAuth Provider Port  	| TCP | CI Server   | OAuth Provider      | CI Server contacts the OAuth provider during authentication. |
| 443   	| TCP | cictl       | CI Server | Start fuzz tests, upload artifacts, monitor status | Port is configurable in CI Fuzz |
| 443	    | TCP | CI/CD Server| CI Server | CI/CD server can initiate jobs on CI Server | Port is configurable in CI Fuzz |
| 6777-7777 | TCP | AUT         | CI Server	| Java agent to contact CI Server with finding and coverage information | Web API fuzzing only. Port range is configurable in CI Fuzz |
| AUT Listening Port  | TCP | CI Server   | AUT       | Send fuzzing inputs to AUT | Port depends on listening port of AUT. Web API fuzzing only. |


A DNS entry for your CI Fuzz server must exist, or you need a custom TLS certificate that is valid for an IP address.

## Step 1: Get a TLS certificate

For secure TLS connections to the fuzzing server, a TLS certificate is needed. Your company may already use a specific CA or run its own. If not, you can get free TLS certificates from Let's Encrypt.

Just install the command line tool certbot and request a certificate with a single command:

```bash
sudo apt install certbot  
sudo certbot certonly --standalone --preferred-challenges http -d <your domain>
```

## Step 2: Run the installer

You should have received a link for the installer and a docker image bundle (CI Fuzz on-premise backend image bundle) from Code Intelligence. Run the installer with the docker image bundle as shown here:

```
chmod +x ci-fuzz-on-prem-installer-<version>-linux
./ci-fuzz-on-prem-installer-<version>-linux --docker-images <path to your bundle>
```

You need at least 10 GB of free disk space to run the installer when using the image bundle, as it needs to create some temporary files. 

## Step 3: Configuration

When CI Fuzz is started it will load all configuration variables found in `/opt/ci-fuzz-<version>/cifuzz.env`. It will then search for any `.env` files in `/etc/cifuzz`. Any variables defined there will overwrite those originally taken from `/opt/ci-fuzz-<version>/cifuzz.env`.

### Data Storage Locations
{: .no_toc }

The following table lists the standard locations and components for persisted data for a CI Fuzz installation. This is a useful reference when planning your initial installation and how you will handle upgrades and backups.

| Data Item |Default Location|Purpose|Configurable|Comments|
|:---------:|:--------------:|:-----:|:----------:|:------:|
| CIFUZZ_DATA_DIR     	| $HOME/.local/share/cifuzz | Directory containing fuzzing artifacts and corpora     | Location can be configured in .env files in /etc/cifuzz | The directory can be a network share. |
| CI Fuzz Database  	| /var/lib/docker/volumes/cifuzz_db-data/_data | Postgres database for storing fuzzing results and user data.   | Not configurable in CI Fuzz      | Backup/tar this directory. See **note** below |
| Configuration files   	| /etc/cifuzz | cictl       | Configuration files for CI Fuzz | CI Fuzz first looks for cifuzz.env and then applies any additional .env files that are found in the directory specified by cifuzz-server --env-files-from | 

**Note**:  If deploying CI Fuzz to a new system / image, it is recommended you first start CI Fuzz to initialize the docker volume directory. Then stop CI Fuzz,  copy over the database backup, and restart CI Fuzz. 

### 3.1 First Time Install
{: .no_toc }

If this is your first time installing CI Fuzz, you will need to create the `/etc/cifuzz` directory.

```bash
sudo mkdir /etc/cifuzz/
```

You can download <a href="/assets/files/cifuzz.env.template">this template</a> to `/etc/cifuzz/cifuzz.env` and then fill in the values as described below. You can use the command:

```bash
sudo wget https://docs.code-intelligence.com/assets/files/cifuzz.env.template -O /etc/cifuzz/cifuzz.env
```

* **CIFUZZ_SERVER_ORIGIN** - change to the hostname of the host where you are running CI Fuzz server. Please set full origin here - protocol (https), hostname and port. If you are using OAuth, then this must match the origin set in your OAuth application.
* **CIFUZZ_SERVER_PORT** you can change the port if needed. Defaults to 8080 if not set.
* **CIFUZZ_CERT_FILE** and **CIFUZZ_CERT_KEY** - absolute paths to the certificate and key that CI Fuzz should use. 
* **CIFUZZ_INTER_SERVICE_AUTH_KEY** - generate a random key using `openssl rand -hex 32` or equivalent.
* **CIFUZZ_NUM_WORKER_THREADS** - set to the number of CPU cores available 
* **CI Fuzz Authentication** - there are several ways to configure authentication to the `CI Fuzz Web Application`. The template contains areas for various OAuth applications. You only need to fill in the values for one of them. See [Authentication]({{ site.baseurl }}{% link docs/ci-fuzz/authentication.md %}) for details on different authentication options. 


### 3.2 Upgrading CI Fuzz
{: .no_toc }

If you are upgrading CI Fuzz from a previous version, there is no need to adjust your configuration options in `/etc/cifuzz/cifuzz.env` unless new functionality has been added that you would like to use. To upgrade CI Fuzz, first stop the service with, then install CI Fuzz with the new docker images, and then restart the service:

```bash
sudo systemctl stop cifuzz
chmod +x ci-fuzz-on-prem-installer-<version>-linux
./ci-fuzz-on-prem-installer-<version>-linux --docker-images <path to your bundle>
sudo systemctl start cifuzz
```


## Step 4: Run server

```bash
sudo systemctl start cifuzz  
sudo systemctl enable cifuzz #to make it autostart after reboot
```

Check if CI Fuzz containers have been started successfully:

```
sudo docker ps
```

The output should look similar to this:

```
CONTAINER ID   IMAGE                                                  COMMAND                  CREATED          STATUS         PORTS                                       NAMES
2833328092d9   f5d6e55758e0d08d.azurecr.io/cifuzz/ci-worker:2.42.0    "/usr/bin/java -cp /…"   11 seconds ago   Up 7 seconds   0.0.0.0:6778->6777/tcp, :::6778->6777/tcp   cifuzz-ci-worker-1
dba452af7e81   f5d6e55758e0d08d.azurecr.io/cifuzz/ci-backend:2.42.0   "/usr/bin/java -cp /…"   11 seconds ago   Up 7 seconds   0.0.0.0:6777->6777/tcp, :::6777->6777/tcp   cifuzz-ci-backend-1
bc3e5af64601   prometheuscommunity/postgres-exporter:v0.10.0          "/bin/postgres_expor…"   11 seconds ago   Up 7 seconds   0.0.0.0:9187->9187/tcp, :::9187->9187/tcp   cifuzz-db-metrics-1
909f0f946f26   f5d6e55758e0d08d.azurecr.io/cifuzz/reporting:2.42.0    "/usr/bin/java -cp /…"   11 seconds ago   Up 8 seconds                                               cifuzz-reporting-1
86d83d710634   postgres:12.7                                          "docker-entrypoint.s…"   11 seconds ago   Up 8 seconds   5432/tcp                                    cifuzz-db-1
6975de4d7388   f5d6e55758e0d08d.azurecr.io/cifuzz/gateway:2.42.0      "/app/cmd/gateway/ba…"   11 seconds ago   Up 8 seconds   0.0.0.0:443->443/tcp, :::443->443/tcp       cifuzz-gateway-1
```
