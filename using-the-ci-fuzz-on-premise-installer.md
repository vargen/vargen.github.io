# Using the CI Fuzz on-premise Installer

The easiest way to start fuzzing is to use our SaaS solution. With this, you can fuzz your code in our cloud. We will take care of hosting the infrastructure so you can focus on fuzzing.

If you need to run the server on-premise or in your own cloud, you can use the CI Fuzz on-prem installer to set it up easily.

This guide covers the new CI Fuzz Clojure backend server. For the legacy GO server, visit [this page](https://help.code-intelligence.com/using-the-ci-fuzz-server-installer).

**Prerequistes**

In order to use the provided install mechanmism and the resulting docker-compose setup, some prerequisites have to be met. We recommend using a server with at least:

> * 32 GB RAM
> * 16 or more CPU cores
> * 100 GB storage
> * A Linux operating system
> * ssh access
> * root privileges (if you want to use port 80/443)
> * One port (443 recommended) open to access from CI/CD runners and users' computers

For a more detailed overview of the hardware and software technical prerequisites see [CI Fuzz Hardware and Software Technical Prerequisites.](https://help.code-intelligence.com/ci-fuzz-hardware-and-software-technical-prerequisites)

**Migrating from legacy CI Fuzz server**

The legacy server used separate ports for HTTP and GRPC. If you are migrating, please note the open port requirement (single port must be reachable from CI/CD runners and users' computers) and edit firewall rules if needed.

#### Linux

The only supported operating system is Linux. Most linux distributions will be fine, while the most comfort can be achieved by using one that supports [systemd](broken-reference/).

#### Docker

Docker-Compose relies on [docker](https://www.docker.com/) being installed on the host system. To do so, follow the provided instructions for your distribution as depicted in the [docker engine documentation](https://docs.docker.com/engine/install/).

Note: Podman on RedHat has not been tested, only regular docker.

#### Docker-Compose

In addition to the [docker engine](broken-reference/), the setup uses [docker-compose](https://docs.docker.com/compose/) to orchestrate its components. To setup docker-compose, follow the [installation instructions](https://docs.docker.com/compose/install/).

The minimum supported docker-compose version is 1.17.0.

#### Systemd

Having docker-compose allows to easily start and stop the service. But to be (re)started at boot, additional work has to be done. The installer provides a shortcut by setting up a systemd system service. This will obviously only work if the operating system supports [systemd](https://systemd.io/).

#### Ports and Network Settings

One port needs to be open to users, we recommend 443. For Web Fuzzing, a range of ports must be open to hosts that run the software under test. Default range is 6777-7777.

A DNS entry for your CI Fuzz server must exist, or you need a custom TLS certificate that is valid for an IP address.

#### Azure Account

If you don't provide the docker images yourself (explained below), you will have to log in to the Azure-Hosted [image registry](https://docs.microsoft.com/en-us/azure/container-registry/container-registry-intro) in order for the installer to be able to download them. This works by Code Intelligence granting access to your Azure account.

#### Azure CLI

Once your Azure account has been granted access to the image registry, you will need to log in to be able to download docker images. To do so, you will need to install the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/) tools.

You can also provide authentication to the Azure container registry in another form. Or circumvent the need by providing the images yourself. See Step 3 for more details.

#### \*\*Step 1: Create an OAuth Application

CI Fuzz uses OAuth for authenticating users. Learn how to set up an OAuth app [here](https://help.code-intelligence.com/create-an-oauth-application).

#### **Step 2: Get a TLS certificate.**

For secure TLS connections to the fuzzing server, a TLS certificate is needed. Probably your company already uses a specific CA or maybe runs its own. If this is not the case, you can get free TLS certificates from [Let's Encrypt](https://letsencrypt.org/).

Just install the command line tool certbot and request a certificate with a single command:

sudo apt install certbot\
sudo certbot certonly --standalone --preferred-challenges http -d

#### Step 3: Run the installer

You should have received a link for the installer. Optionally you have also received a docker image bundle (CI Fuzz on-premise backend image bundle) from Code Intelligence.

**Azure Registry**

If you are using Azure registry images you don't need the docker image bundle:

az login\
az acr login --name cifuzzpublictest\
chmod +x ci-fuzz-on-prem-installer--linux\
./ci-fuzz-on-prem-installer--linux

**Docker Image Bundle**

If you have the docker image bundle, you can skip the az commands and run the installer with the following option:

./ci-fuzz-on-prem-installer--linux --docker-images

If you were updating from previous version:

sudo systemctl daemon-reload\
sudo systemctl restart cifuzz

#### Step 4: Configuration

First create the directory that will contain configuration files:

sudo mkdir /etc/cifuzz/

Then create this file

> /etc/cifuzz/config.env

With the following content:

```
#
# Code Intelligence Fuzzing environment values.
#
#
# Server Setup
#

# Change this value to the domain / IP where the server should be reachable and adjust
# the protocol when TLS encryption is configured.
CIFUZZ_SERVER_ORIGIN=https://cifuzz.company.org:443

# External HTTP/HTTPS port.
CIFUZZ_SERVER_PORT=443

# TLS Setup
#
# Both settings are required for using the TLS enabled docker-compose.tls.yaml file.
# The TLS settings will be applied automatically by the cifuzz-server tool when
# these variables are set.
CIFUZZ_CERT_FILE=
CIFUZZ_CERT_KEY=

#
# OAuth Apps
#

# GitHub
#
# Create a GitHub OAuth app by following the instructions here:
# https://docs.github.com/en/developers/apps/building-oauth-apps/creating-an-oauth-app
#
# As callback URL use `https://<my-domain>/auth/github/callback`.
#
# Enter both the GitHub OAuth app ID and secret in the values below.
CIFUZZ_GITHUB_CLIENT_ID=
CIFUZZ_GITHUB_CLIENT_SECRET=
# Set for GitHub Enterprise
CIFUZZ_GITHUB_BASE_URL=

# GitLab
#
# Create a GitLab OAuth app by following the instructions here:
# https://docs.gitlab.com/ee/integration/oauth_provider.html
#
# As callback URL use `https://<my-domain>/auth/gitlab/callback`.
#
# Enter both the GitLab OAuth app ID and secret in the values below.
CIFUZZ_GITLAB_CLIENT_ID=
CIFUZZ_GITLAB_CLIENT_SECRET=

# Configuration for on-premise GitLab.
CIFUZZ_GITLAB_BASE_URL=
CIFUZZ_GITLAB_ROOT_CAS=

# Bitbucket
#
# Configure an OAuth consumer by following the instructions here:
# https://support.atlassian.com/bitbucket-cloud/docs/integrate-another-application-through-oauth/
#
# As callback URL use `https://<my-domain>:<port>/auth/bitbucket/callback`.
#
# Enter both the Bitbucket OAuth consumer ID and secret in the values below.
CIFUZZ_BITBUCKET_CLIENT_ID=
CIFUZZ_BITBUCKET_CLIENT_SECRET=

# Bitbucket on-premise oauth1
CI_AUTH_BITBUCKET_URL=

# Azure
#
# Create a OAuth / OpenID app by following the instructions here:
# https://docs.microsoft.com/en-us/azure/active-directory/saas-apps/openidoauth-tutorial
#
# As callback URL use `https://<my-domain>/auth/azure/callback`.
#
# Enter both the Azure OAuth App ID and secret in the values below.
CIFUZZ_AZURE_CLIENT_ID=
CIFUZZ_AZURE_CLIENT_SECRET=

#
# Gateway
#

# CHANGE ME!
#
# Shared secret to authenticate the gateway and the backend server. It must be at least
# 32 bytes of hex encoded bytes. If openssl is available you can generate a valid random
# value with `openssl rand -hex 32`.
CIFUZZ_INTER_SERVICE_AUTH_KEY=0ec1302c35fcd8391ea57c2d6fcb63b1d9eca8af5845d79a4a51c1695d2462d3

```

Edit the variables:

CIFUZZ\_SERVER\_ORIGIN - change to the hosname of the host where you are running CI Fuzz server. Please set full origin here - protocol (https), hostname and port. This must match the origin set in your OAuth application.

CIFUZZ\_SERVER\_PORT=443 you can change the port if needed

CIFUZZ\_CERT\_FILE and CIFUZZ\_CERT\_KEY - absolute paths to the certificate and key that CI Fuzz should use.

OAuth Apps - set according to the OAuth app that you have configured in step 1. One OAuth provider is usually enough, you can delete the sections that you will not use.

CIFUZZ\_INTER\_SERVICE\_AUTH\_KEY - please generate a random key with the command suggested in the comment, or otherwise.

Optionally, for other available variables, see this [comprehensive list](https://help.code-intelligence.com/ci-fuzz-server-configuration-variables). Or, for the most up to date list, look in this file:

> /opt/ci-fuzz-/cifuzz.env

#### Step 5: Run server

sudo systemctl start cifuzz\
sudo systemctl enable cifuzz #to make it autostart after reboot

Check if CI Fuzz containers have been started successfully:

sudo docker ps

Output should look similar to this:

CONTAINER ID IMAGE COMMAND CREATED STATUS PORTS NAMES\
c328d2e9ad6b prometheuscommunity/postgres-exporter:v0.10.0 "/bin/postgres\_expor…" 2 days ago Up 2 days 0.0.0.0:9187->9187/tcp, :::9187->9187/tcp cifuzz\_db-metrics\_1\
3ee314af29ce cifuzzpublictest.azurecr.io/cifuzz/ci-backend:2.31.0 "/usr/bin/java -cp /…" 2 days ago Up 2 days 0.0.0.0:6777->6777/tcp, :::6777->6777/tcp cifuzz\_ci-backend\_1\
2e4f9bb6c8e7 postgres:12.7 "docker-entrypoint.s…" 2 days ago Up 2 days 5432/tcp cifuzz\_db\_1\
c24dd46702e8 cifuzzpublictest.azurecr.io/cifuzz/reporting:2.31.0 "/usr/bin/java -cp /…" 2 days ago Up 2 days cifuzz\_reporting\_1\
60d8f8ca1940 cifuzzpublictest.azurecr.io/cifuzz/gateway:2.31.0 "/app/cmd/gateway/ba…" 2 days ago Up 2 days 0.0.0.0:80->80/tcp, :::80->80/tcp cifuzz\_gateway\_1

#### Advanced configuration options

**Using your own reverse proxy/load balancer**

If you want to terminate TLS yourself, comment out CIFUZZ\_CERT\_FILE and CIFUZZ\_CERT\_KEY. Change CIFUZZ\_SERVER\_PORT to some other port, but keep port 443 (or other port where users will be accessing CI Fuzz) in CIFUZZ\_SERVER\_ORIGIN.

Set your reverse proxy or load balancer to direct HTTP and HTTP2/GRPC traffic to CIFUZZ\_SERVER\_PORT. Nginx example:

server {\
listen 443 ssl;\
ssl\_certificate /home/azureuser/tls/cifuzz.onprem.lab\_cert.pem;\
ssl\_certificate\_key /home/azureuser/tls/cifuzz.onprem.lab\_key.pem;\
location / {\
proxy\_pass http://localhost:80/;\
}\
}\
server {\
listen 6443 http2 ssl;\
ssl\_certificate /home/azureuser/tls/cifuzz.onprem.lab\_cert.pem;\
ssl\_certificate\_key /home/azureuser/tls/cifuzz.onprem.lab\_key.pem;\
client\_max\_body\_size 500M;\
location / {\
grpc\_pass grpc://localhost:80;\
}\
}

**Architecture, components troubleshooting**

Technical details about the CI Fuzz backend can be found [here .](https://help.code-intelligence.com/cifuzz-backend-architecture-and-advanced-tools)
